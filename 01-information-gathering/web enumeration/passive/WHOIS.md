**Overview:**
- Query-and-response protocol for accessing registries of internet resources: domain names, IP blocks, and autonomous systems.
- Each record exposes registrant identity, administrative and technical contacts, registrar, name servers, and registration/expiration dates.
- Contact data (names, emails, phone numbers) is directly actionable for social engineering and phishing target identification.
- Name server and IP details reveal network infrastructure layout and potential misconfigurations or entry points.
- Historical record services (e.g. WhoisFreaks) allow tracking ownership and infrastructure changes over time, useful for mapping target evolution.
---
## Enumeration

```bash
whois <domain>
```

Queries the WHOIS registry for registration data on the target domain. Returns registrant, administrative and technical contacts, registrar, name servers, and creation/expiration dates.