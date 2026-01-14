---
Primary_category: "[[WINDOWS PRIVESC]]"
title: "PSCREDENTIAL OBJECT"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS PRIVESC]]

#### *Theory*

---

#### *Abuse*

Imagine we achieved to access the remote system by leveraging an existing flaw on the web application which allowed us to send a *reverse shell* to our *TCP* listener

Once we are in, the first occurrence is to check the privileges associated with the current *access token* as we have a shell as the *service account* running the *web application*, which is likely to have the *seImpersonatePrivilege*

```bash
whoami /priv
```

But this time there is no luck 💀

The current user does not have any sensitive privileges and is not member of any interesting group

So we continue searching throughout the system to see if we anything can help us to privesc, until we find an *XML* file that contains a *PSCredential object*

This file appears to be the result of running the *Export-CLIXML* cmdlet, which exports a *PSCredential* object to an *XML file*

Therefore, whenever we deal with an *XML* file resulting from a *Powershell CLI-XML* operation, we can proceed as follows in order to obtain the plain password from the *PSCredential* object

###### *Importing the PSCredential Object from CLI-XML file into a PS Variable*

> ***[Import-CLIXML](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-clixml?view=powershell-7.5)***

```bash
$cred = Import-CLIXML '<XML_FILE>'
```

###### *Extracting the Plain Password from the PSCredential Object*

> ***[GetNetworkCredential](https://learn.microsoft.com/es-es/dotnet/api/system.management.automation.pscredential.getnetworkcredential?view=powershellsdk-7.4.0)***

```bash
$cred.GetNetworkCredential().password
```