---
Primary_category: "[[LFI]]"
title: "LFI - PHP FILTERS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Input Filters*

If a *.php* string is always appended to the value of the *page* parameter, we are restricted to only list the content of *PHP* scripts

However, if the vulnerable function evaluates the included *PHP* file, we will not be able to read its source code, so we can leverage the *convert PHP filter* to retrieve a *base64* string corresponding to the content of the given *PHP* script

##### *Conversion Filters*

> ***base64-encode***

###### *Structure*

```bash
php://filter/convert.base64-encode/resource=<RESOURCE>
```

###### *Payload*

```bash
?page=php://filter/convert.base64-encode/resource=config
```
