---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: LARAVEL
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - Laravel
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

***Laravel → PHP Web Application Framework***

***[More Information here](https://github.com/laravel/laravelc)***

#### Debugging Mode

If *Laravel Debug Mode* is enabled, quite a lot sensitive information is leaked such as the content of the [[#.Env Configuration File|.env]] configuration file

---

#### .Env Configuration File Leakage

This file contains sensitive information such as →

- ***Database Credentials***

- ***APP Key***

---

#### Insecure Object Deserialization → RCE

The *Laravel App Key* can be used by an attacker to craft a malicious payload and encrypt it with that key before sending it to the *Web Server*

The above situation is exploited via the [[CVE-2018-15133]]

---

#### CVE

- ![](https://img.freepik.com/premium-photo/3d-anonymous-icon-privacy-cybersecurity-illustration-logo_762678-53761.jpg)
	- [[CVE-2018-15133]]