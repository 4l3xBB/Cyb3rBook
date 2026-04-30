---
Primary_category: "[[CRACKING]]"
title: "LINUX CREDENTIALS CRACKING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CRACKING]]

#### *Linux System User Passwords*

> ***Hashes within `/etc/shadow` or `/etc/security/opasswd`***

##### *Generic Hash Format*

```bash
$<HASH_ALGORITHM_TYPE>$<SALT>$<HASH>
```

##### *Unshadow*

Before cracking the hashes, just use `unshadow` to merge both `passwd` and `shadow` files as follows →

```bash
cp /etc/passwd /tmp/passwd.bk && cp /etc/shadow /tmp/shadow.bk
```

```bash
unshadow /etc/passwd.bk /etc/shadow.bk | awk -F: '!/[\*!]/ { printf "%s:%s\n", $1, $2 }' > /tmp/unshadowed.hashes
```

##### *MD5*

> [!BUG]- *Hash Format*
>
> ```bash
> $1$38652870$DUjsu4TTlTsOe/xxZ05uf/
> ```
>

```bash
hashcat --force -O --user --hash-type 500 <HASH> <WORDLIST>
```

###### *Show Password in Plain Text*

```bash
hashcat --force -O --user --hash-type 500 <HASH> <WORDLIST> --show
```

##### *SHA512*

> [!BUG]- *Hash Format*
> 
> ```bash
> $6$72820166$U4DVzpcYxgw7MVVDGGvB2/H5lRistD5.Ah4upwENR5UtffLR4X4SxSzfREv8z6wVl0jRFX40/KnYVvK4829kD1
> ```
>

```bash
hashcat --force -O --user --hash-type 1800 <HASH> <WORDLIST>
```

###### *Show Password in Plain Text*

```bash
hashcat --force -O --user --hash-type 1800 <HASH> <WORDLIST> --show
```