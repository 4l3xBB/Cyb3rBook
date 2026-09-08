---
Primary_category: "[[ZSH]]"
title: "ZSH CUSTOM FUNCTIONS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
  - ZSH
cssclasses:
---

###### PRIMARY CATEGORY → [[ZSH]]

In this [[DESKTOP SETUP|Setup environment]], all the following _[[SHELL SCRIPTING|Shell]] Functions_ are declared in the _[[ZSH#*src/custom.zsh*|custom.zsh]] script_

Then, the _custom.zsh script_ is sourced from the [[ZSH#*.zshrc*|.zshrc]] file as follows →

```bash title="~/.zshrc"
# Custom Functions
if [[ -f /home/al3xbb/.config/zsh/src/custom.zsh ]] ; then
    source /home/al3xbb/.config/zsh/src/custom.zsh
fi
```

> [!IMPORTANT]-
>
>
> In the above code, an _absolute path_ is used instead of the _Tilde Expansion_ `~`
>
> This is because the _Root's .zshrc_ file is a _symbolic link_ to the _non-privilege_ user's _.zshrc_
>
> Note that the `.config/zsh` directory only exists in `/home/al3xbb` but not in `/root`
>
> > ***[[ZSH#Symbolic Link|Reference]]***
>

***Custom.zsh Source File → [See here](https://github.com/4l3xBB/Env-Setup/blob/main/zsh/src/custom.zsh)***

---

#### *Terminal/Screen*

***Shell Functions* related to the *Terminal and Screen* actions →**

##### *clearScreenAndScrollback*

> ***[Reference](https://unix.stackexchange.com/questions/517025/zsh-clear-scrollback-buffer#answer-531178)***

It clears the _Screen Content and Scrollback Buffer_ through the `C-l` keybind

> [!NOTE]- *Function*
>
> ```bash
> clearScreenAndScrollback ()
> {
>     clear && printf '\e[3J'
>     zle && zle .reset-prompt && zle -R
> }
> ```
>
> ```bash
> zle -N clearScreenAndScrollback
> bindkey '^L' clearScreenAndScrollback
> ```
>

> ***More info [[KITTY#Clear Screen and Scrollback Buffer|here]]***

---

#### *Pentesting*

**_Shell Functions_ related to the _[[PENTESTING|Pentesting]] Process_ →**

> Most of the functions are _[[POSIX|Non-POSIX-Compliant]]_ as _ZSH_ is the Targeted _Shell_

##### *mkt*

It creates a _Pentesting Folder Structure_ to store all documentation related to the target 🎯

> [!NOTE]- *Function*
>
> ```bash
 > mkt()
> {
>         local -- _dir=
>         local -a -- _dirs=(
>                 evidence
>                 logs
>                 scans
>                 scope
>                 tools
>         )
>
>         for _dir in "${_dirs[@]}"
>         do
>                 mkdir -p -- "$_dir"
>
>                 [[ $_dir == "evidence" ]] &&
>
>                         mkdir -p -- "$_dir"/{creds,data,screenshots}
>         done
> }
> ```
>


> [!IMPORTANT]-
>
> Note that the above function is _[[POSIX|Non-POSIX Compliant]]_ due to Bash Specific Functionalities such as `local`, `[[ ]]` shell keyword, **Brace Expansion** `{ }` or _indexed arrays_
>
> Therefore, It is not destined to _POSIX Compliant_ shells like _sh_ or _dash_
>
> If _POSIX Compliant_ is needed →
> ```bash
> mkt()
> {
>         _dir= _dirs="evidence logs scans scope tools"
>
>         for _dir in $_dirs
>         do
>                 mkdir -p -- "$_dir"
>
>                 [ "$_dir" = "evidence" ] &&
>
>                         mkdir -p -- "$_dir"/creds \
>                                     "$_dir"/data \
>                                     "$_dir"/screenshot
>         done
>
> }
> ```
> 

##### *extractPorts*

This function mainly displays a **Summary** of the Target's Open Ports

Furthermore, it **copies the Open Ports** to the *System Clipboard*

> [!NOTE]- *Function*
>
> ```bash
 > extractPorts()
> {
>   local -- _file=$1 _ip= _ipAddress= IFS=, \
>            _tmpFile=$( mktemp --quiet --suffix=.log --tmpdir=. )
>   local -a -- _ports=()
>
>   trap 'rm --force "$_tmpFile"' RETURN
>
>   [[ -s $_file ]] || {
>     printf "\n[!] File must be a Nmap Grepable Format :)\n" 1>&2
>     return 1
>   }
>
>   while IFS=$' ' read -r _ _ip
>   do
>     [[ $_ip =~ (^[[:digit:]]{1,3}(\.[[:digit:]]{1,3}){3}) ]] && {
>       _ipAddress=${_ip%%[[:space:]]*}
>       break
>     }
>
>   done < "$_file"
>
>   while IFS= read -r _port
>   do
>     _ports+=( "$_port" )
>
>   done < <( grep --only-matching \
>                  --perl-regexp \
>                  '\s\K\d{1,5}(?=/open)' \
>                  "$_file"
>           )
>
>   cat << PORTS > "$_tmpFile"
>
>   [+] Extracting information...
>
>       [+] IP Address: $_ipAddress
>       [+] Open Ports: ${_ports[*]}
>
>   [+] Ports Copied to Clipboard
>
> PORTS
>
>   command -V xclip &> /dev/null && xclip -sel clip < <( tr -d '\n' <<< "${_ports[*]}" )
>
>   command -V bat &> /dev/null && bat "$_tmpFile"
>
>   return 0
> }
> ```
>

> [!IMPORTANT]-
>
>  An error related to the `trap 'COMMAND' RETURN`  line may occur when callling the function
>
> If this happens, simply replace the above line with `rm --force -- "$_tmpFile"` at the end of the *Function*
>
> Likewise, instead of a function, the above code can be copied into a [[BASH|bash]] script as follows →
>
> ```bash
> sudo nvim /usr/bin/extractPorts
> ```` 
>
> ```bash
> chmod 777 !$
> ```
>
> > ***Script Content***
>
> ```bash
> #!/usr/bin/env bash
>
> extractPorts()
> {
> 	FUNCTION_CONTENT
> }
> 
> extractPorts "$@" || exit 99
> ```
>

##### *validateIP*

This function simply checks if the _IP Address_ entered as an argument is valid

It is used by the _[[ZSH CUSTOM FUNCTIONS#setTarget|setTarget]] function_

> [!NOTE]- *Function*
>
> ```bash
> validateIP ()
> {
>     local _IP=$1 _octet=
>
>     [[ $_IP =~ ^[0-9]+(\.[0-9]+){3}$ ]] || return 1
>
>     while read -rd '.' _octet
>     do
>         (( _octet > 255 )) && return 1
>
>     done <<< "$_IP".
>
>     return 0
> }
> ```
> 

##### *setTarget*

It sets the Target's _IP Address_ and _Hostname_ as one of the _[[POLYBAR#target_to_hack|Polybar]] Bar's Modules_

This function prints the above data into the `/home/al3xbb/.config/bin/target` file

Then the _Polybar Module_ carries out an action based on that _File's Content_ as follows →

- ***If Empty*** → *"No target"* as _Polybar's Bar Content_
- ***If not Empty*** →  _File's Content_ as the _Polybar's Bar Content_, i.e. the _IP Address and Hostame_

> [!NOTE]- *Function*
>
> ```bash
> setTarget ()
> {
>     local -- _IP=$1 _machineName=$2 \
>              _targetFile=/home/al3xbb/.config/bin/target
>
>     (( $# != 2 )) && {
>
>         /bin/cat <<HELP 1>&2
>
>   [!] This function requires two arguments: IP Addresss and Machine Name
>
>     Syntax: setTarget XXX.XXX.XXX.XXX "Machine Name"
> HELP
>         return 1
>     }
>
>     validateIP "$_IP" || {
>         printf "\n%s\n" '[!] First arg must be a valid IP Address' 1>&2
>         return 1
>     }
>
>     [[ -n $_IP && -n $_machineName ]] || {
>
>         printf "\n%s\n" '[!] Args cannot be an empty string' 1>&2
>         return 1
>     }
>
>     printf "%s:%s\n" "$_IP" "$_machineName" > "$_targetFile"
> }
> ```
>

![[customZSHSetTargetFunction.gif|375]]

##### *clearTarget*

This functions empties the `/home/al3xbb/.config/bin/target` file

Therefore, as mentioned in the [[ZSH CUSTOM FUNCTIONS#*setTarget*|setTarget]] function, the *[[POLYBAR#target_to_hack|Polybar]] Module* sets *"No Target"* as _Polybar Bar's Content_

> [!NOTE]- *Function*
>
> ```bash
> clearTarget ()
> {
>     local -- _targetFile=/home/al3xbb/.config/bin/target
>
>     [[ -s $_targetFile ]] && : '' > "$_targetFile" || return 1
> }
> ```
>

![[customZSHclearTargetFunction.gif|375]]