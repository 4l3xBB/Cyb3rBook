---
Primary_category: "[[SETUP]]"
title: SXHKD
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
  - pentesting👹
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**[sxhkd](https://github.com/baskerville/sxhkd) → Simple X Hotkey Daemon**

`sxhkd` process is listen and reacts to input events and execute commands according to them

Most of that commands are `bspc` invocations. That `bspc` commands write in `bspwm`'s socket

```bash
$ pgrep --list-full --exact sxhkd
1643 /usr/local/bin/sxhkd
```

`sxhkd` is launched by [[BSPWM|bspwm]] from [[BSPWM#_bspwmrc_|bspwmrc]] script

All input events such as hotkeys are defined within its configuration file.

It declares all hotkeys related to launch and deploy processes and its functionalities such as:

- [[BSPWM|bspwm]] → Windows Generation, Configuration Daemon Reload, Shortcuts...
<br>
- [[KITTY|kitty]] → Launchs Terminal Emulator
<br>
- [[ROFI|roffi]] → Deploys Program Launcher
<br>
- Other processes such as _Firefox, Burpsuite..._

**Configuration File → `~/.config/sxhkd/sxhkdrc`**

**More information [here](https://github.com/baskerville/sxhkd)**

##### sxhkd Installation

> [!IMPORTANT]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `sxhkd`
>
> There are some dependencies that are needed in the following installation steps
>

Clone the [sxhkd's Github Repository](https://github.com/baskerville/sxhkd) →

```bash
$ cd ~/Downloads
$ git clone https://github.com/baskerville/sxhkd.git sxhkd
```

```bash
$ cd !$ # cd ~/Downloads/sxhkd
$ make
$ sudo make install
```

`~/.config/sxhkd` directory and `sxhkdrc` file →

```bash
$ mkdir -p ~/.config/sxhkd
$ cp ~/Downloads/bspwm/examples/sxhkdrc !$
```

> [!CAUTION]-
>
> Be aware that `sxhkdrc`'s _example/template_ file is copied from the `bspwm` repository rather than from the `sxhkd` one

##### Configuration File

**Copy from [here](https://pastebin.com/qHuKHsX6) the `sxhkdrc` Full Configuration File** 

###### bspwm Section

All _hotkey/key binding_ related to `bspwm` appears in the above link

###### Kitty and Rofi Sections

```bash
################################################
############# WM INDEPENDENT KEYS  #############
################################################

## ----------------------------- ##
## ----- terminal emulator ----- ## KITTY
## ----------------------------- ##

super + Return
  /opt/kitty/bin/kitty

## ---------------------------- ##
## ----- program launcher ----- ## ROFI
## ---------------------------- ##

super + d
  /usr/bin/rofi -show run
```

###### Other Processes Section

```bash
####################################################
#################  OTHER PROGRAMS  #################
####################################################

# Open Firefox (Browser)
super + shift + f
    /usr/bin/firefox

# Open Burpsuite (HTTP Proxy)
super + shift + b
    /usr/bin/burpsuite-launcher

# i3lock-fancy (Screen Lock)
super + shift + x
    /usr/bin/i3lock-fancy
```