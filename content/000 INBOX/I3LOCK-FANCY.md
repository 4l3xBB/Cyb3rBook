---
Primary_category: "[[DESKTOP SETUP]]"
title: I3LOCK-FANCY
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses: 
---

###### PRIMARY CATEGORY → [[DESKTOP SETUP]]

**[I3LOCK-FANCY](https://github.com/meskarune/i3lock-fancy)** → ***Screen Locker***

This *[[BASH|Bash]]* Script performs the following actions →

- ***Destkop Screenshoot***
<br>
- ***Background Blur***
<br>
- ***Lock Icon Adition***

The current user's password must be entered correctly in order to access the system

`i3lock-fancy` is launched by the [[SXHKD|sxhkd]] daemon when a certain input event, such as a *Hotkey*, is performed

```bash title="~/.config/sxhkd/sxhkdrc"
# I3Lock-Fancy Launch
super + shift + x
	/usr/bin/i3lock-fancy
```


![[I3LOCK-FANCY.gif|400]]

**More information [here](https://github.com/meskarune/i3lock-fancy)**

---

#### Installation

> ***[Reference](https://github.com/meskarune/i3lock-fancy#installation)***

##### Dependencies

```bash
sudo apt install -y -- imagemagick util-linux
```

##### *I3Lock-Fancy*

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm]] and the [[SXHKD|sxhkd's]] installation before proceeding with this one related to `I3Lock-Fancy`
>
> There are some dependencies that are needed in the following installation steps
>

Git clone the *[Github Repository](https://github.com/meskarune/i3lock-fancy)* and Install it as follows →

```bash
sudo git clone https://github.com/meskarune/i3lock-fancy /opt/i3lock-fancy
```

```bash
cd !$
sudo make install
```

That's it!

```bash
$ command -V i3lock-fancy
i3lock-fancy is /usr/bin/i3lock-fancy
```

> [!IMPORTANT]-
>
> *I3Lock-Fancy* can also be installed via the *System Package Manager*, such as `apt`
>
> Please note that often the *Package Version* installed from the System Repositories is older than the one installed from the *Project's Github Repository*
>
> It is always recommended to have the latest version of a software installed to avoid unintended aspects such as *Bugs* or *Vulnerabilities*
>

To run the component, simply map its launch to a certain *keybind* through a *Hotkey Daemon* such as [[SXHKD|sxhkd]]

```bash title="~/.config/sxhkd/sxhkdrc"
super + shift + x # Windows + Shift + x
    /usr/bin/i3lock-fancy
```

***Shortcut*** → **`Super-S-x`**