**Overview:**
- PRTG Network Monitor is agentless network monitoring software used to monitor bandwidth, uptime, and collect statistics from routers, switches, servers, and more.
- Uses autodiscovery to scan network areas and build a device list, then gathers further data via ICMP, SNMP, WMI, NetFlow, and REST API.
- Frequently encountered during internal penetration tests.
- Versions before 18.2.39 are vulnerable to CVE-2018-9276, an authenticated command injection in the System Administrator web console.
---
## Discovery

```bash
sudo nmap -sV -p- --open -T4 <target>
```

PRTG is typically found on ports 80, 443, or 8080. Identified via Nmap as `Indy httpd (Paessler PRTG bandwidth monitor)`.

```bash
curl -s http://<target>:<port>/index.htm -A "Mozilla/5.0 (compatible; MSIE 7.01; Windows NT 5.0)" | grep version
```

Confirm version from page source.

---
## Default Credentials

Default credentials are `prtgadmin:prtgadmin`, often pre-filled on the login page and frequently left unchanged.