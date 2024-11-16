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
ping -c1 10.129.135.173
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.135.173 (10.129.135.173) 56(84) bytes of data.
> 64 bytes from 10.129.135.173: icmp_seq=1 ttl=127 time=43.4 ms
> 
> --- 10.129.135.173 ping statistics ---
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
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.135.173
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="Active/scans/AllPorts"
> # Nmap 7.94SVN scan initiated Sat Nov 16 16:31:38 2024 as: nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.135.173
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.135.173 () Status: Up
> Host: 10.129.135.173 () Ports: 53/open/tcp//domain///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-
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
>     [+] IP Address: 10.129.135.173
>     [+] Open Ports: 53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168
>
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="Active/Scans"
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168 -sCV -oN targeted 10.129.135.173
```

> [!NOTES]- *Targeted Output*
>
> ```bash title="Active/scans/Targeted"
> # Nmap 7.94SVN scan initiated Sat Nov 16 16:37:06 2024 as: nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49166,49168 -sCV -oN targeted 10.129.135.173
> Nmap scan report for active.htb (10.129.135.173)
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
printf "%s\t%s" "10.129.135.173" "active.htb" >> /etc/hosts
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
> 10.129.135.173  active.htb
> ```
>

It looks that this host is a *DC (Domain Controller)* judging by the ports It has open

As it has the *SMB* service exposed (*139 and 445 Port*), let's extract some general information about the *target*

```bash title="Active/scans"
netexec smb 10.129.135.173
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.135.173  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> ```
>

The *Target's hostname* is *DC*, therefore, we were right before, it is a *Domain Controller* and the *Domain Name* is ***Active.htb***

Otherwise, nothing interesting, the *SMB is signed* and *SMBv1* is not enabled

##### *53 - DNS*

Let's see if we can gather some information about the *DNS Service* running in this port

###### *Banner Grabbling*

```bash title="Active/scans"
dig version.bind CHAOS TXT @10.129.135.173 +short
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
dig axfr active.htb @10.129.135.173
```

> [!NOTE]- *Command Output*
>
> ```bash title="Active/scans"
> ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> axfr active.htb @10.129.135.173
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
ldapsearch -x -H 'ldap://10.129.135.173' -D '' -w '' -b 'DC=active,DC=htb'
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
> SMB         10.129.135.173  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
> SMB         10.129.135.173  445    DC               [+] active.htb\:
> SMB         10.129.135.173  445    DC               [*] Enumerated shares
> SMB         10.129.135.173  445    DC               Share           Permissions     Remark
> SMB         10.129.135.173  445    DC               -----           -----------     ------
> SMB         10.129.135.173  445    DC               ADMIN$                          Remote Admin
> SMB         10.129.135.173  445    DC               C$                              Default share
> SMB         10.129.135.173  445    DC               IPC$                            Remote IPC
> SMB         10.129.135.173  445    DC               NETLOGON                        Logon server share
> SMB         10.129.135.173  445    DC               Replication     READ
> SMB         10.129.135.173  445    DC               SYSVOL                          Logon server share
> SMB         10.129.135.173  445    DC               Users
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

```bash
smbmap -R -H active.htb -u '' -p ''
```

We list the following resources inside the `Replication` downloaded folder

> [!NOTE]- *Command Output*
>
> ```bash
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

This file is generated automatically when a *GPP* related to *Local User or Groups of the *

---

#### Exploitation

##### *Vulnerability Name or Vuln Chaining*

##### *e.g. RCE via Authenticated File Upload*

---
#### Shell as Web User

Once a connection via *Reverse Shell* is stablished, just proceed as follows to upgrade the obtained shell to a *Fully Interactive TTY*

> ***[Reference](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)***

##### *Script*

```bash title="Target"
script /dev/null -c bash
<C-z>
```

```bash title="Attacker"
stty raw -echo ; fg
reset xterm
```

```bash title="Target"
export TERM=xterm-256color
export SHELL=/bin/bash
. /etc/skel/.bashrc
stty rows <ROWS> columns <COLUMNS>
```

---

#### Privesc #1

***Initial Non-Privileged User → USERNAME***

##### *PRIVESC VECTOR A*

#### Privesc #2 (If exists)

##### *PRIVESC VECTOR A*

---

#### Custom Exploits

##### *EXPLOIT A*

> ***[Reference (if exists)]()***

> [!IMPORTANT]- *EXPLOIT NAME*
>
> ```python
> ```
>

***\[\[ EXPLOIT EXECUTION GIF ]]***