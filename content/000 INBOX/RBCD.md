---
Primary_category: "[[KERBEROS DELEGATIONS]]"
title: "RBCD"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY →  [[KERBEROS DELEGATIONS]]

***RBCD → Resource-based Constrained Delegation***

##### *RBCD-related Attributes*

###### *msDS-AllowedToActOnBehalfOfOtherIdentity*

This attribute was introduce with ***Windows Server 2012***

This implies that ***RBCD*** only works when the ***Domain Controller Functionality Level (DCFL)*** is Windows Server 2012 or higher

---

#### *Workflow*

![[RBCD-20250715190353765.webp|375]]

> ***Zoom In***

---

#### *RBCD*

Let us present a case with the following elements →

- ***Service A*** → *Service Account* for which *RBCD* is configured
- ***Administrator*** → *User Account* that is impersonated
- ***User X*** → *User Account* that has *GenericWrite ACE* on the *Service Account A*
- ***Service B*** → *Computer Account* that *User X* joins to the domain

To configure *RBCD* on a *service account (Service A)*, is mandatory that the attacker has control over an account (*User X*) that has an ***ACE*** such as *GenericWrite* or *GenericAll* on that *service account (Service A)*

Therefore, an attacker could use that controlled account (*User X*) to append the *SID* of another compromised *service account (Service B)* to the *msDS-AllowedToActOnBehalfOfOtherIdentity* attribute of the *Service A*

With that, the newly added *Computer Account (Service B)* can perform a *Full S4U (S4USelf + S4U2Proxy)* to request a *Service Ticket*, on behalf of the *Administrator* user, to the *Service A*

Then, the attacker can carry out a [[PASS THE TICKET|PtT]] with the *S4U2Proxy Service Ticket*

##### *Adding a Computer Account to the Domain*

> ***[Reference](https://www.thehacker.recipes/ad/movement/builtins/machineaccountquota)***

The standard workaround when an attacker does not have control over a *service account* to perform a *Full S4U*, is to leverage the *Domain-Level MachineAccountQuota (MAQ)* attribute of a controlled *user account (User X)* to add an arbitrary *computer account (Service B)* to the domain

> [!INFO]-
>
> This attribute usually allows a non-privileged *user account* to add up to 10 *computer accounts* to an *Active Directory* domain
>

- ***addcomputer.py***

> ***[addcomputer.py](https://github.com/fortra/impacket/blob/master/examples/addcomputer.py?utm_campaign=Image%20Editing%20/%20Aviary%20Launch/)***

```bash
addcomputer.py -computer-name 'SERVICE_B$' -computer-pass 'SERVICE_CPASSWD!' 'domain.local/USER_X:USER_XPASSWD!'
```

##### *Setting up RBCD on the Target Service Account*

*Microsoft* patched the ability of a *service account*  to configure *RBCD* for itself, i.e. the attacker did not need a *domain account* that had an *EACs* such as *GenericWrite* on the *service account* for which *RBCD* was to be configured

However, an attacker currently needs control over a *domain account* that has the mentioned *EACs  (GenericWrite, GenericAll, WriteProperty...)* on the *service account (Service A)*

So, we append the *SID* of the newly added *computer account (Service B)* to the *msDS-AllowedToActOnBehalfOfOtherIdentity*attribute of the *Service A* in order to perform a *Full S4U* with *Service B*

- ***rbcd.py***

> ***[rbcd.py](https://github.com/fortra/impacket/blob/master/examples/rbcd.py)***

```bash
rbcd.py -delegate-to 'SERVICE_B$' -delegate-from 'SERVICE_A' -action write 'domain.local/USER_X:USER_XPASSWD!'
```

#####  *Full S4U (S4U2Self + S4U2Proxy)*

Once *RBCD* is configured for the given *service account (Service A)*, an attacker could perform a *Full S4U* from the controlled *computer account* to request a *Service Ticket*, on behalf of the *Administrator* user, to *Service A*

The attacker can request any *Service Class* of the *Target*, i.e. any *SPN* from the *Service Account A*, such as `HOST/SERVICE_A`, `CIFS/SERVICE_A`

- ***getST.py***

> ***[getST.py](https://github.com/fortra/impacket/blob/master/examples/getST.py)***

```bash
getST.py -spn 'HTTP/SERVICE_A' -impersonate 'Administrator' -dc-ip '<DC>' 'domain.local/SERVICE_B:SERVICE_BPASSWD!'
```

---

#### *Resources*

***[The Hacker Recipes: Resource-based Constrained Delegation - RBCD](https://www.thehacker.recipes/ad/movement/kerberos/delegations/rbcd)***

***[Deep Hacking: Constrained Delegation & RBCD](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/)***

***[You do (not) Understand Kerberos Delegation](https://attl4s.github.io/)***