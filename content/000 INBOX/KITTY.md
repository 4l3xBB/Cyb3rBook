---
Primary_category: "[[SETUP]]"
title: KITTY
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**[kitty](https://github.com/kovidgoyal/kitty) → Fast and Feature-Rich GPU Based Terminal Emulator**

It is an **enhanced terminal** with a _tabbed/tiling_ structure focused in performance and customization

Each tab can be divided into several _panes/windows_

Some of this **features** are →

- **Multiple Instances and Windows**
<br>
- **Setting of _Keybinds/Hotkeys_  via a simple configuration file**
<br>
- **Image and Emojis Display**

```bash
$ kitty +kitten icat /path/to/image
```

- **SSH Protocol support**

```bash
$ kitty +kitten ssh user@hostname
```

Note that the above commands use the syntax `kitty +kitten`

`kittens` are small integrated programs that extend `kitty`'s capacity - [See here](https://sw.kovidgoyal.net/kitty/kittens_intro/)

`kitty` is launched by the [[SXHKD|sxhkd]] daemon when a certain input event, such as a _Hotkey_, is performed

```bash
# ~/.config/sxhkd/sxhkdrc - Kitty Launch Hotkey
super + Return
  /opt/kitty/bin/kitty
```

**Configuration File → `~/.config/kitty/kitty.conf`**

**More information [here](https://github.com/kovidgoyal/kitty)**

>  💫 Awesome Documentation → **[Kitty Site](https://sw.kovidgoyal.net/kitty/)**

---

##### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `sxhkd`
>
> There are some dependencies that are needed in the following installation steps
>

---

##### Configuration File

> **Copy from [here](https://pastebin.com/9PxJas1E) the `kitty` Full Configuration File** 

**Configuration File Documentation → [kitty.conf](https://sw.kovidgoyal.net/kitty/conf/)**

---

##### Actions and Default Shortcuts

**All mappable actions [here](https://sw.kovidgoyal.net/kitty/actions/)**

**All default shortcuts [here](https://sw.kovidgoyal.net/kitty/overview/#tabs-and-windows)**

---

##### Custom Shortcuts





