---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: DisableRestrictedAdminMode
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses: 
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]


#### *Restricted Admin Mode*

#### *Summary*

> ***[Reference](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn283323(v=ws.11)#restrictedadmin-mode-remote-desktop)***

> ***DisableRestrictedAdmin***

> [!TLDR]- *TL;DR*
> ***Credentials are not stored in the Remote Machine (LSASS.exe's memory) when authenticating via RDP (Logon Type 10 - RemoteInteractive) → Credential Delegation***
>

***Disabled by default***

***Default Value → Value not created***

It prevents the ***Credential Theft*** not using the ***Standard Authentication Method*** carried out by ***LSASS.exe*** on the remote machine

I.e., when authenticating via ***RDP (Logon Type 10 - RemoteInteractive)***, the credentials and sensitive information, such as ***NTLM Hashes, Kerberos Keys, Kerberos Tickets (TGTs and TGSs), Kerberos Session Token...***, are stored in the memory space of the ***LSASS.exe*** process

Therefore, if an attacker has compromised that remote machine, he could extract this sensitive information by dumping the ***LSASS.exe***'s memory space or by ***reading/writing*** to it

This could allow ***pivoting*** or ***lateral movement*** using techniques such as ***[[#Pass the Hash (PtH)|PtH]]***, ***[[#Pass the Key (PtK)|PtK]]***, ***[[#OverPass the Hash|OverPass the Hash]]*** or ***[[#Pass the Ticket (PtT)|PtT]]***

So, this directive avoids storing credentials on the remote system when there is a ***RemoteInteractive Authentication (Logon Type 10)***

> [!DANGER]- *Important*
>
> if the ***Restricted Admin Mode*** is enabled on the remote host, an ***RDP*** client can only establish a remote connection if →
>
> ❶ The ***User's Logon Session*** on the client already holds ***valid credentials (NTLM Hash, Kerberos Tickets...)***
>
> ❷ The user is member of the ***Administrators Group*** on the target machine
>

---

#### *Check the Value of DisableRestrictedAdmin*

##### *Reg Query*

```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ | findstr /I 'DisableRestrictedAdmin'
```

##### *Get-ItemProperty*

```bash
(Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa").DisableRestrictedAdmin
```

---

#### *Disable DisableRestrictedAdmin*

##### *Reg Add*

```bash
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 1 /f
```

##### *Set-ItemProperty*

```bash
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "DisableRestrictedAdmin" -Value 1 -Force
```