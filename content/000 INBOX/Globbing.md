---
Primary_category: "[[BASH]]"
title: "Globbing"
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
>
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
> > [!IMPORTANT]
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
> # Include files which starts with one or more .
> # Exclude . and .. directories 
> for _file in ./* ./.[!.]* ./..?*
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
> Both alternatives have similar performance. I'd go with the **Hidden Files Glob Pattern** for **readability** and **to not restore dotglob initial status**

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
- **`./.[!.]* ./..?*` or `dotglob`** → Adds hidden files to glob expansion list
- **`[ -e file ]` or `nullglob`** → Avoids to process non existent files

But, with the above measures applied, the following case may arise →

```bash
# Incorrect, It may cause command hang-up
$ ( shopt -s nullglob ; command -- ./* ./.[!.]* ./..?* )
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

Note that **Globbing** is useful to get a list of unhidden files on a specific directory, but, it has some limitations to overcome like empty matches (`nullglob`), symlinks, non-recursive search  (`globstar`) and hidden files (`dotglob`)

To deal with above aspects, it'd be more feasible to make use of `find` binary

- `Find` → It has several features to perform advanced and recursive searchs on a specific path given certain criteria

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
for _file in ./* ./.[!.]* ./..?* # Non-POSIX Compliant → {*,.[!.]*,..?*} 
do
        [ -e "$_file" ] || continue

        cat -- "$_file"
done
```

>[!NOTE]- Info
> All the following code snippets will include hidden files on globbing expansion. Otherwise, remove `./.[!.]* ./..?*` from them

###### Non Standard Bash Extension For Loop

```bash
foo()
{
        local -- _file= _ns= # Empty Parameters to avoid previous values

        shopt -q nullglob ; _ns=$? # Shell Extension Checking
        (( _ns )) && shopt -s nullglob # Enable if disabled

        for _file in ./* ./.[!.]* ./..?* # Use ./ before globs
        do
                printf "File -> %s\n" "$_file"
        done

        (( _ns )) && shopt -u nullglob # Disable if enabled
}
```

###### Non Standard Bash Extension Oneliner (Command Interface)

```bash
# nullglob to avoid errors on empty matches
# -- as additional protective measure (End of command option arguments)
# /dev/null as non-option arg to avoid command hang-up on stdin

$ ( shopt -s nullglob ; cat -- ./* ./.[!.]* ./..?* /dev/null )
```

>[!CAUTION]-
> Be aware that errors may arise if the list of matches is too long and shell command cannot handle that argument number →
>```bash
> $ cat -- ./* ./.[!.]* ./..?* /dev/null # nullglob should be enabled
> -bash: /bin/cat: Argument list too long
>```
>
> Therefore, in robust scripts, globbing should only be used on **for loops**, unless the number of arguments that the command receives as filenames is known →
> ```bash
> for _file in ./* ./.[!.]* ./..?* # Enable nullglob or [ -e "$_file" ]
> do
>       cat -- "$_file"
> done
> ```

###### Non Standard Bash Extension > v4.0 - Recursive Directory Search

```bash
foo()
{
        local -- _file= _gs=
        shopt -q globstar ; _gs=$?

        (( _gs )) && shopt -s globstar

        for _file in ./** # ./** indicates recursively
        do
                printf "File -> %s\n" "$_file"
        done

        (( _gs )) && shopt -u globstar
}
```

>[!CAUTION]-
>
> Note that `globstar` Bash extension was added on *BASH 4.0 (2006)*. This does not include MACOS devices, whose latest Bash version is 3.2.57, due to licensing issues.
>
> To prevent unintended errors, add some Bash version validation on `.bash` scripts →
> ```bash
> bar()
> {
>       (( BASH_VERSINFO[0] < 4 )) && {
>               printf "Bash Version < v4.0. Exiting...\n" 1>&2
>               return 1
>       }
>
>       return 0
> }
> ```
> 
>
> **[Check release date of Bash Features](https://mywiki.wooledge.org/BashFAQ/061)**
>
> If Bash Version validation is implemented, you should add, prior to the above one, a Shell validation to check if `.bash` script is running in a Bash or not →
> ```bash
> case $( /bin/ps -p "$PPID" -o comm= ) in
>
>         *bash)  return 0
>                 ;;
>
>         *)      printf "Current shell is not a Bash. Exiting...\n" 1>&2
>                 return 1
>                 ;;
> esac
> ```
> It could be in the same function or in a different one, but it must be done prior to any other type of validation
>
> Be aware that `case` is used on prior shell checking instead of other for these reasons:
> - `[[ ]]` → **Not POSIX-Compliant**. Restricted to Korn Shell, Bash and Zsh
> - `[ ]` aka `test` → **POSIX-Compliant** but it does not allow **Pattern Matching** using **Globbing**
>
> > [!HINT]
> > 
> > Note that, in `case` statements, it's not necessary to use doble quotes `""` on Command Substitution or Parameter Expansion to prevent [[Word Splitting]] (according to `IFS` value) or Filename Expansion (Globbing)

By default, this recursive shell search behaves this way →

- **Omit hidden files**
- **Prune dot dirs (does not descend into them)**
- **Since Bash v.4.3, [does not follow symlinks](http://git.savannah.gnu.org/cgit/bash.git/tree/CHANGES?id=d2744e05f24d1116ae8dc012e4b07ff219d0d2c1#n16). This prevents infinite loops and duplicated entries**

**Due to latest point, take into account the use of `globstar` on systems with < Bash v4.3**

Therefore, It'd be feasible to perform a recursive search with `globstar`, **including hidden files and avoiding errors on empty matches,** using `dotglob` and `nullglob` Bash extensions, respectively →

```bash
foo()
{
        # globstar, {dot,null}glob Status Checking

        for _file in ./** # With above Bash extensions enabled
        do
                # [ -e "$_file" ] if nullglob not enabled ( less efficient )
                command -- "$_file"
        done

        # Return Bash extensions to their initial state
}
```

> [!TIP]-
> To avoid having to reset the bash extensions' values above, all commands can be executed on a subhell `( )` instead of grouping them `{ }` →
>
> ```bash
> foo()
> (
>       shopt -s globstar dotglob nullglob
>
>       for _file in ./**
>       do
>               printf "File -> %s\n" "$_file"
>       done
> )
> ```
> Note that both assigments and parameter modifications will only be applied in the subshell's environment, not in Shell parent one

However, `find` offers a more reliable way to do this, being able to deploy several advanced filters that globbing could not

Remember that `globstar` recursive search way is **non-standard** and It offers **poor control over the recursion**

Not to mention that, as long as the number of files handled increase, is way more feasible to make use of `find` in terms of performance

```bash
# Note subshell use in foo to not modify Parent Shell Env attributes
# Being foo function () →
foo ()
(
    shopt -s globstar dotglob nullglob
    for _file in ./**
    do
        printf "File -> %s\n" "$_file"
    done
)
```

```bash
$ export -f -- foo # Bash's Child Process inherit foo func in its env
```

```bash
# Benchmark between foo and 'find .' command over 50k Files
$ hyperfine --shell bash foo --warmup 3 --min-runs 500 -i 'find .'
```

> [!NOTE]- Command Output
> ```bash
> Summary
>   'find .' ran
>     5.39 ± 0.69 times faster than 'foo'
> ```
> | Command | Mean [ms] | Min [ms] | Max [ms] | Relative |
> |:---|---:|---:|---:|---:|
> | `foo` | 173.6 ± 21.8 | 156.6 | 329.1 | 5.39 ± 0.69 |
> | `find .` | 32.2 ± 1.0 | 30.3 | 35.6 | 1.00 |

In the above benchmark, `find` shows a better performance over `globstar - nullglob - dotglob` recursive search on more than 50k files

Note that performance breach between both will increase as long as the number of files increases, so it seems more feasible to make use of `find` for better yield and robustness

`Find` is nearly in every UNIX system while `globstar` is Non-POSIX Compliant and > Bash v4.0 (Infinite Loops) - v4.3

##### FIND

It's an external Binary, not a shell Builting

```bash
$ command -V find
find is /usr/bin/find
```

Take into account that any binary's execution leads to the creation of a Shell's child process through system calls like `fork` or similar (`vfork`, `clone`)

This child process's env is a clone of its parent's env. Then, the binary is executed inside that child process through `execve` syscall

That does not happen on `globstar` way due to none binary is required, only shell functionalities (builtins, keywords, expansions...) are used during it

But since `find` is only executed once or a few times on nearly any context, this does not imply a perceptible yield reduction

>[!IMPORTANT]-
> Subshells or Child Process can be generated due to specific tasks:
>  - **Binary's execution**
>  - **Background Jobs** → `command &`
>  - **Simple or Group command isolation** → `( command )`
> 	 - **Command Substitution** → `$( command )`
> 	 - **Process Substitution** → `<( command ) >( command )`
>  - **Shells's `-c` option argument** → `{sh,bash,ksh} -c 'command'`
> - **Pipes** → `command | command`
> 
> On `command | command`, every command executes on a different subshell
> 
> Any assignment or parameter modification inside above subshells does not take affect on shell parent's env
> 
> As mentioned earlier, as long as subshell generation is done correctly and not in abused way such as in loops, It will not negatively affect performance
>
> ```bash
> foo()
> {
>         for i in {1..5000} # 5k iterations
>         do
>                 : # Shell builtin - Returns true and expand any arg
>         done
> }
> ```
>```bash
> bar()
> {
>         for i in {1..5000}
>         do
>                 (:) # : builtin executed on subshell / iteration
>         done
> }
>```
>
> ```bash
> $ export -f -- foo bar
> ``` 
> ```bash
> $ hyperfine --shell bash foo bar --warmup 3 --min-runs 100 -i
>```
> >[!NOTE] Command Output
> > ```bash
> > Summary
> >   'foo' ran
> >   296.33 ± 26.24 times faster than 'bar'
> > ```
> > | Command | Mean [ms] | Min [ms] | Max [ms] | Relative |
> > |:---|---:|---:|---:|---:|
> > | `foo` | 2.6 ± 0.2 | 2.4 | 5.6 | 1.00 |
> > | `bar` | 781.7 ± 28.0 | 737.1 | 932.2 | 296.33 ± 26.24 |
> As can be seen above, performance is reduced very significantly when subshells are created inside any loop context
>
> It's important to know in which situations shell functionality can be used instead of depend on external binaries. This will allow to not decrease script's yield notoriously
>
> Like the following one, users tend to use external binaries to implement functionalities that can be obtained through shell builtins →
>
> ```bash
> $ export -- _pathname="/etc/ssh/sshd_config"
> ```
>```bash
> # Command 1 → printf "%s\n" "${_pathname##*/}" → Shell Functionality
> # Command 2 → basename "$_pathname" → External Binary
>
> $ hyperfine --shell bash --warmup 3 --min-runs 5000 'printf "%s\n" "${_pathname##*/}"' 'basename "$_pathname"'
>```
> > [!NOTE] Command Output
> > ```bash
> > Summary
> >   'printf "%s\n" "${_pathname##*/}"' ran
> >     5.84 ± 165.51 times faster than 'basename "$_pathname"'
> > ```
> > | Command | Mean [ms] | Min [ms] | Max [ms] | Relative |
> > |:---|---:|---:|---:|---:|
> > | `printf "%s\n" "${_pathname##*/}"` | 0.0 ± 0.0 | 0.0 | 2.0 | 1.00 |
> > | `basename "$_pathname"` | 0.0 ± 0.1 | 0.0 | 2.8 | 5.84 ± 165.51 |

Default behaviour →

- **Does not omit hidden files**. To omit them →

```bash
$ find . -name '.' -o -path '*/.*' -prune -o ... # Omit hidden Files and . dir
```

- **Applies a recursive search.** To limit it to current directory (Non-recursive) →

```bash
$ find . -maxdepth 1 -name '.' -o ... # Non-recursive and Omit . dir
```

- **Is passed a directory and all matches begins with that directory in the command's output**. Therefore, errors probably don't arise when filename starts with a `-`, unlike globbing
<br>
- **`find -exec` option allows to directly run commands with any matched file**. Although, if the pathnames are needed back into the shell, several alternatives arise →

Before proceed to show them, take into account the following stuff →

- **Filenames can contain any char except *Zero Byte (aka Null Byte `\0`*)  and *slash `/`*** 

Hence, since filenames can contain special chars like newlines `\n`, reading files line-by-line will fail → `read`

> [!IMPORTANT]- Why read fails if filename contains newline
> ```bash
> $ touch test$'\n'file # File created with embedded \n on empty dir
> ```
> ```bash
> # Empty IFS to avoid trimming {lead,trail}ing blanks
> while IFS= read -r _file
> do
>         printf "File -> %s\n" "$_file" # One file per line
>
> done < <( find . -name '.' -o -print ) # Proc substitution as loop stdin
> ```
> > [!NOTE] Command Output
> > ```bash
> > File -> ./test
> > File -> file
> > ```
>
> One line is expected as output since only there's only one file in current directory
>
> But that embedded `\n` in its name causes that `read` splits input string into two words due to word splitting
> > [!NOTE] Info
>>
> > Note that `read` reads until a newline `\n` char. This default behaviour can be changed through `-d` option, which changes the delimiter
> >
> > `read` processes input string in the following way →
> > - Process input string until `\n` character
> > - [[Word Splitting]] applies to the processed string according to `IFS`'s value
> > - Resulting string is stored within `read`'s declared parameter
> >
> > If `read`'s input string is divided into words due to Word Splitting, three cases may arise →
> > - **`read -a` option** → Each *field/word* resulting from splitting is stored into an array as one element
>> ```bash
>> $ read -ra _names <<< "Alex John Albert" # Default IFS → $' \t\n'
>> $ printf "|%s| " "${_names[@]}"
>> |Alex| |John| |Albert|
>> ```
> > - **Multiple parameters** → If more than one parameter is declared as a `read` argument, each parameter receives as value one word of the input string
>> ```bash
>> $ read -r _A _B _C <<< "bash zsh ksh" # <<< → Herestring
>> $ printf '|%s| ' "$_A" "$_B" "$_C"
>> |bash| |zsh| |ksh|
>> ```
>>
>> If input string's field number is greater than the number of parameters declared, `read`'s last argument receives the remainder of the string
>> ```bash
>> $ read -r _A _B <<< "Ubuntu Arch Debian Fedora" # Fields > Params
>> $ printf "|%s| " "$_A" "$_B"
>> |Ubuntu| |Arch Debian Fedora| # _A → 1 field ; _B → Remainings
>> ```
>> No Splitting or delimiter consolidation are performed on the remaining part
>>
>> - **One parameter and `IFS` with default value or unset** → Note that, due to [[Word Splitting]] again, **leading and trailing** blanks and `\t` in the input string **are trimmed** and the **internal ones are consolidated**
>> ```bash
>> $ read -r _var <<< $'    \t\t bar  ' # IFS → $' \t\n'
>> $ printf "%q\n" "$_var" # %q → ANSI-C Formatted String
>> bar # Leading and trailing IFS chars stripped out
>> ```
>> While if `IFS ` value is empty, input string does not undergoe word splitting. Thus, the above leading and trailing chars are not removed and the inner ones are not consolidated→
>> ```bash
>> $ IFS= read -r _var <<< $'    \t\t bar  '
>> $ printf "%q\n" "$_var"
>> $'    \t\t bar  '
>> ```
>> Prior behaviour does not happen if `IFS` contains non-whitespace chars in its value (`$' \t\n'`) 
>>
>> No initial or final trimming and no inner consolidation is performed on `IFS` chars →
>> ```bash
>> $ IFS=: read -r _A _B _C _D <<< ":test::foo::::bar::::"
>> $ printf "=%s= " "$_A" "$_B" "$_C" "$_D"
>> == =test= == =foo::::bar::::=
>> ```

Some alternatives such as the following will also fail due to above problem →

```bash
for _file in $( find . -name '.' -o -print )
do
        printf "File -> %s\n" "$_file"
