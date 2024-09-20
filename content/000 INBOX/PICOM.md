---
Primary_category: "[[SETUP]]"
title: PICOM
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**`picom` → Compositor for X**

It manages the way **Windows** and **Graphical Elements** are displayed on the screen

Basically, It combines the above elements, called _**Buffers**_, into a **Final Image** that is displayed to the user

Note that `picom` is a fork of [Compton](https://github.com/yshui/picom/blob/next/History.md), which is already _deprecated_

This binary is executed as a daemon like most of the tools in this [[SETUP|setup]]

```bash
$ pgrep --list-full --exact picom
1930 /usr/local/bin/picom
```

`picom` is executed by [[BSPWM|bspwm]] from the [[BSPWM#*bspwmrc*|bspwmrc]] script through the `picom`'s binary →

```bash
# ~/.config/bspwm/bspwmrc - Picom Launch
checkProcess picom || launchProcess picom
```

> [!INFO]-
>
> Note that `checkProcess` and `launchProcess` are [[SHELL SCRIPTING|shell functions]] declared in the `~/.config/bspwm/src/bspwmrc.sh` [[BSPWM#*src/bspwmrc.sh*|source file]]
>
> Above implementation would be something similar to the usual approach through `pgrep --exact process_name || binary|script_path &` 
>
> ```bash
> #!/usr/bin/env sh
> $ pgrep --exact picom > /dev/null 2>&1 || picom &
> ```

**Configuration File → `~/.config/picom/picom.conf`**

**More information [here](https://github.com/yshui/picom)**

---

##### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `picom`
>
> There are some dependencies that are needed in the following installation steps. That is why  you should see the note above first
>

First of all, install the following dependencies →

> Only _Debian Based_ distros. Check [this](https://github.com/yshui/picom#dependencies) for the _Fedora/RHEL_ ones

```bash
$ apt install -y -- libconfig-dev libdbus-1-dev libegl-dev libev-dev libgl-dev libepoxy-dev libpcre2-dev libpixman-1-dev libx11-xcb-dev libxcb1-dev libxcb-composite0-dev libxcb-damage0-dev libxcb-glx0-dev libxcb-image0-dev libxcb-present-dev libxcb-randr0-dev libxcb-render0-dev libxcb-render-util0-dev libxcb-shape0-dev libxcb-util-dev libxcb-xfixes0-dev meson ninja-build uthash-dev
```

Clone the [picom's Github Repository](https://github.com/yshui/picom) →

```bash
$ cd ~/Downloads
$ git clone https://github.com/yshui/picom.git picom
```

To build →

```bash
$ cd !$ # cd ~/Downloads/picom
$ meson setup --buildtype=release build
$ ninja -C build
```

To install →

```bash
$ ninja -C build install
```

> [!INFO]-
>
> Note that the default installation dir will be `/usr/local`. To change it, just use →
>
> ```bash
> $ meson configure -Dprefix=PATH build
> ```
>

That's it

```bash
$ command -V picom
picom is /usr/local/bin/picom
```

---

##### Configuration File