---
Primary_category: "[[NAMED PIPES ABUSE]]"
title: "NAMED PIPES ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[NAMED PIPES ABUSE]]

#### *Enumeration*

##### *Existing Named Pipes*

> ***PS***

```bash
dir \\.\pipe\
```

> ***[Pipelist.exe](https://learn.microsoft.com/en-us/sysinternals/downloads/pipelist)***

```bash
pipelist.exe /accepteula
```

##### *Named Pipes DACL*

> ***[Accesschk.exe](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk)***

###### *Specific Named Pipe*

```bash
accesschk.exe /accepteula \\.\Pipe\lsass -v
```

###### *All existing Named Pipes*

```bash
accesschk.exe /accepteula \pipe\* -v
```

###### *Existing Named Pipe for which a User has Write Permissions*

- ***Current User***

```bash
accesschk.exe -accepteula -w <NAMED_PIPE> -v
```

- ***Specific User***

```bash
accesschk.exe -accepteula -w <USER> <NAMED_PIPE> -v
```