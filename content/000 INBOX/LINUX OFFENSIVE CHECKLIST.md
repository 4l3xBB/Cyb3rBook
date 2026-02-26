---
Primary_category: "[[CHECKLISTS]]"
title: "LINUX OFFENSIVE CHECKLIST"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CHECKLISTS]]

#### *Mindmap*

---

#### *Enumeration*

##### *HTTP\[s\]*

###### *Web Application Functionality*

- [ ] ***Accesible Web pages functionality while having Burpsuite open***

- [ ] ***Non-hidden HTTP Parameters: Look for any type of injection***

***e.g.***

***GET/POST Parameters in a Login/Registration Form → [[SQLi]]***

***GET/POST Parameters in Contact Form | Comments | Support Tickets → [[XSS]]***

***A web page with GET Parameters such as*** `File`, `Page` ***and so on → [[LFI]] or [[RFI]]***

- [ ] ***Custom HTTP Response Headers and Cookies***

###### *Fuzzing*

- [ ] ***[[FUZZING#Ffuf|Resources: Directories and Files]]***

- [ ] ***[[FUZZING#Virtual Hosts|Virtual Hosts and Subdomains]]***

- [ ] ***[[FUZZING#HTTP Parameters|Hidden Parameters]]***

###### *XSS*

- [ ] ***Web Page with a Contact Form → Check for [[XSS]]***

- [ ] ***Web Page with a Comment Section → Check for [[XSS]]***

- [ ] ***Support Web Page with a Ticket creation Feature → Check for [[XSS]]***

###### *SQLi*

- [ ] ***Login/Registration Panel → Check for [[AUTHENTICATION BYPASS SQLI|Authentication Bypass]] via [[SQLi]]***

- [ ] ***Any POST or GET Parameter. Any Cookie and User-Agent value as well***

###### *File Upload*

- [ ] ***If the website has a login feature and we are logged in, check for [[FILE UPLOAD|File Upload]]***

###### *SSTI*

- [ ] ***If Werkzeug is the Web Server, check for [[SSTI|SSTI (Server Side Template Injection)]]***

##### *FTP*

###### *Non-Credentialed Enumeration*

- [ ] ***Anonymous Access***
- [ ] ***Known CVE (Searchsploit, Google...)***

###### *Credentialed Enumeration*

- [ ] ***If we can upload files, just try to upload them to a location accesible from the Web Server in order to get an RCE through the Web Shell***

---

#### *Exploitation*

##### *HTTP\[s\]*

###### *SQLi*

- [ ] ***Enumeration: Databases, Tables, Columns, Fields***

- [ ] ***Check if the current DB user has Read/Write permissions: FILE Privilege and `secure_file_priv` empty or set to an interesting path***

- [ ] ***If [[UNION BASED SQLI#Reading Files|READ]] permissions → Look for interesting files such as:***

***Web Server Configuration Files***

***Virtual Hosts Configuration Files: Web Root (DocumentRoot) Path, .HTPasswd files or another type of sensitive files***

***Configuration files within the Web Root ( config.php, db_conn.php... )***

***Service Configuration files that are externally accesible such as [[20, 21 - FTP|FTP]], SSH, [[873 - RSYNC|RSYNC]], CIFS, SQUID, SNMP and so on***

***Home Directory Files such as SSH Keys, Shell History Files and so on***

- [ ] ***If [[UNION BASED SQLI#Reading Files|READ]] Permissions via a Non-Blind SQLi ( i.e. Data displayed in the HTTP Response ) → Check for Log Poisoning as it is was [[LFI]]***

- [ ] ***If [[UNION BASED SQLI#Writing Files|WRITE]] Permissions → Web Shell Deployment on Web Root or any directory on which the system user running the DBMS has write permissions***

###### *XSS*

- [ ] ***[[XSS#Web Defacement|Web Defacement]]***

- [ ] ***[[BLIND XSS#Session Hijacking|Session Hijacking]]***

- [ ] ***[[XSS#Login Form Injection|Phishing via Login Form Injection]]***

###### *LFI*

###### *RFI*

###### *File Upload*

---

#### *Privesc | Lateral Movement*

##### *Global*

- [ ] ***Credentials in web application config. files***
- [ ] ***Interesting System Directories ( `/<DIRECTORY>`, `/opt`, `/var` and so on  )***
- [ ] ***Capabilities***
- [ ] **`Sudo` *version vulnerable***
- [ ] ***Monitor processes (Cron Jobs and so on)***
- [ ] ***Writable Docker socket files***
- [ ] ***TMUX or SCREEN sessions***
- [ ] ***NFS Shares with* `no_root_squash`** ***enabled***
- [ ] ***Kernel Vulnerability (e.g. DirtyCow, DirtyPipe and so on)***
- [ ] ***Services not accesible externally***
- [ ] ***Network Traffic with TCPDump or TShard, if installed***
- [ ] ***Run Privesc Enumeration Scripts such as [LINPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)***

##### *Per User*

- [ ] ***Sudo Privileges***
- [ ] ***Groups to which the user belongs to***
- [ ] ***SUID/GUID Binaries (PwnKit as well)***
- [ ] ***Enviromental variables and parameters***
- [ ] ***Files and Directories with Write Permissions***
- [ ] ***Home directory sensitive files ( e.g. Shell History Files, SSH Keys and so on )***
- [ ] ***Access other system user directories within `/home` directory***