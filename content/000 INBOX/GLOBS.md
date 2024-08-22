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

- **`*` → Matches cero or more chars**
- **`?` → Matches one specifically char**
- **`[...]` → Matches one char in a specified set**

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
>>
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

- **`./`** → Prevents file processing, whose name starts with `-`, as a non-option cmd argument
- **`--`** → Denotes the end of option argumentes. It should be used as a additional measure, not the only one
- **`./.[!.]*` or `dotglob`** → Adds hidden files to glob expansion list
- **`[ -e file ]` or `nullglob`** → Avoids to process non existent files

But, with the above measures applied, the following case may arise →

```bash
$ ( shopt -s nullglob ; command -- ./* ./.[!.]* # Bad, it may cause cmd hang-up
```

Above command expects, at least, one file matching the previous pattern. If not, it'll hang trying to read from the standard input (fd 0)

In cases where globbing does not expand to none pathname and `nullglob` shell expansion is enabled, to prevent above case, just add `/dev/null` as final non-option argument

```bash
$ ( shopt -s nullglob ; command -- ./* ./.[!.]* /dev/null )
```

>[!HINT]- Important
> Be aware of `( )` to group inner commands on a subshell (Bash's Child Process). This allows not to modify current shell's environment
>
> It's important to point out that any assignment or parameter modification inside `()` only affects child's environment
>
> ```bash
> $ ( _foo=bar ; declare -p -- _foo ) # Child Env
> declare -- _foo="bar"
> ```
>```bash
> $ declare -p -- _foo # Shell Parent Process Environment
> -bash: declare: _foo: not found
>```

#### Handle Correctly Pathname and Filename

To handle correctly either files with control chars (`"\n", "\t"...`) or another processing-sensitive char, there are different approches to get to the same place ➡️

- `Globbing` → Does not undergoes word splitting cause happens after it. So there's no need to worry about pathnames which contains control chars or others

Note that **Globbing** is useful to get a list of unhidden files on a specific directory, but, it has some limitations to overcome like empty matches, symlinks, recursive search (`globstar`)

To deal with above aspects, it'd be more feasible to make use of `find` binary

- `Find` → It has several features to perform advanced and recursive searchs on a specific path

##### Globbing

###### POSIX Compliant (Portable) - Non-Hidden Files

```bash
for _file in ./* # ./* instead of *
do
        [ -e "$_file" ] || continue # POSIX → [ ] instead of [[ ]]
        
        cat -- "$_file" # -- → referring to end of cmd options
done
```

###### POSIX Compliant (Portable) - Including Hidden Files

```bash
for _file in ./* ./.[!.]*
do
        [ -e "$_file" ] || continue

        cat -- "$_file"
done
```

>[!NOTE]- Info
> All the following code snippets will include hidden files on globbing expansion. Otherwise, remove `./.[!.]*` from them

###### Non Standard Bash Extension For Loop

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

###### Non Standard Bash Extension Oneliner (One command)

```bash
# nullglob to avoid empty matches || -- as additional protective measure
# /dev/null as non-option arg to avoid command hang-up on stdin

$ ( shopt -s nullglob ; cat -- ./* ./.[!.]* /dev/null )
```

###### Non Standard Bash Extension - Recursive Directory Search

By default, it omits dot files (hidden), prune dot dirs and sorts the file list

```bash
globstar **
```

> [!CAUTION]-
> Recursive Directory Search through globstar extension may cause unwanted infinite loop due to wrong filename handling on filename's links
>
> Feature added on BASH 4.0 (2006). This does not include MACOS devices, whose latest bash version is 3.2.57, due to licensing issues.