**Check Open Ports on Remote Systems without using `Nmap`**
### Check TCP Open Port (Single Port - Single Host)

- **/dev/tcp**

	**`echo '' > /dev/tcp/ip_address/port &>/dev/null && echo "[*] OPEN PORT"`**
	<br>
- **Netcat - NC**

	**`nc -nz ip_address port && echo "[*] OPEN PORT"`**
### Check TCP Open Port (Multiple Ports - Single Host)

#### One-Liners

- **/dev/tcp**

	**`for port in $(seq 1 65535); do (timeout 1 bash -c "echo '' > /dev/tcp/192.168.1.1/${port}" &> /dev/null && echo "[*] OPEN PORT - ${port}" &); done`**
	<br>
- **Netcat - NC**

	**`for port in $(seq 1 65535); do (timeout 1 bash -c "nc -nz 192.168.1.1 ${port}" &> /dev/null && echo "[*] OPEN PORT - ${port}" &); done`**
	
### Check TCP Open Port (Multiple Ports - Multiple Hosts)

#### One-Liners

**This OneLiners, first, discover Active Hosts and then Start Port Scan**

- **/dev/tcp**

	**`readarray -t ips < <(for i in $(seq 1 254); do (timeout 1 bash -c "ping -c1 192.168.1.${i}" &> /dev/null && echo "192.168.1.${i}" &); done); for ip in "${ips[@]}"; do for port in $(seq 1 65535); do (timeout 1 bash -c "echo '' > /dev/tcp/${ip}/${port}" &> /dev/null && echo "[+] HOST ${ip} - ${port} OPEN" &); done; done`**
	<br>
- **Netcat**

	**`for i in $(seq 1 254); do (timeout 1 bash -c "ping -c1 192.168.1.${i}" &> /dev/null && echo "192.168.1.${i}" &); done | sort -u | while read ip; do for port in $(seq 1 65535); do (timeout 1 bash -c "nc -nz ${ip} ${port}" &> /dev/null && echo "[+] HOST ${ip}" - PORT ${port} &); done; done`**
	<br>
	**Note -> Define array with `readarray` or `mapfile` and store active hosts is faster than use `|` to send `for` command's stdout to `while read` command's stdin**
