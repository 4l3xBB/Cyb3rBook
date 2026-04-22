---
Primary_category: ""
title: ""
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS SECURITY GROUPS|SECURITY GROUPS]]

#### *Theory*

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

Let's suppose 