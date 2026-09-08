---
Primary_category: "[[SQLi]]"
title: "AUTHENTICATION BYPASS SQLI"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[SQLi]]

#### *Abuse*

There are situations where we face a login panel on a web application and we do not have valid credentials to log in successful

![[SQLi-20260203175946381.webp|300]]

> ***Zoom in***

In this case, we could check for *SQL Injection* by entering one of the following chars

```bash
' # %27
" # %22
# # %23
; # %3B
) # %29
```

If it is vulnerable, we may encounter the following error or something similar instead of a *Login failed* message

![[SQLi-20260203175917535.webp|350]]

> ***Zoom in***

Once we know this, we would need the *SQL Query* always to return *True*, regardless of the username and password entered. To do so, we can leverage the *OR* operator in order to bypass the authentication

Remember that an *OR* operator always returns *TRUE* if one of the two conditions is *TRUE*

Therefore, we can enter the following payload

```sql
admin' OR '1'='1
```

Regardless the first condition, the second one will always return *TRUE (1=1)*

As stated, since the *OR* operator returns *TRUE* if at least one condition is *TRUE*, the authentication attempt will succeed and we will successfully bypass the login form

In this case, this occurs because the *admin* user exists, and therefore returns *TRUE*

![[SQLi-20260203182242985.webp|350]]

> ***Zoom in***

##### *Comments*

We can use comments as well to bypass the login. That is, instead of worrying about closing every single quote that remains open to prevent any syntax error, just add a comment after the entered payload

To do so, we can use **`-- -`** or **`#`**

Thus, the entered data would be the following

```sql
admin' OR 1=1 -- -
```

And it would also bypass the authentication login form since the user exists, which returns *TRUE*, and no other conditions are evaluated due to the added comment

---

#### *Resources*

***[Payload All the Things: Authentication Bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)***