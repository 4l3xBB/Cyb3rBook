---
Primary_category: "[[WINDOWS CREDENTIALED ENUMERATION]]"
title: "WINDOWS PASSWD_NOTREQD"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALED ENUMERATION]]

#### *Theory*

The ***PASSWD_NOTREQD*** is a flag within the *UserAccountControl* attribute of a domain user account

If set, the given user account is not subject to the current *[[WINDOWS PASSWORD POLICY|Password Policy]]*, meaning that it could have a shorter and less complex password or no password at all (If the domain allows empty passwords)

It is important to note that just because this flag is enabled, it does not mean that no password is set for the given account

![[WINDOWS PASSWD_NOTREQD-20251104184026978.webp|200]]

> ***Zoom in***

---

#### *Recon - UNIX-like*

##### *LDAPSearch*

> ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

```bash
ldapsearch -LLL -x -H 'ldap://<TARGET>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(ObjectCategory=person)(UserAccountControl:1.2.840.113556.1.4.803:=32)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' samAccountName dn
```

---

#### *Recon - Windows*

##### *Powershell AD Module*

> ***[Powershell AD Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

```powershell
Get-ADUser -LDAPFilter '(&(ObjectCategory=person)(UserAccountControl:1.2.840.113556.1.4.803:=32)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' | Select samAccountName, userPrincipalName
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

```powershell
Get-DomainUser -UACFilter 'PASSWD_NOTREQD' -LDAPFilter '(!(UserAccountControl:1.2.840.113556.1.4.803:=2))' | Select samAccountName, userAccountControl | fl
```