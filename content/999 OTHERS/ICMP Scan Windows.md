**Discover Active or Up Host on Network through ICMP Sweep using `Ping`**
### CMD

**`for /L %i in (1,1,254) do @ping /n 1 /w 200 192.168.1.%i > NUL && echo 192.168.1.%i - ACTIVE HOST`**

**On CMD is not possible to implemente Parallelism to Boost Scan Speed**
**Powershell and Linux (Bash) supports Parallelism and Threads**

- **Powershell -> Through `Workflow workflow_name {} ; workflow_name`**
- **Linux (Bash) -> Through `&`**

### Powershell

##### Without `Workflow`

**`1..254 | % {if(ping -n 1 -w 200 192.168.1.$_ | Select-String -Pattern 'TTL'){"192.168.1.$($_) - ACTIVE HOST"}}`**

##### With `Workflow`

**`1..6 | % {""}; Workflow ICMPSweep {foreach -Parallel -ThrottleLimit 8 ($i in 1..254){if(ping -n 1 -w 200 192.168.1.$i | Select-String -Pattern 'TTL'){"192.168.1.$($i) - ACTIVE HOST"}}}; ICMPSweep`**
