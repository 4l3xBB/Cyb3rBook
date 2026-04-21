---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "SETAKEOWNERSHIPPRIVILEGE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

This privilege grants a user account the ability to take ownership of any *securable object*, namely →

- ***AD Objects ( Users, Groups... )***

- ***NTFS Files and Directories***

- ***Printers***

- ***Registry Keys ( Hives within HKLM, HKCU... )***

- ***Processes***

That is, this privilege assigns the ***[[GRANT OWNERSHIP (WRITEOWNER)|WriteOnwer]]*** right over a given object

Therefore, if an operator compromise a local or domain user account which has this privilege enabled, he can leverage it to take ownership of any *securable object* of the system by modifying the owner within its *security descriptor*

Once we modify its owner, we implicity have the ***[[GRANT RIGHTS (WRITEDACL)|WriteDACL]]*** right over the given objet. So, we can create, delete or modify any existing *ACE* within the *DACL* of the *object's security descriptor*

From here, it's as simple as create an *ACE* which grants *Full Control* to a principal controlled by the operator

---

#### *Enumeration*

##### *Current User Privileges*

```bash
whoami /priv
```

---

#### *Abuse - Windows*

Let's suppose that 

##### *Enabling SeTakeOwnershipPrivilege*

> ***See [[WINDOWS PRIVESC#Enabling disabled Privileges|Enabling disabled Privileges]]***

##### *Trying to list the Owner of an specific Resource*

> ***A file, directory or named pipe***

```bash
dir -Path '<RESOURCE_PATH>' | Select Fullname, LastWriteTime, Attributes, @{Name=Owner; Expression={ (Get-ACL $_.FullName ).Owner }}
```

##### **