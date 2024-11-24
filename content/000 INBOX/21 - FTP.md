---
Primary_category: "[[PROTOCOLS AND SERVICES]]"
title: "21 - FTP"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[PROTOCOLS AND SERVICES]]

#### Enumeration

##### User Authentication

###### *FTP Client*

```bash
ftp <TARGET> [PORT]
> USER
> PASSWORD
```

######  *Web Browser*

```bash
ftp://<USERNAME>:<PASSWORD>@<TARGET>:[PORT]
```

##### Anonymous Login

```bash
ftp -a <TARGET> [PORT]
```

> ***The `-a` option bypasses the normal login procedure and use anonymous login instead***

##### Banner Grabbling

###### *Netcat*

```bash
nc -nv <TARGET> <PORT> <<< ""
```

---

#### Download Files Recursively

###### *Passive Mode*

```bash
wget --mirror "ftp://anonymous:anonymous@<TARGET>"
```

###### *Active Mode*

```bash
wget --mirror --no-passive-ftp "ftp://anonymous:anonymous@<TARGET>"
```

###### *Special Chars in Credentials*

```bash
wget --mirror [--no-passive-ftp] --user '<USER>' --password '<PASSWORD>' "ftp://<TARGET>"
```