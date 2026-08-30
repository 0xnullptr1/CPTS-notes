**Overview:**
- A subdomain is any website underlying another domain (e.g., photos.google.com is a subdomain of google.com).
- VHosts (Virtual Hosts) are subdomains served on the same server with the same IP. A single IP can serve multiple different websites.
- Key difference: subdomains have DNS records, VHosts may only exist as Host header configurations without public DNS entries.
---
## DNS Configuration

```
sudo sh -c 'echo "<target-ip> <domain>" >> /etc/hosts'
```

Adds DNS entry to resolve domain locally.

---
## Subdomain Fuzzing

```
ffuf -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt:FUZZ -u https://FUZZ.<domain>/
```

Discovers public subdomains via DNS resolution.

---
## VHost Fuzzing

```
ffuf -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u http://<domain>:<port>/ -H 'Host: FUZZ.<domain>' -fs <size>
```

Discovers virtual hosts by fuzzing the Host header. `-fs` filters responses by size to hide false positives.

```
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```

Alternative to ffuf to discover virtual hosts. -The `-u` flag specifies the target URL. The `-w` flag specifies the wordlist file. The `--append-domain` flag appends the base domain to each word in the wordlist.