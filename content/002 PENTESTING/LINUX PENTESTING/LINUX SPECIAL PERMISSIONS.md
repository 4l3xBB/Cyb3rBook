---
Primary_category: "[[LINUX PRIVESC]]"
title: LINUX PERMISSIONS
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Enumeration*

##### *Special Permissions*

###### *SETUID*

This type of permission allows a user to run the given binary with the permissions of another user i.e. the effective user will be the owner whereas the real user will be the current user

```bash
find / -perm -4000 -type f -ls 2> /dev/null
```

###### *SETGID*

On the other hand, when a binary with this permission enabled is executed, the resultant process will run with the permissions of the owning group

```bash
find / -perm -6000 -type f -ls 2> /dev/null
```

##### *Sudo Permissions*

This type of privilege allow a given user to run certain commands in the context of *ROOT* or another privileged account without having to change users or grant excessive rights

> ***The command below will always check the `/etc/sudoers` file***

```bash
sudo -l
```

Sometimes the system will ask for the current user password before running the previous command

That happens if the *NOPASSWD* tag is not specified in any of the existing entries related to the user in question within the **`/etc/sudoers`** file

So, it is always a good safety measure not to add the *NOPASSWD* tag to any *sudoers* entry

In addition, a system administrator should always specify the absolute path of the given binary. Otherwise, an attacker may be able to leverage *PATH* abuse to create a malicious binary that will be executed when the commands runs

---

#### *Abuse*

##### *GTFOBins*

> ***[GTFOBins](https://gtfobins.org/)***

Many programs have additional features that an operator could leverage to run commands

So, once we find out a binary which has any of the previous special permissions enabled or the current user can run a certain command with sudo privileges, we should check if the binary has any feature that we can leverage to carry out certain actions, such as

- ***Break out [[ESCAPE RSHELLS|Restricted Shells]]***
- ***Escalate Privileges***
- ***Spawn a [[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]] connection***
- ***Transfer files***

---

#### *Resources*

***[How to use Special Permissions: SETUID, SETGID and Sticky Bit](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits)***