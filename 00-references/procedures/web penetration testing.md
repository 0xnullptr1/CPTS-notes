## 1 - Information Gathering

- Identify the target's IP address.
- Enumerate DNS records (A, AAAA, MX, TXT, NS, CNAME).
- Perform subdomain enumeration.
- Check for virtual host routing differences (same IP, different `Host` header).
- Check for a `robots.txt`, `sitemap.xml`, and `security.txt`.

## 2 - Web Server & Technology Enumeration

- Fingerprint the web server software and version (headers, error pages, `Server`/`X-Powered-By`).
- Identify the underlying technology stack (language, framework, CMS) via Wappalyzer/WhatWeb.
- Check HTTP response headers for information disclosure and missing security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options).
- Enumerate supported HTTP methods (`OPTIONS`, `PUT`, `DELETE`, `TRACE`).
- Check for verbose error messages (stack traces, debug mode enabled).
- Identify the TLS/SSL configuration; check for weak ciphers and expired/misconfigured certificates.
- Look for default or sample content left on the server (install scripts, default admin pages, README files).

## 3 - Content Discovery & Crawling

- Spider the application manually and with an automated crawler (Burp Spider, ffuf, gobuster).
- Perform directory and file brute-forcing with curated wordlists (SecLists).
- Fuzz for backup files and archives (`.bak`, `.old`, `.zip`, `.tar.gz`, `~`).
- Check for exposed version control directories (`.git`, `.svn`, `.hg`) and dump them if present.
- Check for exposed configuration/environment files (`.env`, `web.config`, `config.php.bak`).
- Enumerate API endpoints (REST/GraphQL/SOAP) via crawling, JS analysis, and dedicated wordlists.
- Review all JavaScript files for hardcoded secrets, hidden endpoints, and internal hostnames.
- Check for exposed admin/management interfaces (`/admin`, `/manager`, `/phpmyadmin`, `/actuator`).
- Map every input point: forms, URL parameters, headers, cookies, JSON bodies, file uploads.
- Build a full site map of unauthenticated vs. authenticated functionality before testing.

## 4 - Authentication

- Enumerate valid usernames via differing error messages, response timing, or verbose registration flows.
- Test for weak/default/guessable credentials.
- Test for brute-force and account lockout protections (and lockout bypass via username case/whitespace tricks).
- Test password reset functionality:
    - Predictable or non-expiring reset tokens.
    - Host header injection in password reset emails.
    - Token leakage via `Referer` header after redirect.
- Test for user enumeration via the "forgot password" flow.
- Check for weak password policy enforcement.
- Test multi-factor authentication for bypass (response manipulation, missing server-side enforcement, reusable/brute-forceable OTP).
- Test "remember me" functionality for predictable or non-expiring tokens.
- Check for authentication bypass via SQL injection, NoSQL injection, or logic flaws (e.g., `admin'--`).
- Check for JWT misconfiguration (`alg: none`, weak signing secret, key confusion, missing signature verification).

## 5 - Session Management

- Analyze session token randomness, length, and predictability.
- Check whether session tokens are sent over HTTP (not just HTTPS).
- Check `Set-Cookie` flags: `HttpOnly`, `Secure`, `SameSite`.
- Test for session fixation (session ID not regenerated after login).
- Test whether sessions are invalidated server-side on logout.
- Test for concurrent session handling and session timeout enforcement.
- Test for CSRF tokens being tied to the session and properly validated (not just present).
- Check for sensitive data stored client-side in cookies, localStorage, or sessionStorage.

## 6 - Injection

- **SQL Injection**
    - Test all input points (GET/POST params, headers, cookies, JSON) with classic payloads (`'`, `"`, `--`, `OR 1=1`).
    - Identify injection type: error-based, union-based, boolean-blind, time-blind.
    - Confirm and exploit with `sqlmap`, tuning for the detected DBMS and WAF evasion if needed.
    - Attempt to read files or write a webshell via `INTO OUTFILE`/`xp_cmdshell` where applicable.
- **Command Injection**
    - Test input fields that may be passed to shell commands (ping, DNS lookup, file conversion utilities).
    - Use command separators/chaining (`;`, `|`, `&&`, `$()`, backticks) and blind indicators (time delay, OOB DNS/HTTP callback).
- **NoSQL Injection**
    - Test for operator injection in JSON bodies (`$ne`, `$gt`, `$regex`) against MongoDB-backed login/search forms.
- **LDAP / XPath Injection**
    - Test login and search forms backed by directory services with wildcard and boolean payloads.
- **Server-Side Template Injection (SSTI)**
    - Identify the template engine via polyglot payloads.
    - Escalate from reflection to remote code execution based on the identified engine.
- **HTTP Header / Host Header Injection**
    - Test for injection via `X-Forwarded-For`, `X-Forwarded-Host`, and `Host` headers (cache poisoning, password reset poisoning, SSRF pivot).

## 7 - File Inclusion

