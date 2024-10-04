---
Primary_category: "[[SETUP]]"
title: BAT
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

***Batcat → A cat(1) clone with Wings 🦇***

This `cat` clone offers features such as →

- ***Syntax Highlighting***

It supports it for a large number of *Programming* and *Markup* Languages

To list all supported →

```bash
$ bat --list-language
```

![[batcat_file_display.gif|400]]

- ***Git Integration***

> ***[Reference](https://github.com/sharkdp/bat#git-integration)***

It displays the modifications compared to the last _Commit_

- ***Non-Printable Characters Display***

> ***[Reference](https://github.com/sharkdp/bat#show-non-printable-characters)***

- ***Pagination***

**More info [here](https://github.com/sharkdp/bat)**

##### Installation

> ***[Reference](https://github.com/sharkdp/bat?tab=readme-ov-file#installation)***

First of all, access to the *[Releases](https://github.com/sharkdp/bat/releases)* page and copy the _Download Link_ of the ***bat_X.X.X_amd64.deb***

Then, install it as follows →

```bash
$ wget -O bat.deb "DOWNLOAD_LINK_BAT_AMD64.DEB"
```

```bash
$ sudo dpkg --install bat.deb
```

That's it!

```bash
$ command -V bat && bat --version
bat is /usr/bin/bat
bat 0.24.0 (fc95468)
```

##### *~~CAT~~ → BAT*

To replace `cat` with `bat`, simply add the following aliases in the _[[ZSH#*.zshrc*|.zshrc]]_ file →

```bash
alias cat='bat' # Standard sustitution
alias catn='bat --style=plain' # Only shows Plain Text (No decorations)
alias catnp='bat --style=plain --paging=never' # Plain Text and No Pagination
```

That's it!

```bash
$ command -V cat
cat is an alias for bat 
```

##### Integration

> ***[Reference](https://github.com/sharkdp/bat?tab=readme-ov-file#integration-with-other-tools)***

***BAT* supports integrations with other *Tools or Functionalities* such as →**

###### *Find ~ FD*

```bash
$ sudo find /path/to/dir ... -exec bat {} +
```

```bash
$ fd . /path/to/dir ... --exec-batch bat {}
```

###### *Command's Help*

- ***Paginate →***

```bash
$ nmap --help | bat --plain --language help # Or bat -pl help
```

- ***Non-Paginate →***

```bash
$ nmap --help | bat --plain --language help --paging=never # Or bat -ppl help
```


###### *Man*

```bash
$ export MANPAGER="sh -c 'col -bx | bat -l man -p'"
$ man nmap
```

To make the above action permanent, add the following line in the _[[ZSH#*.zshrc*|.zshrc]] file_ →

```bash
export MANPAGER="sh -c 'col -bx | bat -l man -p'"
```

###### *Tail -{F,f}*

Specially useful for continuos monitoring a file with _Syntax Highlighting_

```bash
$ tail -F /path/to/file.log | bat --paging=never --language log
```