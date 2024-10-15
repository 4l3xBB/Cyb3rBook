---
Primary_category: "[[SERVER SETUP]]"
title: "TMUX"
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[SERVER SETUP]]

**[TMUX](https://github.com/tmux/tmux) → Terminal Multiplexer**

It enables a number of terminal to be controlled in a single screen

*Tmux* works with *Sessions*

A *Session* can contain one or several *Windows*

Each *Window* can be divided into several *Panes*

![[TMUX-20241015165125813.webp|500]]

***More information [here](https://github.com/tmux/tmux/wiki)***

***[Man Page](https://man7.org/linux/man-pages/man1/tmux.1.html)***

---

#### Installation

##### Dependencies

*Tmux* requires some dependencies to be compiled such as a **C compiler, libraries...**

Install all of them as follows →

```bash
sudo apt install -y -- libevent-dev ncurses-dev build-essential bison pkg-config
```

If any of the above tools are not available as packages, build them from source

> ***[See here](https://github.com/tmux/tmux/wiki/Installing#building-dependencies)***

##### *Tmux*

> ***[Reference](https://github.com/tmux/tmux/wiki/Installing#from-source-tarball)***

Download the *source code* from the *[Releases](https://github.com/tmux/tmux/releases)* Page and Install it as follows →

```bash
wget https://github.com/tmux/tmux/releases/download/X.Xa/tmux-X.Xa.tar.gz
```

```bash
tar -zxf tmux-*.tar.gz
cd tmux-*/
./configure
make && sudo make install
```

```bash
cd - && rm -rf -- ./tmux-*
```

That's it!

```bash
$ command -V tmux
tmux is /usr/local/bin/tmux
```

The default installation directory is `/usr/local`. To change it, add the `--prefix` option to the `./configure` script

```bash
./configure --prefix=/usr/bin
```

###### Uninstallation

According to the *Makefile*, during installation via the `sudo make install` command, the installed files are →

- `/usr/local/bin/tmux`
- `/usr/local/share/man/man1/tmux.1`

To remove them, proceed as follows →

```bash
sudo rm -rf -- /usr/local/bin/tmux /usr/local/share/man/man1/tmux.1
```

> [!INFO]-
>
> If the `--prefix` option was used during the installation process, simply remove the *Tmux* binary from the specified path
>
> Note that all *Installation Paths*, like the ones above, are always specified in the *Makefile*
>
> Therefore, edit that file to check the value of the `Prefix` parameter
>

---

#### Configuration File

##### *Tmux.conf*

> ***[See here]()***

###### Default Prefix Modification

```bash title="~/.tmux.conf"
unbind-key C-b
set-option -g prefix M-i
bind-key M-i send-prefix
```

---

#### Shortcuts

***Default Prefix → `C-b`***

***Custom Prefix → `M-i`***

##### *Shorcuts' Meaning*

| **Key** | **Meaning** | |
| --- | --- | --- |
| ***\<leader\>*** | **`Space`** | **`<leader> + a` → `Space+a`** |
| ***C*** | **`Control`** | **`C-c` → `Control+c`** |
| ***M*** | **`Alt`** | **`M-a` → `Alt+a`** |
| ***S*** | **`Shift`** | **`S-o` → `Shift+o`**  |
| ***Super*** | **`Windows`** | **`Super-s` → `Windows+s`** |
| ***Return*** | **`Enter`** | **`C-S-Return` → `Control+Shift+Enter`** |
| ***-*** | **`+`** | **`C-z` → `Control+z`** |
| ***{a,b,c,d}*** | **`a` `b` `c` `d`** | **`C-{a,b,c,d}` → `C-a` `C-b` `C-c` `C-d`** |

#####  *Windows*

| **Action** | **Shortcut** |
| --- | --- |
| ***Windows Creation*** | **`c`** |
| ***Windows Closing*** | **`&`** |
| ***Windows Rename*** | **`comma`** |

##### *Panes*

| **Action** | **Shortcut** |
| --- | --- |
| ***Vertical Split*** | **`Return`** |
| ***Horizontal Split*** | **`dash`** |
| ***Pane Closing*** | **`x`** |
| ***Close All Panes (Except the focused one)*** | **`C-o`** |
| ***Pane Zoom In/Out*** | **`z`** |
| ***Pane Focus⬆️⬇️⬅️➡️*** | **`C-{k,j,h,l}`** |
| ***Pane Movement⬆️⬇️⬅️➡️*** | **`S-{k,j,h,l}`** |
| ***Pane Resize⬆️⬇️⬅️➡️*** | **`M-{k,j,h,l}`** |
| ***Pane Resize - Reset (Toogle to Tiled Layout)*** | **`r`** |
| ***Toogle between All Layouts*** | **`Space`** |
| ***Pane Detach into a New Window*** | **`!`** |