- Test parameters that reference files/paths for Local File Inclusion (`../../../../etc/passwd`, null byte/path truncation tricks on older stacks).
- Attempt LFI-to-RCE via log poisoning (`/var/log/apache2/access.log` + malicious `User-Agent`), PHP session file poisoning, or `php://filter`/`data://` wrappers.
- Test for Remote File Inclusion where `allow_url_include` may be enabled.
- Check for path traversal in file download/export functionality.

## 8 - File Upload

- Identify client-side vs. server-side validation; bypass client-side checks via proxy.
- Test extension filtering bypass (double extensions, null byte, case variation, trailing dots/spaces, alternate extensions like `.phtml`/`.pht`).
- Test `Content-Type`/magic byte validation bypass (polyglot files).
- Check if the upload directory is web-accessible and executes scripts.
- Test for path traversal in the filename to overwrite arbitrary files.
- Test for malicious metadata/EXIF-based injection (image parsers, ImageTragick).
- Attempt webshell upload and confirm RCE once a bypass is found.

## 9 - Cross-Site Scripting (XSS)

- Test all input points for reflected XSS with context-aware payloads (HTML, attribute, JS, URL contexts).
- Test for stored XSS in persisted data (comments, profile fields, file names, support tickets).
- Test for DOM-based XSS by reviewing client-side JS sinks (`innerHTML`, `document.write`, `eval`, `location`).
- Test filter/WAF bypass techniques (encoding, case variation, event handlers, polyglots).
- Confirm impact via cookie theft or session hijacking.

## 10 - Cross-Site Request Forgery (CSRF)

- Identify state-changing requests lacking anti-CSRF tokens.
- Check whether CSRF tokens are validated server-side and tied to the session (not just present in the request).
- Test `SameSite` cookie bypass scenarios.
- Build and test a proof-of-concept auto-submitting form for a sensitive action (password/email change).

## 11 - XXE & Insecure Deserialization

- **XXE**
    - Identify endpoints accepting XML (direct uploads, SOAP, DOCX/XLSX parsing, SAML).
    - Test for external entity injection to read local files, perform SSRF, or achieve blind OOB exfiltration.
- **Insecure Deserialization**
    - Identify serialized objects in cookies, hidden fields, or request bodies (PHP, Java, Python pickle, .NET ViewState).
    - Fingerprint the language/library and test known gadget chains (`ysoserial` for Java, `phpggc` for PHP).
    - Check `__VIEWSTATE` integrity (MAC validation disabled) on .NET applications.

## 12 - Server-Side Request Forgery (SSRF)

- Identify functionality that fetches a remote resource server-side (URL preview, webhook, PDF generator, image fetcher, import-from-URL).
- Test access to internal-only services and cloud metadata endpoints (`169.254.169.254`).
- Test SSRF filter bypass (alternate IP encodings, DNS rebinding, redirects, URL parser confusion).
- Chain SSRF with internal port scanning to map the internal network.

## 13 - Access Control / Business Logic

- Test for Insecure Direct Object References (IDOR) by manipulating IDs/references across user contexts.
- Test for vertical privilege escalation (low-priv user accessing admin functionality).
- Test for horizontal privilege escalation (accessing another same-level user's data).
- Test for mass assignment (adding unexpected parameters like `isAdmin=true` to requests).
- Test multi-step processes for workflow bypass (skipping payment/approval steps).
- Test rate limiting on sensitive actions (login, OTP, coupon codes, financial transactions).

## 14 - CMS & Third-Party Software

- Identify the CMS/plugin versions in use (WordPress, Joomla, Drupal) and cross-reference against known CVEs.
- **WordPress**: enumerate users (`/wp-json/wp/v2/users`), plugins/themes, check `xmlrpc.php` abuse, review `wp-config.php` backup exposure.
- **Joomla**: check for exposed `configuration.php` backups, enumerate via `/administrator`.
- **Drupal**: check version via `CHANGELOG.txt`, test known RCE chains (Drupalgeddon).
- Check for known default credentials on identified software.
- Search Exploit-DB/searchsploit for the fingerprinted versions before manual testing.

## 15 - API Testing

- Enumerate API documentation (Swagger/OpenAPI, GraphQL introspection) if exposed.
- Test for broken object-level authorization (BOLA/IDOR) across API resources.
- Test for excessive data exposure in API responses (fields not used by the front end).
- Test for missing rate limiting and resource exhaustion.
- Test authentication consistency between the web app and its API (API key leakage, missing auth on API-only endpoints).

## 16 - Source Code Review

- Search the codebase for hardcoded credentials, API keys, and secrets.
- Identify use of dangerous functions (`eval`, `exec`, `unserialize`, raw SQL concatenation).
- Review authentication and session-handling logic for flawed implementation.
- Review access-control checks for missing or inconsistent enforcement across endpoints.
- Trace user input from source to sink for each vulnerability class above.
- Review third-party/vendored library versions against known CVEs.