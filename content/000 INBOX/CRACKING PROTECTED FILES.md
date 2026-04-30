---
Primary_category: "[[CRACKING]]"
title: "CRACKING PROTECTED FILES"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[CRACKING]]

#### *General Workflow*

##### *Search for the Utility*

> ***Multiple "2John" Tools***

```bash
locate *john* | grep -i -- '<FILE_TYPE>'
```

##### *Obtain a Hash from the Provided File*

```bash
<FILE>2john <FILE> > <FILE>.john
```

##### *Crack the Hash with John*

```bash
john --wordlist=<WORDLIST> <FILE>.john
```

##### *Show the obtained Password*

```bash
john --show <FILE>.john
```

```bash
cat ~/.john/john.pot
```

---

#### *ZIP*

##### *Show .ZIP File Technical Metadata and Other information*

```bash
7z l -slt <ZIP_FILE>
```

##### *Obtain a Hash/Digest from the Zip File*

```bash
zip2john <ZIP_FILE> > zip.john
```

##### *Hash Cracking with John*

```bash
john zip.john --wordlist=/usr/share/wordlists/rockyou.txt
```

##### *Show Cracked Hashes/Passwords*

```bash
john --show zip.john
```

```bash
cat ~/.john/john.pot
```

---

#### *GZIP*

##### *Encrypted with OpenSSL*

###### *Check whether the file is encrypted or not*

```bash
file <GZIP_FILE>
```

If encrypted, the output should be similar to the following one →

```bash
GZIP.gzip: openssl enc'd data with salted password
```

###### *Cracking with OpenSSL*

```bash
while IFS= read -r _passwd ; do openssl enc -aes-256-cbc -d -in <GZIP_FILE> -k "$_passwd" 2> /dev/null | tar xz ; done < <WORDLIST>
```

---

#### *BitLocker Encrypted Drives*

> ***[Reference](https://openwall.info/wiki/john/OpenCL-BitLocker)***

##### *Obtain the First Hash (Bitlocker Password) from the Encrypted Virtual Drive*

```bash
bitlocker2john -i Private.vhd 2> /dev/null | grep -i -- '\$bitlocker\$0' > bitlocker.hash # .VHD[X] File
```

> [!DANGER]- *Important*
>
> The output of the above command returns four hashes
>
> The ***first two*** correspond to the ***Bitlocker Password***
>
> The ***remaining two*** are related to the ***Bitlocker Recovery Key***
>
> Since this ***Recovery key*** is very long and randomly generated, It is generally not practial to guess
>

##### *Cracking Bitlocker Hash*

- ***Hashcat***

> ***Hashcat Mode → 22100***

```bash
hashcat --force -O --attack-mode 0 --hash-type 22100 <HASH> <WORDLIST>
```

- ***John the Ripper***

```bash
john --wordlist=<WORDLIST> --format=bitlocker <HASH>
```

##### *Mounting Bitlocker-Encrypted Drives in Windows*

> ***[[VHD - VHDX#Mounting Bitlocker-Encrypted VHD on Windows|Reference]]***

###### *Mount the .VHD File*

![[PASSWORD ATTACKS-20250604174322038.webp|300]]

> ***Zoom In***

###### *Enter the cracked password at the Bitlocker Password Prompt*

![[PASSWORD ATTACKS-20250604174956487.webp|200]]

> ***Zoom In***

![[PASSWORD ATTACKS-20250604175113530.webp|250]]

> ***Zoom In***

##### *Mounting Bitlocker-Encrypted Drives in Linux*

> ***[[VHD - VHDX#Mounting Bitlocker-Encrypted VHD on Linux|Reference]]***

| ***UTILITY*** | ***PURPOSE*** |
| --- | --- |
| **`losetup`** | ***Convert a file (.VHD, .ISO, .IMG...) into a block device*** |
| **`dislocker`** | ***Decrypt and access an encrypted volume with Bitlocker*** |
| **`mount`** | ***Mount the decrypted file system to access all the archives*** |

###### *Dislocker Installation*

> ***[Dislocker](https://github.com/Aorimn/dislocker)***

```bash
apt install -y -- dislocker
```

###### *Loop Device Creation based on the VHD File using losetup*

> ***[Losetup](https://github.com/util-linux/util-linux/blob/master/sys-utils/losetup.c)***

```bash
losetup --find --show --partscan -- <VHD>
```

###### *Check if the created Loop Device is available*

```bash
losetup --all
```

```bash
lsblk -fm | grep -i -- loop
```

###### *Folders Creation to mount the VHD File*

```bash
mkdir -p -- /media/{bitlocker,bitlockermount}
```

###### *Drive Decryption using Dislocker*

```bash
dislocker --volume /dev/loop0p1 --user-password -- /media/bitlocker
> Enter the user password: *****
```

###### *Check the Mounted Device (VHD)*

```bash
mount | grep -i -- dislocker
```

###### *Mount the Decrypted Volume*

```bash
mount --options loop -- /media/bitlocker/dislocker-file /media/bitlockermount
```

```bash
find /media/bitlockermount
```

---

#### *Resources*

> ***[Fileinfo.com](https://fileinfo.com/filetypes/encoded)***