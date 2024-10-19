---
Primary_category: "[[DESKTOP SETUP]]"
title: POLYBAR
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses: 
---

###### PRIMARY CATEGORY → [[DESKTOP SETUP]]

**[POLYBAR](https://github.com/polybar/polybar) → Highly Customizable Status Bars**

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

***Polybar*** is executed by [[BSPWM|bspwm]] from the [[BSPWM#*bspwmrc*|bspwmrc]] script through the `polybar`'s launcher script → `~/.config/polybar/launch.sh`

```bash title="~/.config/bspwm/bspwmrc"
# Polybar Launch
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
> pgrep --exact polybar > /dev/null 2>&1 || ~/.config/polybar/launch.sh &
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
> In addition, the above [[DESKTOP SETUP|setup]] components' installation facilitates the _ZSH Configuration_

```bash
apt install -y -- polybar
```

That's it!

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

As a base for the configuration file's structure, clone the following *Github Repository*

> ***[Reference](https://github.com/VaughnValle/blue-sky)***

```bash
git clone https://github.com/VaughnValle/blue-sky ~/Downloads/blue-sky
```

Then, recursively copy the _polybar's content_ into `~/.config/polybar` →

```bash
cd !$
bash -c "shopt -sq dotglob && cp -rv -- ./polybar/* ~/.config/polybar"
```

> [!INFO]-
>
> In the above command, `bash -c ` is used to run the inner commands inside a [[BASH|bash]]
>
> This is done in order to to be able to use the `shopt` [[SHELL SCRIPTING|shell]] builtin, and then enable the `dotglob` shell extension to expand also the hidden files through [[Globbing|globbing]]
>
> Note that prior action can also be performed in [[Globbing#*POSIX Compliant* - Including Hidden Files|this way]]
>

Once the above is done, insert the following line in the [[BSPWM#*bspwmrc*|bspwmrc]] file related to the _Polybar's Launch_ →

> As already stated, these functions check polybar's status and launch it

```bash title="~/.config/bspwm/bspwmrc"
checkProcess polybar || { launchProcess "$_pbl" ; unset -v -- _pbl ; }
```

Lastly, copy the _polybar's fonts_ (i.e. the _.ttf_ Files) into the system _TrueType Fonts_ directory and reset the _ system's fonts cache_ →

```bash
cp ~/.config/polybar/fonts/* /usr/share/fonts/truetype/
fs-cache -v # Reset the System Fonts Cache
```

---

#### Configuration File

##### Current.ini

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/current.ini)***

Most of the *Bars* and their *Modules* are defined in this ***Configuration File***

##### Workspace.ini

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/workspace.ini)***

This ***Configuration File*** contains the *[[POLYBAR#primary|Main Bar]]*'s definition related to the ***[[BSPWM|Windows Manager]] Workspaces***

---

#### Launcher File

##### Launch.sh

> ***[See here](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/bin/launch.sh)***

This ***[[SHELL SCRIPTING|shell]] script*** launchs all the *Bars* defined in the above *configuration files*

---

#### Bars

A *bar* can contain one or several *[[POLYBAR#Modules|modules]]*

There is a distinction between the **position** of the ***Top Bars*** →

- ***Left Bar***
<br>
- ***Center Bar***
<br>
- ***Right Bar***

![[POLYBAR-20241013161148167.webp|350]]

##### Left

###### os_icon

It is launched from the *[[POLYBAR#launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar os_icon -c ~/.config/polybar/current.ini &
```

***Module processed within the bar → [[POLYBAR#my-text-label|my-text-label]]***

```bash title="~/.config/polybar/current.ini"
modules-center = my-text-label # Module Loaded
```

![[POLYBAR-20241013172458601.webp|70]]

###### ethernet_bar

It is launched from the *[[POLYBAR#Launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar ethernet_bar -c ~/.config/polybar/current.ini &
```

***Module processed within the bar → [[POLYBAR#ethernet_status|ethernet_status]]***

```bash title="~/.config/polybar/current.ini"
modules-center = ethernet_status # Module Loaded
```

![[POLYBAR-20241013172422793.webp|175]]

###### vpn_bar

It is launched from the *[[POLYBAR#Launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar vpn_bar -c ~/.config/polybar/current.ini &
```

***Module processed within the bar → [[POLYBAR#vpn_status|vpn_status]]***

```bash title="~/.config/polybar/current.ini"
modules-center = vpn_status # Module Loaded
```

![[POLYBAR-20241013172336620.webp|175]]

##### Center

###### primary

It is launched from the *[[POLYBAR#Launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar primary -c ~/.config/polybar/workspace.ini &
```

***Module processed within the bar → [[POLYBAR#workspaces|workspaces]]***

```bash title="~/.config/polybar/workspace.ini"
modules-center = workspaces # Module Loaded
```

![[POLYBAR-20241013172919202.webp|350]]

##### Right

###### target_to_hack

It is launched from the *[[POLYBAR#Launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar target_to_hack -c ~/.config/polybar/current.ini &
```

***Module processed within the bar → [[POLYBAR#target_to_hack|target_to_hack]]***

```bash title="~/.config/polybar/current.ini"
modules-center = target_to_hack # Module Loaded
```

![[POLYBAR-20241013173336639.webp|175]]

###### primary

It is launched from the *[[POLYBAR#Launch.sh|launch.sh]] script*  as follows →

```bash title="~/.config/polybar/launch.sh"
polybar primary -c ~/.config/polybar/current.ini &
```

***Module processed within the bar → [[POLYBAR#sysmenu|sysmenu]]***

```bash title="~/.config/polybar/current.ini"
modules-center = sysmenu # Module Loaded
```

![[POLYBAR-20241013173422877.webp|75]]

---

#### Modules

A *module* is a *execution unit* that can perform a specific action within a *[[POLYBAR#Bars|bar]]*, such as *run a script*

One or several *modules* can be processed in the same *bar*

![[POLYBAR-20241013174024652.webp|350]]

##### my-text-label

It simply displays the icon specified in the `content` parameter

***Associated Bar → [[POLYBAR#os_icon|os_icon]]***

***Module definition →***

```bash title="~/.config/polybar/current.ini"
[module/my-text-label]
type = custom/text
content = %{T7}
```

###### Font

> ***See [[KITTY#Hack Nerd Fonts|here]] to install the Hack Nerd Fonts***

The `content` parameter above uses the *`T7` Font Type*

This *Font Type* is not declared by default

***Chosen Font → Hack Nerd Font***

To declare it and set the *size* and *position* of the *OS Icon* →

```bash title="~/.config/polybar/current.ini"
font-7 = "Hack Nerd Font Mono:size=20;6" # Filter by 'font-' in the above file
```

> [!INFO]-
> 
> Note that the `size` parameter sets both the *OS Icon*'s size and position
>
> ```bash
> size=SIZE;POSITION
> ```
>

##### ethernet_status

This module executes the specified *script* through the `exec` parameter every two seconds

***Associated Bar → [[POLYBAR#ethernet_bar|ethernet_bar]]***

***Module definition →***

```bash title="~/.config/polybar/current.ini"
[module/ethernet_status]
type = custom/script
interval = 2
exec = ~/.config/bspwm/bin/ethernet_status.sh
```

###### modules/ethernet_status.sh

> ***[Reference](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/bin/modules/ethernet_status.sh)***

It displays the *IP Address* assigned to a specific *Network Interface*

> [!NOTE]- *ethernet_status.sh*
>
> ```bash
> #!/usr/bin/env bash
>
> printf "%%{F#2495e7}󰈀 %%{F#ffffff}$( /usr/sbin/ifconfig ens33 | awk '/inet\s/ { print $2 }' )%%{u-}\n"
> ```
>

##### vpn_status

This module executes the specified *script* through the `exec` parameter every two seconds

***Associated Bar → [[POLYBAR#vpn_bar|vpn_bar]]***

***Module definition →***

```bash title="~/.config/polybar/current.ini"
[module/vpn_status]
type = custom/script
interval = 2
exec = ~/.config/bspwm/bin/vpn_status.sh
```

###### modules/vpn_status.sh

> ***[Reference](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/bin/modules/vpn_status.sh)***

It displays the *IP Address* assigned to the *VPN Network Interface (Tun0)*

> [!NOTE]- *vpn_status.sh*
>
> ```bash
> #!/usr/bin/env bash
>
> checkVPNiface ()
> {
>   local -- _vpnIface=$( /usr/sbin/ifconfig tun0 2> /dev/null )
>
>   if [[ -n $_vpnIface ]] ; then
>     printf "%%{F#1bbf3e}󰆧 %%{F#ffffff}$( /usr/sbin/ifconfig tun0 | awk '/inet\s/ { print $2 }' )%%{u-}\n"
>   else
>     printf "%%{F#1bbf3e}󰆧 %%{u-} Disconnected\n"
>   fi
> }
>
> checkVPNiface
> ```
>

##### workspaces

It displays all *Workspaces* in a row

***Associated Bar → [[POLYBAR#primary|primary]]***

***Part of the Module definition →***

```bash title="~/.config/polybar/workspace.ini"
[module/workspaces]
type = internal/xworkspaces
```

**To see all the information related to this module → [[POLYBAR#Workspace.ini|workspace.ini]]**

###### Workspaces' States

To change the colour of both *active* and *occupied* *Workspaces* →

```bash title="~/.config/polybar/workspace.ini"
label-active-foreground = ${color.red} # Active
label-occupied-foreground = ${color.yellow} # Occupied
```

##### target_to_hack

This module executes the specified *script* through the `exec` parameter every two seconds

***Associated Bar → [[POLYBAR#target_to_hack|target_to_hack]]***

***Module definition →***

```bash title="~/.config/polybar/current.ini"
[module/target_to_hack]
type = custom/script
interval = 2
exec = ~/.config/bspwm/bin/target_to_hack.sh
```

###### modules/target_to_hack.sh

> ***[Reference](https://github.com/4l3xBB/Env-Setup/blob/main/polybar/bin/modules/target_to_hack.sh)***

This module sets the *IP Address* and *Hostname* of a specific target based on the `~/.config/bin/target` file

> [!NOTE]- *target_to_hack.sh*
>
> ```bash
 > #!/usr/bin/env bash
>
> extractData ()
> {
>   local -- _IP= _machineName= _targetFile=/home/al3xbb/.config/bin/target
>
>   [[ -s $_targetFile ]] || {
>
>     printf "%%{F#e51d0b}󰓾 %%{u-}%%{F#ffffff} No target\n"
>   }
>
>   while IFS=: read -r _IP _machineName
>   do
>     printf "%%{F#e51d0b}󰓾 %%{F#ffffff}%s%%{u-} - %s\n" "$_machineName" "$_IP"
>
>   done < "$_targetFile"
> }
>
> extractData
> ```
>

###### Shell Functions

The following [[SHELL SCRIPTING|shell]] functions performs the following actions to the above file (`target`) →

- ***[[ZSH CUSTOM FUNCTIONS#*setTarget*|setTarget]] → Prints the IP Address and Hostname of the target***

![[customZSHSetTargetFunction.gif|375]]

- ***[[ZSH CUSTOM FUNCTIONS#*clearTarget*|clearTarget]] → Empties the file*** 

![[customZSHclearTargetFunction.gif|375]]

##### sysmenu

This module performs the following actions →

- ***Icon Display via the `content` parameter***
- ***Executes the specified script in the `click-left` parameter***

***Associated Bar → [[POLYBAR#primary|primary]]***

***Module definition →***

```bash title="~/.config/polybar/current.ini"
[module/sysmenu]
type = custom/text
content = 襤
click-left = ~/.config/polybar/scripts/powermenu_alt
```