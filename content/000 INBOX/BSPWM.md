---
Primary_category: "[[SETUP]]"
title: BSPWM
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
  - pentesting👹
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**[bspwm](https://github.com/baskerville/bspwm) → Binary Space Partitioning Window Manager**

`bspwm` is launched as a process and It receives as an argument a _socket_ on which to listen

```bash
$ pgrep --list-full --exact bspwm
2604 bspwm -s /tmp/bspwm_0_0-state -o 4 # Note -s option argument
```

```bash
$ file /tmp/bspwm_0_0-socket
/tmp/bspwm_0_0-socket: socket
```

`bspc`  binary writes messages to `bspwm`'s socket. Those messages are all the commands executed in `bspwmrc` init file

```bash
$ command -V bspc
bspc is /usr/local/bin/bspc
```

> [!IMPORTANT]-
>
> Note that `bspwm` does not handle any input since its _fd 0_ is pointing to `/dev/null`
>
> ```bash
> $ file /proc/$( pgrep --exact bspwm )/fd/0
> /proc/2604/fd/0: symbolic link to /dev/null
> ```
> Thus, [[SXHKD|sxhkd]] translate keyboard and pointers events to `bspc`
>
> `sxhkd` process listens to an input event and executes the command related to it. It can be any command, most of them are the `bspc` ones that write to `bspwm`'s _socket_
>
> That `sxhkd`'s input event is usually a keyboard shortcut
>

**Configuration File → ``$XDG_CONFIG_HOME/bspwm/bspwmrc``**

**More information [here](https://github.com/baskerville/bspwm)**

##### BSPWM Installation

First of all, install the following requirements →

```bash
$ apt install -y -- build-essential git vim xcb libxcb-util0-dev libxcb-ewmh-dev libxcb-randr0-dev libxcb-icccm4-dev libxcb-keysyms1-dev libxcb-xinerama0-dev libasound2-dev libxcb-xtest0-dev libxcb-shape0-dev
```

