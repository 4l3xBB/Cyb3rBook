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

![[KITTY-20240921124521349.webp|350]]

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

Download the binary and its related files from the [project's release page](https://github.com/kovidgoyal/kitty/releases). The required one, in this [[SETUP|Environment Setup]] is the _Linux amd64 binary bundle_

Being _root_ →

```bash
$ mkdir /opt/kitty
$ mv /home/al3xbb/Downloads/kitty-0.36.2-x86_64.txz !$
```

```bash
$ cd !$ && tar --extract --verbose --file kitty-0.36.2-x86_64.txz
$ rm -i -- !$
```

> [!INFO]-
>
> That _.TXZ_ file is simply a _TAR_ package compressed via _XZ_. As the above command does, It can be decompressed and unpacked using the `tar`  command
>
> It can also be done in two steps through →
> ```bash
> $ 7z x file.txz && tar --extract --verbose --file file.tar
> ```
> ```bash
> $ xz --decompress --verbose --threads=0 file.txz
> $ (( $? )) || tar --extract --verbose --file file.tar
> ```

That's it

```bash
$ command -V kitty
kitty is /opt/kitty/bin/kitty
```

As for the _kitty's_ Configuration File, It need to be created

Just create it in `~/.config/kitty` and paste this [[KITTY#Configuration File|Configuration File]]'s content into it

```bash
$ nvim ~/.config/kitty/kitty.conf # Or {nano,vi,emacs,vim...}
```

##### Hack Nerd Fonts

See [here](https://www.nerdfonts.com/font-downloads) to download them

To install them, as _root_ →

```bash
$ mv /home/al3xbb/Downloads/Hack.zip /usr/local/share/fonts
$ cd !$
```

```bash
$ 7z x Hack.zip # Or unzip
$ rm -i -- {LICENSE,README}.md !$
```

Once installed, just check in the `kitty.conf` file that the `font_family` parameter is set as follows →

> ***[Reference](https://sw.kovidgoyal.net/kitty/conf/#opt-kitty.font_family)***

```bash
font_family HackNerdFont
```

More info [[KITTY#Font|here]]

---

#### Configuration File

> **_Kitty.conf_ [Configuration File Sample](https://github.com/CodyReichert/dotfiles/blob/master/kitty/.config/kitty/kitty.conf) Fully documented**

**Copy from [here](https://pastebin.com/La9bEV1c) the `kitty`'s Full Customised Configuration File** 

**Configuration File Documentation → [kitty.conf](https://sw.kovidgoyal.net/kitty/conf/)**

---

#### Parameters

##### Font

> ***[Reference](https://sw.kovidgoyal.net/kitty/conf/#fonts)***

Several parameters related to the _kitty terminal's_ font such as the `font_family` or `font_size` can be specificed in the `kitty.conf` configuration file

Take a look at [[KITTY#Hack Nerd Fonts|this]] installation for more information

##### Shell

> ***[Reference](https://sw.kovidgoyal.net/kitty/conf/#opt-kitty.shell)***

This [[SETUP|Enviroment Setup]] make use of a _ZSH_ as the [[SHELL SCRIPTING|Default Shell]]

> [!IMPORTANT]-
>
> The following action requires the _ZSH_ shell to be installed on the system
>
> Just installed it via the _apt repositories_ as follows →
>
> ```bash
> $ apt install -y -- zsh
> ```
>
> Anyway, **[[ZSH#Installation|See this]]** to get it all set up as far as _ZSH_ is concerned

Then, It's necessary to tell it to _Kitty_. This can be done via the `shell` parameter

```bash
shell zsh
```


##### Background Opacity

> ***[Reference](https://sw.kovidgoyal.net/kitty/conf/#opt-kitty.background_opacity)***

The _kitty terminal_'s opacity is set by the `background_opacity` parameter

Its value ranges from 0, **fully transparent,** to 1, **opaque**

```bash
background_opacity 0.55
```

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

> [!INFO]-
>
> All the following block codes are snippets extracted from the [[KITTY#Configuration File|Kitty's Configuration File]]

##### Layouts

**[Types of Layout](https://sw.kovidgoyal.net/kitty/layouts/)**

**All the Layouts are enabled by default**. To switch between them use default shorcut → **`C-S-l`**

In the [[KITTY#Configuration File|configuration file]], the selected ones are the **[Splits Layout](https://sw.kovidgoyal.net/kitty/layouts/#the-splits-layout)** and the **[Stack Layout](https://sw.kovidgoyal.net/kitty/layouts/#the-stack-layout)**

Most of the [[KITTY#Actions and Default Shortcuts|mappeable actions]] used there have been implemented in the `kitty.conf` file, such as the below ones

```bash
enabled_layouts splits,stack
```

As mentioned in the _reference link_ above, the _Splits Layout_ is the most flexible layout

The _Stack Layout_ is simple enabled to allow zooming in and out a specific _Window_, i.e. switching between the two layouts via the `toogle_layout` mappeable action [[KITTY#Switch to Stack Layout (Window Zoom {in,out}) → C-S-z|below]]

###### Switch to Stack Layout (Window Zoom {in,out}) → C-S-z

> ***[Reference](https://sw.kovidgoyal.net/kitty/actions/#action-toggle_layout)***

**Switch to [Stack Layout](https://sw.kovidgoyal.net/kitty/layouts/#the-stack-layout)**

```bash
map ctrl+shift+z toggle_layout stack
```

> [!IMPORTANT]-
> The `toogle_layout` action switches to the layout specified as argument
>
> A handy action is implemented in the code above, which allows _Zoom_ due to the change to the _Stack Layout_
>
> If the action is performed again being in the specificed layout, It switches to the latest layout
>
> Basically it removes the zoom generated by the Stack Layout

###### Split Rotation → C-S-o

> Shown in the [Split Layout Section](https://sw.kovidgoyal.net/kitty/layouts/#the-splits-layout)

```bash
map ctrl+shift+o layout_action rotate
```

Basically rotates the current **Windows Split** (i.e. **Horizonal ↔ Vertical**)

![[KITTY-20240921192904163.webp|450]]

##### Window Management

> Most of the [[KITTY#Actions and Default Shortcuts|mappeable actions]] below are implemented in the [Split Layout Section](https://sw.kovidgoyal.net/kitty/layouts/#the-splits-layout)

###### Window Split

Different types of Split are implemented through the **[launch action](https://sw.kovidgoyal.net/kitty/launch/#the-launch-command)**

- **Vertical Split** → **`C-S-enter`**

```bash
map ctrl+shift+enter launch --location=vsplit --cwd=current
```

- **Horizontal Split** → **`C-S-dash/hyphen`**

```bash
map ctrl+shift+- launch --location=hsplit --cwd=current
```

- **Adaptative Split** → **`C-S-plus`**

```bash
map ctrl+shift++ launch --location=split --cwd=current
```

The `--cwd=current` option of the `launch` action causes the terminal of the newly generated window to retain the previous _CWD_

> _CWD_ → Current Work Directory

> [!IMPORTANT]-
>
> Take into account that the split orientation is being handled by the `--location` option
>
> See [here](https://sw.kovidgoyal.net/kitty/launch/#cmdoption-launch-location)

###### Windows Movement → C-A-{k,h,l,j}

> ***[Reference](https://sw.kovidgoyal.net/kitty/actions/#action-move_window)***

Moves the Windows **in the specificed direction**. It is like a **Windows Exchange**

**Action** → `move_window`

```bash
map ctrl+alt+k move_window up
map ctrl+alt+h move_window left
map ctrl+alt+l move_window right
map ctrl+alt+j move_window down
```

> [!INFO]-
>
> Note that _Vim-like Motions_ are being used here to increase efficiency

![[KITTY-20240921154947816.webp|345]]

###### Windows Focus → C-S-{k,h,l,j}

> ***[Reference](https://sw.kovidgoyal.net/kitty/actions/#action-neighboring_window)***

**Move the Focus in the specified selection** between the adjacent Windows in the current Tab

**Action** → `neighboring_window`

```bash
map ctrl+shift+h neighboring_window left
map ctrl+shift+l neighboring_window right
map ctrl+shift+k neighboring_window up
map ctrl+shift+j neighboring_window down
```

![[KITTY-20240921160959522.webp|350]]

###### Windows Resize → A-S-{k,h,l,j,r}

> ***[Reference](https://sw.kovidgoyal.net/kitty/layouts/#resizing-windows)***

[[KITTY#Actions and Default Shortcuts|Default Shorcut]] to enter the **Windows Resize Mode** → **`C-S-r`**

Thus, the Window is resized using:

- **`w`** → Wider
- **`n`** → Narrower
- **`t`** → Taller
- **`s`** → Shorter
- **`r`** → Reset Windows Size

**To resize a Windows in a customised way**, use the **`resize_window`** action

As mentioned earlier, _Nvim-like Motions_ are used to increase efficiency. Therefore, instead of use the above ones, use these →

```bash
map alt+shift+h resize_window wider 3
map alt+shift+l resize_window narrower 3
map alt+shift+k resize_window taller 3
map alt+shift+j resize_window shorter 3
```

> [!IMPORTANT]-
>
> The above `resize_window` mappeable action receives two arguments
>
> The First one is simply the windows resize motion to be made (i.e. _Narrower, Taller, Shorter, Wider)_
>
> While the second one controls the resizing increment

**To reset all Windows to their initial size,** use the `reset` option in the `resize_window` actions rather than a specific motion

```bash
map alt+shift+r resize_window reset
```

###### Windows Detach → A-S-w

> ***[Reference](https://sw.kovidgoyal.net/kitty/invocation/#detach-window)***

**Completely detach a Window in a new Window OS**

**Actions** → `detach_window`

[See here](https://sw.kovidgoyal.net/kitty/actions/#action-detach_window)

```bash
map alt+shift+w detach_window
```

![[KITTY-20240921213015367.webp|450]]

##### Tab Management

###### Tab Creation → C-S-t

> ***[Reference](https://sw.kovidgoyal.net/kitty/actions/#action-new_tab_with_cwd)***

[[KITTY#Actions and Default Shortcuts|Default shortcut]] to **open a new tab → `C-S-t`**

The above one keybind is overwritten in the [[KITTY#Configuration File|Configuration File]] by the `new_tab_witch_cmd` mappeable action

This allows to create a new tab while keeping the _CWD_

```bash
map ctrl+shift+t new_tab_with_cwd
```

###### Tab Detach → C-A-o

> ***[Reference](https://sw.kovidgoyal.net/kitty/invocation/#detach-window)***

**Completely detach a tab in a new Window OS**

**Action** → `detach_tab`

[See here](https://sw.kovidgoyal.net/kitty/actions/#action-detach_tab)

```bash
map ctrl+alt+o detach_tab
```

![[KITTY-20240921180937965.webp|450]]

##### *Copy/Paste*

> ***[Reference](https://sw.kovidgoyal.net/kitty/actions/#copy-paste)***

###### Traditional Clipboards → C-S-{c,v}

This [[SETUP|environment setup]] make use of several _Copy/Paste_ mappeable actions such as:

- **[`copy_to_clipboard`](https://sw.kovidgoyal.net/kitty/actions/#action-copy_to_clipboard) → `C-S-c`**
- **[`paste_from_clipboard`](https://sw.kovidgoyal.net/kitty/actions/#action-paste_from_clipboard) → `C-S-v`**

The above ones are mapped with the [[KITTY#Actions and Default Shortcuts|default shortcuts]]

###### Additional Clipboards (Buffers) → {f1,f2,f3,f4}

> ***[Reference](https://sw.kovidgoyal.net/kitty/overview/#multiple-copy-paste-buffers)***

The customised ones are the related to the _Buffers_ 

A _Buffer_ simply is a string that stores the value copied into it. Then, that value can be pasted from the _Buffer_ to any location

- **[`copy_to_buffer <buffer_name>`](https://sw.kovidgoyal.net/kitty/actions/#action-copy_to_buffer) → `f1`**
- **[`paste_from_buffer <buffer_name>`](https://sw.kovidgoyal.net/kitty/actions/#action-paste_from_buffer) → `f2`**

In this case, two _Buffers_ are declared in `kitty.conf` and used →

```bash
map f1 copy_to_buffer a # Buffer a
map f2 paste_from_buffer a
map f3 copy_to_buffer b # Buffer b
map f4 paste_from_buffer b
```

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