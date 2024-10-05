---
Primary_category: "[[PENTESTING]]"
title: SETUP
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
  - pentesting👹
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[PENTESTING]]

#### Environment Customization

![[SETUP-20240921163243628.webp|400]]

| **SOURCES** | | |
| --- | --- | --- |
| **[Parrot](https://parrotsec.org/)** | Operative System | **[See more](https://parrotsec.org/docs/)** |
| **[bspwm](https://github.com/baskerville/bspwm)** | Windows Manager | **[See more](https://wiki.archlinux.org/title/Bspwm)** |
| **[sxhkd](https://github.com/baskerville/sxhkd)** | Hotkey Daemon | **[See more](https://wiki.archlinux.org/title/Sxhkd)** |
| **[Polybar](https://github.com/polybar/polybar)** | Status Bar | **[See more](https://wiki.archlinux.org/title/Polybar)** |
| **[Picom](https://github.com/yshui/picom)** | Windows Visualizer (Compositor) | **[See more](https://wiki.archlinux.org/title/Picom)** |
| **[Rofi](https://github.com/davatorium/rofi)** | Application Launcher | **[See more](https://wiki.archlinux.org/title/Rofi)** |
| **[Feh](https://github.com/derf/feh)** | Image Viewer (Desktop Wallpaper) | **[See more](https://wiki.archlinux.org/title/Feh)** |
| **[Kitty](https://github.com/kovidgoyal/kitty)** | Terminal Emulator | **[See more](https://wiki.archlinux.org/title/Kitty)** |
| **[ZSH](https://wiki.archlinux.org/title/Zsh_(Espa%C3%B1ol))** | [[SHELL SCRIPTING\|Shell]] | **[See more](https://zsh.sourceforge.io/Doc/Release/zsh_toc.html)** |
| **[NEOVIM](https://github.com/neovim/neovim)** | Text Editor | **[See more](https://neovim.io/doc/)** |

##### Components ⟡

- ![](https://i.pinimg.com/originals/b0/3b/da/b03bda611e0210d07641bbb4bddc3c6d.jpg)
	- [[ZSH]]
- ![](https://img.freepik.com/fotos-premium/diseno-cartel-abstracto-fondo-cristal-lugar-texto-geometrico-cristalizado_1197797-166685.jpg)
	- [[BSPWM]]
- ![](https://img.freepik.com/fotos-premium/cubierta-fondo-logotipo-triangulo-circulo-estrellas_1197797-48344.jpg)
	- [[SXHKD]]
- ![](https://i.pinimg.com/736x/ef/31/f8/ef31f8f451d8efe5fe14a94734aa3baf.jpg)
	- [[POLYBAR]]
- ![](https://wallpapercave.com/wp/wp5181877.jpg)
	- [[PICOM]]
- ![](https://e0.pxfuel.com/wallpapers/285/345/desktop-wallpaper-linux-mint-black-1440p-resolution-background-and.jpg)
	- [[ROFI]]
- ![](https://i0.wp.com/endeavouros.com/wp-content/uploads/2022/06/stars.png?resize=750%2C422&ssl=1)
	- [[FEH]]
- ![](https://i.pinimg.com/1200x/06/b5/40/06b5401289915f753505baba16827c6a.jpg)
	- [[KITTY]]
- ![](https://img.freepik.com/premium-photo/latest-drawing-room-interior-decorating-ideas-designs-v-letter-logo-designs_1020867-119148.jpg)
	- [[NEOVIM]]
	
<br>

##### Miscellaneous Δ

- ![](https://mir-s3-cdn-cf.behance.net/projects/404/8b9d2f168858545.6441d700ed582.jpg)
	- [[BAT]]
- ![](https://e1.pxfuel.com/desktop-wallpaper/753/322/desktop-wallpaper-debian-blue-neon-logo-01-debian-blue.jpg)
	- [[LSD]]
- ![](https://p4.wallpaperbetter.com/wallpaper/625/89/275/archlinux-digital-art-linux-arch-linux-tech-hd-wallpaper-preview.jpg)
	- [[FZF]]

<br>

##### Information 🛈

**This Custom Linux Environment is deployed in _Parrot OS_ 🦜**

###### Configuration Files

***[Reference](https://github.com/4l3xBB/Env-Setup)***

All necessary _Configuration Files_ can be found in the above link or in the _Components Documentation_

> [!IMPORTANT]- _Configuration Files Structure_
>
> ```bash
> .
> ├── bspwm
> │   ├── bin
> │   │   └── bspwm_resize.sh
> │   ├── bspwmrc
> │   └── src
> │       └── bspwmrc.sh
> ├── kitty
> │   ├── color.ini
> │   └── kitty.conf
> ├── picom
> │   └── picom.conf
> ├── polybar
> │   ├── bin
> │   │   ├── launch.sh
> │   │   └── modules
> │   │       ├── ethernet_status.sh
> │   │       ├── target_to_hack.sh
> │   │       └── vpn_status.sh
> │   ├── current.ini
> │   └── workspace.ini
> ├── README.md
> ├── sxhkd
> │   └── sxhkdrc
> └── zsh
>     ├── plugins
>     │   └── zsh-syntaxhighlighting
>     │       └── themes
>     │           └── dracula.zsh
>     ├── src
>     │   └── custom.zsh
>     ├── themes
>     │   └── p10k
>     │       ├── root
>     │       │   └── .p10k.zsh
>     │       └── users
>     │           └── .p10k.zsh
>     └── .zshrc
> ```

###### Deployment Flow

**The Deployment Flow would be →**

***\[\[ EXCALIDRAW BANNER ]]***

##### Previous Steps

Before start with the above Components' Installation and Configuration →

```bash
$ sudo apt update # Repositories' update
```

```bash
$ sudo parrot-upgrade
```

> [!CAUTION]-
>
> Be aware that the following upgrading way is not recommended →
>
> ```bash
> $ sudo apt upgrade # Or sudo apt full-upgrade
> ```
> The above way may arises several errors such as the _Kernel Panic_ ones on reboot due to specific packages
>
> Therefore, the [Parrot](https://parrotsec.org/docs/) Team has created `parrot-upgrade`, a simple bash script, to avoid the mentioned problems
>
> ```bash
> $ command -V parrot-upgrade
> parrot-upgrade is /usr/bin/parrot-upgrade
> ```
>
> ```bash
> #!/bin/bash
> set -e
> DEBIAN_FRONTEND="noninteractive"
> DEBIAN_PRIORITY="critical"
> DEBCONF_NOWARNINGS="yes"
> export DEBIAN_FRONTEND DEBIAN_PRIORITY DEBCONF_NOWARNINGS
> apt update || echo failed to update index lists
> dpkg --configure -a || echo failed to fix interrupted upgrades
> apt --fix-broken --fix-missing install || echo failed to fix conflicts
> apt -y --fix-broken --fix-missing full-upgrade
> apt -y full-upgrade
> ```
>
> Thus, on _Parrot OS_, instead of this →
>
> ```bash
> $ sudo apt update && sudo apt upgrade -y # Wrong!
> ```
> Do this →
>
> ```bash
> $ sudo apt update && sudo parrot-upgrade
> ```

 Environment Dependencies →

```bash
$ sudo apt install -y -- zenity build-essential git vim xcb libxcb-util0-dev libxcb-ewmh-dev libxcb-randr0-dev libxcb-icccm4-dev libxcb-keysyms1-dev libxcb-xinerama0-dev libasound2-dev libxcb-xtest0-dev libxcb-shape0-dev
```

Additional Packages →

```bash
$ sudo apt install -y -- scrot scrub xclip bar locate ranger neofetch wmname acpi imagemagick cmatrix
```

---

#### Additional Notes

##### Sessions Management and Shutdown

During the installation of the above components, the user will have to be shutting down, restarting, logging out or simply blocking (i.e _lock_) the current session

This actions can be performed as follows →

###### Shut Down

```bash
$ sudo poweroff
```

###### Restart

```bash
$ sudo reboot
```

###### Log out

```bash
$ loginctl list-sessions # Get the Session's Number
$ loginctl terminate-session <session_number> # Finish a Session
```

> [!INFO]-
>
> If the above command is not available because of the system has not been initialized with _systemd_, use the following one →
> 
> ```bash
> $ pkill -SIGTERM --euid "$( id -u )" # Or pkill -15
> ```
>
> This allows to all the processes related to the _EUID_ to terminate corrrectly and clean up resources
>
> If any process does not respond to the `SIGTERM` signal, just sent a `KILL` signal to it 
>
> ```bash
> $ pkill -KILL --euid "$( id -u )" # Or pkill -9
> ```
>
> Note that all signal types and their associated numbers can be enumerated as follows through these [[Shell Builtins|shell builtins]] →
>
> ```bash
> $ kill -l # Or trap -l
> ```
>

###### Session Lock

It can be handled through the _X Session Manager_ called `lightDM`

To interact with the `lightdm` daemon, the `dm-tool` binary comes into actions

```bash
$ dm-tool switch-to-greeter # Like the Change-User option in Windows
```

```bash
$ dm-tool lock # Like Windows + L in Windows
```

> [!INFO]-
>
> If `lightdm` does not manages the user sessions and the host has been booted with systemd (i.e. ***PID 1***), then try this →
>
> ```bash
> $ loginctl list-sessions
> $ loginctl lock-session SESSION_NUMBER
> ```