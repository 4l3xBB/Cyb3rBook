---
Primary_category: "[[MACHINES]]"
title: NIBBLES
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: []
cssclasses:
---

###### PRIMARY CATEGORY → [[MACHINES]]

***Summary***

![[NIBBLES-20241023174334210.webp|400]]

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Nibbles && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash title="Nibbles"
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

#### Recon

##### *OS Identification*

First, proceed to identify the *Target Operative System*. This can be done by a simple `ping` taking into account the *TTL Unit*

The standard values are →

- ***About 64 → Linux***
- ***About 128 → Windows***

```bash title="Nibbles/scans"
ping -c1 10.129.96.84
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.96.84 (10.129.96.84) 56(84) bytes of data.
> 64 bytes from 10.129.96.84 icmp_seq=1 ttl=63 time=35.3 ms
>
> --- 10.129.96.84 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 35.281/35.281/35.281/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Linux Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash title="Nibbles/scans"
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.96.84
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="Nibbles/scans/AllPorts"
> # Nmap 7.94SVN scan initiated Wed Oct 23 17:50:03 2024 as: nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG AllPorts 10.129.96.84
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.96.84 ()  Status: Up
> Host: 10.129.96.84 ()  Ports: 22/open/tcp//ssh///, 80/open/tcp//http///    Ignored State: closed (65533)
> # Nmap done at Wed Oct 23 17:50:15 2024 -- 1 IP address (1 host up) scanned in 11.17 seconds
> ```
>

**Open Ports → 22, 80**

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash title="Nibbles/Scans"
extractPorts allPorts
```

> [!NOTES]- *Command Output*
>
> ```bash
> [+] Extracting information...
> 
>     [+] IP Address: 10.129.96.84
>     [+] Open Ports: 22,80
> 
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="Nibbles/Scans"
nmap -p22,80 -sCV -oN targeted 10.129.96.84
```

> [!NOTES]- *Command Output*
>
> ```bash
> # Nmap 7.94SVN scan initiated Wed Oct 23 18:14:50 2024 as: nmap -p22,80 -sCV -oN targeted 10.129.96.84
> Nmap scan report for 10.129.96.84 (10.129.96.84)
> Host is up (0.040s latency).
> 
> PORT   STATE SERVICE VERSION
> 22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
> | ssh-hostkey: 
> |   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
> |   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
> |_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
> 80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
> |_http-server-header: Apache/2.4.18 (Ubuntu)
> |_http-title: Site doesn't have a title (text/html).
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Wed Oct 23 18:15:00 2024 -- 1 IP address (1 host up) scanned in 10.33 seconds
> ```
>

##### *OS Version (Codename)*

In *Linux Systems*, the *Operative System Version* could be extracted through *Launchpad*

According to the **Version Column Data** of the [[#Comprehensive Scan]], proceed as follows →

- ***22 - SSH***

> ***[Reference](https://launchpad.net/ubuntu/+source/openssh/1:7.2p2-4ubuntu2.2)***

```bash title="Firefox"
OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 site:launchpad.net
```

- ***80 - HTTP***

> ***[Reference](https://launchpad.net/ubuntu/+source/apache2/2.4.18-2ubuntu3.4)***

```bash title="Firefox"
OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 site: launchpad
```

***Codename → [Ubuntu Xenial](https://releases.ubuntu.com/16.04/)***

This can be verified once the [[SHELL SCRIPTING|shell]] is obtained, i.e. the system has been compromised

There are several ways to carry out it →

```bash
cat /etc/os-release
```

```bash
hostnamectl # If System has been booted via Systemd
```

```bash
lsb_release -a
```

```bash
cat /etc/issue
```

##### *22 - SSH*

***OpenSSH Version → v7.2***

All the *OpenSSH* Versions prior to the *v7.7* one are vulnerable to a**System User Enumeration**

> ***[Reference](https://www.rapid7.com/db/modules/auxiliary/scanner/ssh/ssh_enumusers/)***

**CVE-2018-15473** → ***OpenSSH < v7.7***

```bash
searchsploit ssh user enumeration
```

To get the *ExploitDB* links related to above exploits →

```bash
searchsploit --www ssh user enumeration
```

***Exploit →  OpenSSH < 7.7 - User Enumeration (2)***

To examine it →

```bash
searchsploit --examine linux/remote/45939.py |& cat --language python
```

> **This exploit requires *Python2***

Then, execute it as follows →

```bash title="Nibbles/tools"
searchsploit --mirror linux/remote/45939.py
mv "${_##*/}" ssh_exploit.py
```

```bash="Nibbles/tools"
python2 !$
```

> [!INFO]-
>
> There are situations where, even if the *OpenSSH Version is earlier than the v7.7*, *User Enumration* does not work due due to some configuration or manual patches
>

In this case, nothing interesting is extracted

##### *80 - HTTP*

###### *Web Server Headers (Banner Grabbling)*

```bash
curl --silent --request GET --location --head http://10.129.96.84
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 200 OK
> Date: Wed, 23 Oct 2024 16:59:44 GMT
> Server: Apache/2.4.18 (Ubuntu)
> Last-Modified: Thu, 28 Dec 2017 20:19:50 GMT
> ETag: "5d-5616c3cf7fa77"
> Accept-Ranges: bytes
> Content-Length: 93
> Vary: Accept-Encoding
> Content-Type: text/html
> ```
>

Nothing interesting here

###### *Web Technologies*

- ***Whatweb***

```bash
whatweb http://10.129.96.84
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.96.84 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.96.84]
> ```
>

Nothing interesting here either

- ***Wappalyzer***

![[NIBBLES-20241023191936037.webp|250]]

It extracts that *PHP* is the *Server Language Programming*

This is interesting to know if some attack vector appear such as *File Upload* to chain it with a *RCE*

###### *Browser-Based Web Revision*

Once the web is accessed through `http://10.129.96.84`, the following page content is displayed →

![[NIBBLES-20241023190852646.webp|400]]
> ***Zoom In***

However, the following hint appears in the Page *Source Code* →

![[NIBBLES-20241023191312201.webp|400]]
> ***Zoom In***

The Following *Web Path* is leaked → **`http://10.129.96.84/nibbleblog/ `**

Accessing it returns the following

![[NIBBLES-20241023191602043.webp|400]]
> ***Zoom In***

Let's apply fuzzing to this Website to discover its content (files, directories...)

###### *Web Fuzzing*

Web fuzzing is applied to the leaked Web Path in the above source code using `gobuster`

Note that this *Web Scan* is exported as evidence

```bash title="Nibbles/scans"
gobuster dir --threads 100 --output ./webScan --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://10.129.96.84/nibbleblog
```

> [!NOTE]- *webScan*
>
> ```bash
> /themes               (Status: 301) [Size: 324] [--> http://10.129.96.84/nibbleblog/themes/]
> /admin                (Status: 301) [Size: 323] [--> http://10.129.96.84/nibbleblog/admin/]
> /plugins              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/plugins/]
> /README               (Status: 200) [Size: 4628]
> /languages            (Status: 301) [Size: 327] [--> http://10.129.96.84/nibbleblog/languages/]
> /content              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/content/]
>