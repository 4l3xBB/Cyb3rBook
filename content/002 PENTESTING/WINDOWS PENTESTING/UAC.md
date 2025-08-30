---
Primary_category: "[[WINDOWS MOVEMENT]]"
title: "UAC"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WINDOWS MOVEMENT]]

#### *Summary*

> [!DANGER]- *UAC Enabled*
>
> | ***USER*** | ***LOGON TYPE*** | ***ACCESS TOKEN*** |
> | --- | --- | --- |
> | ***Non RID 500 Local Admin Account*** | ***2 (Interactive) <br> 10 (RemoteInteractive)*** | ***Filtered Token <br> Full Token (Dual Token)*** |
> | ***Non RID 500 Local Admin Account*** | ***3 (Network)*** | ***Filtered Token*** |
> | ***RID 500 Admin Account*** | ***2 (Interactive) <br> 3 (Network) <br> 10 (RemoteInteractive)*** | ***Full Token*** |
> | ***Domain User member of Local Admins Group*** | ***2 (Interactive) <br> 10 (RemoteInteractive)*** | ***Filtered Token <br> Full Token (Dual Token)*** |
> | ***Domain User member of Local Admins Group*** | ***3 (Network)*** | ***Full Token***
>

> [!DANGER]- *UAC not Enabled*
>
> For all the ***Privileged Accounts***, ***LSASS.exe*** creates a ***Full Access Token***, as there is ***no UAC*** which filters that token generating a ***Filtered Access Token***
>
> The above applies regardless of the ***Logon Session Type***, whether it is ***2 (Interactive), 3 (Network)*** or ***10 (RemoteInteractive)***
>
> This ***LocalAccountTokenFilterPolicy*** does not apply if ***UAC is disabled***
>
> | ***USER*** | ***LOGON TYPE*** | ***ACCESS TOKEN*** |
> | --- | --- | --- |
> | ***Non RID 500 Local Admin Account*** | ***2 (Interactive) <br> 3 (Network) <br> 10 (RemoteInteractive)*** | ***Full Token*** |
> | ***RID 500 Admin Account*** | ***2 (Interactive) <br> 3 (Network) <br> 10 (RemoteInteractive)*** | ***Full Token*** |
> | ***Domain User member of Local Admins Group*** | ***2 (Interactive) <br> 3 (Network) <br> 10 (RemoteInteractive)*** | ***Full Token*** |
>

> [!IMPORTANT]- *Summary (UAC Enabled)*
>
> | ***USER***  | ***LOGON TYPE*** | ***POLICY*** | ***ACCESS TOKEN*** | ***Under UAC?*** |
> | --- | --- | --- | --- | :---: |
> | ***Non RID 500 Local Admin Account*** | ***2 (Interactive) <br> 10 (RemoteInteractive)*** | ***LocalAccountFilterTokenPolicy*** ❌ | ***Filtered Token <br> Full Token <br> (Dual Token)*** | 🟢 |
> | ***Non RID 500 Local Admin Account*** | ***3 (Network)*** ❌ | ***LocalAccountFilterTokenPolicy*** 🟢 | ***Filtered Token*** | 🟢 |
> | ***Non RID 500 Local Admin Account*** | ***3 (Network)*** 🟢 | ***LocalAccountFilterTokenPolicy*** ❌ | ***Full Token*** | 🟢 |
> | ***RID 500 Admin Account*** | ***2 (Interactive) <br> 3 (Network)*** 🟢 <br> ***10 (RemoteInteractive)*** | ***FilterAdministratorToken*** ❌ | ***Full Token*** | ❌ |
> | ***RID 500 Admin Account*** | ***2 (Interactive) <br> 10 (RemoteInteractive)*** | ***FilterAdministratorToken*** 🟢 | ***Filtered Token <br> Full Token <br> (Dual Token)*** | 🟢 |
> | ***RID 500 Admin Account*** | ***3 (Network)*** ❌ | ***LocalAccountFilterTokenPolicy*** 🟢 <br> ***FilterAdministratorToken*** 🟢 | ***Filtered Token*** | 🟢 |
> | ***RID 500 Admin Account*** | ***3 (Network)*** 🟢 | ***LocalAccountFilterTokenPolicy*** ❌ <br> ***FilterAdministratorToken*** 🟢 | ***Full Token*** | 🟢 |
> | ***Domain User member of Local Admins Group*** | ***2 (Interactive) <br> 10 (RemoteInteractive)*** | | ***Filtered Token <br> Full Token <br> (Dual Token)*** | 🟢 |
> | ***Domain User member of Local Admins Group*** | ***3 (Network)*** 🟢 | | ***Full Token*** | 🟢 |