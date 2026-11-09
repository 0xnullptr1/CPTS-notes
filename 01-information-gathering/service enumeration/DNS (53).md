**Overview:**
- DNS resolves domain names to IP addresses using a globally distributed hierarchy of servers, with no central database. Queries flow from resolvers through non-authoritative and caching servers up to authoritative nameservers, and finally root servers as a last resort.
- DNS is unencrypted by default, making queries visible to anyone on the local network or upstream. DNS over TLS (DoT) and DNS over HTTPS (DoH) address this, but are not universally enforced.
- BIND9 is the standard DNS server on Linux. Configuration is split across `named.conf.local`, `named.conf.options`, and `named.conf.log`. Zone files must contain exactly one SOA record and at least one NS record. A syntax error renders the entire zone file invalid, causing the server to return SERVFAIL for all queries in that zone.
- Forward records map domain names to IPs. Reverse lookup is handled by PTR records in a separate reverse zone file, mapping the last IP octet back to an FQDN.
- [[DNS enumeration tools]]

**DNS Server Types:**

|Server Type|Description|
|---|---|
|`DNS Root Server`|Top of the hierarchy, queried as last resort. 13 exist globally, coordinated by ICANN|
|`Authoritative Nameserver`|Holds authority for a specific zone. Answers are binding|
|`Non-authoritative Nameserver`|Collects zone info via recursive or iterative queries, not authoritative|
|`Caching DNS Server`|Caches responses for a TTL period set by the authoritative server|
|`Forwarding Server`|Forwards all queries to another DNS server without resolving|
|`Resolver`|Local resolution on the client machine or router|

**DNS Record Types:**

| Record  | Description                                                                       |
| ------- | --------------------------------------------------------------------------------- |
| `A`     | IPv4 address for a domain                                                         |
| `AAAA`  | IPv6 address for a domain                                                         |
| `MX`    | Responsible mail servers for the domain                                           |
| `NS`    | Authoritative nameservers for the domain                                          |
| `TXT`   | Arbitrary text. Used for SPF, DMARC, SSL validation, and third-party verification |
| `CNAME` | Alias pointing one domain to another                                              |
| `PTR`   | Reverse lookup. Maps an IP address back to a domain name                          |
| `SOA`   | Zone authority record. Contains admin contact and zone metadata                   |

---
## Dangerous Settings

|Setting|Description|
|---|---|
|`allow-query`|Defines which hosts can send queries to the server|
|`allow-recursion`|Defines which hosts can send recursive queries|
|`allow-transfer`|Defines which hosts can request zone transfers|
|`zone-statistics`|Enables collection of per-zone statistical data|

---
## Enumeration

```bash
dig ns <domain> @<nameserver>
```

Query for NS records. Returns the authoritative nameservers responsible for the target domain.

```bash
dig any <domain> @<nameserver>
```

Query for all available record types at once. Returns a combined view of A, MX, NS, TXT, SOA, and any other records the server will expose.

```bash
dig axfr <domain> @<nameserver>
```

Request a full zone transfer. AXFR is a replication mechanism that dumps all records in a zone. Succeeds only if the server allows transfers from the requesting host, making it a high-value misconfiguration to test.

Example:

```bash
dig axfr example.com @ns1.example.com
```

---

## Brute force

```bash
dnsenum --dnsserver <nameserver> --enum -p 0 -s 0 -o found_subdomains.txt -f ~/subdomains.list <domain>
```

Brute-force subdomains against the target nameserver using a wordlist. `-p 0 -s 0` disables Google scraping and whois lookups, keeping the scan DNS-only. Results saved to `found_subdomains.txt`.