---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: LocalAccountTokenFilterPolicy
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]

#### *UAC Remote Restriction*

#### *Summary*

> ***LocalAccountTokenFilterPolicy***

> ***Path***

```bash
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy
```

***Enabled by default***

***Default Value → 0***

It only applies if/for →

- ***UAC Enabled***
- ***Logon Type 3 (Network)***
- ***Non RID 500 Local Administrator Account***
- ***RID 500 Local Admin Account (Administrator), if [[FilterAdministratorToken]] enabled, as this directive applies UAC to this account***

I.e. It applies to ***All Local Admin Accounts*** under ***UAC***

By default, ***Non RID 500 Local Admin Accounts*** are under ***UAC*** via the ***enabledLUA*** directive, while the ***RID 500 Admin Account*** is not under ***UAC*** unless the ***[[FilterAdministratorToken]]*** is enabled

This directive restricts ***Local Admin Accounts*** to stablish a ***Remote Connection (Logon Type 3 - Network)*** with a ***Full Access Token***

I.e., this ***Full Token*** is filtered by the ***UAC***, becoming a ***Filtered Access Token***

So, if ***LocalAccountTokenFilterPolicy*** is disabled (1) and a client remotely connects as a ***Non-RID 500 Local Admin Account***, ***LSASS.exe*** will create a ***Full Access Token*** instead of a filtered one

> [!DANGER]- *RID 500 Local Admin Account*
>
> By default, the ***RID 500 Local Admin Account*** is not affected by the ***UAC***
>
> Therefore, ***LSASS.exe*** always creates a ***Full Access Token*** if a client authenticates with this account, regardless of the ***Logon Session Type***
>
> Since the ***UAC*** does not apply to this account, the ***LocalAccountFilterTokenPolicy*** does not apply either
>
> However, if the [[#Admin Approval Mode|FilterAdministratorToken]] directive is enabled, the ***UAC*** is applied to this account, and thus, the other also applies
>
> So, if [[#UAC Remote Restriction|LocalAccountTokenFilterPolicy]] and [[#Admin Approval Mode|FilterAdministratorToken]] directives are enabled, the ***Administrator Account*** receives a ***Filtered Access Token*** when ***Logon Session Type 3 (Network)*** is created
>
> That means that any ***Network Authentication*** for remote login (***WinRM, Psexec, Net User, WMI...***) will result in an ***Access Denied Error***, as ***LSASS.exe*** will only create a ***Filtered Access Token*** with ***privileges disabled*** and ***privileged Groups***, such as ***Administrators***, with a ***Deny-Only*** status
>

---

#### *Check the State of LocalAccountTokenFilterPolicy*

##### *Reg Query*

```bash
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\" | findstr /I 'LocalAccountTokenFilterPolicy'
```

##### *Get-ItemProperty*

```bash
(Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System").LocalAccountTokenFilterPolicy
```

#### *Disable LocalAccountTokenFilterPolicy*

##### *Reg Add*

```bash
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

##### *Set-ItemProperty*

```bash
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name 'LocalAccountTokenFilterPolicy' -Value 1 -Type DWord -Force
```
