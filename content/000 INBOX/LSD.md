---
Primary_category: "[[SETUP]]"
title: LSD
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

***LSD → The next Gen LS Command 🗂️***

This `ls` fork offers features such as ***colors, icons, tree-view and more...***

![[lsd_directory_display.gif|485]]

**More info [here](https://github.com/lsd-rs/lsd)**

#### Installation

First of all, access to the _[Releases](https://github.com/lsd-rs/lsd/releases)_ page and copy the _Download Link_ of the _**lsd_X.X.X_amd64.deb**_

Then, install it as follows →

```bash
wget -O lsd.deb "DOWNLOAD_LINK_LSD_AMD64.DEB"
```

```bash
sudo dpkg --install lsd.deb
```

That's it!

```bash
$ command -V lsd && lsd --version
lsd is /usr/bin/lsd
lsd 1.1.5
```

##### *~~LS~~ → LSD*

To replace `ls` with `lsd`, simply add the following aliases in the _[[ZSH#*.zshrc*|.zshrc]]_ file →

```bash title="~/.zshrc"
alias ll='lsd -lh --group-dirs=first'
alias la='lsd -a --group-dirs=first'
alias l='lsd --group-dirs=first'
alias lla='lsd -lha --group-dirs=first'
alias ls='lsd --group-dirs=first'
```

That's it!

```bash
$ command -V ls
ls is an alias for lsd --group-dirs=first
```