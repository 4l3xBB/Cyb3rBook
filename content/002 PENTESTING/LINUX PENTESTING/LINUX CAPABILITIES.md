---
Primary_category: "[[LINUX PRIVESC]]"
title: "LINUX CAPABILITIES"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Theory*

##### *Capability Values*

###### *Sets*

> ***e, p & i***

| **SET** | **DESCRIPTION** |
| --- | --- |
| ***Effective ( `e` )*** | ***Currently enabled capabilities for the process*** | 
| **Permitted ( `p` )***  | ***Maximum set of capabilities for the process*** |
| ***Inheritable ( `i` )*** | ***Capabilities that can be inherited by child processes*** |

###### *Operators*

> ***= & +***

| **OPERATOR** | **DESCRIPTION** |
| --- | --- |
| *`=ep`* | ***Enabled for the Effective ( `e` ) and Permitted ( `p` ) sets. It also disables the Inheritable ( `i` ) set***
| **`+ep`** | ***Add the given capability to the Effective ( `e` ) and Permitted ( `p` ) sets, without altering the Inheritable ( `i` ) set*** |

##### *Sensitive Capabilities*

| **CAPABILITY** | **DESCRIPTION** |
| --- | --- |
| **`cap_sys_admin`** | ***Perform actions with admin privileges ( e.g. Modify system files, change system settings... )***
| **`cap_sys_chroot`** | ***Change the root directory for the current process, allowing it to access any file and directory*** |
| **`cap_sys_trace`** | **Attach and debug other processes, allowing it to gain access to sensitive information** |
| **`cap_sys_nice`** | ***Raise or lower the process priority, potentially allowing it to gain access to restricted resources*** |
| **`cap_sys_time`** | ***Modify the System Clock, so it can modify timestamps and so on*** |
| **`cap_resource`** | ***Modify system resource limits, such as the maximum number of open FD*** |
| **`cap_sys_module`** | ***Load and unload kernel modules, potentially allowing it to gain access to sensitive data*** |
| **`cap_net_bind_service`** | ***Bind to network ports, allowing it to gain access to sensitive information*** |

##### *Privesc Capabilites*

| **CAPABILITY** | **DESCRIPTION** |
| --- | --- |
| **`cap_setuid`** | ***The given process is able to set its effective user ID*** |
| **`cap_setgid`** | ***The given process is able to set its effective group ID*** |
| **`cap_sys_admin`** | ***Perform actions with admin privileges ( e.g. Modify system files, change system settings... )***
| **`cap_dac_override`** | ***Allows bypassing of file read, write and execute permissions check*** |


---

#### *Enumeration*

```bash
getcap -r / 2> /dev/null
```

---

#### *Abuse*

##### *CAP_DAC_OVERRIDE*

Imagine we land on a target through a reverse shell as a non-privileged user, then we initialize our enumeration process in order to achieve a quick privilege escalation

We start by listing the system binaries with an assigned capability and its corresponding sets

This time there is a binary with the **`cap_dac_override`** capability set, which allows the process to bypass file permissions such as read, write and execute for the given resource

The binary in question is a text editor, namely **`vim.basic`**

Therefore, we could edit a file such as **`/etc/passwd`** and delete the *x* in the second column of the row corresponding to the *ROOT* user, so that we can subsequently use the **`su`** command without being asked for a password

```bash
/usr/bin/vim.basic /etc/passwd
```

```bash
su -
```