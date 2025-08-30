---
Primary_category: "[[KERBEROS]]"
title: OVERPASS THE HASH
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[KERBEROS]]

#### *Theory*

> ***[Reference](https://github.com/GhostPack/Rubeus?tab=readme-ov-file#example-over-pass-the-hash)***

This technique only makes use of the ***RC4_HMAC or NTLM Hash***

It converts a ***NTLM Hash*** derived from a ***principal's password*** into a valid ***Ticket Granting Ticket (TGT)***

---

#### *OverPassing The Hash*

##### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***Privileges needed as the `sekurlsa` module is used (LSASS.exe Memory Manipulation)***

It creates a ***New Logon Type 9 (NewCredentials)*** with dummy credentials (Another ***LUID***) to not interfere with the current ***Logon Session***

> ***An Active Logon Session can only have ONE Ticket Granting Ticket  (TGT)***

Then, opens the ***LSASS.exe*** to write to its memory and injects the provided ***RC4_HMAC Hash*** into the appropiate section of the created ***Logon Session***

```bash
mimikatz.exe 'privilege::debug' 'token::elevate' 'sekurlsa::pth /user:<USERNAME> /domain:<DOMAIN> /rc4:<NTLM_HASH> /run:<COMMAND>' exit # By default → cmd.exe
```

This causes the normal Kerberos authentication process to kick off as normal as if the user had normally logged on, turning the supplied hash into a fully-fledged TGT

> ***The `sekurlsa::pth` method performs both OverPass the Hash/Pass the Key and Pass the Ticket Techniques***

##### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

> ***[Reference](https://github.com/GhostPack/Rubeus?tab=readme-ov-file#asktgt)***

It builds a ***Raw AS_REQ*** for the specified ***Kerberos Principal*** and ***Encryption Key (RC4_HMAC)***

If the ***Kerberos Authentication*** is sucessfull, the resulting ***AS_REP*** is parsed and the ***TGT*** is extracted as a ***Base64 Blob***

```bash
rubeus.exe asktgt /user:<USERNAME> /rc4:<NTLM_HASH> /nowrap
```