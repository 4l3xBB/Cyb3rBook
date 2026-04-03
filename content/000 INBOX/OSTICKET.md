---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: "OSTICKET"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Discovery | Footprinting | Enumeration*

##### *Agent Login Panel*

```bash
<URL>/scp/login.php
```

##### *Detecting an OSTicket Site*

###### *Websiteiiiiiiii Footer*

Almost, most of the *OSTicket* installations will showcase the *OSTicket* logo followed by the phrase *Powered by* in the page's footer

![[OSTICKET-20260330175006004.webp|350]]

> ***Zoom in***

We can send an *HTTP* request as well and filter the response as follows

```bash
curl --silent --location --request GET '<URL>' | grep -i --color -- 'osticket'
```

---

#### *Attacking OSTicket*

##### *Abusing Temporary Email Accounts assigned to Tickets*

> ***[Reference](https://medium.com/intigriti/how-i-hacked-hundreds-of-companies-through-their-helpdesk-b7680ddc2d4c)***

Let's suppose we find an exposed service such as a company's *Slack* or *Gitlab*, which requires a valid company email in order to register a new account and join the organization

On the other hand, we discover another target that is running an *OSTicket* web application where users can sign up and log in to create any type of ticket related to an inquiry

![[OSTICKET-20260330190457352.webp|350]]

> ***Zoom in***

After creating an account on the support platform and log in, we realize that a temporary email account is assigned to the new ticket

![[OSTICKET-20260330190510110.webp|350]]

> ***Zoom in***

We can enter the *Ticket ID* to check its status and all the information related to it. This time the company set up their helpdesk system *( OSTicket )* to correlate ticket numbers with emails, so any email sent to this account will appear in the menu below

![[OSTICKET-20260330190727125.webp|350]]

> ***Zoom in***

Therefore, we could access to the *Gitlab* instance's login page and select the *Register now* option as we do not have valid credentials for an existing account

![[OSTICKET-20260330191720236.webp|350]]

> ***Zoom in***

However, we can sign up using the email account associated with the ticket

![[OSTICKET-20260330191839306.webp|350]]

> ***Zoom in***

Once we are logged in, we may uncover certain repositories which only accounts of the given company have access to