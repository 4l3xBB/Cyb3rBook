---
Primary_category: "[[SETUP]]"
title: ROFI
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**[Rofi](https://github.com/davatorium/rofi) → Application Launcher**

Basically, It's an application launcher and a ssh-launcher

Displays to the user a list of options from which one or more options can be selected

In this case, It is a list of the applications that are installed on the system

`rofi` is launched by the [[SXHKD|sxhkd]] daemon when a certain input event, such as a _Hotkey_, is performed

```bash title="~/.config/sxhkd/sxhkdrc"
# Rofi Launch
super + d
  /usr/bin/rofi -show run
```


![[ROFI_LAUNCHER 1.gif|400]]

**Configuration File → `~/.config/rofi/config.rasi`**

> [!IMPORTANT]-
>
> It is not necessary to modify the above configuration file in this [[SETUP|setup]] environment

**More information [here](https://github.com/davatorium/rofi)**

---

#### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `rofi`
>
> There are some dependencies that are needed in the following installation steps
>

```bash
apt install -y -- rofi
```

That's it

```bash
$ command -V rofi
rofi is /usr/bin/rofi
```

> [!IMPORTANT]-
>
> Installation can be done either through the OS package manager or via the `git clone` command
>
> Note that the default OS repositories listed in `/etc/apt/sources.list` file and in `/etc/apt/sources.list.d` directory may have older versions unlike the `rofi`'s official Github repository
>
> As It is always is desirable to have the latest versions of any _package_ or _binary_ installed, I'd recommend installing them via their Github Repositories
>
> Although, in this case the _package_'s version installed from the `apt` Package Manager differs only slightly from the github one

##### Themes

> ***[Reference](https://github.com/newmanls/rofi-themes-collection)***

Create the `~/.config/rofi/themes` to store  *.RASI File* for each *ROFI Theme* →

```bash
mkdir -p -- ~/.config/rofi/themes
```

Clone the Github Repository below into `/opt` and copy all *ROFI Themes* to the directory created above →

```bash
sudo git clone https://github.com/newmanls/rofi-themes-collection /opt/rofi
```

```bash
cp -rv !$/themes/* ~/.config/rofi/themes
```

Open the *ROFI Theme Selector* and choose the *Theme* you like the most →

```bash
command -V -- rofi-theme-selector &> /dev/null && rofi-theme-selector
```

![[ROFI_Theme_Selector.gif|400]]