**Overview:**
- Automated process of systematically browsing a target website by following links and collecting data. Sends real HTTP requests, leaving traces in server logs.
- Extracts internal/external links, comments, metadata, and sensitive files (`.bak`, `.old`, `web.config`, `settings.php`, log files) that may expose credentials, API keys, or source code.
- `robots.txt` (site root) lists disallowed paths, often revealing hidden directories, admin panels, and backup files the owner wants excluded from search indexing.
- `/.well-known/` (RFC 8615) centralizes service metadata and configuration files. The `openid-configuration` URI is particularly useful for discovering authentication endpoints.
---
## Installation

```bash
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
```

Downloads and extracts ReconSpider.

---
## Crawling

```bash
python3 ReconSpider.py <target>
```

Crawls the target and saves results to `results.json`.