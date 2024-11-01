---
Primary_category: "[[EASY]]"
title: "ACADEMY"
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

###### PRIMARY CATEGORY → [[MEDIUM]]

###### PRIMARY CATEGORY → [[HARD]]

###### PRIMARY CATEGORY → [[INSANE]]

#### Summary

- ***Summary A***
- ***Summary B***
- ***Summary C***
- ***Summary D***
- ***Summary E***

![[ACADEMY-20241101164644590.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir MACHINE_NAME && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash title="MACHINE_NAME"
mkt
```

> [!IMPORTANT]- *Tree*
>
> ```bash
> .
> ├── evidence
> │   ├── creds
> │   ├── data
> │   └── screenshots
> ├── logs
> ├── scans
> ├── scope
> └── tools
> ```
>

---

#### Recon

##### *OS Identification*

First, proceed to identify the *Target Operative System*. This can be done by a simple `ping` taking into account the *TTL Unit*

The standard values are →

- ***About 64 → Linux***
- ***About 128 → Windows***

```bash title="MACHINE_NAME/scans"
ping -c1 10.129.96.84
```

> [!NOTE]- *Command Output*
>
> ```bash
> ```
>

As mentioned, according to the TTL, It seems that It is a ***________ Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash title="MACHINE_NAME/scans"
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn -oG allPorts TARGET
```

> [!NOTE]- *AllPorts Output*
>
> ```bash title="Nibbles/scans/AllPorts"
> ```
>

**Open Ports →**

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash title="MACHINE_NAME/Scans"
extractPorts allPorts
```

> [!NOTES]- *Command Output*
>
> ```bash
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash title="MACHINE_NAME/Scans"
nmap -p22,80 -sCV -oN targeted TARGET
```

> [!NOTES]- *Command Output*
>
> ```bash
> ```
>

##### *OS Version (Codename)*

In *Linux Systems*, the *Operative System Version* could be extracted through *Launchpad*

According to the **Version Column Data** of the [[#Comprehensive Scan]], proceed as follows →

- ***PORT NUMBER - PROTOCOL NAME***

> ***[Reference](LINK TO LAUNCHPAD)***

```bash title="Firefox"
SERVICE VERSION e.g. OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 site:launchpad.net
```

- ***PORT NUMBER - PROTOCOL NAME***

> ***[Reference](LINK TO LAUNCHPAD)***

```bash title="Firefox"
SERVICE_VERSION
```

***Codename → [Distro Codename e.g. Debian Buster](LINK TO INFO ABOUT THE CODENAME)***

This can be verified once the [[SHELL SCRIPTING|shell]] is obtained, i.e. the system has been compromised

There are several ways to carry out it →

```bash
cat /etc/os-release
```

```bash
hostnamectl # If System has been booted via Systemd
```

```bash
lsb_release -a
```

```bash
cat /etc/issue
```

```bash
cat /proc/version
```

##### *22 - SSH*

***OpenSSH Version → v7.2***

All the *OpenSSH* Versions prior to the *v7.7* one are vulnerable to a **System User Enumeration**

> ***[Reference](https://www.rapid7.com/db/modules/auxiliary/scanner/ssh/ssh_enumusers/)***

**CVE-2018-15473** → ***OpenSSH < v7.7***

```bash
searchsploit ssh user enumeration
```

To get the *ExploitDB* links related to above exploits →

```bash
searchsploit --www ssh user enumeration
```

***Exploit →  OpenSSH < 7.7 - User Enumeration (2)***

To examine it →

```bash
searchsploit --examine linux/remote/45939.py |& cat --language python
```

> **This exploit requires *Python2***

Then, execute it as follows →

```bash title="MACHINE_NAME/tools"
searchsploit --mirror linux/remote/45939.py
mv "${_##*/}" ssh_exploit.py
```

```bash title="MACHINE_NAME/tools"
python2 !$
```

> [!INFO]-
>
> There are situations where, even if the *OpenSSH Version is earlier than the v7.7*, *User Enumeration* does not work due due to some configuration or manual patches
>

In this case, nothing interesting is extracted

##### *PORT NUMBER - PROTOCOL NAME*

##### *e.g. 80 - HTTP*

---

#### Exploitation

##### *Vulnerability Name or Vuln Chaining*

##### *e.g. RCE via Authenticated File Upload*

---
#### Shell as Web User

Once a connection via *Reverse Shell* is stablished, just proceed as follows to upgrade the obtained shell to a *Fully Interactive TTY*

> ***[Reference](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)***

##### *Script*

```bash title="Target"
script /dev/null -c bash
<C-z>
```

```bash title="Attacker"
stty raw -echo ; fg
reset xterm
```

```bash title="Target"
export TERM=xterm-256color
export SHELL=/bin/bash
. /etc/skel/.bashrc
stty rows <ROWS> columns <COLUMNS>
```

---

#### Privesc #1

***Initial Non-Privileged User → USERNAME***

##### *PRIVESC VECTOR A*

#### Privesc #2 (If exists)

##### *PRIVESC VECTOR A*

---

#### Custom Exploits

##### *EXPLOIT A*

> ***[Reference (if exists)]()***

> [!IMPORTANT]- *EXPLOIT NAME*
>
> ```python
> ```
>

***\[\[ EXPLOIT EXECUTION GIF ]]***