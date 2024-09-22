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

##### Plugins

Most of the above features can be implemented or enhanced through _ZSH Plugins_ that are sourced from the `.zshrc` [[ZSH#Configuration File|configuration file]]

###### ZSH-autocomplete

> ***[Reference](https://github.com/marlonrichert/zsh-autocomplete)***

It enhances the _ZSH_'s inherent auto-complete capability

The _.zsh_ file related to the plugin is sourced from the `.zshrc` configuration file

```bash
if [[ -f /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh ]]; then
	source /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh
fi
```


###### ZSH-autosuggestions

> ***[Reference](https://github.com/zsh-users/zsh-autosuggestions/tree/master)***

It suggests commands as the user types based on history and the previous completions

![[zsh-autocomppletion.svg|400]]

---

##### Installation

To install this 

---

##### Configuration File

**[.zshrc File](https://paste.mozilla.org/ZNBfNfOc)**