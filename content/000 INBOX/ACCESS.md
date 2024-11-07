---
Primary_category: "[[EASY]]"
title: ACADEMY
draft: true
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

![[ACCESS-20241107182702583.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Acess && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash title="MACHINE_NAME"
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

```bash title="Access/scans"
ping -c1 10.129.232.68
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.232.68 (10.129.232.68) 56(84) bytes of data.
> 64 bytes from 10.129.232.68: icmp_seq=1 ttl=127 time=45.3 ms
>
> --- 10.129.232.68 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 45.280/45.280/45.280/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash title="Access/scans"
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.232.68
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="MACHINE_NAME/scans/AllPorts"
> # Nmap 7.94SVN scan initiated Thu Nov  7 19:05:42 2024 as: nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.232.68
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.232.68 ()  Status: Up
> Host: 10.129.232.68 ()  Ports: 21/open/tcp//ftp///, 23/open/tcp//telnet///, 80/open/tcp//http///    Ignored State: filtered (65532)
> # Nmap done at Thu Nov  7 19:06:08 2024 -- 1 IP address (1 host up) scanned in 26.49 seconds 
> ```
>

**Open Ports → 21, 32, 80**

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash title="Access/Scans"
extractPorts allPorts
```

> [!NOTES]- *ExtractPorts Output*
>
> ```bash title="Access/scans"
> [+] Extracting information...
>
>     [+] IP Address: 10.129.232.68
>     [+] Open Ports: 21,23,80
>
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="Access/Scans"
nmap -p21,23,80 -sCV -oN targeted 10.129.232.68
```

> [!NOTES]- *Targeted Output*
>
> ```bash title="Access/scans/Targeted"
> # Nmap 7.94SVN scan initiated Thu Nov  7 19:13:05 2024 as: nmap -p21,23,80 -sCV -oN targeted 10.129.232.68
> Nmap scan report for 10.129.232.68 (10.129.232.68)
> Host is up (0.064s latency).
>
> PORT   STATE SERVICE VERSION
> 21/tcp open  ftp     Microsoft ftpd
> | ftp-syst:
> |_  SYST: Windows_NT
> | ftp-anon: Anonymous FTP login allowed (FTP code 230)
> |_Can't get directory listing: PASV failed: 425 Cannot open data connection.
> 23/tcp open  telnet?
> 80/tcp open  http    Microsoft IIS httpd 7.5
> |_http-server-header: Microsoft-IIS/7.5
> | http-methods:
> |_  Potentially risky methods: TRACE
> |_http-title: MegaCorp
> Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
>
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Nov  7 19:16:06 2024 -- 1 IP address (1 host up) scanned in 181.40 seconds
> ```
>

##### *80 - HTTP*

###### *General Information*

- ***Banner Grabbling***

```bash title="Access/scans"
nc -nv 10.129.148.205 80 <<< ""
```

Nothing reported

- ***Server HTTP Response Headers***

```bash title="Access/scans"
curl --silent --request GET --location --head "http://10.129.232.68"
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 200 OK
> Content-Type: text/html
> Last-Modified: Thu, 23 Aug 2018 23:33:43 GMT
> Accept-Ranges: bytes
> ETag: "44a87bb393bd41:0"
> Server: Microsoft-IIS/7.5
> X-Powered-By: ASP.NET
> Date: Thu, 07 Nov 2024 18:24:02 GMT
> Content-Length: 391
> ```
>

According to some of the above headers, we extract the following information →

- ***Web Server → Microsoft IIS***
- ***Server-Side Programming Language → ASP.NET***

Therefore, we can start thinking about using *.ASP* as a valid extension when performing the *Web Resources Enumeration*  via *Gobuster, wfuzz...*

###### *Web Technologies #0*

Let's see the Web Technologies →

```bash title="Access/scans"
whatweb "http://10.129.232.68"
```

> [!NOTES]- *Command Output*
>
> ```bash
> http://10.129.232.68 [200 OK] Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/7.5], IP[10.129.232.68], Microsoft-IIS[7.5], Title[MegaCorp], X-Powered-By[ASP.NET]
> ```
>

Practically the same information is obtained, *IIS* as *Web Server* and *ASP.NET* as *Server-Side Programming Language*

###### *Nmap Fuzzing*

Before proceeding with _Directory Enumeration_ using _known Fuzzers_, such as `gobuster` or `wfuzz`, run the _Nmap Small Fuzzer_ to get an idea of the available resources

```bash title="Access/scans"
nmap -p80 --script http-enum -oN simpleWebScan 10.129.232.68
```

> [!NOTE]- *Command Output*
>
> ```bash

> ```
>

###### *Web Technologies #1*

Once you access the website through the browser, just check again the *Web Technologies* reported by *Wappalyzer*

They may differ from those reported by *Whatweb*

![[ACCESS-20241107194147563.webp|275]]
> ***Zoom In***

But not in this case, nothing new here

###### *Browser-Based Web Enumeration*

The only thing displayed in the Web is the following image →

![[ACCESS-20241107194449955.webp|400]]
> ***Zoom In***

Nothing interesting is shown in the *HTML Souce Code* either

###### *Web Resources Fuzzing*

Let's check any available resources, let's go first with the directory enumeration using *WFuzz* →

```bash title="Access/scans"
wfuzz -c --hc 404 -t 20 -f fullWebScan -w /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.232.68/FUZZ
```

> [!NOTE]- *Command Output*
>
> ```bash
> Target: http://10.129.232.68/FUZZ
> Total requests: 220545
> ==================================================================
> ID    Response   Lines      Word         Chars          Request
> ==================================================================
>
> Total time: 0
> Processed Requests: 156221
> Filtered Requests: 156220
> Requests/sec.: 0
> ```
>

No directory has been found with the above scan, let's try again but specifying some extensions as follows, this time using *gobuster* →

```bash title="Access/scans"
gobuster dir --threads 200 --output fullWebScan --extensions asp,aspx,html --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://10.129.232.68
```

> [!NOTE]- *Command Output*
>
> ```bash
> /index.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> /Index.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> /INDEX.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> ```
>

Nothing interesting either, let's check the services running on the remaining ports

##### *23 - Telnet*



##### *21 - FTP*



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