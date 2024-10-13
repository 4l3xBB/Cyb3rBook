---
Primary_category: "[[SETUP]]"
title: SESSION MANAGEMENT
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - CustomEnvironment🦜
cssclasses:
---

###### PRIMARY CATEGORY → [[SETUP]]

During the installation of the *[[SETUP#Components ⟡|Setup Components]]*, the user will have to be shutting down, restarting, logging out or simply blocking (i.e _lock_) the current session

##### Actions

This actions can be performed as follows →

###### Shut Down

```bash
$ sudo poweroff
```

###### Restart

```bash
$ sudo reboot
```

###### Log out

```bash
$ loginctl list-sessions # Get the Session's Number
$ loginctl terminate-session <session_number> # Finish a Session
```

> [!INFO]-
>
> If the above command is not available because of the system has not been initialized with _systemd_, use the following one →
> 
> ```bash
> $ pkill -SIGTERM --euid "$( id -u )" # Or pkill -15
> ```
>
> This allows to all the processes related to the _EUID_ to terminate corrrectly and clean up resources
>
> If any process does not respond to the `SIGTERM` signal, just sent a `KILL` signal to it 
>
> ```bash
> $ pkill -KILL --euid "$( id -u )" # Or pkill -9
> ```
>
> Note that all signal types and their associated numbers can be enumerated as follows through these [[Shell Builtins|shell builtins]] →
>
> ```bash
> $ kill -l # Or trap -l
> ```
>

###### Session Lock

It can be handled through the _X Session Manager_ called `lightDM`

To interact with the `lightdm` daemon, the `dm-tool` binary comes into actions

```bash
$ dm-tool switch-to-greeter # Like the Change-User option in Windows
```

```bash
$ dm-tool lock # Like Windows + L in Windows
```

> [!INFO]-
>
> If `lightdm` does not manages the user sessions and the host has been booted with systemd (i.e. ***PID 1***), then try this →
>
> ```bash
> $ loginctl list-sessions
> $ loginctl lock-session SESSION_NUMBER
> ```
>

Alternatively, components such as *[[I3LOCK-FANCY|I3Lock-Fancy]]* can also be used to perform a *Screen Lock*