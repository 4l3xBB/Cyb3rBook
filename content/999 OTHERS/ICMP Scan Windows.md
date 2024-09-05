---
Primary_category: ""
title: ""
draft: true
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags: 
cssclasses:
---

**Discover Active Host in Networks through ICMP Sweep using `Ping`**

#### CMD

```bash
for /L %i in (1,1,254) do @ping /n 1 /w 200 192.168.1.%i > NUL && echo 192.168.1.%i - ACTIVE HOST
```

**On CMD is not possible to implemente Parallelism to Boost Scan Speed**

**Both Powershell and [[SHELL SCRIPTING|Shell Scripting]] supports Parallelism and Threads**

- **Powershell**. As follows →

```powershell
Workflow workflow_name {} ; workflow_name
```

- **Shell Scripting** → Through `&`

#### Powershell

##### Without Workflow

```powershell
1..254 | % {if(ping -n 1 -w 200 192.168.1.$_ | Select-String -Pattern 'TTL'){"192.168.1.$($_) - ACTIVE HOST"}}
```

##### With Workflow

```powershell
1..6 | % {""}; Workflow ICMPSweep {foreach -Parallel -ThrottleLimit 8 ($i in 1..254){if(ping -n 1 -w 200 192.168.1.$i | Select-String -Pattern 'TTL'){"192.168.1.$($i) - ACTIVE HOST"}}}; ICMPSweep
```