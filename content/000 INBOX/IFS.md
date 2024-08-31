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

**Last variable gets the remaining words** if resulting fields' number is greater than the number of parameters defined by `read`

```bash
$ read -r _A _B <<< "Alex John Doe" # IFS Default Value
$ printf "=%s= " "$_A" "$_B"
=Alex= =John Doe= # _B receives the remaining args
```

> [!IMPORTANT]-
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
> > Be aware that `IFS=' :'` modifies `IFS`'s value globally, which is often not what is desired, below there're several ways to handle this situation correctly

##### Unquoted Shell Expansion

When **Shell Expansion** is performed in shell's parsing, all unquoted expansion undergoes [[Word Splitting]] and [[Globbing]]

> [!INFO]-
> There're different types of shell expansion and they all occur in a set order after command has been splitted into tokens
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
> With all above actions done, **Parameter Assignment**, **Redirections** and **Quote Removal** are performed prior to command execution

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

> [!INFO]
> In **Command Substitution**, trailing newlines are stripped out in subshell's output
> 
> Likewise, output's embedded strings may be removed due to [[Word Splitting]] if `IFS` is unset, default or with a `\n` explicitly assigned
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
> > Note the following notation with `batcat --show-all` :
> > - `·` → _Blank/Whitespace_
> > - `├──┤` → Tab `\t`
> > - `␊` → Newline `\n`
>
> Above file's content has several blanks, tabs and newlines, both internal and leading/trailing
> 
> If `foo` file has to be processed line-by-line, below command is incorrect due to several reasons →
> ```bash
> for _line in $( < ./foo )
> do
>         printf "%s\n" "$_line"
> done
> ```
> ```bash
> # Command Output
> Ubuntu
Debian
foo
Arch
Fedora
CentOS
Parrot
> ```
> Above way use unquoted command substitution to process file's lines. Thus, prior expansion undergoes [[Word Splitting]] and [[Globbing]]
>
> Leading `\n` and _leading/trailing/inner_ blanks and `\t` are treated as delimiters due to `IFS` default value
>
> Due to Word splitting applied on command substitution output, **the inner ones are consolidated** and **the _leading/trailing_ are chopped off**
>
> As mentioned earlier, trailing newlines are trimmed in command substitution output too
>
> Note that there's one line (the `foo` one) that has been generated through Globbing due to `*` char after splitting
> 
> Para solucionar los problemas anteriores, dejamos una versión del bloque de comandos anterior pero con mejoras como la limitación de IFS a saltos de línea y set -f para evitar el globbing
> 
> Recordemos que la eliminación de los saltos de línea finales en el resultado de la sustitución de comandos no se puede tratar ya que es una características interna de dicha funcionalidad
>
> Para solucionar el problema anterior, simplemente no hacer uso de sustitución de comandos para procesar las líneas de un fichero y, por lo tanto, utilizar bucle `while IFS= read -r` proporcionando como entrada el fichero `done < ./file`
>
> Con ello evitamos word splitting ya que read lee y procesa hasta un salto de línea ( linea por linea del fichero) . Evitamos el trimming inicial y final de los espacios y tabuladores gracias a `IFS=`, lo igualamos a una cadena vacía para ello.
> 
> Tampoco actua el globbing puesto que no se lleva a cabo ningún tipo de expansión, simplemente tener en cuenta el entrecomillar la referencia a la variable cuando se vaya a usar para imprimir cada una de las líneas


#### Ways to set `IFS`'s values