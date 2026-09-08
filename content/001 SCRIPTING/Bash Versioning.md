---
Primary_category: "[[BASH]]"
title: Bash Versioning
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - Bash
cssclasses:
---


###### PRIMARY CATEGORY → [[BASH]]

| **RESOURCES**| |
| --- | --- |
| **BASH Versioning** | [See here](https://mywiki.wooledge.org/BashFAQ/061)


If a [[SHELL SCRIPTING|Shell Script]] is not _POSIX Compliant_ and a  specific _Non-POSIX_ Shell Functionality is being used (e.g. `local` in [[BASH|Bash]]), It may arises some errors and unintended actions if that script is interpreted by a _POSIX Compliant_ shell like _sh or dash_

The same applies if the script implement some functionality that is specific of the shell for which it is being created

Therefore, It's necessary to perform some checks at the beginning of the script to prevent above problems 

##### Shell Check

> **_POSIX Compliant_**

```bash
foo()
{
        case $( /bin/ps -p "$PPID" -o comm= ) in

                *bash)  return 0
                        ;;

                *)      printf "Shell not allowed. Exiting...\n" 1>&2
                        return 1
                        ;;
        esac
}
```

> [!INFO]-
>
> Above code extracts the executed command which created the Script's Parent Process
>
> If It is not run by a [[BASH|Bash]], returns _false_. Thus, can be chained with →
>
> ```bash
> $ foo || exit 99 # Script exits with a 99 Error code if not a Bash
>```
>
> It's _POSIX Compliant_ since only a `case` statement is used with **Command Substitution** and Standard [[Globbing]]
>
> Note that It's not necessary to quote that expansion as `case` statement's parameter is one of the fews situations which not undergoes [[Word Splitting]] and Globbing

> [!CAUTION]-
>
> Be aware that this check must be performed before any other one, as It checks the shell that should run the script

##### Bash Version Check

It can be done through `BASH_VERSION` escalar parameter and `BASH_VERSINFO` indexed array

- `BASH_VERSION` →

```bash
$ declare -p -- BASH_VERSION
declare -- BASH_VERSION="5.1.16(1)-release"
```

- `BASH_VERSINFO` →

```bash
$ printf "=%s= " "${BASH_VERSINFO[@]}"
=5= =1= =16= =1= =release= =x86_64-pc-linux-gnu=
```

###### vX.0

> **_e.g. Bash v4.0_**

```bash
bar()
{
      (( BASH_VERSINFO[0] < 4 )) && {
              printf "Bash Version < v4.0. Exiting...\n" 1>&2
              return 1
      }
 
      return 0
}
```

> [!INFO]-
>
> It is necessary when some implemented functionality is not present in previous versions
>
> Above command uses **Arithmetic Operation** `(( ))` to perform Bash Version Checking through `BASH_VERSINFO` indexed array

###### vX.X

> **_e.g. Bash v4.4_**

```bash
bar()
{
        (( BASH_VERSINFO[0] < 4 || ( BASH_VERSINFO[0] == 4 && BASH_VERSINFO[1] < 4 ) )) && {

                printf "Bash Version < v4.4. Exiting...\n" 1>&2
                return 1
        }
}
```
