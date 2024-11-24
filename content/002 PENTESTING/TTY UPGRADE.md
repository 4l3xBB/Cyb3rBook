---
Primary_category: "[[PENTESTING ROOT]]"
title: "TTY/PTY UPGRADE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[PENTESTING ROOT]]

#### Linux Terminal Upgrade

> ***[Reference](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#method-2-using-socat)***

Once a connection is stablished from/to the *Target*, It is recommended to upgrade the *Basic/Limited Shell* obtained into a *Fully Interactive TTY*

##### *TTY/PTY Creation*

###### *Script*

```bash title="Target"
script /dev/null -c bash # Spawn a Bash within a PTY ~ Pseudoterminal
C-z # SIGTSTP to background the previous Bash Process
```

###### *Python3*

```python
python3 -c 'import pty ; pty.spawn("/bin/bash")' # Spawn a Bash within a PTY
```

##### *TTY/PTY Upgrade*

Once a *PTY* i.e. a *Pseudoterminal* is generated, just proceed to upgrade it as follows →

> ***Attacker*** 🗡️

```bash
stty raw -echo # Disable Terminal Input Processing and disable input's echo
fg # Foregrounds the Shell send to the background
reset xterm # Restore the Term to its default value while maintaining some Raw behaviour
```

> ***Target*** 🎯 

```bash
export TERM=xterm-256color
export SHELL=/bin/bash
stty rows <NUMBER> columns <NUMBER>
. /etc/skel/.bashrc # Same as → source /etc/skel/.bashrc
```

---

#### Miscellaneous

##### *Windows*

###### *CMD to Powershell*

```powershell
powershell.exe -File -
```