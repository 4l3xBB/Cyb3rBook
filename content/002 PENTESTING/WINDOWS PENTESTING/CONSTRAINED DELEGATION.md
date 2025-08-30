---
Primary_category: "[[KERBEROS DELEGATIONS]]"
title: "CONSTRAINED DELEGATION"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[KERBEROS DELEGATIONS]]

***KCD → Kerberos Constrained Delegation***

##### *KCD-Related Attributes and Flags*

###### *msDS-AllowedToDelegateTo*

> ***Object Attribute***

###### *TrustedToAuthForDelegation*

> ***Protocol Transition-related Flag/Value within the UserAccountControl Attribute***

#### *Workflow*

![[CONSTRAINED DELEGATION-20250715190320940.webp|400]]

> ***Zoom In***

---

#### *KCD with Protocol Transition*

> ***[Reference](https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained#with-protocol-transition)***

![[CONSTRAINED DELEGATION-20250715200809844.webp|250]]

> ***Zoom In***

Let us present a case with the following elements →

- ***Service A*** → Service Account with *KCD with protocol transition* enabled
- ***Administrator*** → Impersonated User Account (Victim)
- ***Service B*** → Service Account for which the ticket resulting from the ***S4U2Proxy*** is created for

##### *S4U → S4U2Self + S4U2Proxy*

Since ***Protocol Transition*** is enabled, the *service account (Service A)* will receive a ***Forwardable Service Ticket*** as a result of the ***S4U2Self***, unless the impersonated  user (Administrator) belongs to the ***Protected Users*** group or *"is sensitive and cannot be delegated"*

The *service account* with the *KCD* enabled (Service A) can present this ticket as ***additional-ticket*** to the ***KDC's TGS*** during a ***S4U2Proxy***, and will receive a ***Service Ticket*** for the requested ***SPN*** (related to Service B), which must be an ***SPN*** of the ***msDS-allowedToDelegateTo*** attribute of that *service account*


###### *getST - Impacket*

> ***[getST.py](https://github.com/paranoidninja/Pandoras-Box/blob/master/python/impacket-scripts/getST.py)***

```bash
getST.py -spn 'cifs/SERVICE_B' -impersonate Administrator 'domain.local/SERVICE_A:<PASSWD>'
```

###### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

```bash
rubeus.exe s4u /nowrap /msdsspn:'cifs/SERVICE_B' /impersonateuser:'Administrator' /user:'SERVICE_A' /domain:'domain.local' /password:'<PASSWD>'
```

#####  *Pass the Ticket*

Then, perform ***[[PASS THE TICKET|PtT]]*** with the obtained ***Service Ticket*** to the ***delegated service (AP Exchange)*** i.e. *Service B*

---

#### *KCD w/o Protocol Transition*

> ***[Reference](https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained#without-protocol-transition)***

![[CONSTRAINED DELEGATION-20250717191420395.webp|250]]

> ***Zoom In***

Let us present a case with the following elements →

- ***Service A*** → Service Account with *KCD w/o protocol transition* enabled
- ***User X*** → User Account that has *GenericWrite* on *Service A* to configure *RBCD* on it
- ***Administrator*** → Impersonated User Account (Victim)
- ***Service B*** → Service Account for which the ticket resulting from the ***S4U2Proxy*** is created for
- ***Service C*** → Computer account joined to the domain by the attacker to be able to perform *RBCD* on *Service A*

If a *service account* (service A) is configured with ***KCD w/o Protocol Transition***, the *Service Ticket* resulting from an ***S4U2Self*** will not be ***Forwardable***, because the ***TrustedToAuthForDelegation*** flag is not set in the ***UserAccountControl*** attribute of the *service account*

This *Service Ticket* also cannot be *forwardable* if the *impersonated user (Administrator)* belongs to the *protected users* group or *"is sensitive and cannot be delegated*

This means that the requirements for a successful ***S4U2Proxy*** are not met. Therefore, there is not a direct way to obtain a ***Forwardable Ticket*** from a *service account* with *KCD w/o Protocol Transition* enabled

However, there are different approaches that attackers can carry out to get a *forwardable ticket*, on behalf of a user, to the requesting service

The most feasible one, since no user interaction is needed, is by carrying out an [[RBCD]] attack to the *service account* with *KCD* enabled

##### *RBCD + KCD Kerberos Only*

The *service account* with *KCD w/o Protocol Transition (Service A)* must be configured for *RBCD*

In the end, our goal is to obtain a *Forwardable Service Ticket*, on behalf of a user (Administrator), to the *KCD-enabled Service Account (Service A)*, to use it as *additional ticket* during a *S4U2Proxy*

So, we need to add a *service account (Service C)* of our control to the *msDS-AllowedToActOnBehalfOfOtherIdentity* attribute of the *KCD-enabled Service Account  (Service A)*

In this way, an attacker can perform a ***S4U2Self + S4U2Proxy***, from the controlled *service account (Service C)*, to obtain a *forwardable service ticket* as the impersonated user (Administrator) to the *KCD-enabled Service Account (Service A)*

And then use this *forwardable ticket* on a ***S4U2Proxy*** from the *KCD service account (Service A)*, on behalf of the user *Administrator*, to the *Service B*. Resulting in a *Service Ticket* that an attacker can use to authenticate as *Administrator* to *Service B*

> [!IMPORTANT]-
>
> In order to configure *RBCD* on a *service account*, an attacker must have control over an account (User X) that has specific rights, such as *GenericWrite*, *GenericAll* or *WriteProperty* (among others), on that *service account*
>
> There is a *self-RBCD* approach where any *service account* could set up *RBCD* for itself, but *Microsoft* seems to have patched it
>

###### *Adding a Computer Account to the Domain*

If an attacker does not have control over a *service account*, but has compromised any *user account*, the attacker can leverage of the Domain-Level *MachineAccountQuota* attribute of that *user account* to join a *Computer Account (Service C)*  to the domain

- ***addcomputer.py***

> ***[addcomputer.py](https://github.com/fortra/impacket/blob/master/examples/addcomputer.py?utm_campaign=Image%20Editing%20/%20Aviary%20Launch/)***

```bash
addcomputer.py -computer-name 'SERVICE_C$' -computer-pass 'SERVICE_C$PASSWD!' 'domain.local/USER_X:USER_XPASSWD!'
```

###### *Setting Up RBCD on KCD-enabled Service Account*

Having control over an account (User X) that has *GenericWrite* on the *KCD-enabled Service Account* allows an attacker to configure *RBCD* on that *service account* by adding the newly added*computer account* to the *msDS-ActOnBehalfOfOtherIdentity* attribute

Thus, when the attacker performs a *S4U2Self + S4U2Proxy* from *Service C*, on behalf of the user *Administrator*, to the *KCD-enabled Service*, a *Forwardable Service Ticket* will be issued

- ***rbcd.py***

> ***[rbcd.py](https://github.com/fortra/impacket/blob/master/examples/rbcd.py)***

```bash
rbcd.py -delegate-to 'domain.local/SERVICE_A' -delegate-from 'SERVICE_C$' -action write 'domain.local/USER_X:USER_XPASSWD!'
```

###### *S4U2Self + S4U2Proxy (Full S4U) from the Controlled Service Account to the KCD-enabled Service Account*

> ***Controlled Service Account = Newly added Computer Account***

Once *RBCD* is configured on the *KCD-enabled Service Account (Service A)*, just perform a ***Full S4U*** from the newly added *Computer Account (Service C)*, as the *Administrator User (impersonated)*, to that *Service Account (Service A)*

The *SPN* specified during the *S4U2Proxy* can be any (e.g. `CIFS/TARGET`, `HOST/TARGET`)

- ***getST.py***

> ***[getST.py](https://github.com/fortra/impacket/blob/master/examples/getST.py)***

```bash
getST.py -spn 'HOST/SERVICE_A' -impersonate 'Administrator' -dc-ip <DC> 'domain.local/SERVICE_C$:SERVICE_CPASSWD!'
```

We will receive a *Forwardable Service Ticket* issued as *Administrator* to the *KCD-enabled Service*

###### *Additional S4U2Proxy*

The attacker uses the received *Forwardable Service Ticket* to *Service A* as *additional-ticket* on a *S4U2Proxy* to request *Service Ticket* from the *KCD-enabled Service Account (Service A)*, on behalf of *Administrator*, to the *Service B*

```bash
getST.py -spn 'HOST/SERVICE_B' -impersonate 'Administrator' -additional-ticket <SERVICE_A_ST.CCACHE> 'domain.local/SERVICE_A:SERVICE_APASSWD!'
```

A *Service Ticket* to *Service B*, in a *CCACHE* format, is obtained

###### *Optional: Modifying Service Ticket's SNAME*

> ***anySPN***

> ***[Reference](https://www.thehacker.recipes/ad/movement/kerberos/ptt#modifying-the-spn)***

Unlike the ***PAC***, the *SNAME* field of a *Service Ticket* is not in the *encrypted* section of the ticket. This information is basically related to the requested *SPN* by the client

Therefore, an attacker could tamper it modifying its value to indicate another *Service Class* for the *Service Ticket* resulting from the ***S4U2Proxy***

> ***Related option***  → **`-altservice`**

```bash
getST.py -spn 'HOST/SERVICE_B' -altservice 'CIFS/SERVICE_B' -impersonate 'Administrator' -additional-ticket <SERVICE_A_ST.CCACHE> 'domain.local/SERVICE_A:SERVICE_APASSWD!'
```

######  *Pass the Ticket*

Then, perform ***[[PASS THE TICKET|PtT]]*** with the obtained ***Service Ticket*** to the ***delegated service (AP Exchange)*** i.e. *Service B*

---

#### *Resources*

***[The Hacker Recipes: Constrained Delegation](https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained)***

***[Deep Hacking: Constrained Delegation & RBCD](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/)***

***[You do (not) Understand Kerberos Delegation](https://attl4s.github.io/)***