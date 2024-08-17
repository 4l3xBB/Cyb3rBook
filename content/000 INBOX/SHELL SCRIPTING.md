---
Primary_category: "[[ROOT]]"
title: <% tp.file.title %>
Creation_date: <% tp.file.creation_date() %>
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - index_category
cssclasses:
  - card-list
---
###### PRIMARY CATEGORY → [[ROOT]]

#### SHELLS

- ![](https://img.freepik.com/premium-photo/robot-with-hoodie-that-says-robot-it_937795-644.jpg)
	- [[BASH]]

<br>

```bash
$ export -- EDITOR=nvim
$ C-x C-e
```
```bash
foo(){
        local -a -- _shells=()
        local -- IFS=: _shell=

        while IFS= read -r _shell
        do
                [[ $_shell == \#* ]] && continue

                _shells+=("${_shell##*/}")

        done < /etc/shells

        printf "%s\n" "${_shells[*]}"
}
```
```bash
$ foo
sh:bash:bash:rbash:rbash:sh:dash:dash:tmux:screen
```