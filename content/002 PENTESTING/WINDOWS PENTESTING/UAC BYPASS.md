---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "UAC BYPASS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Binary Self-Elevation*

There are different type of contexts in which this technique comes handy

##### *Logon Session 2 (Interactive) via Runas.exe - Filtered Token*

Imagine an attacker logs in to the a remote system as a ***Non-Privileged User***

Then, ***LSASS.exe*** creates a ***Non-Privileged Access Token***, which has a ***Medium IL (Integrity Level)***

The attacker runs the following command to search for credentials stored in the form of ***encrypted Blobs*** or stored as ***.VCRD*** files within the ***Windows Credential Lockers (Windows Vaults)***

```bash
cmdkey.exe /list
```

Or just checks the ***Windows Credential Manager***

And he found an stored credential related to a ***Local Admin Account***

Despite not knowing the plain text password, the following command could be executed to get a ***Logon Session Type 2 (Interactive)*** as this user

```bash
runas.exe /savecred /user:<ADMIN_USER> powershell.exe
```

The above command launch a *powershell.exe* as the ***Local Admin User***

Since the obtained session is an ***Interactive*** one and the user belongs to the ***Administrators*** group, the ***User Account Control*** is applied

Therefore, the launched *powershell.exe* instance has a ***Medium IL*** as this process has been created under the context of a ***Filtered Token***



> [!DANGER]- *Important*
>
> By default, the ***User Account Control (UAC)*** applies for all the ***Admin Local Accounts*** except for the ***RID 500 Account (Administrator)***, as long as the [[WINDOWS MOVEMENT#Admin Approval Mode|Admin Approval Mode]] is not enabled
>
> Therefore, ***LSASS.exe*** creates a ***Dual Token (Filtered and Full Access Token)*** when a client logs in as a ***Non-RID 500 Admin Local Account*** if the ***Logon Session Type*** is ***Interactive (2)*** or ***RemoteInteractive (10)***
>

In practical terms, it is like being logged in via ***WinRM (Logon Type 3 - Network)***, as a ***Local Admin User*** having applied the ***[[WINDOWS MOVEMENT#UAC Remote Restriction|LocalAccountTokenFilterPolicy]]***

I.e., the client also receives a ***Filtered Access Token***

But the difference in this case is that the attacker comes from an ***Interactive Logon Session*** as the ***Non-Privileged User***

Thus, we can leverage of the ***Binary Autoelevation*** and search for binaries that bypass the ***User Account Control***

The current situation is that an attacker has a process, the *powershell.exe*, with a ***Medium IL*** under a ***Filtered Token*** context

But this access token is related to the ***Local Admin User***, which means that there is a ***Full Access Token*** associated with the ***LUID*** of this user stored in the memory of ***LSASS.exe***

So, an attacker could launch a process marked as ***Self-Elevating***, which bypasses the ***UAC*** and therefore launches under the context of the ***Full Access Token***

From this process, any launched process have a ***High IL*** as this process will be created under the context of the ***Full Access Token***

The, the attacker could spawn another *powershell.exe* from that process

##### *MSConfig*

The ***msconfig.exe*** binary is one of them

Therefore, the entire process would be →

###### *Launch a Process as the Non-Privileged User*

> ***Non-Privileged Access Token → Medium IL***

```bash
powershell.exe
```

###### *Check for Stored Credentials*

> ***Local Admin Account's Stored Credentials found***

```bash
cmdkey.exe /list
```

###### *Create an Interactive Session as the Local Admin User*

> ***Filtered Access Token (Dual Token) → Medium IL***

```bash
runas.exe /savecred /user:<ADMIN_USER> powershell.exe
```

###### *Launch the Self-Elevating Binary*

> ***UAC Bypass***

> ***Full Access Token → High IL***

```bash
msconfig.exe
```

###### *Spawn another Process from the Elevated Process*

> ***Full Access Token → High IL***

![[UAC BYPASS-20250612185917145.webp|350]]

> ***Zoom In***