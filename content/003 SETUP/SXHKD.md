---
Primary_category: "[[DESKTOP SETUP]]"
title: SXHKD
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses: 
---

###### PRIMARY CATEGORY → [[DESKTOP SETUP]]

**[SXHKD](https://github.com/baskerville/sxhkd) → Simple X Hotkey Daemon**

`sxhkd` process is listen and reacts to input events and execute commands according to them

Most of that commands are `bspc` invocations. That `bspc` commands write in `bspwm`'s socket

```bash
$ pgrep --list-full --exact sxhkd
1643 /usr/local/bin/sxhkd
```

`sxhkd` is launched by [[BSPWM|bspwm]] from [[BSPWM#*bspwmrc*|bspwmrc]] script →

```bash
# ~/.config/bspwm/bspwmrc - sxhkd Binary Launch
checkProcess sxhkd || launchProcess sxhkd
```

> [!INFO]-
>
> Note that `checkProcess` and `launchProcess` are [[SHELL SCRIPTING|shell functions]] declared in the `~/.config/bspwm/src/bspwmrc.sh` [[BSPWM#*src/bspwmrc.sh*|source file]]
>
> Above implementation would be something similar to the usual approach through `pgrep --exact process_name || binary|script_path &` 
>
> ```bash
> #!/usr/bin/env sh
> pgrep --exact sxhkd > /dev/null 2>&1 || sxhkd &
> ```

All input events such as hotkeys are defined within its configuration file.

It declares all hotkeys related to launch and deploy processes and its functionalities such as:

- *[[BSPWM|bspwm]]* → Windows Generation, Configuration Daemon Reload, Shortcuts...
<br>
- *[[KITTY|kitty]]* → Launchs Terminal Emulator
<br>
- *[[ROFI|roffi]]* → Deploys Program Launcher
<br>
- Other processes such as _Firefox, Burpsuite..._

**Configuration File → `~/.config/sxhkd/sxhkdrc`**

**More information [here](https://github.com/baskerville/sxhkd)**

---

#### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `sxhkd`
>
> There are some dependencies that are needed in the following installation steps
>

Clone the [sxhkd's Github Repository](https://github.com/baskerville/sxhkd) →

```bash
cd ~/Downloads
git clone https://github.com/baskerville/sxhkd.git sxhkd
```

```bash
cd !$ # cd ~/Downloads/sxhkd
make
sudo make install
```

`~/.config/sxhkd` directory and `sxhkdrc` file →

```bash
mkdir -p ~/.config/sxhkd
cp ~/Downloads/bspwm/examples/sxhkdrc !$
```

> [!IMPORTANT]-
>
> Be aware that `sxhkdrc`'s _example/template_ file is copied from the `bspwm` repository rather than from the `sxhkd` one
>

That's it

```bash
$ command -V sxhkd
sxhkd is /usr/local/bin/sxhkd
```

---

#### TL;DR ⌨

##### Shorcuts' Meaning

| **Key** | **Meaning** | |
| --- | --- | --- |
| ***C*** | **`Control`** | **`C-c` → `Control+c`** |
| ***M*** | **`Alt`** | **`M-a` → `Alt+a`** |
| ***S*** | **`Shift`** | **`S-o` → `Shift+o`**  |
| ***Super*** | **`Windows`** | **`Super-s` → `Windows+s`** |
| ***Return*** | **`Enter`** | **`C-S-Return` → `Control+Shift+Enter`** |
| ***-*** | **`+`** | **`C-z` → `Control+z`** |
| ***{a,b,c,d}*** | **`a` `b` `c` `d`** | **`C-{a,b,c,d}` → `C-a` `C-b` `C-c` `C-d`** |

##### Sxhkd

> ***[[SXHKD#sxhkd Section|Reference]]***

| **Action** | **Shortcut** |
| --- | --- |
| ***[[SXHKD#Reload sxhkd's Configuration File → Super-Esc\|Reload Sxhkd Configuration File]]*** | **`Super-Esc`** |

##### Bspwm 

> ***[[SXHKD#bspwm Section|Reference]]***

| **Action** | **Shortcut** |
| --- | --- |
| ***[[SXHKD#Quit/Restart → Super-S-{q,r}\|Quit Bspwm (Return to the greeter)]]*** | **`Super-S-q`** |
| ***[[SXHKD#Quit/Restart → Super-S-{q,r}\|Restart Bspwm]]*** | **`Super-S-r`** |

##### OS Window

> ***[[SXHKD#bspwm Section|Reference]]***

| **Action** | **Shortcut** |
| --- | --- |
| ***[[SXHKD#OS Windows - Open/Close → Super-Return/Super-q\|OS Windows Closing]]*** | **`Super-q`** |
| ***[[SXHKD#OS Windows State - Monocle Layout (Zoom {in,out}) → Super-m\|OS Windows - Monocle Layout]]*** | **`Super-m`** |
| ***[[SXHKD#OS Windows State - Tiled → Super-t\|OS Windows - Tiled Layout]]*** | **`Super-t`** |
| ***[[SXHKD#OS Windows State - Floating → Super-s\|OS Windows - Floating Layout]]*** | **`Super-s`** |
| ***[[SXHKD#OS Windows State - Full-Screen → Super-f\|OS Windows - Full Screen]]*** | **`Super-f`** |
| ***[[SXHKD#OS Windows - Focus → Super-{h,j,k,l}\|Os Windows Focus ⬅️⬇️⬆️➡️ ]]*** | **`Super-{h,j,k,l}`** |
| ***[[SXHKD#OS Windows - Movement → Super-S-{h,j,k,l}\|OS Windows Movement ⬅️⬇️⬆️➡️]]*** | **`Super-S-{h,j,k,l}`** |
| ***[[SXHKD#OS Windows - Resize → Super-M-{h,j,k,l}\|OS Windows Resize ⬅️⬇️⬆️➡️]]*** | **`Super-M-{h,j,k,l}`** |
| ***[[SXHKD#OS Windows - Move it to another Desktop → Super-S-desktop_number\|OS Windows to another Desktop]]*** | **`Super-S-DESKTOP_NUMBER`** |
| ***[[SXHKD#OS Desktop - Focus → Super-desktop_number\| Desktop Focus]]*** | **`Super-DESKTOP_NUMBER`** |
| ***[[SXHKD#OS Desktop - Focus → Super-desktop_number\|Last Desktop Focus]]*** | **`Super-TAB`**

##### Program Launch

> ***[[SXHKD#Kitty Section|Reference I]]&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[[SXHKD#Rofi Section|Reference II]]&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[[SXHKD#Other Processess Section|Reference III]]***

| **Action** | **Shortcut** |
| --- | --- |
| ***[[SXHKD#Launch kitty Terminal → Super-Return\|Kitty]]*** | **`Super-Return`** |
| ***[[SXHKD#Rofi Section\|Rofi]]*** | **`Super-d`** |
| ***[[SXHKD#Launch Firefox → Super-S-f\|Firefox]]*** | **`Super-S-f`** |
| ***[[SXHKD#Launch Burpsuite → Super-S-b\|Burpsuite]]*** | **`Super-S-b`** |
| ***[[SXHKD#Launch I3Lock-Fancy → Super-S-x\|I3Lock-Fancy]]*** | **`Super-S-x`** |

---

#### Configuration File

**Copy from [here](https://github.com/4l3xBB/Env-Setup/blob/main/sxhkd/sxhkdrc) the `sxhkdrc` Full Customised Configuration File** 

##### sxhkd Section

###### Reload sxhkd's Configuration File → Super-Esc

> Note that _super_ refers to the _Windows_ key

```bash
super + Escape
  pkill -USR1 -x sxhkd
```

##### bspwm Section

> The rest of the _key bindings_ related to `bspwm` in the `sxhkdrc` file are in the link above

###### Quit/Restart → Super-S-{q,r}

To **Return to the greeter closing the _Windows Manager_ → `W-S-q`**

To **Restart the _Windos Manager_ → `W-S-r`**

```bash
super + shift + {q,r}
    bspc {quit,wm -r}
```

###### OS Windows - Open/Close → Super-Return/Super-q

- **Open → Check the [[SXHKD#Kitty Section|Kitty Section]]**
- **Close**, as follows →

```bash
super + {_,shift + }q
    bspc node -{c,k}
```

###### OS Windows State - Monocle Layout (Zoom {in,out}) → Super-m

> See the different _OS Windows Layouts/States_ for _bspwm_ [[BSPWM#OS Windows *Layouts/States*|here]]

It switches the Focused Windows display to a **Monocle State**

```bash
super + z
    bspc desktop -l next
```

###### OS Windows State - Tiled → Super-t

> _bspwm Layouts/States_ [[BSPWM#OS Windows *Layouts/States*|here]]

It switches the Focused Windows display to its default state, the **Tiled** one

```bash
super + {t,shift + t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating,fullscreen}
```

###### OS Windows State - Floating → Super-s

> _bspwm Layouts/States_ [[BSPWM#OS Windows *Layouts/States*|here]]

It switches the Focused Windows display to a **Floating State**

```bash
super + {t,shift + t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating,fullscreen}
```

###### OS Windows State - Full-Screen → Super-f

> _bspwm Layouts/States_ [[BSPWM#OS Windows *Layouts/States*|here]]

It switches the Focused Windows display to a **Full-Screen State**

```bash
super + {t,shift + t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating,fullscreen}
```

###### OS Windows - Focus → Super-{h,j,k,l}

To change the focus between the different _OS Windows_ on a specific _Desktop_ →

```bash
super + {_,shift + }{h,j,k,l}
    bspc node -{f,s} {west,south,north,east}
```

![[SXHKD-20240922121029718.webp|350]]

###### OS Windows - Movement → Super-S-{h,j,k,l}

Moves the focused _OS Window_ in the specificed direction. It is like a Windows Exchange

```bash
super + shift + {Left,Down,Up,Right}
    bspc node -v {-20 0,0 20,0 -20,20 0}
```

![[SXHKD-20240922113044366.webp|450]]

###### OS Windows - Resize → Super-M-{h,j,k,l}

In the `bspwm` keybinding section, a custom [[BASH|bash]] script is set as the command to be executed when the following input events (hotkeys) are performed →

```bash
super + alt + {h,j,k,l}
    /home/al3xbb/.config/bspwm/bin/bspwm_resize.sh {west,south,north,east}
```

![[SXHKD-20240922113749584.webp|450]]

The above code allows to resize an _OS Window_ with the **`super + alt`**  shortcut regardless of the _[[BSPWM#OS Windows *Layouts/States*|layout/state]]_

Create a directory inside `~/.config/bspwm` called `bin` and store the `bspwm_resize.sh`'s content as a script (i.e. an **executable** file)

> *bspwm_resize.sh* Content [[BSPWM#*bin/bspwm_resize.sh*|here]]

```bash
$ mkdir ~/.config/bspwm/bin
$ touch bspwm_resize.sh
$ chmod +x !$
$ nvim $
```


###### OS Windows - Move it to another Desktop → Super-S-desktop_number

Moves the focused _OS Windows_ to another _bspwm_ Desktop

```bash
super + {_,shift + }{1-9,0}
    bspc {desktop -f,node -d} '^{1-9,10}'
```

###### OS Desktop - Focus → Super-desktop_number

To move between the different _OS Desktops_ via the Desktop Number → **`Super-desktop_number`**

```bash
super + {_,shift + }{1-9,0}
    bspc {desktop -f,node -d} '^{1-9,10}'
```

To move to the last _OS Desktop_ → **`Super-Tab`**

```bash
super + {grave,Tab}
    bspc {node,desktop} -f last
```

##### Kitty Section

###### Launch kitty Terminal → Super-Return

```bash
super + Return
  /opt/kitty/bin/kitty
```

##### Rofi Section

###### Launch Rofi → Super-d

```bash
super + d
  /usr/bin/rofi -show run
```

##### Other Processess Section

###### Launch Firefox → Super-S-f

```bash
super + shift + f
    /usr/bin/firefox
```

###### Launch Burpsuite → Super-S-b

```bash
super + shift + b
    /usr/bin/burpsuite-launcher
```

###### Launch I3Lock-Fancy → Super-S-x

```bash
super + shift + x
    /usr/bin/i3lock-fancy
```