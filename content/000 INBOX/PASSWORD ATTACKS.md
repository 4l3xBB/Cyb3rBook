---
Primary_category: "[[PENTESTING ROOT]]"
title: "PASSWORD ATTACKS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[PENTESTING ROOT]]

#### Cracking Offline

##### *ZIP*

###### *Show .ZIP File Technical Metadata and Other information*

```bash
7z l -slt <ZIP_FILE>
```

###### *Obtain a Hash/Digest from the Zip File*

```bash
zip2john <ZIP_FILE> > zip.john
```

###### *Hash Cracking with John*

```bash
john zip.john --wordlist=/usr/share/wordlists/rockyou.txt
```

###### *Show Cracked Hashes/Passwords*

```bash
john --show zip.john
```

```bash
cat ~/.john/john.pot
```