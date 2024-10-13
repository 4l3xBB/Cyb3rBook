---
Primary_category: "[[SETUP]]"
title: FEH
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**[FEH](https://github.com/derf/feh) → Fast and Light Image Viewer**

With this tool, images can be displayed on the terminal as [[KITTY|kitty]] does through `kitten icat`

But, in this [[SETUP|setup environment]], It is used to manage the **Desktop Wallpaper** as [[BSPWM|bspwm]] cannot do it itself

It has several image display modes and allows the background image to be changed dinamically

`feh` is launched by [[BSPWM|bspwm]] from the [[BSPWM#*bspwmrc*|bspwmrc]] script as follows →

```bash title="~/.config/bspwm/bspwmrc"
# Feh Binary Launch
/usr/bin/feh --bg-fill /home/al3xbb/Desktop/4l3xBB/Wallpapers/Fondo.jpg &
```

**Wallpaper's Directory → `~/Desktop/USER/Wallpapers/`**

**More info [here](https://github.com/derf/feh)**

**[Project Documentation Page](https://feh.finalrewind.org/)**

As always, all the information needed to handle this tool can be found in its **[Man Page](https://man.finalrewind.org/1/feh/)**

> [!INFO]-
>
> Note that the _help panel_ is not compiled by default as mentioned [here](https://github.com/derf/feh#contributing)
>
> Just see the _Man Page_ linked above or →
>
> ```bash
> $ man feh # :)
> ```

---

#### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm]] and the [[SXHKD|sxhkd's]] installation before proceeding with this one related to `feh`
>
> There are some dependencies that are needed in the following installation steps
>


```bash
apt install -y -- feh
```

That's it

```bash
$ command -V feh
feh is /usr/bin/feh
```

> [!IMPORTANT]-
>
> Installation can be done either through the OS package manager or via the `git clone` command
>
> Note that the default OS repositories listed in `/etc/apt/sources.list` file and in `/etc/apt/sources.list.d` directory may have older versions unlike the `feh`'s official Github repository
>
> As It is always is desirable to have the latest versions of any _package_ or _binary_ installed, I'd recommend installing them via their Github Repositories
>
> Although, in this case the _package_'s version installed from the `apt` Package Manager differs only slightly from the github one

Once installed, download the desired image and save it in the following directory →

```bash
mkdir -p ~/Desktop/4l3xBB/Wallpapers
mv ~/Downloads/Image.jpg !$
```

To set it as the _Desktop Wallpaper_ →

```bash
feh --bg-fill /home/al3xbb/Desktop/4l3xBB/Wallpapers/Fondo.jpg
```

> [!INFO]-
>
> Note that, as mentioned earlier, the above command has to be implement in the [[BSPWM#*bspwmrc*|bspwmrc]] file to set the image as the _Desktop Wallpaper_ automatically
>
> ```bash title="~/.config/bspwm/bspwmrc"
> /usr/bin/feh --bg-fill /home/al3xbb/Desktop/4l3xBB/Wallpapers/Fondo.jpg &
> ```
>
 