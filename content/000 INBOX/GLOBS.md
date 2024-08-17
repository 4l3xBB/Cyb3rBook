---
Primary_category: "[[BASH]]"
title:
  - - GLOBS
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
  - Bash
cssclasses:
---

###### PRIMARY CATEGORY → [[BASH]]

| **Globs** | **Meaning** | **e.g** |
| :---: | :---: | :---: | 
| **\*** | **Match any str, of any lenght** | **foo** |
| **foo\*** | **Match any str beginning with foo** | **footer** |
| **bar?** | **Match bar followed by one or cero chars** | **bart** |
| **[kz]sh** | **Match sh beginning with k or z** | **ksh** |

**Globbing, Globs, Filename Expansion** (All the same) happens after [[Word Splitting]], which means that any file expanded through **Globbing** corresponds to One Word for Bash.

```bash
for _file in ./*
do
        printf "%s\n" "$_file"
done
```

Above code will cause `$_file` parameter to be expanded as `./filename`  instead of `filename` 

>[!CAUTION]-
> **Note that this is incorrect 🔴**
> ```bash
> for _file in *
> do
>       printf "%s\n" "$_file"
> done
> ```
> **Previous Example may lead to a command interpreting a file, which name begins with a dash (-), like an option given the following reasons:**
> - **A file can contain any char in its name except a Null Byte `\0`**
> - **Rarely a command is not going to interpret a string beginning with a dash like an option**
>
> **The following Syntax `--`  is often used on a wide variety of commands to indicate end of option processing.**
> 
>**This can be added to previous code as an additional measure to ensure command to not process File as option due to leading dash**
> ```bash
> for _file in ./*
> do
>       cat -- "$_file"
> done
>
> # As result -> cat -- ./filename
> ```
> >[!IMPORTANT]
> > **Note that `--` syntax should never replace `./` measure, cause not all commands have implemented `--` to indicate end of options, such as `echo` command**

> It's the latest Type of Expasion that happens during **Bash Parsing Process**, aka [[Bash Parser]]

##### Correct Pathname and File Processing

Note that 