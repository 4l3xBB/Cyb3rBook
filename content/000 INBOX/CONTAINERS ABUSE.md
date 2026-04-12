---
Primary_category: "[[LINUX PRIVESC]]"
title: "CONTAINERS ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *LXD | LXC*

##### *LXD Group*

> ***See [[LINUX PRIVILEGED GROUPS#LXC LXD|Linux Privileged Groups]]***

---

#### *Docker*

##### *Docker Shared Directories*

It's important to take into account the presence of *shared directories* or *volume mounts* within the container itself

This type of feature allows that specific directories or files in the host system are accesible from the container

So, a *system administrator* or *devops* could have set up a *shared directory* between the host system and the container during the creation of the latter

Therefore, let's suppose we have compromised a given web application and stablished a remote connection to the target through a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***, let's say by leveraging an arbitrary ***[[FILE UPLOAD|File Upload]]***

In this case the web application was deployed using a *docker container*, so we have landed within a container file system

However, one of the first things we need to do from an attacker's perspective when enumerating a docker container is to look for non-standard directories, as these are likely to be *shared directories* between the host system or other containers and the container in question

So, we could start by listing the content of the container's *root* directory

```bash
ls -l /
```

```bash
find / -maxdepth 2 \( -path '/usr' -o -path '/opt' -o -path '/proc' \) -prune -o -perm -0001 -type d 2> /dev/null
```

##### *Docker Sockets*



##### *Docker Group*