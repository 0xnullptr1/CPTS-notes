_**Overview:**
- Extracts technical details about web server software, versions, and underlying technologies from HTTP responses and page content.
- Knowing the exact stack narrows exploits to those affecting specific versions, increasing compromise probability significantly.
- Exposes misconfigured or outdated software that broader recon methods would miss.
- Combined with passive recon data, builds a comprehensive attack surface profile.
---
## Banner Grabbing

```
curl -I <target>
```

`-I` sends a HEAD request and returns only HTTP response headers. Reveals server software and version via `Server:`, `X-Powered-By:`, and similar headers.

---
## WAF Detection

```
wafw00f <target>
```

Detects WAF presence and identifies vendor. Run before aggressive fingerprinting as a WAF may block or distort subsequent probes.

---
## Software Identification

```
nikto -h <target> -Tuning b
```

Runs only Software Identification modules. `-Tuning b` limits scope to technology detection. Identifies outdated software and version disclosures.