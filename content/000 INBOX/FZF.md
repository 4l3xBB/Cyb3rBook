---
Primary_category: "[[SETUP]]"
title: FZF
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

***FZF → A General-Purpose Command-Line Fuzzy Finder 🔎*** 

It is an interactive ***Text Filter*** program for any kind of list such as →

- ***Files***

![[FZF_Files.gif|375]]

- ***Command History → `C-R`***

![[zsh_fzf.gif|375]]

- ***System Processes → `Tab`***

![[FZF_Processes.gif|375]]

- ***SSH Connections → `Tab`***

![[FZF_SSH.gif|375]]

###### References

***[Github Repository](https://github.com/junegunn/fzf)***

***[FZF Documentation](https://junegunn.github.io/fzf/)***

***Check [this](https://www.redhat.com/sysadmin/fzf-linux-fuzzy-finder) out too***

##### Installation

> ***[Reference](https://github.com/junegunn/fzf?tab=readme-ov-file#installation)***

Clone the _Github Repository_ and run the _Install Script_ as follows →

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
```

```bash
~/.fzf/install
```

That's it!

```bash
$ command -V fzf
fzf is /home/al3xbb/.fzf/bin/fzf
```

To enable some handy features of _FZF_, add the following lines in the _[[ZSH#*.zshrc*|.zshrc]] file_ →

```bash
export FZF_DEFAULT_OPTS="--height 40% --border --preview 'bat --color=always {} 2> /dev/null'" # File Preview with Bat
```

The above one modifies the *FZF's Display Format and enables* *File Preview Mode with* *BAT*

![[FZF_FILE_DISPLAY.gif|375]]


```bash
export FZF_DEFAULT_COMMAND="fd --type f" # FD instead of Find
```

> [!IMPORTANT]-
>
> Note that the above actions require _[[BAT|Bat]]_ and _FD_ to be installed in the system
>

##### *Integration*

***FZF* supports integrations with some *Tools and Functionalities* such as →**

###### *Nvim/Vim/Vi*

```bash
$ nvim $( fzf )
```

![[FZF_NVIM.gif|375]]

###### *Bat/Cat*

```bash
$ bat $( fzf )
```

![[FZF_BAT-ezgif.gif|375]]