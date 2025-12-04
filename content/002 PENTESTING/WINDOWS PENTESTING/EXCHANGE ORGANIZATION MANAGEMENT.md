---
Primary_category: "[[EXCHANGE GROUPS]]"
title: "EXCHANGE ORGANIZATION MANAGEMENT"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[EXCHANGE GROUPS]]

#### *Theory*

The ***Exchange Organization Management*** group is another powerful and privileged group within the *AD Enviroment*

Any domain account belonging to this group can access the *mailboxes* of any domain user account

Furthermore, this *Exchange* group has *FullControl (i.e. GenericAll)* of the *Organizational Unit (OU)* called *Microsoft Exchange Security Groups*, containing the *[[EXCHANGE WINDOWS PERMISSIONS|Exchange Windows Permissions]]* group

Therefore, members of this group can takeover the entire domain by performing the same *TTPs* as with the latter

> ***See [[EXCHANGE WINDOWS PERMISSIONS#Abuse - Windows|here]] or [[EXCHANGE WINDOWS PERMISSIONS#Abuse - UNIX-like|here]]***