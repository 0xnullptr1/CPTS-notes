**Overview:**
- Directory fuzzing discovers hidden directories, files, and endpoints on web servers by testing common paths from a wordlist.
- Useful for finding admin panels, backup files, API endpoints, and other resources not linked in the application.
- /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
---
## Directory Fuzzing

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http://<target>:<port>/FUZZ
```

Fuzzes directories at root level.

```bash
gobuster dir -u http://<target>:<port> -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
```

Alternative to ffuf to discover directories. The `-u` flag specifies the target URL. The `-w` flag specifies the wordlist file. 

---
## Extension Fuzzing

```bash
ffuf -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http://<target>:<port>/indexFUZZ
```

Fuzzes file extensions (e.g., index.php, index.html).

---
## Page Fuzzing

```bash
ffuf -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http://<target>:<port>/blog/FUZZ.php
```

Fuzzes page names within a known directory.

---
## Recursive Fuzzing

```bash
ffuf -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http://<target>:<port>/FUZZ -recursion -recursion-depth 1 -e .php -v
```

Recursively fuzzes directories. `-recursion-depth` controls depth, `-e` adds extensions, `-v` verbose output.