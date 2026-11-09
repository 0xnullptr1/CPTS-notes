**Overview:**
- Starting point for engagements where no domain credentials are provided. Covers passive host identification, network poisoning, active host discovery, and Kerberos-based user enumeration.
- Passive measures (traffic capture, poisoning) collect data without generating noisy scans. Active measures (ping sweeps, nmap, kerbrute) validate and expand on those results.
- Key targets are domain controllers, file servers, SQL servers, web servers, and Exchange servers. Key services to identify are Kerberos, NetBIOS, LDAP, and DNS.
- Valid usernames collected here feed directly into password spraying in the next phase.
- Mitre ATT&CK lists this technique as [ID: T1557.001](https://attack.mitre.org/techniques/T1557/001), `Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay`.
---
## Key Data Points

|Data Point|Description|
|---|---|
|`AD Users`|Valid user accounts targeted for password spraying.|
|`AD Joined Computers`|Domain controllers, file servers, SQL servers, web servers, Exchange servers.|
|`Key Services`|Kerberos, NetBIOS, LDAP, DNS.|
|`Vulnerable Hosts`|Quick-win targets for initial foothold.|

---
## Passive Traffic Analysis

```bash
sudo tcpdump -i <interface>
```

Capture raw traffic on the interface to identify live hosts and network topology without sending any packets. Look for ARP requests and replies, MDNS, and other layer 2 traffic to map the local segment.

---
## Network Poisoning

>**Responder:**

```bash
sudo responder -I tun0
```

Poison LLMNR, NBT-NS, and MDNS queries on the interface to capture NTLMv2 hashes from hosts attempting name resolution. Run during business hours to maximize capture volume.

>**Inveigh:**

```powershell
Import-Module .\Inveigh.ps1
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

PowerShell equivalent of Responder for Windows-based attack hosts. `-NBNS Y` enables NBT-NS poisoning, `-ConsoleOutput Y` prints captures to the console, `-FileOutput Y` writes results to disk. Type `GET NTLMV2UNIQUE` in the interactive console to view deduplicated captured hashes.

---
## Host Discovery

```bash
fping -asgq <CIDR>
```

Ping sweep across the target subnet. `-a` shows only alive hosts, `-s` prints summary stats, `-g` generates the target list from the CIDR range, `-q` suppresses per-target output. Feed results into a host list for nmap.

```bash
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

Loop used on a Linux-based system to discover devices in a specified network segment.

```
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

Loop used on a Windows-based system to discover devices in a specified network segment.

---
## Host Enumeration

```
sudo nmap -v -A -iL <hosts-file> -oN <out-file>
```

Service and OS detection against discovered hosts. `-v` enables verbose output, `-A` enables OS detection, version detection, script scanning, and traceroute, `-iL` reads targets from file, `-oN` writes output in normal format. Used to identify domain controllers, web servers, and potentially vulnerable services.

---
## Kerberos User Enumeration

```
kerbrute userenum -d <domain> --dc <dc-ip> <wordlist> -o <out-file>
```

Enumerate valid domain usernames by abusing Kerberos pre-authentication. Does not trigger standard failed-login lockout policies since it operates at the Kerberos layer before authentication completes. `-d` specifies the domain, `--dc` specifies the domain controller IP, `-o` writes confirmed valid users to file.