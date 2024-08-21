---
Primary_category: "[[BASH]]"
title:
  - - GLOBS
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - Bash
cssclasses:
---

###### PRIMARY CATEGORY → [[BASH]]

| **Globs**  |                 **Meaning**                 |  **e.g**   |
| :--------: | :-----------------------------------------: | :--------: |
|   **\***   |      **Match any str, of any lenght**       |  **foo**   |
| **foo\***  |    **Match any str beginning with foo**     | **footer** |
|  **bar?**  | **Match bar followed by one or cero chars** |  **bart**  |
| **[kz]sh** |     **Match sh beginning with k or z**      |  **ksh**   |

**`*` → Matches cero or more chars**
**`?` → Matches one specifically char**
**`[...]` → Matches one char in a specified set**

**Globbing, Globs, Filename Expansion** (All the same) happens after [[Word Splitting]], which means that any file expanded through **Globbing** corresponds to One Word for Bash and not undergoes any **word** or **field separation**.

```bash
for _file in ./*
do
        rm -f -- "$_file"
done
```

Above code will cause `$_file` parameter to be expanded as `./filename`  instead of `filename`.

> **Globbing** is the latest Type of Expasion that happens during **Bash Parsing Process**, aka [[Bash Parser]]

>[!CAUTION]-
> **Note that this is incorrect 🔴**
> ```bash
> for _file in * # Always use ./* instead of *
> do
>       printf "%s\n" "$_file
> done
> ```
> ```bash
> cat * > ../file # Wrong!
> ```
> **Previous Example may lead to a command interpreting a file, which name begins with a dash (-), like an option, given the following reasons:**
> - **A file can contain any char in its name except a Null Byte `\0` and Slash `/`**
> - **Rarely a command is not going to interpret a string beginning with a dash like an option (i.e. `command -opt -- arg`)**
>
> **The following Syntax `--`  is often used on a wide variety of commands to indicate end of option processing.**
> 
>**This can be added to previous code as an additional measure to ensure command to not process File as an option due to leading dash.**
> ```bash
> for _file in ./*
> do
>       cat -- "$_file"
> done
>
> # As result -> cat -- ./filename
> ```
> >[!IMPORTANT]
> > **Be aware `--` syntax should never replace `./` measure, cause not all commands have implemented `--` to indicate end of options, such as `echo` command**

Be aware that, in previous example, if directory has no files, `*` glob pattern will return the pattern instead `./*`, then `rm` will receive as a non-option argument a non-existent file.

To avoid this issue, check file existence before executing any command which has that file as argument →

##### POSIX Compliance

```bash
for _file in ./*
do
        [ -e "$_file" ] || continue # Use [ ] Instead of [[ ]]

        printf "File -> %s\n" "$_file"
done
```

> [!IMPORTANT]-
> Use `test` or `[ ]` [[Shell Builtins]] instead of `[[ ]]` non-standard Bash

##### Non-Standard Shell Extension → `Nullglob`

```bash
foo()
{
        local -- _file= _ns= # Empty Parameters to avoid previous values

        shopt -q nullglob ; _ns=$? # Shell Extension Checking
        (( _ns )) && shopt -s nullglob # Enable if disabled

        for _file in ./* # Use ./* instead of *
        do
                printf "File -> %s\n" "$_file"
        done

        (( _ns )) && shopt -u nullglob # Disable if enabled
}
```

>[!IMPORTANT]-
> Check shell extension first, then, do actions and restore initial value. It's always advisable to keep or restore values to the previous ones
>
> By default, **globbing** does not expand to hidden files, this behaviour can be changed by the following glob patterns →
> ```bash
> for _file in ./* ./.[!.]* # Include hidden files || Exclude . ..
> do
>         printf "File -> %s\n" "$_file"
> done
> ```` 
> Similar results can be achieved using `dotglob` Shell Expansion
>```bash
> foo()
> {
>         local _file= _dg=
>
>         shopt -q dotglob ; _dg=$?
>         (( _dg )) && shopt -s dotglob
>
>         for _file in ./*
>         do
>                 printf "File -> %s\n" "$_file"
>         done
>
>         (( _dg )) && shopt -u dotglob
> }
>```
> Both alternatives have similar performance. I'd go with the **Hidden Files Glob Pattern** for **readibility**

> [!NOTE]- Tip
> On **Arithmetic Operators or Expansions** is not necessary to use following syntax →
> ```bash
> # declare -- _var=1
> : $(( $_var + 1 )) # $(( $_var )) or (( $_var ))
> ```
> Instead use `$(( _var ))` or `(( _var ))` →
> ```bash
> : $(( _var + 1 )) # Inside (( )) or $(( )), use _var instead of $_var
> ```
> **Overview →** Omit `$` char inside `$(( ))` or `(( ))`

This **`Nullglob` Non-Standard Shell Expansion** results way more efficient than the **POSIX** one. This is due to non requiring File Existence Check for each iteration (i.e. `[ -e "$_file ]`)

Note that **Globbing** should only be used on `for` loops. If used as non-option command argument, and expasion results on a too long Filename List, command may not handle correctly all arguments

- `./.[!.]*` or `dotglob` add hidden files to glob expansion list
- `[ -e file ]` or `nullglob` avoid to process non existent files

But, with the above measures applied, the following case may arise →

```bash
$ ( shopt command ./* ./.[!.]*
```

Above command does not 

#### Handle Correctly Pathname and Filename

To handle correctly either files with control chars (`"\n", "\t"...`) or another processing-sensitive char, there are different approches to get to the same place ➡️

- Filename Processing through 

##### 