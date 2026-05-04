---
Primary_category: "[[CREDENTIAL HUNTING]]"
title: "NETWORK CREDENTIAL HUNTING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CREDENTIAL HUNTING]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp[[LINUX PRIVESC]]

#### *Theory*

We can only extract data from the captured packets when a given protocol is not end-to-end encrypted

That is, if an attacker intercepts packets from an ***HTTP Connection*** between a client and a Web Server, the data contained in these packets cannot be extracted unless the attacker knows the key

In that case, since the ***Encryption Method*** used in a client-server architecture is usually a ***Symmetric Cypher***, the attacker could use that key to decrypt those packets and extract all the data in plain format

---

#### *TShark*

> ***[TShark](https://www.wireshark.org/docs/man-pages/tshark.html)***

##### *HTTP*

> ***Encrypted Counterpart → HTTPS***

###### *HTTP POST Data (POST Parameters' value)*

> ***From a given Capture File e.g. Pcap***

```bash
tshark -r <CAPTURE_FILE> -Y 'http.request.method == POST' -T fields -e http.file_data 2> /dev/null
```

> ***From Live Traffic***

```bash
tshark --interface <INTERFACE> -Y 'http.request.method == POST' -T fields -e http.file_data 2> /dev/null
```

##### *SNMP*

> ***Encrypted Counterpart → SNMPv3***

###### *SNMP Community String*

> ***From a given Capture File e.g. Pcap***

```bash
tshark -r <CAPTURE_FILE> -Y 'snmp' -T fields -e snmp.community 2> /dev/null | sort -u
```

> ***From Live Traffic***

```bash
tshark --interface <INTERFACE> -Y 'snmp' -T fields -e snmp.community 2> /dev/null
```

##### *FTP*

> ***Encrypted Counterpart → FTPS***

###### *FTP Login (User and Password)*

> ***From a given Capture File e.g. Pcap***

```bash
tshark -r <CAPTURE_FILE> -Y 'ftp.request.command == USER or ftp.request.command == PASS' -T fields -e ftp.request.command -e ftp.request.arg -E separator=: 2> /dev/null
```

> ***From Live Traffic***

```bash
tshark --interface <INTERFACE> -Y 'ftp.request.command == USER or ftp.request.command == PASS' -T fields -e ftp.request.command -e ftp.request.arg -E separator=: 2> /dev/null
```

---

#### *PCredz*

> ***[PCredz](https://github.com/lgandx/PCredz)***

##### *Setup*

```bash
apt install -y -- python3-pip libpcap-dev file
```

```bash
python3 -m venv .venv
source !$/bin/activate
```

```bash
pip3 install Cython python-libpcap
```

```bash
git clone https://github.com/lgandx/PCredz PCredz
```

##### *Creds Extraction*

> ***From a given Capture File e.g. Pcap***

```bash
python3 PCredz -v -f <CAPTURE_FILE>
```