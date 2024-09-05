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

- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[Globbing]]
- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[IFS]]
- ![](https://i.pinimg.com/736x/6b/e7/f7/6be7f7b015f91caab4a092f6f11de5a1.jpg)
	- [[Bash Versioning]]

<br>

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
