**Overview:**
- Passive recon conducted before any active engagement, used to validate scoping documents, confirm target boundaries, and surface publicly accessible information that can affect the outcome of the test.
- Goal is to map IP space, domain structure, email schema, and any leaked credentials without touching the target directly.
- Breach data and credential dumps are especially valuable here. Even old passwords can unlock VPN portals, OWA, or other AD-authenticated externally facing services.
- Username lists built during this phase feed directly into password spraying, credential stuffing, and brute force attacks against exposed login portals.
---
## Target Data Points

|Data Point|Description|
|---|---|
|`IP Space`|Valid ASN, netblocks, cloud hosting providers, DNS records.|
|`Domain Information`|Subdomains, mail servers, DNS, VPN portals, OWA. Can reveal defensive infrastructure (SIEM, IPS/IDS, AV).|
|`Schema Format`|Email naming convention and AD username format, used to build valid username lists for spraying.|
|`Data Disclosures`|Public files (.pdf, .pptx, .docx, .xlsx) containing intranet listings, user metadata, credentials, or internal hostnames.|
|`Breach Data`|Leaked usernames, cleartext passwords, or hashes from prior breaches that may still be valid.|

---
## Recon Resources

|Resource|Examples|
|---|---|
|`ASN / IP Registrars`|IANA, ARIN (Americas), RIPE (Europe), BGP Toolkit (bgp.he.net).|
|`Domain Registrars & DNS`|Domaintools, PTRArchive, ICANN Lookup, manual nslookup against 8.8.8.8.|
|`Social Media`|LinkedIn, Twitter, Facebook, news articles. Useful for employee names and org structure.|
|`Public Company Websites`|About Us and Contact Us pages often expose email formats, office locations, and staff names.|
|`Cloud & Dev Storage`|GitHub, AWS S3 / Azure Blob (grayhatwarfare.com), Google Dorks (exploit-db.com/google-hacking-database).|
|`Breach Data Sources`|HaveIBeenPwned (email exposure check), Dehashed (cleartext passwords and hashes).|

---
## ASN and IP Enumeration

```bash
nslookup <domain>
```

Resolve domain to IP and identify nameservers. Run against each NS record returned by BGP Toolkit to validate.

---
## Google Dork Enumeration

```
filetype:pdf inurl:<domain>
```

Hunt for publicly indexed PDFs. Useful for finding documents with internal metadata, usernames, or intranet references.

```
intext:"@<domain>" inurl:<domain>
```

Surface email addresses indexed by search engines. Reveals naming convention (e.g. first.last, f.last).

---
## Username Enumeration

```bash
python3 linkedin2username.py -c <company-name>
```

Scrape LinkedIn to generate username permutations (flast, first.last, f.last, etc.) for use in password spraying.

---
## Breach Data

```bash
sudo python3 dehashed.py -q <domain> -p
```

Query Dehashed for cleartext passwords or hashes associated with the target domain. Try recovered passwords against externally facing portals using AD authentication (OWA, Citrix, RDS, VPN, VMware Horizon).
