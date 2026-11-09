**Overview:**
- IIS tilde enumeration is a technique used to uncover hidden files, directories, and short file names (8.3 format) on vulnerable Microsoft IIS web servers.
- When a file or folder is created on an IIS server, Windows generates a short file name in 8.3 format (8 chars + `.` + 3 chars extension). These short names can grant access to their corresponding files even if they were meant to be hidden.
- The tilde (`~`) character followed by a sequence number represents a short file name in a URL (e.g. `TRANSF~1.ASP`). If a valid short name is found, it can be used to access the resource or enumerate further.
- Enumeration involves sending HTTP requests with distinct character combinations to identify valid short file names, then using that information to access resources or enumerate the directory structure further.
---
## Tilde Enumeration

```bash
java -jar iis_shortname_scanner.jar 0 5 http://<target>/
```

Automates sending character combinations via HTTP to identify valid short file names.

---
## Wordlist Generation

If direct access to the discovered short name is blocked (e.g. `TRANSF~1.ASP`), brute-force the full filename using a targeted wordlist.

```bash
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```

Filters wordlists for entries starting with the known short name prefix and saves them to a custom list.

```bash
gobuster dir -u http://<target>/ -w /tmp/list.txt -x .aspx,.asp
```

Enumerate the target using the generated wordlist to find the full filename.