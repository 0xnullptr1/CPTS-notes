**Overview:**
- WordPress is an open-source CMS written in PHP, typically running on Apache with MySQL as the backend. Used for blogs, forums, and general-purpose sites.
- Its extensible nature makes it prone to vulnerabilities through third-party themes and plugins.
- Key directories: plugins at `wp-content/plugins`, themes at `wp-content/themes`. Both should be carefully enumerated as vulnerable plugins/themes can lead to RCE.
---
## Discovery & Footprinting

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/
Sitemap: https://inlanefreight.local/wp-sitemap.xml
```

Browsing to `/robots.txt` often reveals WordPress-specific paths.
Presence of `/wp-admin` and `/wp-content` confirms a WordPress installation.

---
## Enumeration

```bash
curl -s http://<target> | grep WordPress
```

Confirm WordPress and identify version from page source.

```bash
curl -s http://<target> | grep themes
curl -s http://<target> | grep plugins
```

Identify active themes and plugins. Use version numbers found to search for known vulnerabilities.

---
## WPScan

```bash
sudo wpscan --url http://<target> --enumerate
```

Enumerate WordPress users, plugins, themes, and vulnerabilities.

```bash
sudo wpscan --password-attack xmlrpc -t 20 -U <user> -P /usr/share/wordlists/rockyou.txt --url http://<target>
```

Password attack via XML-RPC. `-t` sets threads, `-U` specifies the username, `-P` the wordlist.