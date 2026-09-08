---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: WINDOWS TRUSTS
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]

#### *Theory*

##### *Trust Types*

##### *Transitivity*

##### *SID Filtering*

##### *SID History*

This *AD* attribute comes into play when a domain user account or group is migrated to one domain or forest to another. Therefore, it supports both migration scenarios and allows such domain objects to retain access to certain resources after being moved by mapping their old *Security Identifier (SID)* to the *SIDHistory* attribute of the given object in the new domain or forest

That is, if a user is migrated to another domain, a new account is created on it. Then, the original user's *SID* will be added to the *SIDHistory* attribute of the new account, ensuring that this account can still access resources in the original domain

This attribute is intended to work across domains but can work within the same domain

---

#### *Components* ⟡

> ***Trust Flavor***

- ![](https://i.gifer.com/HsWw.gif)
	- [[WINDOWS TRUSTS: CHILD → PARENT|CHILD → PARENT]]
- ![](https://i.gifer.com/HsWw.gif)
	- [[WINDOWS TRUSTS: CROSS-FOREST|FOREST]]

<br>

---

#### *Components* ⟡

> ***Trust Attack***

- ![](https://64.media.tumblr.com/51209ffa1af161126909f47a373e2c45/tumblr_oordk7Nyv91w3y4ilo1_500.gif)
	- [[EXTRASIDS]]

<br>

---
#### *Enumeration - UNIX-like*

##### *LDAPSearch*

> ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

```bash
ldapsearch -LLL -x -H 'ldap://<DC>' -D '<USER>' -w '<PASSWD>' -b 'DC=<DOMAIN>,DC=<TLD>' '(objectClass=trustedDomain)'
```

##### *LDAPDomaindump*

> ***[LDAPDomaindump](https://github.com/dirkjanm/ldapdomaindump)***

###### *Setup*

```bash
git clone "https://github.com/dirkjanm/ldapdomaindump" ldapdomaindump
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install -r requirements.txt
```

###### *Usage*

```bash
python3 ldapdomaindump.py --user '<DOMAIN>\<USER>' --password '<PASSWD>' --no-json --no-grep '<DC>'
```

##### *BloodHound.py*

> ***[BloodHound.py](https://github.com/dirkjanm/BloodHound.py)***

###### *Setup*

```bash
git clone "https://github.com/dirkjanm/BloodHound.py" BH.py
cd !$ && git checkout bloodhound-ce
python3 -m venv .venv
. !$/bin/activate && pip3 install .
```

###### *Usage*

```bash
python3 bloodhound.py --collectionmethod All --domain '<DOMAIN>' --username '<USER>' --password '<PASSWD>' --zip --nameserver '<DC>' --domain-controller '<DC_FQDN>'
```

###### *Domain | Forest Trusts Edges on BloodHound-CE*

![[WINDOWS TRUSTS-20251106203312499.webp|350]]

> ***Zoom in***

---

#### *Enumeration - Windows*

##### *Powershell AD Module*

> ***[Powershell AD Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

> ***Get-ADTrust***

```powershell
Get-ADTrust -Filter *
```

##### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

###### *Getting all Trusts for the current Domain*

> ***Get-DomainTrust***

```powershell
Get-DomainTrust
```

###### *Getting all Trusts for the current Forest*

> ***Get-ForestTrust***

```powershell
Get-ForestTrust
```

###### *Enumerating Users who are in Groups outside of their Principal Domain*

> ***Get-DomainForeignUser***

```powershell
Get-DomainForeignUser
```

###### *Building a relational mapping of all domain trusts*

> ***Get-DomainTrustMapping***

```powershell
Get-DomainTrustMapping
```

##### *Netdom*

> ***[Netdom](https://learn.microsoft.com/es-es/windows-server/administration/windows-commands/netdom)***

###### *Querying Domain | Forest Trusts*

```bash
netdom query /domain:'<DOMAIN>' trust
```

---

#### *References*

***[A Guide to Attacking Domain Trusts](https://harmj0y.medium.com/a-guide-to-attacking-domain-trusts-ef5f8992bb9d)***