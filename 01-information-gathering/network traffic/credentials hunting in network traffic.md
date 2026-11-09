**Overview:**
- Legacy systems, misconfigured services, and internal test environments frequently use unencrypted protocols. These expose credentials in cleartext to anyone with network access or a captured pcap.
- Wireshark is the standard tool for manual traffic analysis. Pcredz automates credential extraction from both live interfaces and pcap files, covering NTLM, Kerberos, FTP, HTTP, SNMP, and mail protocols.
- Focus on POST requests over HTTP, SNMP community strings, and mail protocols first. These are the most common sources of cleartext credentials in modern environments.

|Unencrypted Protocol|Encrypted Counterpart|
|---|---|
|HTTP|HTTPS|
|FTP|FTPS / SFTP|
|SNMP|SNMPv3 (with encryption)|
|POP3|POP3S|
|IMAP|IMAPS|
|SMTP|SMTPS|
|LDAP|LDAPS|
|RDP|RDP with TLS|
|DNS|DNS over HTTPS (DoH)|
|SMB|SMB over TLS (SMB 3.0)|
|VNC|VNC with TLS/SSL|

---
## Wireshark Filters

|Filter|Description|
|---|---|
|`ip.addr == <ip>`|All traffic to or from a specific IP|
|`ip.src == <ip> && ip.dst == <ip>`|Traffic between two specific hosts|
|`tcp.port == 80`|Filter by port|
|`eth.addr == <mac>`|Filter by MAC address|
|`http`|All HTTP traffic|
|`http.request.method == "POST"`|HTTP POST requests. May contain credentials if sent over plaintext HTTP|
|`http contains "passw"`|HTTP packets containing the string `passw`|
|`dns`|DNS traffic|
|`icmp`|ICMP traffic|
|`tcp.flags.syn == 1 && tcp.flags.ack == 0`|SYN packets. Useful for detecting scans or connection attempts|
|`tcp.stream eq <n>`|Isolate a specific TCP stream between two hosts|

To search for strings manually: `Edit > Find Packet` → search for `passw` or similar keywords across packet bytes or strings.

---
## Pcredz

```bash
./Pcredz -f <file>.pcapng -t -v
```

Extracts credentials from a pcap file. `-t` enables credit card scanning, `-v` enables verbose output. Supports: FTP, HTTP Basic/NTLM/Forms, POP/IMAP/SMTP, SNMP community strings, NTLMv1/v2 (SMB, LDAP, MSSQL, HTTP, DCE-RPC), and Kerberos AS-REQ hashes.