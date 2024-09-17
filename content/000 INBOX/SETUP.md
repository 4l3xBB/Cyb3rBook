---
Primary_category: "[[PENTESTING]]"
title: "SETUP"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - pentesting👹
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[PENTESTING]]

#### Environment Customization

| **SOURCES** | | |
| --- | --- | --- |
| **[Parrot](https://parrotsec.org/)** | Operative System | **[See more](https://parrotsec.org/docs/)** |
| **[bspwm](https://github.com/baskerville/bspwm)** | Windows Manager | **[See more](https://wiki.archlinux.org/title/Bspwm)** |
| **[sxhkd](https://github.com/baskerville/sxhkd)** | Hotkey Daemon | **[See more](https://wiki.archlinux.org/title/Sxhkd)** |
| **[Polybar](https://github.com/polybar/polybar)** | Status Bar | **[See more](https://wiki.archlinux.org/title/Polybar)** |
| **[Picom](https://github.com/yshui/picom)** | Windows Visualizer (Compositor) | **[See more](https://wiki.archlinux.org/title/Picom)** |
| **[Rofi](https://github.com/davatorium/rofi)** | Rofi | **[See more](https://wiki.archlinux.org/title/Rofi)** |
| **[Feh](https://github.com/derf/feh)** | Image Viewer (Wallpaper) | **[See more](https://wiki.archlinux.org/title/Feh)** |
| **[Kitty](https://github.com/kovidgoyal/kitty)** | Terminal Emulator | **[See more](https://wiki.archlinux.org/title/Kitty)** |
| **[ZSH](https://wiki.archlinux.org/title/Zsh_(Espa%C3%B1ol))** | [[SHELL SCRIPTING\|Shell]] | **[See more](https://zsh.sourceforge.io/Doc/Release/zsh_toc.html)** |

##### Components

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
- ![250](https://i.pinimg.com/1200x/06/b5/40/06b5401289915f753505baba16827c6a.jpg)
	- [[KITTY]]
	
<br>

This Custom Linux Environment is deployed in _Parrot OS_ 🦜

**The Deployment Flow would be →**

**bwpwm & sxhkd →**

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
> $ sudo apt update && sudo apt upgrade -y
> ```
> Do this →
>
> ```bash
> $ sudo apt update && sudo parrot-upgrade
> ```

##### Customised Functions

###### Pentesting Folder Structure

```bash
mkt()
{
        local -- _dir=
        local -a -- _dirs=(
                evidence
                logs
                scans
                scope
                tools
        )

        for _dir in "${_dirs[@]}"
        do
                mkdir -p -- "$_dir"

                [[ $_dir == "evidence" ]] &&

                        mkdir -p -- "$_dir"/{creds,data,screenshots}
        done
}
```

> [!IMPORTANT]-
>
> Note that above code is _[[POSIX|Non-POSIX Compliant]]_ due to Bash Specific Functionalities such as `local`, `[[ ]]` shell keyword, **Brace Expansion** `{ }` or _indexed arrays_
>
> Therefore, It is not destined to _POSIX Compliant_ shells like _sh_ or _dash_
>
> If _POSIX Compliant_ is needed →
> ```bash
> mkt()
> {
>         _dir= _dirs="evidence logs scans scope tools"
>
>         for _dir in $_dirs
>         do
>                 mkdir -p -- "$_dir"
>
>                 [ "$_dir" = "evidence" ] &&
>
>                         mkdir -p -- "$_dir"/creds \
>                                     "$_dir"/data \
>                                     "$_dir"/screenshot
>         done
>
> }
> ```
> 

###### Set Target Host - Polybar Right Module

```bash
setTarget()
{
  local -- _IP=$1 _machineName=$2 _IPPattern='[0-9]{1,3}(\.[0-9]{1,3}){3}'

  (( $# != 2 )) && {
    /bin/cat << HELP 1>&2

  [!] This function requires two arguments: IP Addresss and Machine Name

    Syntax: setTarget XXX.XXX.XXX.XXX "Machine Name"
HELP
    return 1
  }

  [[ $_IP =~ $_IPPattern ]] || { printf "\n%s\n" '[!] First args must be an IP' 1>&2 ; return 1 ; }

  [[ -n $_IP && -n $_machineName ]] || { printf "\n%s\n" '[!] Args cannot be an empty string' 1>&2 ; return 1 ; }

  printf "%s\n%s\n" "$_IP" "$_machineName" > /home/al3xbb/.config/bin/target
}
```