done
```

>[!CAUTION] Wrong
> Note that if any pathname contains a space, `\n` or  `\t`, its name will be split into more than one word. Likewise, if pathname contains any globbing chars (`*`, `?`), shell will try to expand it to any matched file
>
> Furthermore, `$( )` expansion chop off any trailing newline
> 
> Previous situation can be improved to handle correctly filenames with embedded spaces and tabs. It can be also manage globbing expansion → 
>
> ```bash
> (
>         IFS=$'\n' # Word Splitting not applied on \t or blanks
>         set -f # Filename|Globbing expansion disabled
>
>         for _file in $( find . -name '.' -o -print )
>         do
>                 printf "File -> %s\n" "$_file"
>         done
> )
> ```
> > [!INFO]
> > There're several standard ways to assign values to `IFS` parameter
>> - [_Non-POSIX Compliant_](http://austingroupbugs.net/view.php?id=249). But It's quite expanded →
>> ```bash
>> $ IFS=$' \t\n' # ANSI-C Quoting
>> ``` 
>> - _POSIX Compliant_. Command substitution and Parameter Expansion →
>> ```bash
>> $ IFS=$( printf '\nX' ) ; IFS=${IFS%X} # CORRECT
>> ```
>> - _POSIX Compliant_. Command Substitution →
>>```bash
>> $ eval "$( printf 'IFS="\n"' )" # CORRECT
>>```
>>
>> Be aware that it cannot be done through the following way since **command substitution** trims trailing `\n` →
>> ```bash
>> $ IFS=$( printf '\n' ) # WRONG!!
>> ```
>>
>> To prevent that above expansion trims trailing newline, a character must be appended after `\n`
>> 
>> After that, the character is trimmed using `${var%string}` parameter expansion syntax
>>
>> Note that double quotes are not used on escalar assignments's right hand such as the prior ones and in the following situations →
>> ```bash
>> $ foo=$HOSTNAME
>> $ foo=$( hostname )
>> $ foo=${HOSTNAME:-$(hostname)}
>> ```
>> ```bash
>> case $foo in ... # Parameter in case statement
>> ```
>>```bash
>> $ [[ -n $foo ]] && ... # [[ Shell Keyword
>>```
>> [More cases 😊](https://unix.stackexchange.com/questions/68694/when-is-double-quoting-necessary/68748#68748)
>>
>> Above situations do not undergoe [[Word Splitting]] and Globbing. They're like double quote contexts, therefore, It's not necessary to use double quotes
>>
>> However, remember that when in doubt, **always quote parameters references !!**
>
> Be aware that above `for loop` will break up pathnames that contain newlines `\n` and all parameter creation and modification is not saved due to `( )` subshell
>
> To prevent subshell's above problem, `local` or `declare` shell builtins can be used to restrict to a local scope the shell parameters modification →
> ```shell
> foo()
> {
>         local -- IFS=$'\n' \ # IFS Local Scope modification
> 		         _oldSetOptions=$( set +o ) # Save Shell options
>         set -f # Disable Globbing
>
>         for _file in $( find . -name '.' -o -print )
>         do
>                 printf "File -> %s\n" "$_file"
>         done
>
>         eval "$_oldSetOptions" # Restore Shell options
> }
> ```
> > [!INFO]
> > To restore shell options to their previous values, It cannot be like this →
> > ```bash
> > foo()
> > {
> >     set -f
> >     : # Stuff here !
> >     set +f
> > }
> > ```
>> In the above function globbing is enabled once all stuff is done, but what if globbing was already disabled prior to `set -f `
>>
>> To prevent that misleading situation, store shell options in a parameter through **command substitution**
>>
>> With all required actions done, just restore shell options from prior parameter through `eval` →
>> ```shell
> > foo()
> > {
> >         local -- _oldSetOptions=$( set +o ) # Store Shell Opts
> >
> >         set -f
> >         : # Stuff here !
> >         eval "$_oldSetOptions" # Restore Shell Opts
> >
> > }
>> ```
>
> Note that above way _Non-POSIX Compliant_ due to `local`, same applies with `declare` and `typeset`
>
> One _POSIX Compliant_ way to store `IFS`'s prior value →
> ```bash
> foo()
> {
>         _file= _savedIFS= _oldSetOpts=$( set +o ) # Store Shell Opts
>
>         [ -n "${IFS+set}" ] && _savedIFS=$IFS # Save IFS Initial value
>
>         set -f # Disable filename expansion
>         eval "$( printf 'IFS="\n"' )" # Assign \n to IFS as value
>
>         for _file in $( find . -name '.' -o -print )
>         do
>                 printf "File -> %s\n" "$_file"
>         done
>
>         eval "$_oldSetOpts" # Restore Shell Opts
>         unset IFS 
>
>		# Restore IFS value if It was not unset
>         [ -n "${_savedIFS+set}" ] && { IFS=$_savedIFS ; unset _savedIFS; }
> }
> ```