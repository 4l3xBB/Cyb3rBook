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

![[Excalidraw/Drawing 2024-09-21 12.07.48.excalidraw.md#^group=t1N4qrO2inu-0N-uPYbeY]]

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

#### Installation

> [!CAUTION]-
>
> First, see the [[BSPWM|bspwm's installation]] before proceeding with this one related to `sxhkd`
>
> There are some dependencies that are needed in the following installation steps
>

---

#### Configuration File

> **Copy from [here](https://pastebin.com/9PxJas1E) the `kitty`'s Full Customised Configuration File** 

**Configuration File Documentation → [kitty.conf](https://sw.kovidgoyal.net/kitty/conf/)**

---

#### Actions and Default Shortcuts

**All mappable actions [here](https://sw.kovidgoyal.net/kitty/actions/)**

**All default shortcuts [here](https://sw.kovidgoyal.net/kitty/overview/#tabs-and-windows)** related to Windows, Tabs, Layouts...

---

#### Custom Shortcuts

Note that, as mentioned earlier, a keybind is mapped to a [[KITTY#Actions and Default Shortcuts|specific action]]

**Keybind → Action**

```bash
map KEYBIND ACTION # An Action may has arguments
```

##### Layouts

##### Window Management

######

##### Tab Management

##### *Copy/Paste*

##### Miscellaneous

###### Clear Screen and Scrollback Buffer

> ***[Reference](https://sw.kovidgoyal.net/kitty/conf/#shortcut-kitty.Reset-the-terminal)***

```bash
map ctrl+l clear_terminal to_cursor active
```

> [!IMPORTANT]-
>
> The same or similar actions can also be performed via functions declaration in a shell rc file such as the [[ZSH#Configuration File|zshrc]]
>
> - **Clear Screen** →
> ```bash
> clearOnlyScreen() {
>     printf "\e[H\e[2J"
> 	zle && zle .reset-prompt && zle -R
> }
> ```
> - **Clear Screen and Scrollback Buffer** →
> ```bash
> clearScreenAndScrollback() { 
>     printf "\e[H\e[3J"
>     zle && zle .reset-prompt && zle -R	
> }
> ```
> - **Clear Screen saving its content in Scrollback Buffer** →
> ```bash
> clearScreenSavingContentsInScrollback() {
>     printf "\e[H\e[22J"
>     zle && zle .reset-prompt && zle -R
> }
> ```
>
> Note that the _ZSH Line Editor_ `zle` is used in the above functions to prevent some undesired actions and perform a more optimal cleanup
> 
> - `zle` → Verify if the _ZSH Line Editor_ is available
> <br>
> - `zle .reset-prompt` → Prompt Restart to prevent corruption due to clear or reset actions
>
> See both [issue1](https://github.com/sorin-ionescu/prezto/issues/1026) and [issue2](https://github.com/zsh-users/zsh-autosuggestions/issues/107#issuecomment-183824034)
>
> - `zle -R` → Prompt Redisplay to prevent visual problems
>
> See this [pull request discussion](https://github.com/Powerlevel9k/powerlevel9k/pull/1176#discussion_r299303453)

In this [[SETUP|environment setup]], this action is implemented in the [[ZSH#Configuration File|zshrc]] as follows →

> ***[Reference](https://unix.stackexchange.com/questions/517025/zsh-clear-scrollback-buffer#answer-531178)***

```bash
clearScreenAndScrollback () {
	clear && printf '\e[3J' # Or printf "\e[H\e[3J"
	zle && zle .reset-prompt && zle -R # Avoid Screen Element/Prompt Corruption
}
```

```bash
zle -N clearScreenAndScrollback 
bindkey '^L' clearScreenAndScrollback
```

> [!INFO]-
>
> The last commands above perform the following actions:
>
> - `zle -N FUNCTION_NAME` → Function register as a New Widget in _ZSH Line Editor_
> <br>
> - `bindkey KEYBIND FUNCTION_NAME` → A certain keybind runs the passed function

######