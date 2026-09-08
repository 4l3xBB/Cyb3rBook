---
Primary_category: "[[MITM & COERCED AUTHS]]"
title: WPAD SPOOFING
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[MITM & COERCED AUTHS]]

#### *Theory*

##### *WPAD*

The *Web Proxy Auto-Discovery Protocol ( WPAD )* allows a web client to automatically discover a proxy configuration

To do so, the client search for a *WPAD.dat* file in the following ways →

- ***DHCP ( 252 Option )***
- ***DNS → Query name to the Primary DNS Server trying to resolve `wpad` or `wpad.domain.internal`***
- ***[[LLMNR - NBTNS - MDNS SPOOFING|ANYCAST Protocols]] as a fallback***

By default, any *Windows* host has the *Automatically detect Proxy settings* option enabled in *Internet Options*

![[WPAD SPOOFING-20260423205241872.webp|200]]

> ***Zoom in***

Therefore, a *Windows* client search for a *proxy configuration* automatically by trying to discover a *PAC* file

As stated, this discovery is carried out through ***[[53 - DNS|DNS]]***, *DHCP* or an *ANYCAST* protocol

Once the name resolution is completed and the client knows the *IP Address* of the given proxy server, it tries to download via *HTTP* a *JS PAC* file, which is a file containing all the configuration related to the proxy server

```bash
http://wpad.domain.local/wpad.dat
```

However, if an operator controls the client *DNS* resolution, either through ***[[DHCPV6 SPOOFING|DHCPv6 Spoofing]]*** or by being able to create a *WPAD DNS* record in the domain zone, and points the *WPAD* record to the *IP Address* of a machine under his control, the given client can be forced to establish a connection to the latter

Moreover, this connection requires a subsequent authentication, so an attacker could set up an *HTTP* server in order to receive the incoming authentication and ***[[NTLM CAPTURE|capture]]*** the ***[[WINDOWS CREDENTIALS CRACKING#Net-NTLMv2 Response|Net-NTLMv2]]*** hashes or perform a ***[[NTLM RELAY|NTLM Relay]]*** attack

##### *Global Query Block List*

To prevent this issue, *Microsoft* introduced a *Global DNS Query Block list* starting with *Windows Server 2008*. By default, this list contains two values →

- ***WPAD***
- ***ISATAP ( Intra-Site Automatic Tunnel Addressing Protocol ). Also vulnerable***

Therefore, any client that sends a *query name* containing *wpad.domain.internal* to the *DC* will receive an *NXDOMAIN* response indicating that the provided name does not exist, even if it really exists

This means that if an attacker manages to create a *DNS* record named *WPAD* in the domain *DNS* zone pointing to itself, it will not have any effect at all

---

#### *Components* ⟡

- ![](https://64.media.tumblr.com/51209ffa1af161126909f47a373e2c45/tumblr_oordk7Nyv91w3y4ilo1_500.gif)
	- [[DNS ADMINS#WPAD ABUSE|DNS ADMINS]]