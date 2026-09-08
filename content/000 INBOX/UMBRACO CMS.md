---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: "UMBRACO CMS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Theory*

##### *Backend URL*

```bash
http[s]://<TARGET>/umbraco
```

##### *Interesting Files*

###### *Umbraco 7 or lower*

Most *Umbraco CMS* installations used to work with *SQL Server Compact Edition ( SQL CE )*

The connection string points to an *SDF* resource located within the **`/App_Data`** directory

We can extract some sensitive information from the resource above, such as hashes of *Umbraco* user's password or *API Keys*

To do, proceed as follows

```bash
strings --data all -n 4 > sdf.data
```

###### *Since Umbraco 8*

In this case, the connection string refers to an *SQLite* file instead of an *SDF* file, also located within the **`/App_Data`** directory

---

#### *Discovery | Footprinting*

There are different ways to verify if the web application we are facing is an *UMBRACO CMS* or not

##### *Backend URL*

Like most *CMSs*, it has a *Control Panel / Backend* to manage all aspects related to the web application itself

It can be accessed from the *URL* below

```bash
http[s]://<TARGET>/umbraco
```

##### *Source Code*

We can also request the content of the home page and filter by the word *Umbraco*

```bash
curl --silent --location --request GET 'http://<TARGET>' |& grep -i --color -- 'umbraco'
```