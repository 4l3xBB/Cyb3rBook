---
Primary_category: "[[OSINT]]"
title: "SEARCH ENGINE DISCOVERY"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[OSINT]]

|  **REFERENCES** | |
| --- | --- |
| ***Google Dorking for Pentesters*** | ***[See here](https://www.freecodecamp.org/news/google-dorking-for-pentesters-a-practical-tutorial/)*** |

#### Google Dorks

##### *Search Operators*

| ***Operator*** | ***Description*** | ***Example*** |
| --- | --- | --- |
| **`site:<DOMAIN>`** | ***Limit results to a specific website or domain*** | **`site:domain.tld`** |
| **`inurl:<STRING>`** | ***Find pages with a specific string in URL*** | **`inurl:login`**
| **`filetype:<FILE_EXT>`** | ***Search for files of a particular type*** | **`filetype:php`** |
| **`intitle:<STRING>`** | ***Find pages with a specific string in Title*** | **`intitle:"My Login Panel"`** |
| **`intext:<STRING>`** <br> **`inbody:<STRING>`** | ***Search for a string within the body text of the pages*** | **`intext:"Password reset"`** <br> **`inbody:"Password reset"`** |
| **`link:<DOMAIN>`** | ***Find pages that link to a specific webpage*** | **`link:domain.tld`** |
| **`related:<DOMAIN>`** | ***Find websites related to a specific page*** | **`related:domain.tld`** |
| **`info:<DOMAIN>`** | ***Provide a summary of information about a webpage*** | **`info:domain.tld`** |
| **`define:<STRING>`** | ***Provide definition of a word or phrase*** | **`define:"Laravel Framework"`** |
| **`allinurl:<STRING>`** | ***Find pages containing all specified words in the URL*** | **`allinurl:"admin panel"`** |
| **`allintext:<STRING>`** | ***Find pages containing all specified words in the body text*** | **`allintext:"admin password reset"`** |
| **`allintitle:<STRING>`** | ***Find pages containing all specified words in the title*** | **`allintitle:"Confidential Report 2023"`** |

##### *Use Cases*

> ***[GHD ExploitDB](https://www.exploit-db.com/google-hacking-database)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[GHD Pentest Tool](https://pentest-tools.com/information-gathering/google-hacking)*** 

###### *Finding Login Pages*

```bash
site:domain.tld inurl:login
site:domain.tld (inurl:login OR inurl:admin)
```

###### *Identifying Exposed Files*

```bash
site:domain.tld filetype:pdf
site:domain.tld (filetype:docx OR filetype:xls)
```

###### *Uncovering Configuration Files*

```bash
site:domain.tld inurl:config.php
site:domain.tdl (ext:conf OR ext:cnf)
```

###### *Locating Database Backups*

```bash
site:domain.tld inurl:backup
site:domain.tld filetype:sql
```