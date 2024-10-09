---
Primary_category: "[[SETUP]]"
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

###### PRIMARY CATEGORY → [[SETUP]]

***Neovim → Vim Fork focused on extensibility and usability***

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

**More info [here](https://github.com/neovim/neovim)**

***[Neovim Documentation](https://neovim.io/doc/)***

#### Installation

##### *Neovim*

> ***[Reference](https://github.com/neovim/neovim/blob/master/INSTALL.md#linux)***

If *Neovim* is already installed, It is probably an older version that the one to be installed below. So, just uninstall it →

```bash
sudo apt remove -y -- neovim
```

Access to the *[Releases](https://github.com/neovim/neovim/releases)* to copy the _Download Link_ of the *nvim-linux64.tar.gz* to `wget` it or simply proceed as follows →

```bash
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz
sudo rm -rf /opt/nvim
sudo tar -C /opt -xzf nvim-linux64.tar.gz
```

Then, add this to the _Shell Configuration File_ → _[[ZSH#*.zshrc*|.zshrc]]_

```bash title="~/.zshrc"
export PATH="$PATH:/opt/nvim-linux64/bin"
export EDITOR=/opt/nvim-linux64/bin/nvim
```

Also execute the above command to apply changes in the current _Shell Context_ or →

```bash
source ~/.zshrc
```

That's it!

```bash
$ command -V nvim
nvim is /opt/nvim-linux64/bin/nvim
```

> ***To setup a Neovim Distro, See [[#Distributions|Here]]***

#### Distributions

To improve even more the ***Editor Base Experience***, just try one of the following _distros_ →

- ![](https://img.freepik.com/premium-photo/living-bust-portrait-v-vendetta-illustration-high-quality-detailed-art-nouveau-style_1157627-165.jpg)
	- [[NVCHAD]]

<br>

#### *Nvim Cheatsheet*

![[NEOVIM-20241006135518276.webp]]
