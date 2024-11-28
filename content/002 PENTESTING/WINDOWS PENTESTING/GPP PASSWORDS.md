---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "GPP PASSWORDS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

| **REFERENCES** | |
| --- | --- |
| ***Reference I*** | ***[See here](https://vk9-sec.com/exploiting-gpp-sysvol-groups-xml/)*** |
| ***Reference II*** | ***[See here](https://adsecurity.org/?p=2288)*** |

***GPO (Group Policy Objects)*** and ***GPP (Group Policy Preferences)*** configuration templates are stored in *SYSVOL* shared folder

This directory ***is replicated in all DCs*** and ***all authenticated domain users*** have read permissions on it

All *Domain Computers* need to access *SYSVOL* in order to download and apply the established *GPOs* and *GPPs* by the *DC*

***Related Path → `\\DOMAIN.LOCAL\SYSVOL\DOMAIN.LOCAL\Policies\`***

When *GPPs* related to *Local User Accounts or Passwords* are configured, this creates a `Groups.xml` file

This file is part of the stored configuration in *SYSVOL* and can contain some sensible data such as → 

- ***Username***
- ***CPassword***

The last one stores the *Cyphered User Password*. But, since ***[Microsoft published the Simmetric Cyphering Key](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be)*** used to encrypt that password, anyone can get it in plain text

As all authenticated domain users has read perms on *SYSVOL*, any user can access to this file

---

#### GPP Decryption

##### *gpp-decrypt*

```bash
gpp-decrypt <CPASSWORD> 
```