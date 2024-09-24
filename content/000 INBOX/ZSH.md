---
Primary_category: "[[SETUP]]"
title: ZSH
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - ZSH
  - pentesting👹
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

**_ZSH_ → Z Shell**

This **[[SHELL SCRIPTING|shell]]** is prefered over _[[BASH|bash]]_ or _fish_ because of all the handy functionalities and customisation It offers to the users

It implements features such as →

- **Advanced Autocomplete**
- **Autosuggestions**
- **Syntactical Corrections**
- **Advanced [[Globbing]]**
- **Shared History between different terminals**
- **Syntax Highlighting**

#### Plugins

Most of the above features can be implemented or enhanced through _ZSH Plugins_ that are sourced from the `.zshrc` [[ZSH#Configuration File|configuration file]]

> The _.zsh_ files related to the plugins below are sourced from the `.zshrc` configuration file

##### ZSH-autocomplete

> ***[Reference](https://github.com/marlonrichert/zsh-autocomplete)***

It enhances the _ZSH_'s inherent auto-complete capability

![[zsh-autocomppletion.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash
if [[ -f /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh ]]; then
	source /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh
fi
```

##### ZSH-autosuggestions

> ***[Reference](https://github.com/zsh-users/zsh-autosuggestions/tree/master)***

It suggests commands as the user types based on history and the previous completions

![[zsh-autosuggestions.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash
if [[ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]]; then
  source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
fi
```

##### ZSH-syntax-highlighting

> ***[Reference](https://github.com/zsh-users/zsh-syntax-highlighting)***

It enables highlighting of commands while typing them in an interactive terminal

![[zsh-syntaxhighlighting.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash
if [[ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]]; then
  source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
fi
```

###### Dracula Theme

There are different themes for this plugin, the one used in this [[SETUP|setup enviroment]] is the **[Dracula Theme](https://github.com/dracula/dracula-theme)**

> ***[Reference](https://draculatheme.com/zsh-syntax-highlighting)***

To use it, just `source`, in the `.zshrc` file, the _.zsh_ script resulting from the steps in the link above 

```bash
if [[ -f /home/al3xbb/.config/zsh/zsh-syntaxhighlighting/themes/dracula.zsh ]] ; then
  source /home/al3xbb/.config/zsh/zsh-syntaxhighlighting/themes/dracula.zsh
fi
```

> [!IMPORTANT]-
> 
> Note that the `dracula.zsh` script should only be sourced if the _zsh-syntax-highlighting_'s script has been sourced correctly previously
>
> The above action is performed in the _ZSH Custom [[ZSH#Configuration File|Configuration File]]_. It implements something like this →
>
> ```bash
> $ [[ -f /plugin.zsh ]] && . /plugin.zsh && . /theme.zsh # Or source
> ```
>
>
> Moreover, It is necessary to create a directory where to store the `dracula.zsh` script after clone the repository →
>
> ```bash
> $ mkdir -p ~/.config/zsh/zsh-syntaxhighlighting/themes/ ; cd !$
> ```
>```bash
> $ git clone https://github.com/dracula/zsh-syntax-highlighting.git dracula
>```
> ```bash
> $ cd !$ && mv zsh-syntax-highlighting.sh ../dracula.zsh
> ```

##### ZSH-sudo

> ***[Reference](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/sudo/sudo.plugin.zsh)***

This plugins automatically places the _sudo_ binary at the beginnng of the line by pressing the **`Esc`** key twice

![[zsh-sudo.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash
if [[ -f /usr/share/zsh-sudo/sudo.plugin.zsh ]]; then
  source /usr/share/zsh-sudo/sudo.plugin.zsh
fi
```

---

#### Installation

To install this 

---

#### Configuration File

**[.zshrc File](https://pastebin.com/sg98cGRu)**