---
Primary_category: "[[PASSWORD ATTACKS]]"
title: "WORDLISTS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PASSWORD ATTACKS]]

#### *Wordlists Generation*

##### *Cewl*

> ***[Cewl](https://github.com/digininja/CeWL)***

Wordlists generation from a given web page providing its *URL*

```bash
cewl --depth <INTEGER> --min_word_length <INTEGER> --lowercase --write <OUTPUT_FILE> <URL>
```

##### *Username Anarchy*

> ***[Username Anarchy](https://github.com/urbanadventurer/username-anarchy)***

> ***Username Format Generator***

It is really useful when trying to discover the naming convention used in the *AD* enviroment

```bash
./username-anarchy --input-file <USERSLIST>
```

##### *CUPP*

> ***[CUPP](https://github.com/Mebus/cupp)***

```bash
python3 cupp.py --interactive
```

---

#### *Passwords Mutation*

##### *Hashcat*

> ***[Hashcat](https://github.com/hashcat/hashcat)***

> ***[Reference](https://hashcat.net/wiki/doku.php?id=rule_based_attack)***

###### *Generate a Custom Wordlist*

```bash
hashcat --force --rules-file=<RULES_FILE> --stdout <WORDSLIST> | sort -u > <OUTPUT_FILE>
```

> [!BUG]- *Rules Wordlists*
>
> | ***Wordlist*** | |
> | --- | --- |
> | **`/usr/share/hashcat/rules/best64.rule`** | |
> | **`/usr/share/john/rules/best64.rule`** | |
>

###### *Generate a Custom Wordlist and Crack on the Fly*

```bash
hashcat --force -O --attack-mode <ATTACK_MODE> --hash-type <HASH_TYPE> --rules-file <RULES_FILE> <HASH_FILE> <WORDLIST>
```