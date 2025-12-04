---
Primary_category: "[[WINDOWS RECONAISSANCE]]"
title: "WINDOWS DNS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS RECONAISSANCE]]

#### *Theory*

By default, when a *Windows Server* host is promoted to *Domain Controller (DC)*, by installing the *Active Directory Domain Services (AD DS)* feature, other services are also installed such as *DHCP*, *DNS* and so on

Therefore, the *DC* ends up being the *primary nameserver* of any *domain-joined* host

The *DNS-related* feature on *AD* is called ***Active Directory Integrated Domain Name System***

In the other hand, it is important to note that any authenticated domain account has sufficient rights over the *domain DNS zone* and its children *(i.e. DNS records)* in order to query and retrieve the value of any existing record

Furthermore, any authenticated domain account can add new records to the *domain DNS zone*, which could prove useful in some situations such as *[[WEB CLIENT ABUSE (WEBDAV)|webDAV coercion]]*

Take into account that on an *AD enviroment*, when the primary nameserver *(DC)* does not know how to resolve the name requested by the client, because it may not exists on its *DNS zone*, a *Windows* client will fallback into *Multicast Name Resolution Protocols* such as *LLMNR, NBT-NS* and *mDNS*

---

#### *Recon - UNIX-like*

##### *Adidnsdump*

> ***[ADIDNSDump](https://github.com/dirkjanm/adidnsdump)***

###### *Setup*

```bash
git clone https://github.com/dirkjanm/adidnsdump ADIDNSDump
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install .
```

###### *Usage*

> ***By default, this tool exports the DNS records found to a CSV file***

```bash
python3 adidnsdump/dnsdump.py --user '<DOMAIN>\<USER>' --password '<PASSWD>' --resolve '<DC>'
```
