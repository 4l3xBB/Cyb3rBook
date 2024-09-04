---
Primary_category: "[[BASH]]"
title: "IFS"
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
>
> In previous command, `$foo` parameter expansion is not double quoted to enable [[Word Splitting]] (also [[Globbing]])

There's no [[Word Splitting]] if `IFS` has as value an empty string →

```bash
$ ( IFS='' ; printf "=%s= " $foo ) # IFS with empty string as value
=This is a test= # 1 Word
```

> [!INFO]-
>
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

**Last variable gets the remaining words** if resulting fields' number is greater than the number of parameters defined by `read`

```bash
$ read -r _A _B <<< "Alex John Doe" # IFS Default Value
$ printf "=%s= " "$_A" "$_B"
=Alex= =John Doe= # _B receives the remaining args
```

> [!IMPORTANT]-
>
> No inner consolidation is performed in the remaining part of the string if number of fields is greater than than number of `read`'s variables
> ```bash
> $ read -r foo bar <<< "  bash  dash   ksh   zsh   "
> $ printf "=%s= " "$foo" "$bar"
> =bash= =dash   ksh   zsh=
> ```
> Take into account that leading and trimming whitespaces continues to be performed even in the remaining part

if `read -a` is specified, each field resulting from field splitting is assigned into an array as element, according to `IFS` →

```bash
$ read -ra _distros <<< "Ubuntu Arch Debian Fedora"
$ printf "=%s= " "${_distros[@]}"
=Ubuntu= =Arch= =Debian= =Fedora=
```

Any leading and trailing blanks and tabs `\s` are trimmed from input string if `IFS` is set to those values (e.g. **`IFS` default value** or **Unset**)

```bash
$ read -r _A <<< $'     \t \t foo \t   ' # IFS → $' \t\n'
$ printf "%q\n" "$_A" # ANSI-C Formatted String
foo
```

> [!INFO]-
>
> Note that `$' '` ANSI-C Quoting is used in the input string to interpret sequence chars such as `\n` or `\t`
>
> The following way is incorrect because sequence chars are treated as literals due to single quotes `' '` →
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

> [!IMPORTANT]-
>
> Above situation changes if `IFS`' value is a mixture of whitespace and non-whitespace chars `IFS=' :'` →
> ```bash
> $ IFS=' :' read -r foo bar doe <<< $'  zsh:  ksh :  bash    '
> $ printf "=%s= " "$foo" "$bar" "$doe"
> =zsh= =ksh= =bash=
> ```
>
> Any non-whitespace char (Only one) plus all adjacent whitespaces ones act as a single delimiter, they get consolidated
>
> > [!INFO]
>>
> > Trimming at the beginning and the end of input string continues as long as whitespace chars are in `IFS`'s value
>
> If there is more than one consecutive non-whitespace char and It's inside `IFS`' values, those ones are treated as single delimiters (no consolidation) and one empty _Word/Field_ is created →
> ```bash
> $ IFS=' :'
> $ foo="  ::This is:: a test::"
> $ printf "=%s= " $foo
> == == =This= =is= == =a= =test= == # Two adjacent : create one Word
> ``` 
> > [!CAUTION]
>> 
> > Be aware that `IFS=' :'` modifies `IFS`'s value globally, which is often not what is desired, below there're several ways to handle this situation correctly

##### Unquoted Shell Expansion

When **Shell Expansion** is performed in shell's parsing, all unquoted expansion undergoes [[Word Splitting]] and [[Globbing]]

