---
Primary_category: "[[BASH]]"
title: "[[IFS]]"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - Bash
cssclasses:
---

###### PRIMARY CATEGORY → [[BASH]]

`IFS` as **Internal/Input Field Separator**, basically a set of chars that acts as delimiters when [[Word Splitting]] is performed on an input string or parameter reference

By default, `IFS` parameter is assigned as value a blank, tab char and newline →

```bash
$ printf "%q\n" "$IFS" # or printf "%s\n" "${IFS@Q}" | ANSI-C output Format
$' \t\n'
```

If `IFS` is not set, It behaves such as its default value →

```bash
# IFS default value → $' \t\n'
$ foo="This is a test"
$ printf "=%s= " $foo # Not double quoting
=This= =is= =a= =test= # 4 Words/Fields
```

```bash
$ ( unset -v -- IFS ; printf "=%s= " $foo ) # IFS Unset
=This= =is= =a= =test= # 4 Words
```

> [!IMPORTANT]-
> In previous command, `$foo` parameter expansion is not double quoted to enable [[Word Splitting]] (also [[Globbing]])

There's no [[Word Splitting]] if `IFS` has as value an empty string →

```bash
$ ( IFS='' ; printf "=%s= " $foo ) # IFS with empty string as value
=This is a test= # 1 Word
```

> [!INFO]-
> Note that `( )` is used in above commands to prevent modifying `IFS` globally. Thus, this parameter modification only affects Subshell's environment
>
> The above concepts may better understood as follows →
> ```bash
> foo() # Args counter and splitter
> {
>       printf "Args -> %d |" "$#"
>       printf " =%s=" "$@" ; echo
> }
> ```
> - `IFS` with default value →
> ```bash
> $ foo $foo
> Args -> 4 | =This= =is= =a= =test=
> ```
> - `IFS` unset →
> ```bash
> $ ( unset -v -- IFS ; foo $foo )
> Args -> 4 | =This= =is= =a= =test=
> ```
> - `IFS` with an empty string as value →
> ```bash
> $ ( IFS='' ; foo $foo )
> Args -> 1 | =This is a test=
> ```

#### When IFS is relevant ?

##### `Read` Shell builtin

`read`'s input string processing steps →

- `read` receives a string as input
- That string undergoes [[Word Splitting]] according to `IFS` values
- Every field resulting from prior splitting is assigned to a `read`'s parameter

```bash
$ read -r _A _B _C <<< "Ubuntu Arch Debian" # IFS Default Value
$ printf "=%s= " "$_A" "$_B" "$_C"
=Ubuntu= =Arch= =Debian=
```

```bash
$ IFS=: read -r _A _B _C <<< "Ubuntu:Arch:Debian" # : as IFS's value
$ printf "=%s= " "$_A" "$_B" "$_C"
=Ubuntu= =Arch= =Debian=
```

> [!INFO]-
> `read -r` takes `\` char as literal. Therefore, It prevents escaping any chars and maintains the input string format

**Last variable gets the remaining words** if fields' number is greater than the number of parameters defined by `read`

```bash
$ read -r _A _B <<< "Alex John Doe" # IFS Default Value
$ printf "=%s= " "$_A" "$_B"
=Alex= =John Doe= # _B receives the remaining args
```

if `read -a` is specified, each field resulting from field splitting is assigned into an array as element, according to `IFS` →

```bash
$ read -ra _distros <<< "Ubuntu Arch Debian Fedora"
$ printf "=%s= " "${_distros[@]}"
=Ubuntu= =Arch= =Debian= =Fedora=
```

Any leading and trailing blanks and tabs `\s` are trimmed from input string if `IFS` is set to those values (**`IFS` default value** or **Unset**)

```bash
$ read -r _A <<< $'     \t \t foo \t   ' # IFS → $' \t\n'
$ printf "%q\n" "$_A" # ANSI-C Formatted String
foo
```

> [!INFO]-
> Note that `$' '` ANSI-C Quoting is used in the input string to interpret sequence chars such as `\n` or `\t`
>
> The following way is incorrect because sequence chars are treated as literals →
> ```bash
> $ read -r _A <<< ' \t\t foo \n \t   '
> $ printf "%q\n " "$_A"
> \\t\\t\ foo\ \\n\ \\t
> ```

Likewise, string's inner blanks and `\t` are consolidated into one char →

```bash
$ read -r _A _B <<< $'  foo    bar   '
$ printf "=%s= " "$_A" "$_B"
=foo= =bar= # 
```

While if `IFS` is set to an empty string and therefore field splitting is not performed, the following situations do not arise →

- Leading and Trailing blanks and tabs are not stripped out
<br>
- The inner ones are not consolidated

```bash
$ IFS='' read -r _A <<< $'     \t \t foo \t   ' # IFS → empty string
$ printf "%q\n " "$_A"
$'     \t \t foo \t   ' # N
```

A similar case arise when `IFS` has as its value non-whitespace characters. Again, no initial or final trimming and no inner consolidation is performed on `IFS` chars →

```bash
$ IFS=: read -r _A _B _C _D <<< ":test::foo::::bar::::"
$ printf "=%s= " "$_A" "$_B" "$_C" "$_D"
== =test= == =foo::::bar::::=
```

##### Shell Parameter Expansion

#### Ways to set `IFS`'s values