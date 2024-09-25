---
Primary_category: "[[SETUP]]"
title: POLYBAR
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**`polybar` → Highly Customizable Status Bars**

Each bar has its own modules and It is executed as a daemon

```bash
$ pgrep --list-full --exact polybar
1631 polybar log -c /home/al3xbb/.config/polybar/current.ini
1632 polybar ethernet_bar -c /home/al3xbb/.config/polybar/current.ini
1633 polybar vpn_bar -c /home/al3xbb/.config/polybar/current.ini
```

> [!INFO]-
>
> In above command's output, each process corresponds to a specific bar with its name and its configuration file where the bar and its module/s are declared
>
> ```bash
> polybar log -c /home/al3xbb/.config/polybar/current.ini
>          |                                   |
>          +---> Bar's Name                    +---> Config File
> ```
> - `-c FILE | --config=FILE` → Path to the configuration file

A module can execute any action such as a command, scripts...

![[POLYBAR-20240925175458343.webp|350]]

> Each bar with its configuration file as an argument is launched by the below script

`polybar` is executed by [[BSPWM|bspwm]] from the [[BSPWM#*bspwmrc*|bspwmrc]] script through the `polybar`'s launcher script → `~/.config/polybar/launch.sh`

```bash
# ~/.config/bspwm/bspwmrc - Polybar Launch
checkProcess polybar || { launchProcess "$_pbl" ; unset -v -- _pbl ; }
```

> [!INFO]-
>
> Note that `checkProcess` and `launchProcess` are [[SHELL SCRIPTING|shell functions]] declared in the `~/.config/bspwm/src/bspwmrc.sh` [[BSPWM#*src/bspwmrc.sh*|source file]]
>
> The `$_pbl` parameter is also declared in that file being the **Polybar Launcher Path**
>
> Above implementation would be something similar to the usual approach through `pgrep --exact process_name || binary|script_path &` 
>
> ```bash
> #!/usr/bin/env sh
> $ pgrep --exact polybar > /dev/null 2>&1 || ~/.config/polybar/launch.sh &
> ```

**Configuration File → Any file which a `.ini` extension**

**Launcher File → `~/.config/polybar/launch.sh`**

**More information [here](https://github.com/polybar/polybar/wiki)**

**[Polybar Site](https://polybar.github.io/)**

---

#### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm]], [[SXHKD|sxhkd]] and  [[KITTY|kitty]] installations before proceeding with this one related to `polybar`
>
> There are some dependencies that are needed in the following installation steps
>
> In addition, the above [[SETUP|setup]] components' installation facilitates the _ZSH Configuration_

```bash
$ apt install -y -- polybar
```

That's it

```bash
$ command -V polybar
polybar is /usr/bin/polybar
```

> [!IMPORTANT]-
>
> Installation can be done either through the OS package manager or via the `git clone` command
>
> Note that the default OS repositories listed in `/etc/apt/sources.list` file and in `/etc/apt/sources.list.d` directory may have older versions unlike the `polybar`'s official Github repository
>
> As It is always is desirable to have the latest versions of any _package_ or _binary_ installed, I'd recommend installing them via their Github Repositories
>
> Although, in this case the _package_'s version installed from the `apt` Package Manager differs only slightly from the github one

As a base for the configuration file's structure, clone the following [Github Repository](https://github.com/VaughnValle/blue-sky)

```bash
$ git clone https://github.com/VaughnValle/blue-sky ~/Downloads/blue-sky
```

Then, recursively copy the _polybar's content_ into `~/.config/polybar` →

```bash
$ cd !$
$ bash -c "shopt -sq dotglob && cp -rv -- ./polybar/* ~/.config/polybar"
```

> [!INFO]-
>
> In the above command, `bash -c ` is used to run the inner commands inside a [[BASH|bash]]
>
> This is done in order to to be able to use the `shopt` [[SHELL SCRIPTING|shell]] builtin, and then enable the `dotglob` shell extension to expand also the hidden files through [[Globbing|globbing]]
>
> Note that prior action can also be performed in [[Globbing#*POSIX Compliant* - Including Hidden Files|this way]]

Once the above is done, insert the following line in the [[BSPWM#*bspwmrc*|bspwmrc]] file related to the _Polybar's Launch_ →

> As already stated, these functions check polybar's status and launch it

```bash
checkProcess polybar || { launchProcess "$_pbl" ; unset -v -- _pbl ; }
```

Lastly, copy the _polybar's fonts_ (i.e. the _.ttf_ Files) into the system _TrueType Fonts_ directory and reset the _ system's fonts cache_ →

```bash
$ cp ~/.config/polybar/fonts/* /usr/share/fonts/truetype/
$ fs-cache -v # Reset the System Fonts Cache
```

#### Configuration File



#### Launcher File

