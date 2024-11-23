---
Primary_category: "[[PROTOCOLS AND SERVICES]]"
title: "53 - DNS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[PROTOCOLS AND SERVICES]]

#### Enumeration

##### Banner Grabbling

###### *DIG*

```bash
dig version.bind CHAOS TXT @<TARGET> +short
```

###### *Nmap*

```bash
nmap -p<DNS_PORT> --script dns-nsid -vvv -Pn <TARGET>
```

---

#### Zone Transfer

```bash
dig axfr @<TARGET> +short
dig axfr @<TARGET> <DOMAIN> +short
```
