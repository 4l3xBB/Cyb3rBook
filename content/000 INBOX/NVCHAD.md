---
Primary_category: "[[NEOVIM]]"
title: NVCHAD
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[NEOVIM]]

**NVChad** → ***Fast and Beautiful Neovim Distribution***

It is a ***Blazing Fast*** *Neovim* config which provides a **Solid Defaults** and a **Beautiful UI**

It simply enhances the **Neovim's User Experience**

It has several handy features such as →

- ***Beautiful UI (User Interface)***

![[NVCHAD-20241007205955337.webp|500]]

- ***Theme Switcher***
> ***[Reference](https://nvchad.com/themes/)***

![[NEOVIM_Theme.gif|450]]

- ***Status Line***

![[NVCHAD-20241008165947682.webp|1000]]

**Take a look at the rest of the features [here](https://nvchad.com/docs/features/)**

**[Github Repository](https://github.com/NvChad/NvChad)**

**[NVChad Documentation](https://nvchad.com/)**

#### Installation

> ***[Reference](https://nvchad.com/docs/quickstart/install)***

##### Pre-requisites

There are several required aspects such as →

- ***[[NEOVIM|Neovim]] v0.10 or higher***

> [!CAUTION]-
>
> Note that installing *Neovim* through the default repositories using the `apt` *Package Manager* may result in a *Neovim* older version than the most recent ones listed [here](https://github.com/neovim/neovim/releases)
>
> Therefore, try to install it through the steps mentioned in the link above
>

- ***[[KITTY#Hack Nerd Fonts|Nerd Font]] as the Terminal Font***
<br>
- ***GCC***

```bash
sudo apt install -y -- build-essential 
```

##### *NVChad*

First of all, remove any previous *Neovim Configuration* →

```bash
rm -rf ~/.config/nvim
rm -rf ~/.local/state/nvim
rm -rf ~/.local/share/nvim
```

Then, proceed with the installation as follows →

```bash
git clone https://github.com/NvChad/starter ~/.config/nvim && nvim
```

Once the ***Lazy.nvim's*** Plugins Installation is finished, simply run →

```bash title="nvim"
:MasonInstallAll
```

Remove the `~/.config/nvim/.git`

```bash
rm -rf ~/.config/nvim/.git
```

##### Theme Selection

To select any ***NVChad Theme***, inside *Neovim* → **`Space-t-h`**

##### *Nvchad* Update

```bash title="nvim"
:lazy sync
```

#### Configuration Files

##### *Init.lua*

###### *Relative Numbers*

> ***[Reference](https://samolusola.me/how-to-enable-relative-line-numbers-in-vim-or-neovim)***

```lua title="~/.config/nvim/init.lua"
vim.wo.relativenumber = true
```

###### *$ at each EOL*

To disable `$` char at the end of each line →

```lua title="~/.config/nvim/init.lua"
vim.opt.listchars = "tab:»·,trail:·"
```

###### *# → Autocomment at each Newline*

- ***Problem***

If a line is commented using the `#` character, the next line created is automatically commented in the same way

- ***POC***

![[NVChad_Autocomment.gif|400]]

- ***Workaround***

> ***[Reference](https://www.reddit.com/r/neovim/comments/12gfkmg/question_how_to_disable_auto_comment_in_the_next/)***

To disable autocommenting from a commented line →

```bash title="~/.config/nvim/init.lua"
vim.cmd([[autocmd FileType * set formatoptions-=ro]])
```

##### *Mappings.lua*

###### *; → ~~Command Mode~~*

Disable `;` *Command Mode* mapping in order to allow the `;` char's default behaviour

**`;` → *~~Command Mode~~***

**`;` → *Repeat latest `f`, `t`, `F` or `T` N times (Default Behaviour)***

```lua title="~/.config/nvim/lua/mappings.lua"
map("n", ":", ":", { desc = "CMD enter command mode" })
```

###### *C-h → Left Character Deletion in Insert Mode*

Disable `C-h` mapping in order to allow _Character Deletion_ in *Insert Mode* as with `Backspace`  →

**`C-h` → *~~Cursor N Chars to the left in Insert Mode~~***

**`C-h` → *Left Character Deletion in Insert Mode***

```lua title="~/.config/nvim/lua/mappings.lua"
vim.keymap.set('i', '<C-h>', '<BS>', { noremap = true, silent = true })
```

##### Miscellaneous

###### Disable *Neovim CMP*

> ***About the Plugin → [Reference](https://github.com/hrsh7th/nvim-cmp)***

***NVChad* installs the *NVim CMP Plugin***

To disable it, simply delete the lines below in the indicated file →

> ***See [here](https://pastebin.com/sW0XJ58j) too***

> [!IMPORTANT]- *CMP Plugin Lines*
>
> ```bash title="~/.local/share/nvim/lazy/NvChad/lua/nvchad/plugins/init.lua"
 > -- load luasnips + cmp related in insert mode only
> {
>   "hrsh7th/nvim-cmp",
>   event = "InsertEnter",
>   dependencies = {
>     {
>       -- snippet plugin
>       "L3MON4D3/LuaSnip",
>       dependencies = "rafamadriz/friendly-snippets",
>       opts = { history = true, updateevents = "TextChanged,TextChangedI" },
>       config = function(_, opts)
>         require("luasnip").config.set_config(opts)
>         require "nvchad.configs.luasnip"
>       end,
>     },
>
>     -- autopairing of (){}[] etc
>     {
>       "windwp/nvim-autopairs",
>       opts = {
>         fast_wrap = {},
>         disable_filetype = { "TelescopePrompt", "vim" },
>       },
>       config = function(_, opts)
>         require("nvim-autopairs").setup(opts)
>
>         -- setup cmp for autopairs
>         local cmp_autopairs = require "nvim-autopairs.completion.cmp"
>         require("cmp").event:on("confirm_done", cmp_autopairs.on_confirm_done())
>       end,
>     },
>
>     -- cmp sources plugins
>     {
>       "saadparwaiz1/cmp_luasnip",
>       "hrsh7th/cmp-nvim-lua",
>       "hrsh7th/cmp-nvim-lsp",
>       "hrsh7th/cmp-buffer",
>       "hrsh7th/cmp-path",
>     },
>   },
>   opts = function()
>     return require "nvchad.configs.cmp"
>   end,
> },
> ```
>

![[NVChad_CMP_Plugin.gif|400]]

#### Shortcuts ~ TL;DR

As with [[TMUX]], in **NVChad** all shortcuts are preceded by a certain prefix

***Default Prefix → `Space`***

##### *Terminology*

> ***[Reference](https://neovim.io/doc/user/windows.html#_1.-introduction)***

| **Elements** | |
| --- | --- |
| ***Buffer (Item/File)*** | **File opened in Memory** |
| ***Window*** | **Buffer Visualization** |
| ***Tab*** | **Group or Set of Windows** |

##### *Shorcuts' Meaning*

| **Key** | **Meaning** | |
| --- | --- | --- |
| ***\<leader\>*** | **`Space`** | **`<leader> + a` → `Space+a`** |
| ***C*** | **`Control`** | **`C-c` → `Control+c`** |
| ***M*** | **`Alt`** | **`M-a` → `Alt+a`** |
| ***S*** | **`Shift`** | **`S-o` → `Shift+o`**  |
| ***Super*** | **`Windows`** | **`Super-s` → `Windows+s`** |
| ***Return*** | **`Enter`** | **`C-S-Return` → `Control+Shift+Enter`** |
| ***-*** | **`+`** | **`C-z` → `Control+z`** |
| ***{a,b,c,d}*** | **`a` `b` `c` `d`** | **`C-{a,b,c,d}` → `C-a` `C-b` `C-c` `C-d`** |

##### *NVimTree*

| **Action** | **Shortcut** |
| --- | --- |
| ***Open/Close FileTree Window*** | **`C-n`** |
| ***Toogle to FileTree Window*** | **`<leader> + e`**
| ***Switch between FileTree and Editor⬅️➡️*** | **`C-{h,l}` <br> `C-w-{h,l}`** |
| ***Move Cursor⬆️⬇️*** | **`k` `j`** |
| ***Open/Rename a Directory/File*** | **`Return` `r`** |
| ***Create/Delete a File*** | **`a` `d`** |
| ***Copy/Paste a File*** | **`c` `p`** |
| ***Search a File (Filter Mode)*** | **`f`** |
| ***Select a File (Mark Mode)*** | **`m`** |

##### *Telescope*

> ***[Reference I](https://github.com/nvim-telescope/telescope.nvim?tab=readme-ov-file#default-mappings)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference II](https://github.com/nvim-telescope/telescope.nvim/blob/df534c3042572fb958586facd02841e10186707c/lua/telescope/mappings.lua#L147c)*** 

###### *Finders*

| **Action** | **Shortcut** |
| --- | --- |
| ***Find Files*** | **`<leader> + ff`** |
| ***Find All Files*** | **`<leader> + fa`** |
| ***Find Buffers*** | **`<leader> + fb` <br> `:ls`** |
| ***Find in Current Buffer*** | **`<leader> + fz` <br> `/` `?`** |

###### *Finder Navigation*

| **Action** | **Shortcut** |
| --- | --- |
| ***Confirm Item Selection (Open Item - Current Window)*** | **`Return`** |
| ***Open Item - Horizontal/Vertical Split*** | **`C-x` `C-v`** |
| ***Open Item - New Tab*** | **`C-t`** |
| ***Select Item and Go Forward/Backward*** | **`TAB` `S-TAB`** |
| ***Next/Previous Item (Insert Mode)*** | **`C-n` `C-p`** <br> `Down` `Up`|
| ***Next/Previous Item (Normal Mode)*** | **`j` `k` <br> `Down` `Up`** |
| ***High/Middle/Low Position (Normal Mode)*** | **`H` `M` `L`** |
| ***First/Last Item (Normal Mode)*** | **`gg` `G`** |
| ***Scroll Preview Window⬆️⬇️⬅️➡️*** | **`C-{u,d,f,k}`** |
| ***Scroll Results Window⬅️➡️*** | **`M-{f,k}`** |
| ***Close Telescope (Insert/Normal Mode)*** | **`C-c` `ESC`** |

##### *Buffers*

| **Action** | **Shortcut** |
| --- | --- |
| ***List Buffers*** | **`<leader> + fb` → Telescope <br> `:ls` → OG** |
| ***Next/Previous Buffer*** | **`TAB` `S-TAB` <br> `:bn` `:bp`** |
| ***Open/Close a Buffer*** | **`<leader> + b` `<leader> + x` <br> `:e` `:bd`**
| ***Go to Specific Buffer*** | **`:b <BUFFER_NUMBER>`** |

##### *Windows*

***[Reference](https://neovim.io/doc/user/windows.html#windows)***

| **Action** | **Shortcut** |
| --- | --- |
| ***Vertical Split*** | **`C-w-v`** |
| ***Horizontal Split*** | **`C-w-s`** |
| ***Windows Closing*** | **`C-w-q` <br> `ZQ` `:q`** |
| ***Windows Movement⬆️⬇️⬅️➡️*** | **`C-w-S-{k,j,h,l}`** |
| ***Windows Focus⬆️⬇️⬅️➡️*** | **`C-w-{k,j,h,l}`** |
| ***Windows Resize⬆️⬇️⬅️➡️*** | **`NUMBER-C-w-{plus,dash,<,>}`** |
| ***Windows Resize - Reset*** | **`C-w-=`** |

##### Misc

| **Action** | **Shortcut** |
| --- | --- |
| ***Toogle Line Number*** | **`<leader> + n` <br> `:set nu`** |
| ***Toogle Relative Number*** | **`<leader> + rn` <br> `:set rnu`** |
| ***NVChad Cheatsheet*** | **`<leader> + ch`** |
| ***Clear Highlight*** | **`Esc` <br> `:noh`** |

#### *NVChad Cheatsheet*

![[NVCHAD-20241009205008486.webp|500]]