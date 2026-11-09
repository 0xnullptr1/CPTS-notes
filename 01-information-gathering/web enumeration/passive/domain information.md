**Overview:**
- No direct interaction with the target. Data comes from third-party services, public records, and open sources. Start with the main website to infer technologies and infrastructure structure before touching any tool.
- SSL certificates often cover multiple domains via SAN fields, revealing additional targets outside normal DNS enumeration. Certificate Transparency logs (crt.sh) expose every certificate ever issued for a domain, including subdomains never meant to be public.
- After collecting subdomains, resolving them to IPs separates self-hosted assets from third-party providers. Self-hosted IPs are the primary attack surface.
- Cross-referencing resolved IPs with Shodan surfaces open ports, banners, and known vulnerabilities without sending a packet to the target. DNS records (MX, NS, TXT, SOA) further map mail infrastructure, name server authority, and SPF/DMARC/DKIM policies.
---
## SSL Certificate Enumeration

```bash
curl -s https://crt.sh/\?q\=<domain>\&output\=json | jq
```

Query Certificate Transparency logs via crt.sh and format the output as JSON. Reveals subdomains included in issued certificates.

---
## Subdomain Resolution

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep <domain> | cut -d" " -f1,4; done
```

Resolve each subdomain in the list and filter for entries pointing to the target domain. Identifies hosts directly reachable from the internet and separates them from third-party-hosted assets.

---
## Shodan Correlation

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep <domain> | cut -d" " -f4 >> ip-addresses.txt; done
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

Extract resolved IPs into a list, then query each against Shodan. Surfaces open ports, running services, banners, and known vulnerabilities for each internet-facing host.

---
## DNS Enumeration

```bash
dig any <domain>
```

Request all available DNS record types for the target domain. Returns A, MX, NS, TXT, and SOA records in a single query.

|Record|What it reveals|
|---|---|
|`A`|IP addresses mapped to a (sub)domain|
|`MX`|Mail servers handling email for the domain|
|`NS`|Name servers authoritative for the domain|
|`TXT`|SPF, DMARC, DKIM policies and third-party verification keys|
|`SOA`|Primary name server and zone administrative contact|
