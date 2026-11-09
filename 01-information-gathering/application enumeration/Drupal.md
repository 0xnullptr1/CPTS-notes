**Overview:**
- Drupal is an open-source CMS written in PHP, supporting MySQL, PostgreSQL, or SQLite as the backend.
- Like WordPress, it supports themes and modules, making third-party extensions a common attack surface.
- Drupal indexes content using nodes (e.g. `/node/<id>`), which can hold blog posts, articles, polls, and more.
- Drupal can be identified via the `Powered by Drupal` header/footer, the standard logo, presence of `CHANGELOG.txt` or `README.txt`, page source inspection, or `/node` references in `robots.txt`
---
## Discovery

```bash
curl -s http://target.example.com | grep Drupal
```

Confirm Drupal and identify version from page source.

---
## Enumeration

```bash
droopescan scan drupal -u http://target.example.com
```

Fingerprints Drupal version, installed modules, themes, and known vulnerabilities.