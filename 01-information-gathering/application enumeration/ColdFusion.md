**Overview:**
- ColdFusion is a Java-based web application development platform using CFML (ColdFusion Markup Language), a tag-based language similar to HTML that simplifies database integration, web services, and email management.
- Used to build dynamic and interactive web applications connected to databases such as MySQL, Oracle, and Microsoft SQL Server. Can run on Windows, Mac, or Linux and deploy to cloud platforms like AWS and Azure.
- CFML reduces the amount of code needed for complex tasks through built-in tags and functions. It also supports JavaScript and Java within the ColdFusion environment.
- Historically vulnerable to SQLi, XSS, directory traversal, authentication bypass, and arbitrary file uploads.
- Known CVEs: `CVE-2021-21087` (arbitrary JSP upload), `CVE-2020-24450` (command injection), `CVE-2020-24449` (arbitrary file read), `CVE-2019-15909` (XSS).
---
## Default Ports

|Port|Protocol|Description|
|---|---|---|
|80|HTTP|Web server|
|443|HTTPS|Secure web server|
|8500|SSL|ColdFusion server communication|
|5500|Server Monitor|Remote administration|
|1935|RPC|Client-server communication|
|25|SMTP|Email|

---
## Discovery & Enumeration

|Method|Indicator|
|---|---|
|Port scan|Port 8500 (SSL), 80, 443|
|File extensions|`.cfm`, `.cfc`|
|HTTP headers|`Server: ColdFusion`, `X-Powered-By: ColdFusion`|
|Default paths|`/CFIDE/administrator/index.cfm`, `admin.cfm`|
|Error messages|References to ColdFusion tags or functions|

```bash
nmap -p- -sC -Pn <target> --open
```

Identify open ports. Port 8500 is a strong indicator of ColdFusion. Browse to `<target>:8500`  The presence of `/CFIDE` and `/cfdocs` directories confirms it.

Navigate to `/CFIDE/administrator` to reach the admin login page and confirm the exact version.