> [!INFO]-
>
> There're different types of shell expansion and they all occur in a set order after command has been split into tokens
> 
> Ordered from the first to the last:
>
> - **Brace Expansion**. _Non-POSIX Compliant_ →
> ```bash
> $ echo a{b,c,d}e # {1..5} → 1 2 3 4 5
> abe ace ade
> ```
> - **Tilde Expansion** → [Check GNU Bash Manual](https://www.gnu.org/software/bash/manual/bash.html#Tilde-Expansion)
> ```bash
> $ echo ~ # → $HOME
> $ echo ~+  # → $PWD
> $ echo ~-  # → $OLDPWD
> ```
> Note that one `$` char preceding a string introduce differente types of expansion like the following ones 
>
> - **Parameter Expansion** →
> ```bash
> $ foo="test"
> $ printf "%s\n" "$foo"
> test
> ```
> Parameter expansion may be enclosed between brackets to perform any specific action such as string manipulation of the parameters' value
> ```bash
> $ echo "${foo%t}" # → tes | Suffix trimming
> $ echo "${foo#t}" # → est | Prefix trimming
> $ echo "${foo/t/f}" # → fest | Substitution
> $ echo "${foo:2}" # → st | Substring expansion
> ```
>
> There're [much more](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion), nearly all related to string manipulation
> 
> - **Command Substitution →** Command execution inside a subshell `( )`. Any trailing newlines in command output are trimmed
>
>
> ```bash
> $ echo $( command ) # Or `command`
> ```
>
> > [!CAUTION]
> >
> > Be aware that embedded newlines `\n` in output command may be deleted due to [[Word Splitting]]
>
> - **Arithmetic Expansion →**
> ```bash
> $ echo (( 1 + 2 ))
> ```
>
> - **Process Substitution**. _Non-POSIX Compliant_. Command is executed in a subshell `()` asynchronously. Its input or output appears as filename
> ```shell
> $ echo <( command ) # Or >( command )
> ```
> Once any of above expansions occur, if not quoted `' ', " ", $' '`, both [[Word Splitting]] and [[Globbing]] apply to them
>
> Note that Globbing (_aka Filename Expansion_) occurs after Word Splitting.
>
> Therefore, when Globbing is used to expand files, each matched files is taken as a single _Word/Field_ by the shell due to no Word splitting after that
>
> With all above actions done, **Parameter Assignment**, **Redirections** and **Quote Removal** are performed prior to the command execution

As mentioned above, word splitting splits the resulting expansion string into different fields or words through `IFS` parameter's value

Therefore, following case undergoes word splitting →

```bash
$ foo="God's time is perfect"
$ printf "|%s| " $foo # Variable reference not quoted
|God's| |time| |is| |perfect|
```

While this one does not →

```bash
$ printf "|%s| " "$foo" # Variable reference quoted
|God's time is perfect|
```

Same happens with other Shell Expansions like **Command Substitution**

Be aware that any variable reference inside command substitution **must be quoted even if command subtitution itself is quoted** →

```bash
$ file="Lord of the Rings.mp3"
$ echo "$( rm -f -- $file )" # Wrong!
$ echo "$( rm -f -- "$file" )" # Correct! Inner variable reference quoted
```

> [!INFO]-
>
> In **Command Substitution**, trailing newlines are stripped out in subshell's output
> 
> Likewise, output's embedded newlines may be removed due to [[Word Splitting]] if `IFS` is unset, default or with a `\n` explicitly assigned
>
> A situation related to the above actions that may arise repeteadly is the following:
>
> Having a file named `foo` with this content →
> ```bash
> $ batcat --show-all -- ./foo # Show non-printable chars (\n, \t, blanks)
> ␊
> ····Ubuntu·├──┤Debian├──┤␊
> ␊
> *␊
> ├──┤Arch···Fedora··␊
> ··CentOS␊
> ␊
> Parrot␊
> ├──┤␊
> ····␊
> ␊
> ```
> > [!IMPORTANT]
> >
> > Note the following notation with `batcat --show-all` :
> > - `·` → _Blank/Whitespace_
> > - `├──┤` → Tab `\t`
> > - `␊` → Newline `\n`
>
> Above file's content has several blanks, tabs and newlines, both internal and leading/trailing
> 
> If `foo` file has to be processed line-by-line, below command is incorrect due to several reasons (each _field/word_ is stored in `$_line` per iteration) →
> ```bash
> for _line in $( < ./foo )
> do
>         printf "%s\n" "$_line"
> done
> ```
> ```bash
> # Command Output
> Ubuntu
> Debian
> foo
> Arch
> Fedora
> CentOS
> Parrot
> ```
> Above way uses unquoted command substitution to process file's lines. Thus, that expansion undergoes [[Word Splitting]] and [[Globbing]]
>
> Leading `\n` and _leading/trailing/inner_ blanks and `\t` are treated as delimiters due to `IFS` default value
>
> Due to Word splitting applied on command substitution output, **the inner ones are consolidated** and **the _leading/trailing_ are chopped off**
>
> Same applies with inner newlines, they're stripped due to word splitting
>
> As mentioned earlier, trailing newlines are trimmed in command substitution output too
>
> Note that there's one line (the `foo` one) that has been generated through Globbing due to `*` char. It expands to all files in the current directory
> 
> Above situation can be improved as follows →
> ```bash
> ( 
>        IFS=$'\n'
>        set -f
>
>        for _line in $( < ./foo )
>        do
> 		      printf "Line -> %s\n" "$_line"
>        done
> )
>```
> > [!IMPORTANT]
> >
> > Subshell used to prevent Global modifications like the `IFS` and [[Globbing]] ones
>
> `IFS` parameter is set to `\n`. Therefore, [[Word Splitting]] only acts when a newline char is found
> 
> Filename expansion is disabled due to `set -f`, which means that when a globbing char is found (`*`, `?`, `[ ]`), It does not expand to any file in CWD
>
> But above way continues to be incorrect →
> - Empty lines (_leading/trailing/inner_ `\n`) are removed due to Word Splitting as `IFS` value is `\n`
>
> Remember that adjacent inner whitespace chars are consolidated into a single delimiter. Thus, they do not create any additional words
>
> - Command substitution keeps trimming trailing newlines. In fact, that trimming is performed before the Word Splitting one since command substitution occurs before the split
>
> Command Substitution expansion cannot be quoted as its output would be processed once, no line-by-line (i.e. it'd be taken as a single word)
>
> Likewise, an empty string cannot be assigned to `IFS` parameter since no Word splitting would act and, therefore, the same as above would occur (all output as one single word)
>
> To the above problems, It can be added that all is executed into a subshell `( )`. Hence, any parameter modification or creation is not reflected in the Parent Shell's env
>
> This can be modifiy as follows →
> ```bash
> foo()
> {
>         local -- _line \ # Or declare | typeset
>                  IFS=$'\n' _oldSetOptions=$( set +o ) # Store prior Opts
>         set -f # Disable globbing
>
>         for _line in $( < ./foo )
>         do
>                 printf "%s\n" "$_line"
>         done
>
>         eval "$_oldSetOptions" # Restore initial Opts
> }
> ```
> With above code, It's no longer necessary to use a subshell `( )`
> - **`IFS` modification scope is limited locally** to the function due to `local` shell builtin
> <br>
> - **Initial Shell Options** like the globbing one are stored in a parameter. Once all stuff is done, they're restored to their prior values through `eval`
>
> > [!IMPORTANT]
> >
> > Be aware that  above code is _non-POSIX-Compliant_ due to following reasons →
>> - `local`, `declare` and `typeset`. They are supported by many shell but, strictly, not _POSIX_
>> <br>
>> - ANSI-C Quoting to assing value to `IFS` . Bash extension, not _POSIX_
>> <br>
>> - `$( < file )` . Bash extension, not _POSIX_
>> 
>> A _POSIX-Compliant_ code would be this one →
>> ```bash
> > bar()
> > {
> >         _line= _savedIFS= _oldSetOptions=$( set +o ) # Store Opts
> >
> >         [ -n "${IFS+set}" ] && _savedIFS=$IFS # Store Initial IFS
> >
> >         set -f ; eval "$( printf 'IFS="\n"' )" # Instead of $'\n'
> >
> >         for _line in $( cat ./foo ) # instead of $(< file)
> >         do
> >                 printf "%s\n" "$_line"
> >         done
> >
> >         unset -v -- IFS # IFS Cleanup
> >
> >         [ -n "${_savedIFS+set}" ] && {
> >
> >                  IFS=$_savedIFS ; unset _savedIFS ; } # Restore IFS
> >
> >         eval "$_oldSetOptions" # Restore Shell Options
> > }
>> ``` 
>> Simple parameter assignments are used instead of `local``cat` command in **command subtitution** rather than `$(< file)`
>>
>> That `IFS` assignment is treated [[#Ways to set `IFS`'s values|here]]
>
> Although, behaviour remains the same, i.e. empty lines are still removed by Word splitting and Command Substitution, as in the last examples above
> 
> Having seen all incorrect examples, this is the correct way to handle and process line-by-line an input (e.g. A file)  →
> ```bash
> while IFS= read -r _line # Empty IFS
> do
>         printf "%s\n" "$_line" # Always quote variable references!
>
> done < ./foo # File passed directly as input
> ```
> Note that no expansion is performed as input unlike above examples
> 
> `File` is passed directly as `read`'s input
> 
> By default, `read` processes all input line-by-line (until `\n`)
>
> In this case, input string does not undergoes [[Word Splitting]] since `IFS` is empty. Therefore, no leading and trailing whitespaces chars are trimmed
>
> It's true that previous code blocks stdin's `read while loop`. To keep stdin free and be able to read from it →
> ```bash
> while IFS read -r _line <&4 # Read stdin (FD0) is taken from FD4
> do
>         printf "%s\n" "$_line"
>
> done 4< ./foo # FD4 assigned to ./foo file opened in read mode
> ```
> What is happening above is that `./foo` file is being opened in read mode. That open resource is assigned to file descriptor 4
>
> That file descriptor only refers to `./foo` file in the `while loop` context. Thus, `while loop`'s stdin remains free to be used

##### $* Quoted Form

When positional parameters are expanded through **Parameter Expansion**, if expansion is not quoted, [[Word Splitting]] applies on the resulting string according to `IFS` values

To expand all positional parameters, It can be done using `$*` or `$@` expansions and their quoted forms, `"$*"` and `"$@"`, respectively

Let's create a function that prints the Args number and each of them to better appreciate the following behavior →

```bash
foo ()
{
    printf "Args -> %d |" "$#" # Number of arguments
    printf " =%s=" "$@" ; echo # Each argument
}
```

```bash
$ set -- "Ubuntu Focal" "Linux Mint" "Debian Bookworm" # Set Positional Args
```

- `$*` unquoted expansion undergoes word splitting. Therefore, that action is applied for each positional parameter

```bash
$ foo $*
Args -> 6 | =Ubuntu= =Focal= =Linux= =Mint= =Debian= =Bookworm=
```

- Likewise, `$@` unquoted expansion undergoes word splitting. As mentioned above repeteadly, not being quoted means that **split-glob** is performed on that expansion

```bash
$ foo $@
Args -> 6 | =Ubuntu= =Focal= =Linux= =Mint= =Debian= =Bookworm=
```

If above expansions are quoted:

- `"$*"` → Expands all positional parameters as a single string where each parameter is separated by the first `IFS` value

```bash
$ export -f -- foo
$ ( IFS=: ; foo "$*" )
Args -> 1 | =Ubuntu Focal:Linux Mint:Debian Bookworm=
```

- `"$@"` → Expands all positional parameters, but, unlike the above one, each positional parameter is treated as a single quoted word

```bash
$ foo "$@"
Args -> 3 | =Ubuntu Focal= =Linux Mint= =Debian Bookworm=
```

That is, `"$@"` is the same as `"$1" "$2" "$3" ...`

Same occurs with array expansion to extract all array elements →

```bash
$ declare -a -- _names=( "John Doe" "Richard Stallman" )
```

```bash
$ foo ${_names[*]} # Same as $*
Args -> 4 | =John= =Doe= =Richard= =Stallman=
$ foo ${_names[@]} # Sames as $@
Args -> 4 | =John= =Doe= =Richard= =Stallman=
```

```bash
$ ( IFS=: ; foo "${_names[*]}" ) # Sames as "$*"
Args -> 1 | =John Doe:Richard Stallman= 
$ foo "${_names[@]}" # Sames as "$@"
Args -> 2 | =John Doe= =Richard Stallman=
```

#### Ways to set `IFS`'s values

There are several ways to assign values to `IFS` parameter, both _POSIX_ and _Non-POSIX Compliant_

Take into account that It may arise situations where It's necessary to modify `IFS` without affect its value globally, like as follows:

- Any function that _returns/prints_ array elements as a single string with each element separated by `IFS`'s first value →

```bash
(
        declare -a -- _array=( A B C )
        IFS=: # : as IFS value
        printf "%s\n" "${_array[*]}"
)
```

- Any [[Globbing#Handle Correctly Pathname and Filename|filename handling process]] through **command substitution** or another expansion →

```bash
(
        IFS=$'\n' ; set -f # Set IFS to newline and Disable Globs
        for _file in $( find . -name '.' -o -print ) # Omit . directory
        do
                printf "File -> %s\n" "$_file"
        done
)
```

That is, perform any actions with system files. [[Globbing|See Globbing]] for more information

-  As explained in the sections at the beginning, any File processing line-by-line via **command subtitution** →

```bash
(
        IFS=$'\n' ; set -f # IFS to \n and Globbing Disabled
        for _line in $(< ./foo ) # Same as $( cat ./foo )
        do
                printf "%s\n" "$_line"
        done
)
```

Note that all above examples are executed inside subshell `( )` to avoid modify `IFS`'s value globally and Shell Options such as the globbing one (`set -f`)

Also as mentioned several times, **all parameter creation or modification inside a subshell is not reflected outside it** (i.e. Those changes do not apply in Shell Parent's env)

To handle that situation, there are several ways to **perform `IFS` and Shell Options modification without affect them globally** plus **keep any parameters modification outside child processes**

> [!INFO]-
>
> In the below sections, only a newline `\n` char is assigned to `IFS` parameter to make it more complicated due to trailing newline trimming which command substitution performs
>
> Likewise, they're gonna take the last above code related to File Processing line-by-line as example
> 
> The one related to array elements or positional arguments returned as a single string through `"S*"` expansion is [[#$* Quoted Form|explained in above sections]] 
>
> All related to handle filenames correctly through **globbing** and `find` command is [[Globbing#Handle Correctly Pathname and Filename|here]]
>

##### Local Shell Builtin + C-ANSI Quoting

> **_Non-POSIX Compliant_**

```bash
foo()
{
        local -- _line= \
                 IFS=$'\n' \ # \n as IFS value through ANSI-C Format
                 _oldSetOptions=$( set +o ) # Store Shell Options
        set -f # Disable Globbing (noglob opt)

        for _line in $( < ./foo ) # Non-POSIX Compliant
        do
                printf "%s\n" "$_line"
        done

        eval "$_oldSetOptions" # Restore Shell Options
}
```

> [!IMPORTANT]- Explanation
>
> `IFS`'s modification scope is limited locally to the function due to `local` shell builtin
> 
> > [!CAUTION]
> >
> > Be aware that `IFS` modification will retains its value in the loop context
> 
> A newline `\n` character is assigned to `IFS` through **_C-ANSI Quoting_**
>
> Shell Options are saved for later restoration `set +o`. Therefore, It can proceed to [[Globbing]] disabling with no problem `set -f`
>
> Command Substitution is performed and `for loop` takes it as input. Each _field/word_ resulting from that **unquoted expansion** is assigned to `_line` parameter for each iteration
> 
> Thus, `IFS`'s value has to be modified only to a newline. This way each expansion's output line is treated as a single line after [[Word Splitting]] and `_line` receives an entire line rather than word-by-word (i.e. If blanks were also part of `IFS`'s value)
> 
> Remember that above code cannot handle _leading/trailing/inner_ empty lines →
>
> - Since `IFS`'s value is `\n`, Word Splitting takes that value as delimiter and deletes it
>
> That occurs due to any sequence of whitespace chars is consolidated and taken as a single delimiter. Furthermore, adjacent whitespaces characters do not form a word
>
> - Command Substitution trims any trailing newline from its output. In fact, this is the reason why final empty lines are removed since prior expansion happens before [[Word Splitting]]
>

> [!CAUTION]-
>
> Note that above code is _Non-POSIX Compliant _ due to:
> 
> - `Local` Shell Builtin → Limits `IFS`'s modification Scope to the function
> <br>
> - _ANSI-C Quoting_ → `IFS`'s value is assigned through this way
> <br>
> - `$( < ./file )` → Generates the `For loop`'s input
>
> Those actions can be implemented in a _POSIX Compliant_ way using a simple parameter assignment rather than `local`
>
> It's not possible to limit `IFS` scope but, if assigned, its initial value can be saved in a variable for later restoration
>
> There're several _POSIX-Compliant_ ways to assign a `\n` to `IFS` instead of _ANSI-C Quoting_. [[#Eval + Printf|See below]]
>
> Just use `$( cat ./file )` rather than `$(< ./file )`
>
> Take into account that above code is correct as long as any [[Bash Versioning#Shell Check|Shell validation]] is implemented before it to prevent unintended results
>
> Since _POSIX-oriented_ shells such as _sh_ or _dash_ do not support above _Bash_-like functionality
> ```bash
> bar()
> {
>         case $( /bin/ps -p "$PPID" -o comm= ) in
>
>                 *bash)  return 0
>                         ;;
>                 *)      printf "Not allowed Shell. Try with Bash...\n" 1>&2
>                         return 1
>                         ;;
>         esac
> }
> ```
>
> This way the command which generated script's Parent Process is extracted through command substitution and evaluated in a `case` statement

##### Eval + Printf

> **_POSIX Compliant_**

```bash
foo()
{
        _line= _savedIFS=
        _oldSetOptions=$( set +o ) # Save Shell Options

        [ -n "${IFS+set}" ] && _savedIFS=$IFS # If set, saves IFS's value

        set -f # Disable Globbing
        eval "$( printf 'IFS="\n"' )" # Assing \n to IFS as its value

        for _line in $( cat < ./foo ) # Command Substitution processing
        do
                printf "%s\n" "$_line"
        done

        unset -v -- IFS # IFS Cleanup

        [ -n "${_savedIFS+set}" ] && { # If IFS was set, restore it

                IFS=$_savedIFS ; unset -- _savedIFS ; }

        eval "$_oldSetOptions" # Restore Shell Options
}
```

> [!IMPORTANT]- Explanation
>
> All initial Shell Options are stored in a variable for later restoring (i.e. `set +o` and `eval "$oldSetOptions`)
>
> The same applies with `IFS`, **but only if `IFS` is set.** No value can be stored if a parameter is unassigned
>
> Therefore, `IFS`'s value is stored for further restoring, regardless of whether its value is an empty string or other. [See This](https://unix.stackexchange.com/questions/640062/how-to-temporarily-save-and-restore-the-ifs-variable-properly)
>
> A newline is assigned to `IFS` as a value through `eval` and `printf`
>
> > [!CAUTION]
> >
> > This is the correct way to assing a `\n` to `IFS` parameter →
> > ```bash
> > $ eval "$( printf 'IFS="\n"' )"
> > ```
> >  Note that the following ways are incorrect due to several reasons →
> > ```bash
> > $ IFS=\n # Wrong!
> > ```
> > Above code assigns the `n` character as `IFS`'s value. Backslash char `\` does not remain because It's sintactically interpred as an escape character by the shell
> >
> > Remember any character preceded by a `\` is treated as literal (i.e. a `\` is sintactically meaningfull for the shell)
> >
> > ```bash
> > $ IFS='\n' # Wrong!
> > ```
> >
> > As the other one, this in incorrect due to any character in quotes is treated as literal and It loses its syntactic meaning if it has one
> >
> > Therefore, `IFS` receives as value literally `\n` . Note that It's not an escape sequence
> >
> > ```bash
> > $ IFS=$( printf '\n' )
> > ``` 
> > Due to `printf`'s behavior, `\n` is interpreted as an escape sequence (i.e. a newline) and not as literal
> >
> > The problem in above assignment is that **Command Substitution trims any trailing newline in its output**
> > 
> > Thus, because of this deletion, an empty string is assigned to `IFS`
> > 
> >  ```bash
> > $ IFS=$( printf '"\n"' )
> >  ```
> > Likewise, that does not work either. Here, the same applies as when this is is done `IFS='\n'`, that is, `IFS` get as value literally `\n` (**Not an escape sequence!**)
> >
> > All this being said, the right way comes →
> >  ```bash
> > $ eval "$( printf 'IFS="\n"' )"
> > ```
> > Several thing happens in above command:
> > 
> > - The **Command Substitution**'s resulting string is the following → `IFS="\n"`
> >
> > No trailing newline trimming is performed as last output char is a double quote `"` and not a newline `\n`
> >
> > Note that, as mentioned above, although `\n` sequence is in quotes, `printf` behaviour does that `\n` is interpreted as an escape sequence rather than a literal
> >
> > So, on paper, `IFS="\n"` is a valid assignment to get a newline inside `IFS`. It only remains for that `printf` string to be interpreted as a command
> >
> > That requirement is satisfied by `eval` shell builtin. It does that assigment to `IFS`
>>
>> [More info. here](https://unix.stackexchange.com/questions/477959/n-in-ifs-n-is-a-variable)
>
>
> After the line-by-line file processing is done, `IFS` parameter is unset. This allows to restore its previous state in case It was unassigned
> 
> Regardless of that, if the parameter that stored initial `IFS`'s value is set, which also means that `IFS` was set, `IFS` recovers its previous value. Otherwise, `IFS` keeps unset due to above step
> 
> Shell options are restored to their initial value through `eval "$_oldSetOptions"`

##### Printf + Parameter Expansion

> **_POSIX Compliant_**

```bash
$ IFS=$( printf '\nX' )
$ IFS=${IFS%X}
```

> [!IMPORTANT]- Explanation
>
> Same as the `eval` + `printf ` one. Only `IFS` assignment is differente
>
> A character is placed after the newline `\n` to prevent that the expansion trims trail newline
>
> Therefore, that char is the last one in the **Command Substitution**'s output rather than `\n`
>
> After that, a type of **Parameter Expansion** is used to remove that trailing char `X`

> [!INFO]-
>
> Note that It's not necessary to quote above **Command Substitution** since It occurs in an parameter assigment
>
> Escalar parameter assigment, together with `case` statements, `[[ ]]` shell keyword and other cases, does not undergoe [[Word Splitting]] and [[Globbing]]
> ```bash
> $ foo=$HOSTNAME # Parameter Expansion (Without {})
> $ foo=$( hostname ) # Command Substitution
> $ foo=${HOSTNAME:-$(hostname --long)} # Parameter Expansion
> ```
> ```bash
> $ case $foo in ... # Parameter reference in case statement
> ```
> ```bash
> $ [[ -n $foo ]] && ... # [[ Shell Keyword
> ``` 
>
> They’re like in a double quote contexts, therefore, It’s not necessary to use double quotes
> 
> However, remember that when in doubt, **always quote parameters references !!**
> 
> [More Cases here](https://unix.stackexchange.com/questions/68694/when-is-double-quoting-necessary/68748#68748c)

---

Having seen the above situations, It has to be said that no one should read file lines with a `for loop` since this way need to process a **Command Substitution**'s output

That expansion cannot be quoted as It will be treated as a single string. Therefore, only one iteration will be done with that string as the `for loop`'s parameter value

Thus, [[Word Splitting]] and [[Globbing]] will be performed together with command subtitution trimming trailing newlines from its output

Also, remember that, once expansion is performed and above actions are occurs on output's string, the `for loop` processes each resulting _word/field_ assigning it to the declared parameter

```bash
$ for _line in $(< ./foo) ; do printf "%s\n" "$_line" ; done
```

As mentioned earlier, this situation can be improved **modifying `IFS` to a newline** and **disabling globbing** with `set -f` sentence

```bash
(
        IFS=$'\n'
        set -f

        for _line in $( < ./foo)
        do
                printf "%s\n" "$_line"
        done
)
```

With above code, because of `IFS` limited to just a newline, a line with blanks between non-whitespace chars is not split into several lines due to [[Word Splitting]] and the `foor loop` processing

Globbing character are not interpreted neither, therefore, no filename expansion is performed and no line is generated for each file matched with that glob pattern

Although, expansion continues trimming trailing newlines from its output

Likewise, since `IFS` has `\n` as its value and newlines characters are considered whitespace chars, any consecutive sequence of newlines is consolidated as one single delimiter

In other words, above situation causes the empty lines to be skipped

> You cannot possibly preserve blank lines if you are relying on IFS to split on newlines

Moreover, that `IFS` modification will be remain in the loop context, which means that any unquoted expansion or other situations where [[Word Splitting]] acts, will be taken according to that `IFS` value

**That why FOR LOOP IS NOT A RECOMMENDED WAY TO PROCESS FILE LINES**

[More info. here](https://mywiki.wooledge.org/DontReadLinesWithFor)

Instead of the above one, this is the recommended way →

#### Correct Processing of a File's Lines

> **_POSIX Compliant_**

```bash
while IFS= read -r _line
do
        printf "%s\n" "$_line"

done < ./foo
```

**That's all.**

> [!IMPORTANT]- Explanation
>
> No resulting string from an expansion is taken as input
>
> `read`, as its default behavior, process input until a newline `\n`
>
> That input string processed by `read` undergoes [[Word Splitting]] but not [[Globbing]]
>
> Once above actions are performed, that string is assigned to `read`'s declared parameter
>
> To prevent that leading and trailing blanks and tabs are trimmed, `IFS` is set to an empty string (`read`'s default behavior)
> 
> With that, no splitting, or globbing or final `\n` trimming or empty lines deletion is performed
>
> Note that `read -r` option makes that `read` treats `\` chars as literals and not as escape sequence such as newlines or tabs

Note that above correct way to handle file lines is way more reliable and shorter than [[#Local Shell Builtin + C-ANSI Quoting|this one]]. The same applies with [[#Eval + Printf|this one]]

If a file's last line does not end with a newline character `\n`, `read` process it but returns false

Since `while loop` iterates until `read` returns false, the remaning line is stored in `read`'s declared parameter but It's not processed inside the loop itself

To prevent above situation, just process that remaning line individually →

```bash
while IFS= read -r _line
do
        printf "%s\n" "$_line"

done < ./foo

[[ -n $_line ]] && printf "%s\n" "$_line"
```

`while loop` stops if `read` returns _false_, which means that It has reached the _EOF_ (i.e. a line which end with `\n`)

Then, `$_line`'s value is checked. It it has content (i.e. no empty string), then It's processed

Basically, It process last line which lacks a trailing newline but it has been read by `read`

The same applies to →

```bash
while IFS= read -r _line || [[ -n $_line ]]
do
        printf "%s\n" "$_line"

done < ./foo
```

`While loop` iterates until `read` returns _false_, then `[[ ]]` keyword checks if there was content after last `\n` processed by `read`. If _true_, that content (parameter's value) is processed

This causes that `while loop` continues iterating one last time even if `read` command returns _false_, so that the last line with no newline can be processed

Note that this does not work  →

```bash
printf 'foo\nbar' | while IFS= read -r _line
do
        printf "%s\n" "$_line"
done
```

> [!CAUTION]- Wrong
>
> As mentioned above, `read` process input string until a `\n`
>
> Therefore, `read` process and stores the remaining string after newline but returns _false_
>
> Which makes that `while loop` iterates no more. So `$_line` receives a value that will not be processed

And this does not work either →

```bash
printf 'foo\nbar' | while IFS= read -r _line
do
        printf "%s\n" "$_line"
done

[[ -n $_line ]] && pritnf "%s\n" "$_line"
```

> [!CAUTION]- Wrong
>
> All the `while loop` context is run in a subshell `( )` created when a pipeline `|` is used
>
> As mentioned in the beginning sections, any parameter assignment, such as the `read` one, is not reflected in the Parent Shell's environment
>
> Therefore, when `[[ ]]` check is performed, `$_line` expands to an empty string because It is not set in the process parent's environment

 ---
 
 The following would be the correct way to iterate over an input stream using _pipelines_ to redirect first command's output as input of the `while loop` →

```bash
printf 'foo\nbar' | {

        while IFS= read -r _line # Or || [[ -n $_line ]] ; do ...
        do
                printf "%s\n" "$_line"
        done

        [[ -n $_line ]] && printf "%s\n" "$_line"
}
```

> [!IMPORTANT]- Correct
>
> Braces `{ }` allows to group all commands inside the same shell context (i.e. execution environment)
>
> Therefore, `[[ ]]` recognizes an assigned `_line` parameter and returns _True_

[More information related to this topic](https://mywiki.wooledge.org/BashFAQ/001)