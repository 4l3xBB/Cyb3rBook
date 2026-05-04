---
Primary_category: "[[CHECKLISTS]]"
title: "WEB OFFENSIVE CHECKLIST"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CHECKLISTS]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WEB PENTESTING]]

#### *Mindmap*

> ***[Web Application Pentesting Mindmap](https://github.com/eMVee-NL/MindMap/blob/main/image/Mindmap%20Web%20Application%20Pentesting.png)***

![[WEB OFFENSIVE CHECKLIST-20260504211440702.webp|250]]

> ***Zoom in***

---

#### *Enumeration*

##### *Web Application Functionality*

- [ ] ***Accesible Web pages functionality while having Burpsuite open***

- [ ] ***Non-hidden HTTP Parameters: Look for any type of injection***

***e.g.***

***GET/POST Parameters in a Login/Registration Form → [[SQLi]]***

***GET/POST Parameters in Contact Form | Comments | Support Tickets → [[XSS]]***

***A web page with GET Parameters such as*** `File`, `Page` ***and so on → [[LFI]] or [[RFI]]***

***A web application where a logged-in user can update its profile picture by uploading a file → [[FILE UPLOAD|File Upload]]***

- [ ] ***Custom HTTP Response Headers and Cookies***

##### *Fuzzing*

- [ ] ***[[FUZZING#Ffuf|Resources: Directories and Files]]***

- [ ] ***[[FUZZING#Virtual Hosts|Virtual Hosts and Subdomains]]***

- [ ] ***[[FUZZING#HTTP Parameters|Hidden Parameters]]***

##### *XSS*

- [ ] ***Web Page with a Contact Form → Check for [[XSS]]***

- [ ] ***Web Page with a Comment Section → Check for [[XSS]]***

- [ ] ***Support Web Page with a Ticket creation Feature → Check for [[XSS]]***

##### *SQLi*

- [ ] ***Login/Registration Panel → Check for [[AUTHENTICATION BYPASS SQLI|Authentication Bypass]] via [[SQLi]]***

- [ ] ***Any POST or GET Parameter. Any Cookie and User-Agent value as well***

##### *LFI*

- [ ] ***Any POST or GET parameter***

##### *File Upload*

- [ ] ***If the website has a login feature and we are logged in, check for [[FILE UPLOAD|File Upload]]***

##### *Command Injection*

- [ ] ***Any POST or GET Parameter***

##### *HTTP Verb Tampering*

- [ ] ***Try the same request with a different HTTP method (e.g. GET ↔ POST, POST ↔ PUT, GET ↔ HEAD )***

##### *IDOR*

- [ ] ***Look for any direct object reference in the web application to test for its system control access***

##### *XXE*

- [ ] ***Search for any HTTP request which sends an XML data structure on its body***

---

#### *Exploitation*

##### *SQLi*

- [ ] ***Enumeration: Databases, Tables, Columns, Fields***

- [ ] ***Check if the current DB user has Read/Write permissions: FILE Privilege and `secure_file_priv` empty or set to an interesting path***

- [ ] ***If [[UNION BASED SQLI#Reading Files|READ]] permissions → Look for interesting files such as:***

***Web Server Configuration Files***

***Virtual Hosts Configuration Files: Web Root (DocumentRoot) Path, .HTPasswd files or another type of sensitive files***

***Configuration files within the Web Root ( config.php, db_conn.php... )***

***Service Configuration files that are externally accesible such as [[20, 21 - FTP|FTP]], SSH, [[873 - RSYNC|RSYNC]], CIFS, SQUID, SNMP and so on***

***Home Directory Files such as SSH Keys, Shell History Files and so on***

- [ ] ***If [[UNION BASED SQLI#Writing Files|WRITE]] Permissions → Web Shell Deployment on Web Root or any directory on which the system user running the DBMS has write permissions***

##### *XSS*

- [ ] ***[[XSS#Web Defacement|Web Defacement]]***

- [ ] ***[[BLIND XSS#Session Hijacking|Session Hijacking]]***

- [ ] ***[[XSS#Login Form Injection|Phishing via Login Form Injection]]***

##### *LFI*

- [ ] ***Try different [[LFI - BASIC BYPASSES|Bypasses]]***

***Once we are able to exploit the LFI to point to other web files***

- [ ] ***Information Leakage e.g. Configuration files***

- [ ] ***Source code of Webroot scripts e.g. Other PHP scripts***

***RCE***

- [ ] ***[[LFI TO RCE - PHP WRAPPERS|PHP Wrappers]] e.g. ( `data:// | input:// | expect://` )***

- [ ] ***[[LFI TO RCE - FILE UPLOAD|File Upload]]***

- [ ] ***[[LFI TO RCE - RFI|RFI]]***

- [ ] ***[[LFI TO RCE - LOG POISONING|Log Poisoning]]***

- [ ] ***[[LFI TO RCE - PHP SESSION POISONING|PHP Session Poisoning]]***

##### *File Upload*

- [ ] ***If the Web Application runs X (e.g. PHP), try uploading an X script (e.g. test.php)***

***If the given upload is not allowed, security filters may have been set up. If so →***

- [ ] ***[[FILE UPLOAD - FILTER BYPASSES#Client-Side Validation|Client-Side Validation Bypass]]***
- [ ] ***[[FILE UPLOAD - FILTER BYPASSES#Blacklist|Blacklist Bypass]]***

- [ ] ***[[FILE UPLOAD - FILTER BYPASSES#Whitelist|Whitelist Bypass]]***

- [ ] ***[[FILE UPLOAD - FILTER BYPASSES#Content-Type Header|Content-Type Bypass]]***

- [ ] ***[[FILE UPLOAD - FILTER BYPASSES#MIME Type|MIME Type Bypass]]***

***If none of the previous bypasses work →***

- [ ] ***Fuzz for allowed extensions (e.g. SVG)***

- [ ] ***Check [[FILE UPLOAD - LIMITED FILE UPLOADS|Limited File Uploads]]***

***Furthermore, we have to bear in mind that if we discover an [[LFI]] vulnerability, we can just fuzz for allowed extensions, add a code snippet (e.g. mini WebShell) within the file before uploading it and request it from a web client. We will gain RCE***

##### *Command Injection*

- [ ] ***Try a [[COMMAND INJECTION#Exploitation|basic command injection]] to see if there is any input validation or sanitizacion in place***

***if not, reverse shell and pwned. If so, proceed as follows***

- [ ] ***Try to bypass them with different [[COMMAND INJECTION#Filter Evasion - Blacklist|filter evasions]] and [[COMMAND INJECTION#Advanced Command Obfuscation|command obfuscation]]***

##### *HTTP Verb Tampering*

- [ ] ***Send an OPTIONS HTTP request to know the supported HTTP methods by the web server***

- [ ] ***In case of any error or unauthorized operation, simply try changing the HTTP request method to look for differences in the HTTP response***

##### *IDOR*

This attack vector results from a bad access control system or its absence and a direct object reference

- [ ] ***[[IDOR - INFORMATION DISCLOSURE#Insecure Parameters|Simple Direct Object Reference]] (e.g. <URL\>?id=1 ) → Just try changing its value to something else to test the access control system***

- [ ] ***[[IDOR - INFORMATION DISCLOSURE#Bypassing Encoded References|More Secure Direct Object Reference]] (e.g. <URL\>?id=`6B29FC40-CA47-1067-B31D-00DD010662DA` or <URL\>?id=098f6bcd4621d373cade4e832627b4f6 ) → Check if the reference is being created in the frontend ( e.g. JS Function ) before being sent to the server***

Once we discover the *IDOR* → 

- [ ] ***[[IDOR - INFORMATION DISCLOSURE#Mass Enumeration|Mass Enumeration]]***

- [ ] ***[[IDOR#Insecure Function Calls|Insecure Function Calls]]***

##### *XXE*

***Once we have located an HTTP request that sends XML data within its body*** 

***Data Disclosure →***

- [ ] ***[[XXE - DATA DISCLOSURE#Simple XXE|Simple XXE]]***

- [ ] ***If the web application runs PHP, try [[XXE - DATA DISCLOSURE#PHP Filter Wrapper XXE|XXE using PHP Filter Wrapper]]***

- [ ] ***If not, try [[XXE - DATA DISCLOSURE#CDATA XXE|XXE using CDATA]]***

- [ ] ***If our input is not reflected anywhere in the HTTP response, check if the web application handles error properly***

- [ ] ***If not, try [[XXE - DATA DISCLOSURE#Error Based XXE|Error based XXE]]***

- [ ] ***If we are dealing with a Blind XXE, see [[XXE - DATA DISCLOSURE#Blind OOB XXE|Blind OOB XXE]]***

- [ ] ***Bear in mind that we can achieve [[XXE - RCE|RCE]] as well***