---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: "WINDOWS GROUP POLICIES"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]

#### *Theory*

A *Domain Group Policy Object* encompasses a series of directives, restrictions and configurations that apply to a specific set of *AD objects*, whether it is a user account or a computer account

Therefore, if an attacker takes control over a domain account that have certain rights or privileges over a *GPO*, it can modify it by adding or editing certain settings or features in order to compromise one or all object for which the *GPO* is applied and linked

Some actions that can be taken by abusing certain rights over a *GPO* are →

- ***Adding additional rights to a User Account (such as*`SeDebugPrivilege`, `SeTakeOwnershipPrivilege`, `SeImpersonatePrivilege`*)***

- ***Adding a local admin user to one or more hosts***

- ***Creating an inmediate scheduled task to perform any action on the given host***

##### *Interesting ACES over a GPO*

- ***WriteProperty*** to the ***GPC-File-Sys-Path*** property of a GPO

- ***GenericAll, GenericWrite*** and ***WriteProperty*** to any property

- ***[[GRANT RIGHTS (WRITEDACL)|WriteDACL]]*** and ***WriteOwner***

---

#### *Recon - UNIX-like*

##### *LDAPSearch*

> ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

###### *Enumerating GPOs*

```bash
ldapsearch -LLL -x -H 'ldap://<DC>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'CN=Policies,CN=System,DC=lab,DC=local' displayName | awk -F: -v IGNORECASE=1 '/displayName/ { print $2 }'
```

##### *Go-Windapsearch*

> ***[Go-Windapsearch]()***

```bash
go-windapsearch --domain '<DOMAIN>' --dc '<DC>' --username '<USER>' --password '<PASSWD>' --module 'gpos'
```

---

#### *Recon - Windows*

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

###### *Enumerating GPOs*

> ***Get-DomainGPO***

```powershell
Get-DomainGPO | Select displayName
```

###### *Listing the GPO's ACEs for a specific account*

> ***Get-DomainGPO + Get-ObjectACL***

```powershell
$sid = Convert-NameToSID '<ACCOUNT_OR_GROUP>'
```

```powershell
Get-DomainGPO | Get-ObjectACL | ? { $_.securityIdentifier -eq $sid }
```

##### *Group Policy Module*

> ***[Group Policy Module](https://learn.microsoft.com/en-us/powershell/module/grouppolicy/?view=windowsserver2022-ps)***

###### *Enumerating GPOs*

```powershell
Get-GPO -All | Select displayName
```

###### *Converting GPO GUID to Name*

> ***Get-GPO***

```powershell
Get-GPO -Guid '<GPO_GUID>'
```

##### *Group3r*

> ***[Group3r](https://github.com/Group3r/Group3r)***

```powershell
group3r.exe -f '<LOG_FILE>'
```

---

#### *Abuse - Windows*

##### *SharpGPOAbuse*

> ***[SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse)***

```powershell
```