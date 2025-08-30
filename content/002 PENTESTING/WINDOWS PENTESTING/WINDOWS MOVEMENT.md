---
Primary_category: "[[WINDOWS PENTESTING]]"
title: "WINDOWS MOVEMENT"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WINDOWS PENTESTING]]

#### *Components* ⟡

- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[WINDOWS CREDENTIALS|CREDENTIALS]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[NTLM]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[KERBEROS]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[DACL ABUSE]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[MITM & COERCED AUTHS|MITM - COERCION]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[ADCS]]
- ![](https://media0.giphy.com/media/26xBQxJc5JzAtLx1C/giphy.gif?cid=6c09b952uz02zvkfe1bx3fxt82i5u7ul58fncsegw7tkzfrs&ep=v1_internal_gif_by_id&rid=giphy.gif&ct=g)
	- [[SCHANNEL]]

<br>

---

#### *Protection Mechanisms*

> ***Credentials and Processes***

- ![](https://i.gifer.com/embedded/download/KNT9.gif)
	- [[UAC]]
- ![](https://i.gifer.com/embedded/download/KNT9.gif)
	- [[LSA PROTECTION]]
- ![](https://i.gifer.com/embedded/download/KNT9.gif)
	- [[CREDENTIAL GUARD]]

<br>

---

#### *Lateral Movement Mitigations*

> ***Policies and Directives***

- ![](https://i.pinimg.com/originals/55/61/f9/5561f9ff936b7f4553387851f3f1a365.gif)
	- [[LocalAccountTokenFilterPolicy|UAC RESTRICTIONS]]
- ![](https://i.pinimg.com/originals/55/61/f9/5561f9ff936b7f4553387851f3f1a365.gif)
	- [[FilterAdministratorToken|ADMIN APPROVAL MODE]]
- ![](https://i.pinimg.com/originals/55/61/f9/5561f9ff936b7f4553387851f3f1a365.gif)
	- [[DisableRestrictedAdminMode|RESTRICTED ADMIN MODE]]

<br>

---

#### *Windows Authentication Process*

> ***[Reference I](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961760(v=technet.10)?redirectedfrom=MSDN)&nbsp;&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;&nbsp;[Reference II](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication)***

##### *Interactive Logon*

> ***Logon Type 2 (Interactive)***

> ***Logon Type 10 (RemoteInteractive)***

> ***SSO***

![[WINDOWS LATERAL MOVEMENT-20250605161031849.webp|450]]

> ***Zoom In***

![[WINDOWS LATERAL MOVEMENT-20250605162505687.webp|300]]

> ***Zoom In***

##### *Non-Interactive Logon*

> ***Logon Type 3 (Network)***

###### *SSPI (Security Support Provider Interface) + SSP*

> ***Credentials are not stored on the Target Server***

> ***There are exceptions such as RDP with CredSSP or [[KERBEROS DELEGATIONS|Kerberos Delegations]]***