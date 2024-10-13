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
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[SETUP]]

**[ZSH](https://github.com/zsh-users/zsh) → Z Shell**

This **[[SHELL SCRIPTING|Shell]]** is prefered over _[[BASH|bash]]_ or _fish_ because of all the handy functionalities and customisation It offers to the users

It implements **handy features** such as →

**[[ZSH#ZSH-autocomplete|Autocompletion]] &nbsp;\~&nbsp; [[ZSH#ZSH-autosuggestions|Autosuggestions]] &nbsp;\~&nbsp; Syntactical Corrections &nbsp;\~&nbsp; Advanced [[Globbing]]**

**Shared History between different terminals &nbsp;\~&nbsp; [[ZSH#ZSH-syntax-highlighting|Syntax Highlighting]]**

**Main File → `~/.zshrc`**

**Source File → `~/.config/zsh/src/custom.zsh`**

**Theme's Configuration File (*Powerlevel10k*) → `~/.p10k.zsh`**

**More info [here](https://github.com/zsh-users/zsh)**

**_[ZSH Manual](https://zsh.sourceforge.io/Doc/)_**

---

#### Plugins

Most of the above features can be implemented or enhanced through _ZSH Plugins_ that are sourced from the `.zshrc` [[ZSH#Configuration File|configuration file]]

> The _.zsh_ files related to the plugins below are sourced from the `.zshrc` configuration file

##### ZSH-autocomplete

> ***[Reference](https://github.com/marlonrichert/zsh-autocomplete)***

It enhances the _ZSH_'s inherent auto-complete capability

![[zsh-autocomppletion.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash title="~/.zshrc"
if [[ -f /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh ]]; then
	source /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh
fi
```

##### ZSH-autosuggestions

> ***[Reference](https://github.com/zsh-users/zsh-autosuggestions/tree/master)***

It suggests commands as the user types based on history and the previous completions

![[zsh-autosuggestions.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash title="~/.zshrc"
if [[ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]]; then
  source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
fi
```

##### ZSH-syntax-highlighting

> ***[Reference](https://github.com/zsh-users/zsh-syntax-highlighting)***

It enables highlighting of commands while typing them in an interactive terminal

![[zsh-syntaxhighlighting.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash title="~/.zshrc"
if [[ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]]; then
  source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
fi
```

###### Dracula Theme

There are different themes for this plugin, the one used in this [[SETUP|setup enviroment]] is the **[Dracula Theme](https://github.com/dracula/dracula-theme)**

> ***[Reference](https://draculatheme.com/zsh-syntax-highlighting)***

To use it, just `source`, in the `.zshrc` file, the _.zsh_ script resulting from the steps in the link above 

```bash title="~/.zshrc"
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

##### ZSH-sudo

> ***[Reference](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/sudo/sudo.plugin.zsh)***

This plugins automatically places the _sudo_ binary at the beginnng of the line by pressing the **`Esc`** key twice

![[zsh-sudo.svg|350]]

_.zsh_ file is sourced from `.zshrc`

```bash title="~/.zshrc"
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
sudo apt install -y -- zsh
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

> Plugins' Path → `/usr/share/ZSH_PLUGIN_NAME`

To install most of the plugins → _[[ZSH#ZSH-autocomplete|Autocomplete]] ~ [[ZSH#ZSH-autosuggestions|Autosuggestions]] ~ [[ZSH#ZSH-syntax-highlighting|Syntax-hightlighting]]_

```bash
sudo apt install -y -- zsh-{autocomplete,autosuggestions,syntax-highlighting}
```

> [!INFO]-
>
> In this [[SETUP|Environment Setup]], the only _ZSH Plugins_ to be used are _ZSH-Syntax-highlighting_ and _ZSH-Sudo_

###### *ZSH-Sudo*

As _Root_, Install the _[[ZSH#zsh-sudo|ZSH-Sudo]] Plugin_, as follows  →

```bash
mkdir -p /usr/share/zsh-sudo ; (( $? )) || cd !$
```

```bash
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/plugins/sudo/sudo.plugin.zsh
```

###### *ZSH-Syntax-Highlighting*

It has been previously installed via `apt` in the `/usr/share/zsh-syntax-highlighting/`

As mentioned in the *[[ZSH#ZSH-syntax-highlighting|ZSH-Syntax-Highlighting]] Plugin's Section*, the *Theme* selected for this plugin is the *[[ZSH#Dracula Theme|Dracula Theme]]*

Install the above _Syntax Theme_ as follows →

```bash
mkdir -p ~/.config/zsh/zsh-syntaxhighlighting/themes/ ; cd !$
```

```bash
git clone https://github.com/dracula/zsh-syntax-highlighting.git dracula
```

```bash
mv ./dracula/zsh-syntax-highlighting.sh ./dracula.zsh && rm -rf !$
```

> [!CAUTION]-
>
> Note that the `dracula.zsh` script should only be sourced if the _zsh-syntax-highlighting_'s script has been sourced correctly previously
>
> See the *[[ZSH#Dracula Theme|Dracula theme]] Section* for more information about the _Source Code Block_

##### Theme

> ***[Reference](https://github.com/romkatv/powerlevel10k)***

**The _ZSH Theme_ to install → _Powerlevel10k_**

> [!IMPORTANT]-
>
> Note that the following steps related to the _ZSH Theme's Installation_ should be applied for both _non-privileged_ and _privileged_ users
>
> Being in this case for **_Root_** and **_al3xbb_**
>

Manual installation as follows →

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
```

To source the _powerlevel10k script_ , insert this line at the **beginning** of the `.zshrc` file →

```bash title="~/.zshrc"
source /home/al3xbb/powerlevel10k/powerlevel10k.zsh-theme
```

Then, to start the interactive _powerlevel10k's setup_ →

> _Hack Nerd Fonts_'s previous installation is required. See [[KITTY#Hack Nerd Fonts|here]]

```bash
source ~/.zshrc
```

> Do not forget to repeat the above steps for the other users

To set up a **more granular configuration**, just edit the [[ZSH#*.p10k.zsh*|.p10k.zsh]] file as follows →

###### *.p10k.zsh - Modified Sections*

In that _powerlevel10k file_, for this [[SETUP|Setup Enviroment]], as the _non-privileged user_, add/edit as follows →

- **Left Prompt Elements**

```bash title="~/.p10k.zsh"
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
	os_icon
	dir
	vcs	
	context
	command_execution_time
	status
)
```

- **Righ Prompt Elements** → All of them should be commented as in the [[ZSH#*.p10k.zsh*|configuration file]]

Regarding to the _[[ZSH#DIR|dir]] segment_, the _bold font_ can be disabled as follows →

```bash title="~/.p10k.zsh"
typeset -g POWERLEVEL9K_DIR_ANCHOR_BOLD=false # Or true to enable it
```

> Repeat all the above steps as _Root_

Only as _Root_, add/edit as follows to add an icon as a [[ZSH#CONTEXT|context]]→ 

> ***[Icon's Source](https://www.nerdfonts.com/cheat-sheet)***

```bash title="~/.p10k.zsh"
typeset -g POWERLEVEL9K_CONTEXT_ROOT_TEMPLATE='󰈸' # Custom Icon
typeset -g POWERLEVEL9K_CONTEXT_PREFIX='' # Empty it
```

##### *.ZSHRC File*

###### Symbolic Link

To avoid having to modify, when a change is made, both the _Root_ and the _Non-privileged User_'s [[ZSH#*.zshrc*|.zshrc]] file, simply proceed as follows →

> As _Root_

```bash
ln --symbolic --force -- /home/al3xbb/.zshrc ~/.zshrc
```

> [!INFO]-
>
> Above command just creates a _symbolic link_ as `~/.zshrc` which points to `/home/al3xbb/.zshrc`
>
> Therefore, any changes made to the _.zshrc_ file apply to both users

###### _ZSH_ Plugins

As mentioned in the [[ZSH#Plugins|Plugins]]' Section, each _Plugin's .zsh script_ must be sourced from the `.zshrc` file

In this [[SETUP|environment setup]], the sourced ones are → _[[ZSH#ZSH-syntax-highlighting|zsh-syntax-highlighting]] ~ [[ZSH#ZSH-sudo|zsh-sudo]]_

Check if exist and source them →

- ***ZSH-Sudo***

```bash title="~/.zshrc"
if [[ -f /usr/share/zsh-sudo/sudo.plugin.zsh ]] ; then
	source /usr/share/zsh-sudo/sudo.plugin.zsh
fi
```

- ***ZSH-Syntax-Highlighting → 🦇 Dracula Theme 🧛🏻‍♂️***

```bash title="~/.zshrc"
if [[ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]]; then
  source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

  if [[ -f /home/al3xbb/.config/zsh/zsh-syntaxhighlighting/themes/dracula.zsh ]] ; then
    source /home/al3xbb/.config/zsh/zsh-syntaxhighlighting/themes/dracula.zsh
  fi
fi
```

> _ZSH Plugin and Plugin's Theme sourced_

---

#### Configuration File

##### *ZSH*

###### *.zshrc*

**_ZSH_ Configuration File → [.zshrc](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/.zshrc)**

###### *src/custom.zsh*

**_ZSH_ Source File → [custom.zsh](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/src/custom.zsh)**

##### *Powerlevel10k*

###### *.p10k.zsh*

**_Non-Privileged User's Powerlevel10k_ Configuration File → [.p10k.zsh](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/themes/p10k/users/.p10k.zsh)**

**_Root's Powerlevel10k_ Configuration File →[.p10k.zsh](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/themes/p10k/root/.p10k.zsh)**

---

#### Parameters

##### *Z Shell ~ .zshrc*

###### PATH

> _PATH Enviromental Parameter_

```bash
export PATH=/opt/kitty/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/sbin:/opt/nvim/nvim-linux64/bin:/home/al3xbb/.fzf/bin
```

In addition to the usual paths such as `/usr/bin:/bin:/usr/sbin:/usr/local/bin`, others are added →

- _[[KITTY|Kitty]]_ → `/opt/kitty/bin`
- _[[NEOVIM|Neovim]]_ → `/opt/nvim/nvim-linux64/bin`
- _[[FZF]]_ → `/home/al3xbb/.fzf/bin`

> **This allows to use their binaries from any path relatively**

###### Aliases

Various _aliases_ are defined to carry out the following sustitutions →

- **`batcat` rather than `cat`**

```bash
alias cat='bat' # Standard sustitution
alias catn='bat --style=plain' # Only shows Plain Text (No decorations)
alias catnp='bat --style=plain --paging=never' # Plain Text and No Pagination
```

> ***[[BAT|Reference]]***

- **`lsd` rather than `ls`** 

```bash
alias ll='lsd -lh --group-dirs=first'
alias la='lsd -a --group-dirs=first'
alias l='lsd --group-dirs=first'
alias lla='lsd -lha --group-dirs=first'
alias ls='lsd --group-dirs=first'
```

> ***[[LSD|Reference]]***

To remove the `lsd`'s _Bold_ applied to the font and file icons, just declare the `LS_COLORS` parameter as It appears in the [[ZSH#*.zshrc*|.zshrc]] configuration file

###### Command History

**Command History File** → `.zsh_history`

```bash
HISTFILE=~/.zsh_history
```

**Command History File's Size** → Number of entries in `.zsh_history`

```bash
SAVEHIST=10000
```

**Command History's Memory entries** → Number of entries in the History Memory

```bash
HISTSIZE=10000
```

To **ignore duplicate entries** in the _Command History_ and **Synchronise** it between the open terminals →

```bash
setopt histignorealldups sharehistory # Respectively
```

###### Command Line Editor

Use the following _ZSH Builints_ to enable the _Command Line Editor_ →

```bash
autoload -U edit-command-line # Loads the ZSH Function deferred
zsl -N edit-command-line # Widget Creation from this ZSH Function
binkey '\C-x\C-e' edit-command-line # Widget-Keybind association
```

The above code ensures that the _Command Line Editor_ can be accessed through `C-x C-e` as in [[BASH|bash]]

> [!IMPORTANT]-
>
> Note that the _edit-command-line ZSH Function_ opens the _Command Line Editor_ with the text editor set as value in the _EDITOR_ parameter

It is necessary to stablish the _Text Editor_ used in the _Command Line Editor_

This is carried out through the `EDITOR` parameter

**Editor → [[NEOVIM|Neovim]]**

```bash
export EDITOR=/opt/nvim/nvim-linux64/bin/nvim
```

> This is extremely useful when e.g. doing `C-x C-e` to open the _Command Line Editor_

![[zsh_terminal_editor.gif|375]]

###### Autocompletion System

To enable the _ZSH's Modern Autocompletion System_  →

```bash
autoload -Uz compinit && compinit
```

> [!INFO]-
>
> The above command deferred loads the _ZSH's compinit function_ and, if _true_, calls it
>
> Note that the `-U` option causes its function reference to not be loaded into the _ZSH Hash Table_
>
> While the `-z` option only checks that the function provided as an argument is strictly for _ZSH_
>

If the `compinit` function is loaded and called correctly, then various additional configurations are applied to the _ZSH Autocompletion System_

These can be found in the [[ZSH#*.zshrc*|.zshrc]] configuration file

###### FZF - _Fuzzy Finder_

> ***[[FZF|Reference]]***

This _Fuzzy Finder_ is mainly used to modify the `C-r` [[SHELL SCRIPTING|Shell]]'s shortcut for _reverse history search_

To load all _FZF Functionality and Shortcuts_ in the _ZSH Process Context_ →

```bash
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh 
```

Then, to enable some _FZF_'s handy features →

```bash
export FZF_DEFAULT_OPTS="--height 40% --border --preview 'bat --color=always {} 2> /dev/null'" # File Preview with BAT
```

```bash
export FZF_DEFAULT_COMMAND="fd --type f" # FD instead of Find
```

![[zsh_fzf.gif|375]]

###### JAVA Troubleshooting

There are _Windows Managers_, such as [[BSPWM|bspwm]], that do not perform any _Windows Reparenting_

Therefore, any _Java Application_ that uses _JWT_, may experience some integration problems in an environment with these _Windows Managers_

In order to avoid the previous problems →

```bash
export _JAVA_AWT_WM_NONREPARENTING=1
```

The above code simply tells _Java_ to carry out the _Windows Reparenting_ instead of the _Windows Manager_

> [!IMPORTANT]-
>
> Note that, in addition to the above measure, another is implemented in the [[BSPWM#*bspwmrc*|bspwmrc]]
>
> ```bash
> wmname LG3D &
> ```
> This changes the current _Windows Manager_'s name, which the System recognises, to _LG3D_
>
> _LG3D_ is an old _Windows Manager_, and the _Java Applications_ created with _JWT_ run correctly on it
>
> So changing the _Windows Manager_'s Name tricks *Java* into thinking It is running in a supported environment
>

###### Custom Functions

All the _Custom Functions_ related to the [[ZSH#*.zshrc*|.zshrc]] file are sourced from the [[ZSH#*src/custom.zsh*|custom.zsh]] script

There are several handy functions such as →

- ***clearScreenAndScrollback → [[KITTY#Clear Screen and Scrollback Buffer|Reference]]***

The remaining functions are related to the _[[PENTESTING|Pentesting]] Index Category_

- ![](https://wallpapercave.com/wp/wp2661451.jpg)
	- [[ZSH CUSTOM FUNCTIONS]]

<br>

##### *Powerlevel10k* ~ *.p10k.zsh*

> ***[Reference](https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#batteries-included)***

This _ZSH Theme_ is extremely customizable

The basic units are the _segments_, which display specific information from different sources at the _user's prompt_

![[ZSH-20240929121722873.webp|400]]

The _segments_ can be located in the left or right side  (_i.e. the left or right prompt_)

In this [[SETUP|environment setup]], only those on the right are enabled →

###### OS_ICON

> [!INFO]-
>
> Note that the below code snippets are related to the _.p10.zsh_ file
>

It displays the _Operative System Icon_

To enable it →

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( os_icon )
```

To modify the _displayed icon_ and its colour →

```bash
typeset -g POWERLEVEL9K_OS_ICON_CONTENT_EXPANSION='' # Custom Icon
typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=255 # Icon's Colour
```

###### DIR

It displays the _Current Work Directory_

To enable it →

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( dir )
```

![[zsh_p10k_dir.gif|400]]

> Note how the _Current Work Directory_ changes continously

###### VCS

If the _Current Work Directory_ is a _Github Repository_, **It displays de Git Status**

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( vcs )
```

![[zsh_p10k_git.gif|375]]

> See → _"on😺🌿master"_ 

###### CONTEXT

In this case, if someone is connected to other host remotely, **It shows the current user and hostname**

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( context )
```

![[zsh_p10k_context.gif|375]]

> See → _"with test@parrot"_

###### COMMAND_EXECUTION_TIME

It displays the last command's time duration, from a `sleep` command to an _SSH Session_

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( command_execution_time )
```

![[zsh_p10k_cmdExectime.gif|375]]

> See → _"Took⌛12s"_

###### STATUS

It displays the _Exit Code_ of the last command

```bash
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=( status )
```

![[zsh_p10k_status.gif|375]]

> See → _"❌127/1/INT/TSTP ~ ✅1"_