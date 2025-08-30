---
Primary_category: "[[OSINT]]"
title: "CERTIFICATE TRANSPARENCY"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[OSINT]]

#### Subdomain Enumeration

##### *Crt.sh*

> ***[crt.sh](https://crt.sh/)***

For a given domain, obtain all ***Common Names (CN)*** for which a valid ***TLS Certificate***  has been issued

```bash /<DOMAIN>/
curl --silent --request GET --location "https://crt.sh?q=<DOMAIN>&output=json" | jq --raw-output '.[] | .common_name, .name_value' | sort -u
```

---

#### IP Addresses Extraction from Subdomains

Once the subdomain are gathered, just use `host` or `dig` to get the *IP Address* to which they resolve

##### *Crt.sh + Host*

```bash /<DOMAIN>/
while IFS= read -r _domain ; do host "$_domain" ; done < <( curl --silent --request GET --location "https://crt.sh?q=<DOMAIN>&output=json" | jq --raw-output '.[] | .common_name, .name_value' | sort -u ) | awk '/digitaldot.es.*has address/ {print $1,$4}'
```

##### *Crt.sh + Dig*

```bash /<DOMAIN>/
while IFS= read -r _domain ; do printf "%s %s\n" "$_domain" "$(dig $_domain +short)"; done < <( curl --silent --request GET --location "https://crt.sh?q=<DOMAIN>&output=json" | jq --raw-output '.[] | .common_name, .name_value' | sort -u )
```

---

#### Information Extraction from IP Address

##### *Shodan*

> ***[Shodan](https://www.shodan.io/)***

###### *IP Addresses from a File*

```bash
while IFS= read -r _ip ; do shodan host "$_ip" ; done < <FILE>
```
