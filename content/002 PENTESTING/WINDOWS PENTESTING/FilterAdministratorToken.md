---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: FilterAdministratorToken
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]

#### *Admin Approval Mode*

#### *Summary*

> ***FilterAdministratorToken***

> [!IMPORTANT]-
> ***UAC does not apply to RID 500 Admin Account if FilterAdministratorToken directive is disabled***

> ***Path***

```bash
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\FilterAdministratorToken
```

***Disabled by default***

***Defaul Value → 0***

It only applies if/for →

- ***UAC Enabled***
- ***Logon Type 2 (Interactive) and 10 (RemoteInteractive)***
- ***RID 500 Local Administrator Account***

It makes that ***LSASS.exe*** creates a ***Dual Token***, i.e. a ***Filtered Access Token*** and a ***Full Access Token***, when a client starts a ***Logon Session Type 2 (Interactive)*** as the ***RID 500 Local Admin Account***

This directive basically ***enables*** the ***UAC*** for the ***RID 500 Administrator Local Account***

By defaulf, the ***RID 500 Admin Local Account*** is not affected by the ***UAC***

- This account receives a ***Full Access Token*** in ***any Logon Session Type***
- There is no ***Dual Token***

But, if ***Admin Approval Mode*** is enabled, this force the ***RID 500 Admin Account*** to behave as a ***Non-RID 500 Local Admin Account***, i.e. ***UAC*** is applied

- ***Dual Token*** created on a ***Logon Session Type 2 (Interactive)***

***Filtered Access Token + Full Access Token***

- Process launched under ***Full Access Token*** if ***UAC is accepted***
- ***[[LocalAccountTokenFilterPolicy]]*** applies to the ***Administrator Account***, so a client cannot perform a successfull ***Network Authentication*** with this account since a ***Filtered Access Token*** is created

I.e, a client cannot use ***WinRM, PSExec, Net Use or WMI*** to connect remotely, as the ***RID 500 Local Admin Account***, to the machine when both policies (***[[FilterAdministratorToken]] and [[LocalAccountTokenFilterPolicy]]***) are enabled

---

#### *Check the State of FilterAdministratorToken*

##### *Reg Query*

```bash
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" | Select-String -Pattern 'FilterAdministratorToken'
```

##### *Get-ItemProperty*

```bash
(Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System").FilterAdministratorToken
```

---

#### *Disable FilterAdministratorToken*

If enabled, proceed as follows to disable it →

##### *Reg Add*

```bash
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v FilterAdministratorToken /t REG_DWORD /d 0 /f
```

##### *Set-ItemProperty*

```bash
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name FilterAdministratorToken -Value 0 -Type DWord -Force
```
