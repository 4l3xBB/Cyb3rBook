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

It implements **handy features** such as →

**Advanced Autocomplete ~ Autosuggestions ~ Syntactical Corrections ~ Advanced [[Globbing]]**

**Shared History between different terminals ~ Syntax Highlighting**

**Main File → `~/.zshrc`**

**More info [here](https://github.com/zsh-users/zsh) and in the _[ZSH Manual](https://zsh.sourceforge.io/Doc/)_**

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

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm]], [[SXHKD|sxhkd]] and  [[KITTY|kitty]] installations before proceeding with this one related to _ZSH_
>
> The above [[SETUP|setup]] components facilitate the _ZSH Configuration_
>

```bash
$ sudo apt install -y -- zsh
```

##### User's Default Shell

To change the _users' default shell_ to a _ZSH_ one, as _Root_ →

```bash
for _user in al3xbb root
do
    usermod --shell "$( command -v zsh )" "$_user"
done
```

##### Plugins

To install most of the plugins → _[[ZSH#ZSH-autocomplete|Autocomplete]] ~ [[ZSH#ZSH-autosuggestions|Autosuggestions]] ~ [[ZSH#ZSH-syntax-highlighting|Syntax-hightlighting]]_

```bash
$ sudo apt install -y -- zsh-{autocomplete,autosuggestions,syntax-highlighting}
```

As _Root_, Install the _[[ZSH#zsh-sudo|ZSH-Sudo]] Plugin_, as follows  →

```bash
$ mkdir -p /usr/share/zsh-sudo ; (( $? )) || cd !$
```

```bash
$ wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/plugins/sudo/sudo.plugin.zsh
```

> [!INFO]-
>
> In this [[SETUP|Environment Setup]], the only _ZSH Plugins_ to be used are _ZSH-Syntax-highlighting_ and _ZSH-Sudo_

##### Theme

> ***[Reference](https://github.com/romkatv/powerlevel10k)***

**The _ZSH Theme_ to install → _Powerlevel10k_**

Manual installation as follows →

```bash
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
```

To source the _powerlevel10k script_ , insert this line at the **beginning** of the `.zshrc` file →

```bash
source /home/al3xbb/powerlevel10k/powerlevel10k.zsh-theme
```

Then, to start the interactive _powerlevel10k's setup_ →

> _Hack Nerd Fonts_'s previous installation is required. See [[KITTY#Hack Nerd Fonts|here]]

```bash
$ source ~/.zshrc
```

---

#### Configuration File

**_ZSH_ Configuration File → [.zshrc](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/.zshrc)**

**_Powerlevel10k_ Configuration File → [.p10k.zsh](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/themes/.p10k.zsh)**