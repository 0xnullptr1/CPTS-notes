**Overview:**
- Parameter fuzzing discovers hidden, undocumented, or vulnerable parameters in web applications by testing large numbers of requests.
- Web applications use parameters in URLs, query strings, POST data, and headers to pass data between client and server.
- Some parameters are exposed in the user interface, while others remain hidden or undocumented, these can reveal sensitive functionality or data.
---
## GET Parameters

```
ffuf -w /home/kali/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://<target>:<port>/admin/admin.php?FUZZ=<test-value> -fs <size>
```

Fuzzes GET parameter names. `-fs` filters response size to hide false positives.

## POST Parameters

```
ffuf -w /home/kali/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://<target>:<port>/admin/admin.php -X POST -d 'FUZZ=<test-value>' -H 'Content-Type: application/x-www-form-urlencoded' -fs <size>
```

Fuzzes POST parameter names.

---
## Value Fuzzing

```
ffuf -w <wordlist>:FUZZ -u http://<target>:<port>/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs <size>
```

Fuzzes parameter values after discovering a valid parameter name.

---
## Manual Testing

```
curl http://<target>:<port>/admin/admin.php -X POST -d 'id=<value>' -H 'Content-Type: application/x-www-form-urlencoded'
```

Manually tests a discovered parameter with curl.
