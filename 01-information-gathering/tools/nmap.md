**Overview:**
- Nmap is an open-source network analysis and security auditing tool written in C, C++, Python, and Lua. It uses raw packets to identify live hosts, open ports, running services and versions, and operating systems.
- Common use cases: network mapping, firewall and IDS/IPS auditing, vulnerability assessment, penetration test simulation, and response analysis.
- Nmap offers multiple firewall and IDS/IPS evasion techniques including packet fragmentation, decoys, source IP spoofing, and source port manipulation.
---
## Syntax

```bash
nmap <scan-types> <options> <target>
```

---
## Scanning Options

| Option               | Description                                     |
| -------------------- | ----------------------------------------------- |
| `10.10.10.0/24`      | Target network range                            |
| `-sn`                | Disable port scanning                           |
| `-Pn`                | Disable ICMP Echo Requests                      |
| `-n`                 | Disable DNS resolution                          |
| `-PE`                | ICMP Echo Request ping scan                     |
| `--packet-trace`     | Show all packets sent and received              |
| `--reason`           | Display reason for each result                  |
| `--disable-arp-ping` | Disable ARP ping requests                       |
| `--top-ports=<num>`  | Scan the most frequent top ports                |
| `-p-`                | Scan all ports                                  |
| `-p22-110`           | Scan port range 22 to 110                       |
| `-p22,25`            | Scan specific ports                             |
| `-F`                 | Scan top 100 ports                              |
| `-sS`                | TCP SYN scan                                    |
| `-sT`                | TCP connect scan (full three-way handshake)     |
| `-sA`                | TCP ACK scan                                    |
| `-sU`                | UDP scan                                        |
| `-sV`                | Service version detection                       |
| `-sC`                | Default script scan                             |
| `--script <script>`  | Run specified NSE script or category            |
| `-O`                 | OS detection                                    |
| `-A`                 | OS detection, service detection, and traceroute |
| `-D RND:5`           | Use 5 random decoys                             |
| `-e`                 | Specify network interface                       |
| `-S <ip>`            | Spoof source IP address                         |
| `-g`                 | Specify source port                             |
| `--source-port`      | Scan from specified source port                 |
| `--dns-server <ns>`  | Use specified DNS server for resolution         |
| `-iL`                | Read targets from file                          |

---
## NSE Script Categories

|Category|Description|
|---|---|
|`auth`|Authentication credential determination|
|`broadcast`|Host discovery via broadcast|
|`brute`|Brute-force credential attacks against services|
|`default`|Default scripts run with `-sC`|
|`discovery`|Accessible service evaluation|
|`dos`|Denial of service vulnerability checks|
|`exploit`|Exploit known vulnerabilities on scanned ports|
|`external`|Scripts using external services for processing|
|`fuzzer`|Identify vulnerabilities via unexpected packet fields|
|`intrusive`|Scripts that may negatively affect the target|
|`malware`|Check for malware infection on the target|
|`safe`|Non-intrusive, non-destructive scripts|
|`version`|Service detection extension|
|`vuln`|Specific vulnerability identification|

---
## Performance Options

|Option|Description|
|---|---|
|`--max-retries <num>`|Max retries per port|
|`--stats-every=5s`|Print scan status every 5 seconds|
|`-v / -vv`|Verbose output|
|`--initial-rtt-timeout 50ms`|Initial RTT timeout|
|`--max-rtt-timeout 100ms`|Maximum RTT timeout|
|`--min-rate 300`|Minimum packets sent simultaneously|
|`-T <0-5>`|Timing template (0=paranoid, 5=insane)|

---
## Output Options

|Option|Description|
|---|---|
|`-oA <file>`|Save in all formats|
|`-oN <file>`|Save in normal format|
|`-oG <file>`|Save in grepable format|
|`-oX <file>`|Save in XML format|

```bash
xsltproc target.xml -o target.html
```

Convert XML output to HTML for browser viewing.

---
## Firewall and IDS/IPS Evasion

 - Firewalls inspect and filter packets based on configured rules. 
-  IDS scans for attack patterns and reports them. 
 - IPS extends IDS by actively blocking detected threats based on signature matching.
- The TCP ACK scan (`-sA`) is harder to filter than SYN or connect scans because it sends only the ACK flag, which many firewalls and IDS/IPS treat as established-connection traffic.

```bash
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

SYN scan using random decoys to obscure the real source. Decoys work with SYN, ACK, ICMP, and OS detection scans.

```bash
sudo nmap <target> -n -Pn -p <port> -O -S <spoofed-ip> -e tun0
```

Example:

```bash
sudo nmap 192.0.2.80 -n -Pn -p 80 -O -S 203.0.113.99 -e tun0
```

OS detection scan using a spoofed source IP (Nmap uses this address as the apparent source of the scan packets, instead of the real attack host). `-S` sets the source IP, `-e` specifies the interface.

---
## DNS Proxying

 - DNS servers are often more trusted by firewalls than arbitrary external hosts. Using `TCP port 53` as a source port can cause firewall rules and IDS/IPS filters to treat packets as legitimate DNS traffic and pass them through.

```bash
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace
```

SYN scan against a filtered port to confirm its state.

```bash
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

Repeat using source port `53` to bypass firewall rules trusting DNS traffic.

```bash
nc -nv --source-port 53 <target> <port>
```

Connect to the filtered port using source port `53` once the firewall is confirmed to accept it.


