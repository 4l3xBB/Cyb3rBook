---
Primary_category: Dante Root
title: Dante Root
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

#### *Networks*

```bash
fping --alive --quiet --stats --generate 10.10.110.100/24
```

```bash
bash -c 'for _ip in 10.10.110.{1..245} ; do ( ping -c 1 -W 1 "$_ip" &> /dev/null && printf "%s\n" "$_ip" & ) ; done '
```

> ***ICMP Blocking Test***

```bash
nmap -p21,22,25,23,53,67,80,88,143,110,443,444,445,111,135,139,389,636,8000,8080,8888,65000,3306,1433,5432,6379,5985,3389,5986,1521,873 --open -sS --min-rate 5000 -vvv -n -Pn --disable-arp-ping 10.10.110.0/24
```

##### *10.10.110.100/24*

- ![](https://99px.ru/sstorage/86/2018/02/image_861602180019251883900.gif)
	- [[10.10.110.2 - OOC]]
- ![](https://99px.ru/sstorage/86/2018/02/image_861602180019251883900.gif)
	- [[10.10.110.100]]

<br>