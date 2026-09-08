---
Primary_category: "[[002 PENTESTING/PROTOCOLS AND SERVICES/PROTOCOLS AND SERVICES|PROTOCOLS AND SERVICES]]"
title: "22 - SSH"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[002 PENTESTING/PROTOCOLS AND SERVICES/PROTOCOLS AND SERVICES|PROTOCOLS AND SERVICES]]

#### *Theory*

> 🛠️⌛

***SSH → Secure Shell***

---

#### *SSH Credential Harvesting*

##### *Workflow*

In this case, we can either force a remote *SSH* client to connect to us through a certain port, such as the *SSH* standard port, or wait for an incoming connection

Regardless of how we receive the connection and its subsequent authentication, it would be interesting to view the plain credentials of any client that authenticates against our *SSH* server

The caveat is that this is not possible with the default configuration of the standard *OpenSSH* server

However, we can set up either a *MITM* tool that listens for incoming authentications on the *SSH* port *( e.g. TCP Port 22 )* and redirects them to the *OpenSSH* server, such as ***[SSH-MITM](https://github.com/ssh-mitm/ssh-mitm)***, or a *honeypot SSH Server* that receives authentications and emulates a *UNIX* system in high interaction mode, such as ***[Cowrie](https://github.com/cowrie/cowrie)***

##### *MITM*

 - ***[SSH-MITM](https://github.com/ssh-mitm/ssh-mitm)***

***Setup***

```bash
curl --silent --location --request GET 'https://github.com/ssh-mitm/ssh-mitm/releases/latest/download/ssh-mitm-x86_64.AppImage' --output ssh-mitm
```

```bash
chmod 700 !$
```

***Usage***

```bash
./ssh-mitm-x86_64.AppImage server --listen-address '<ADDRESS>' --listen-port '<PORT>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> ./ssh-mitm-x86_64.AppImage server --listen-address 0.0.0.0 --listen-port 2222
> ```
>

##### *Honeypot*

- ***[Cowrie](https://github.com/cowrie/cowrie)***

> ***Deploy as a user other than Root***

***Setup***

```bash
mkdir Cowrie
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install cowrie
cowrie init
cowrie start
```

***Usage***

```bash
tail -F var/log/cowrie/cowrie.log
```