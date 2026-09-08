---
Primary_category: "[[WINDOWS RECONAISSANCE]]"
title: "WINDOWS PASSWORD POLICY"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS RECONAISSANCE]]

#### *Theory*

When assesing an *Active Directory* environment, starting from an anonymous position, there is a point where an operator is able to glean several domain user accounts either by *[[OSINT]]* or by performing an exhaustive user enumeration with *[kerbrute](https://github.com/ropnop/kerbrute)* and a wordlist such as one from *[statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames)*

At that point, one *TTP* to take into account is the *Password Spraying*, where an adversary tries several logon attempts against a certain service providing a user list and one password

Before proceeding with it, the *domain password policy* must be checked in order to avoid locking out any domain user account

Any authenticated domain user can list this information. However, there are situations where an operator can list this data through an *SMB Null Authentication* or *LDAP Anonymous Bind*

###### *SMB*

> ***Null Authentication***

```bash
nxc smb <TARGET> --username '' --password '' --pass-pol
```

###### *RPC*

```bash
rpcclient --user '' --no-pass --command 'getdompwinfo' <TARGET>
```

###### *LDAP*

> ***Anonymous Bind***

```bash
ldapsearch -x -H 'ldap://<TARGET>' -b 'DC=<DOMAIN>,DC=<TLD>' | grep -m 1 -B 10 pwdHistoryLength
```

---

#### *Enumeration - Linux*

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb <TARGET> --username '<USER>' --password '<PASSWD>' --pass-pol
```

##### *RPCclient*

```bash
rpcclient --user '<USER>%<PASSWD>' --command 'getdompwinfo' <TARGET>
```

##### *Ldapsearch*

```bash
ldapsearch -x -H 'ldap://<TARGET>' -D '<USER>@<DOMAIN>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' | grep -m 1 -B 10 pwdHistoryLength
```

---

#### *Enumeration - Windows*

##### *Net Command*

```bash
net accounts
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

```bash
Get-DomainPolicy
```