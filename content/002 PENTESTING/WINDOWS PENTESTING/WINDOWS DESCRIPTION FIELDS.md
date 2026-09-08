---
Primary_category: "[[WINDOWS CREDENTIALED ENUMERATION]]"
title: "WINDOWS DESCRIPTION FIELDS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALED ENUMERATION]]

#### *Theory*

Most of the *Domain Objects* on an *AD Enviroment* have a *description* attribute/field

This attribute usually refers to the role that the given object has on the domain and describes its purpose briefly

However, sensitive information such as account passwords are sometimes found in the user account description

![[WINDOWS DESCRIPTION FIELDS-20251104180025270.webp|200]]

> ***Zoom in***

---

#### *Recon - UNIX-like*

##### *LDAPSearch*

> ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

```bash
ldapsearch -LLL -x -H 'ldap://<TARGET>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(ObjectCategory=person)(description=*))' samAccountName description
```

##### *RPCClient*

> ***[RPCClient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)***

```bash
rpcclient --user '<USER>%<PASSWD>' --command 'querydispinfo' '<TARGET>'
```

---

#### *Recon - Windows*

##### *Powershell AD Module*

> ***[Powershell AD Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

> ***Get-ADUser***

```powershell
Get-ADUser -Filter * -Properties * | ? { $_.description -ne $null } | Select samAccountName, description
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***Get-DomainUser***

```powershell
Get-DomainUser | ? { $_.description -ne $null } | Select samAccountName, description
```