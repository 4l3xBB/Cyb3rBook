---
Primary_category: "[[DESKTOP SETUP]]"
title: NEOVIM
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORIES → [[DESKTOP SETUP]]&nbsp;&nbsp;•&nbsp;&nbsp;[[SERVER SETUP]]

**[NEOVIM](https://github.com/neovim/neovim) → *Vim Fork focused on extensibility and usability***

It greatly improves the user experience with a _Text Editor_ and enhances aspects such as ***Efficiency and Performance***

##### Compared to *Vim*

- ***More Modern***
<br>
- ***More Extensible***
<br>
- ***More Efficient***

##### Modal Editor

Therefore, each mode has its own _keybinds_ mapped to a specifics actions →

- ***Normal Mode*** 
<br>
- ***Insert Mode***
<br>
- ***Visual Mode***

**More information [here](https://github.com/neovim/neovim)**

***[Neovim Documentation](https://neovim.io/doc/)***

---

#### Installation

##### *Neovim - Standard Installation*

> ***[Reference](https://github.com/neovim/neovim/blob/master/INSTALL.md#linux)***

If *Neovim* is already installed, It is probably an older version that the one to be installed below

So, just uninstall it →

```bash
sudo apt remove -y -- neovim
```

Access to the *[Releases](https://github.com/neovim/neovim/releases)* to copy the _Download Link_ of the *nvim-linux64.tar.gz* to `wget` it or simply proceed as follows →

```bash
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
sudo rm -rf /opt/nvim
sudo tar -C /opt -xzf nvim-linux-x86_64.tar.gz
```

Then, add this to the _Shell Configuration File_ → _[[ZSH#*.zshrc*|.zshrc]]_

```bash title="~/.zshrc"
export PATH="$PATH:/opt/nvim-linux-x86_64/bin"
export EDITOR="/opt/nvim-linux-x86_64/bin/nvim"
```

Also execute the above command to apply changes in the current _Shell Context_ or →

```bash
source ~/.zshrc
```

That's it!

> ***To setup a Neovim Distro, See [[#Distributions|Here]]***

```bash
$ command -V nvim
nvim is /opt/nvim-linux64/bin/nvim
```

##### *Neovim - Installation From Source*

If *Neovim* is already installed, It is probably an older version that the one to be installed below

So, just uninstall it →

```bash
sudo apt remove -y -- neovim
```

###### *Prerequisites*

> ***[Reference](https://github.com/neovim/neovim/blob/master/BUILD.md#build-prerequisites)***

```bash
sudo apt install -y -- ninja-build gettext cmake unzip curl build-essential
```

###### *CMake Installation*

> ***[Reference](https://cmake.org/download/)***

This binaries suite is necessary in order to correctly compile from source the ***[neovim](https://github.com/neovim/neovim)*** project

First, take a look at the ***[releases](https://cmake.org/download/)*** and download the latest version whose installation is performed by a *.sh* script

Then, proceed as follows →

```bash
mkdir /opt/cmake
cd !$ && wget -O install.sh "https://github.com/Kitware/CMake/releases/download/vX.XX.X/cmake-X.XX.X-linux-x86_64.sh"
chmod 700 install.sh && ./install.sh
```

Note that the installation is performed on the `/opt/cmake` directory. So, simply add the previous *directory path* to the *PATH* environment parameter

> [!IMPORTANT]-
>
> Be aware that the path where the `cmake` installation was performed must be set before the standard `cmake` path in the *PATH* env parameter in order to use the installed `cmake` version
>

###### *Neovim Installation*

> ***[Reference I](https://github.com/neovim/neovim/blob/master/INSTALL.md#install-from-source)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference II](https://github.com/neovim/neovim/blob/master/BUILD.md)*** 

```bash
git clone https://github.com/neovim/neovim Neovim
```

```bash
cd !$ && make CMAKE_BUILD_TYPE=RelWithDebInfo
```

```bash
make install
```

After this, the `neovim` binary is installed in the `/usr/loca/` directory, so there is no need to modify the *PATH* environment parameter since the above path is already in it

```bash
$ command -V nvim
nvim is /usr/local/nvim
```

---

#### Distributions

To improve even more the ***Editor Base Experience***, just try one of the following _distros_ →

- ![](https://img.freepik.com/premium-photo/living-bust-portrait-v-vendetta-illustration-high-quality-detailed-art-nouveau-style_1157627-165.jpg)
	- [[NVCHAD]]

<br>

#### *Nvim Cheatsheet*

![[NEOVIM-20241006135518276.webp]]