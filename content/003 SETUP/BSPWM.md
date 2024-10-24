---
Primary_category: "[[DESKTOP SETUP]]"
title: BSPWM
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses: 
---

###### PRIMARY CATEGORY → [[DESKTOP SETUP]]

**[BSPWM](https://github.com/baskerville/bspwm) → Binary Space Partitioning Window Manager**

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

This _Windows Manager_ generates and divides each _OS Window_ in half (_Binary Partitioning_)

![[BSPWM-20240922093153092.webp|500]]

---

#### OS Windows *Layouts/States*

There are several types of _OS Windows_ Layouts →

- **Tiled**
- **Monocle**
- **Floating**
- **Full-Screen** 

![[BSPWM-20240922102829181.webp|500]]

**Configuration File → ``~/.config/bspwm/bspwmrc``**

**More information [here](https://github.com/baskerville/bspwm)**

---

#### Installation

First of all, install the following requirements →

```bash
sudo apt install -y -- build-essential git vim xcb libxcb-util0-dev libxcb-ewmh-dev libxcb-randr0-dev libxcb-icccm4-dev libxcb-keysyms1-dev libxcb-xinerama0-dev libasound2-dev libxcb-xtest0-dev libxcb-shape0-dev
```

Clone the [bspwm's Github Repository](https://github.com/baskerville/bspwm) →

```bash
cd ~/Downloads
git clone https://github.com/baskerville/bspwm.git bspwm
```

```bash
cd !$ # cd ~/Downloads/bspwm
make
sudo make install
```

That's it

```bash
$ command -V bspwm
bspwm is /usr/local/bin/bspwm
```

###### Configuration Files

Create the `~/.config/bspwm` directory and store the `bspwmrc` [[BSPWM#Configuration Files|configuration file]] in it

```bash
mkdir -p ~/.config/bspwm
cp ~/Downloads/bspwm/examples/bspwmrc !$
```

###### Source/Binary Files

Same as above, but as follows →

```bash
mkdir -p ~/.config/bspwm/{bin,src}
```

```bash
touch ~/.config/bspwm/bin/bspwm_resize.sh
chmod +x !$
```

```bash
touch ~/.config/bspwm/src/bspwmrc.sh
```

Just copy the following files' content and paste into the above created ones →

- **`bspwmrc.sh`** → [[BSPWM#*src/bspwmrc.sh*|Source File]]
- **`bspwm_resize.sh`** → [[BSPWM#*bin/bspwm_resize.sh*|Binary File]]

---

#### Configuration Files

> [!IMPORTANT]-
>
> `bspwmrc` file is executed with the following command by `lightdm` process →
>
> ```bash
> $ bspwm -c /path/to/bspwmrc
> ```
>
> Therefore, the shell interpreter that executes `bspwmrc` is the one indicated by the _shebang_
>
> It's recommended to use a _[[POSIX|POSIX Compliant]]_ shell such as _sh_ or _dash_
>
> ```bash
> #!/usr/bin/env sh 
> ````
> ```bash
> #!/usr/bin/env dash
> ```

**Two files are required →**
- **`~/.config/bspwm/bspwmrc`** → `bspwm`'s Configuration File
- **`~/.config/bspwm/src/bspwmrc.sh`** → `bspwmrc` sources it

###### *bspwmrc*

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/bspwm/bspwmrc)***

> [!NOTE]-  _POSIX Compliant_ Version
>
> ```bash title="~/.config/bspwm/bspwmrc"
> #!/usr/bin/env sh                                   
>
> # Import required Functions and Parameters
> if [ -f ~/.config/bspwm/src/bspwmrc.sh ] ; then
>   . ~/.config/bspwm/src/bspwmrc.sh
> fi
>
> bspc monitor -d I II III IV V VI VII VIII IX X
>
> bspc config border_width         2
> bspc config window_gap          12
>
> bspc config split_ratio          0.52
> bspc config borderless_monocle   true
> bspc config gapless_monocle      true
>
> bspc rule -a Gimp desktop='^8' state=floating follow=on
> bspc rule -a Chromium desktop='^2'
> bspc rule -a mplayer2 state=floating
> bspc rule -a Kupfer.py focus=on
> bspc rule -a Screenkey manage=off
>
> # No Windows Border
> bspc config border_width 0
>
> # VMWare Bidirectional Clipboard
> vmware-user-suid-wrapper &
>
> # Sxhkd Init
> checkProcess sxhkd || launchProcess sxhkd
>
> # Polybar Init
> checkProcess polybar || { launchProcess "$_pbl" ; unset -v -- _pbl ; }
>
> # Picom Init
> checkProcess picom || launchProcess picom
>
> # Feh Init (Desktop Wallpaper)
> /usr/bin/feh --bg-fill /home/al3xbb/Desktop/Fondos/Fondo.jpg &
> # Avoid Compatibility errors with Java applications (Burpsuite...)
> # Change the Windows Manager Name from bspwm to LG3D (Only the name)
> wmname LG3D &
> ```
>

> [!CAUTION]- Non _POSIX Compliant_ Version
>
> ```bash title="~/.config/bspwm/bspwmrc"
> #!/usr/bin/env bash
>
> # Import required Functions and Parameters
> if [[ -f ~/.config/bspwm/src/bspwmrc.sh ]] ; then
>   . ~/.config/bspwm/src/bspwmrc.sh
> fi
>
> bspc monitor -d I II III IV V VI VII VIII IX X
>
> bspc config border_width         2
> bspc config window_gap          12
>
> bspc config split_ratio          0.52
> bspc config borderless_monocle   true
> bspc config gapless_monocle      true
>
> bspc rule -a Gimp desktop='^8' state=floating follow=on
> bspc rule -a Chromium desktop='^2'
> bspc rule -a mplayer2 state=floating
> bspc rule -a Kupfer.py focus=on
> bspc rule -a Screenkey manage=off
>
> # No Windows Border
> bspc config border_width 0
>
> # VMWare Bidirectional Clipboard
> vmware-user-suid-wrapper &
>
> # Sxhkd Init
> checkProcess sxhkd || launchProcess sxhkd
>
> # Polybar Init
> checkProcess polybar || { launchProcess "$_pbl" ; unset -v -- _pbl ; }
>
> # Picom Init
> checkProcess picom || launchProcess picom
>
> # Feh Init (Desktop Wallpaper)
> /usr/bin/feh --bg-fill /home/al3xbb/Desktop/Fondos/Fondo.jpg &
> # Avoid Compatibility errors with Java applications (Burpsuite...)
> # Change the Windows Manager Name from bspwm to LG3D (Only the name)
> wmname LG3D &
> ```
>

###### *src/bspwmrc.sh*

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/bspwm/src/bspwmrc.sh)***

> [!NOTE]- _POSIX Compliant_ Version
> ```bash title="~/.config/bspwm/src/bspwmrc.sh"
> ####################################################
> ###################  PARAMETERS  ###################
> ####################################################
> 
> _pbl=~/.config/polybar/launch.sh
> 
> #####################################################
> ####################  FUNCTIONS  ####################
> #####################################################
> 
> # ------- Get an External Binary's Path ------- #
> getCmd ()
> {
>   _cmdPath=$( command -v "$1" )
>   [ -n "$_cmdPath" ] && return 0 || return 1
> }
> 
> # ------- Check if a Process is running or not ------- #
> checkProcess ()
> {
>   pgrep -x "$1" > /dev/null 2>&1 && return 0 || return 1
> }
> 
> # ------- Launch a {script,binary} as a process ------- #
> launchProcess ()
> {
>   _cmd=''
>   case $1 in
>     */*)  [ -f "$1" ] && _cmd=$1 || return 1
>           ;;
>     *)    getCmd "$1" || return $?
>           _cmd=$_cmdPath ; unset -v -- _cmdPath
>           ;;
>   esac
>   [ -x "$_cmd" ] && "$_cmd" &
>   unset -v -- _cmd
> }
> ```
>

> [!CAUTION]- Non _POSIX Compliant_ Version
>
> ```bash title="~/.config/bspwm/src/bspwmrc.sh"
> ####################################################
> ###################  PARAMETERS  ###################
> ####################################################
> 
> _pbl=~/.config/polybar/launch.sh
> 
> #####################################################
> ####################  FUNCTIONS  ####################
> #####################################################
>
> # ------- Get an External Binary's Path ------- #
> checkCmd ()
> {
>   local -n -- _cmdPath=$2
>   _cmdPath=$( command -v "$1" )
>   [[ -n $_cmdPath ]] && return 0 || return 1
> }
> 
> # ------- Check if a Process is running or not ------- #
> checkProcess ()
> {
>   pgrep -x "$1" &> /dev/null && return 0 || return 1
> }
> 
> # ------- Launch a {script,binary} as a process ------- #
> launchProcess ()
> {
>   local -- _cmd=
>   if [[ $1 = */* ]] ; then
>     [[ -f $1 ]] && _cmd=$1 || return 1
>   else
>     checkCmd "$1" _cmd || return $?
>   fi
>   [[ -x $_cmd ]] && "$_cmd" &
> }
> ```
>

---

#### Binary Files

**One file is required →**

- **`~/.config/bspwm/bin/bspwm_resize.sh`** → _[[SXHKD#OS Windows - Resize → W-A-{h,j,k,l}|sxhkd]]_ relates a specific _input event (Hotkey)_ to this _script_ execution

###### *bin/bspwm_resize.sh*

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/bspwm/bin/bspwm_resize.sh)***

> [!NOTE]- _POSIX Compliant_ Version
>
> ```bash title="~/.config/bspwm/bin/bspwm_resize.sh"
> #!/usr/bin/env bash
>
> if bspc query -N -n focused.floating > /dev/null ; then
>     step=20
> else
>     step=100
> fi
>
> case $1 in
>     west)   dir=right; falldir=left; x=-$step; y=0
>             ;;
>
>     east)   dir=right; falldir=left; x=$step; y=0
>             ;;
>
>     north)  dir=top; falldir=bottom; x=0; y=-$step
>             ;;
>
>     south)  dir=top; falldir=bottom; x=0; y=$step
>             ;;
> esac
>
> bspc node -z "$dir" "$x" "$y" || bspc node -z "$falldir" "$x" "$y"
> ```
>
