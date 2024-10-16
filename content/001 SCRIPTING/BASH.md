---
Primary_category: "[[SHELL SCRIPTING]]"
title: "BASH"
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - "#Bash"
cssclasses:
  - card-list
  - purple-style
---
###### PRIMARY CATEGORY → [[SHELL SCRIPTING]]

#### Components ⟡

- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[Globbing]]
- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[IFS]]
- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[Bash Versioning]]

<br>

#### Shortcuts ⌨

> ***[Reference I](https://www.enlinux.com/bash-keyboard-shortcuts/)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference II](https://www.gnu.org/software/bash/manual/bash.html#Bindable-Readline-Commands)***

**Default Mode** → ***Emacs***

##### *Shorcuts' Meaning*

| **Key** | **Meaning** | |
| --- | --- | --- |
| ***C*** | **`Control`** | **`C-c` → `Control+c`** |
| ***M*** | **`Alt`** | **`M-a` → `Alt+a`** |
| ***S*** | **`Shift`** | **`S-o` → `Shift+o`**  |
| ***Super*** | **`Windows`** | **`Super-s` → `Windows+s`** |
| ***Return*** | **`Enter`** | **`C-S-Return` → `Control+Shift+Enter`** |
| ***-*** | **`+`** | **`C-z` → `Control+z`** |
| ***{a,b,c,d}*** | **`a` `b` `c` `d`** | **`C-{a,b,c,d}` → `C-a` `C-b` `C-c` `C-d`** |

##### *Emacs*

###### Motion

| **Action** | **Shorcut** |
| --- | --- |
| ***Move to the Beginning/End of the Line*** | **`C-a` `C-e`** |
| ***Move Backward/Forward One Character*** | **`C-b` `C-f`** |
| ***Move Backward/Forward One Word*** | **`M-b` `M-f`** |
| ***Move to the Beginning of the line <br> Return to the previous Position*** | **`C-x` `C-x`** |

###### Edition

| **Action** | **Shortcut** |
| --- | --- |
| ***Delete all Characters Before/After the Cursor*** | **`C-u` `C-k`** |
| ***Delete One Word Backward/Forward*** | **`C-w` `M-d`** |
| ***Delete One Character Backward/Forward*** | **`C-h` `C-d`** |
| ***Yank (Paste the last element Deleted or Cut)*** | **`C-y`** |
| ***Convert Word to Uppercase/Lowercase*** | **`M-u` `M-l`** |

##### *History*

| **Action** | **Shortcuts** |
| --- | --- |
| ***Move to Previous/Next Command*** | **`C-p` `C-n`** |
| ***Reverse/Forward Search (History Searching Mode)*** | **`C-r` `C-s`** |
| ***Leave History Searching Mode*** | **`C-g`** |
| ***Reverse Search - Run the Current Command*** | **`C-o`** |

##### *Misc*

| **Action** | **Shortcuts** |
| --- | --- |
| ***Execute Command (Sames as `Return`)*** | **`C-j` <br> `C-m`** |
| ***Clear the Screen*** | **`C-l`** |
| ***Interrupt Current Process (SIGINT)*** | **`C-c`** |
| ***Suspend Current Process (SIGTSTP)*** | **`C-z`** |
| ***Last Command Repetition (History Expansion)*** | **`!!`** |
| ***Paste Previous Command's Last Argument*** | **`M-dot` <br> `!$` <br> `$_`** |
| ***Paste N Command's Last Argument*** | **`!-N:$`** |
| ***Toogle to Command Line Editor*** | **`C-x C-e`** |

---

#### Code Snippet

```bash
$ nvim foo.bash
```

```bash
#!/usr/bin/env bash

bash()
{
        case $( /bin/ps -p "$PPID" -o comm= ) in

                *bash)  return 0
                        ;;

                *)      printf "Shell is not a Bash. Exiting...\n" 1>&2
                        return 1
                        ;;
        esac
}

bash || exit 99
```

```bash
$ chmod u+x foo.bash
$ dash -c './foo.bash'
Shell is not Bash. Exiting...
```
