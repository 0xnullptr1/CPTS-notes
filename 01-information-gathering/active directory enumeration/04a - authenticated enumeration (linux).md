**Overview:**
- Enumeration performed after obtaining valid domain credentials at any privilege level. Covers users, groups, shares, and privileged accounts.
- Minimum requirement is a cleartext password, NTLM hash, or SYSTEM access on a domain-joined host.
- Every domain object is identified by a SID combined with a RID. The RID is a unique decimal suffix appended to the domain SID to form a full object SID. Built-in accounts have fixed RIDs regardless of domain (e.g. built-in Administrator is always RID 500 / `0x1f4`).
- Key groups to note during enumeration: `Administrators`, `Domain Admins`, `Backup Operators`, `Executives`, and any group containing privileged IT staff. These are primary targets for privilege escalation.
---
## User and Group Enumeration

```bash
rpcclient -U <user>%<password> <dc-ip>
```

Authenticate to the DC via RPC. Run `enumdomusers` to list all users with their RIDs, `queryuser <RID>` to retrieve detailed attributes for a specific user, `enumdomgroups` to list all domain groups.

```bash
nxc smb <dc-ip> -u <user> -p <password> --users
```

Enumerate all domain users with their attributes via SMB. Replace `--users` with `--groups` to enumerate groups or `--loggedon-users` to list active sessions on the target host.

```bash
python3 windapsearch.py --dc-ip <dc-ip> -u <user>@<domain> -p <password> --da
```

Enumerate members of the Domain Admins group. Replace `--da` with `-PU` to enumerate all users with elevated privileges that may not appear in standard admin groups.

---
## Share Enumeration

```bash
nxc smb <dc-ip> -u <user> -p <password> --shares
```

List accessible SMB shares on the target host.

```bash
nxc smb <dc-ip> -u <user> -p <password> -M spider_plus --share '<share-name>'
```

Recursively list all readable files within a share. `spider_plus` module digs through each accessible share and outputs a full file listing.

```bash
smbmap -u <user> -p <password> -d <domain> -H <target-ip>
```

Enumerate shares and permissions on the target within the context of the provided credentials. `-H` specifies the target host.

```bash
smbmap -u <user> -p <password> -d <domain> -H <target-ip> -R '<share-name>' --dir-only
```

Recursively list directories within a specific share. `--dir-only` suppresses file output and shows directory structure only.

---
## Remote Execution

```bash
psexec.py <domain>/<user>:<password>@<target-ip>
```

Spawn an interactive shell on the target via the `ADMIN$` share. Requires local admin rights. Drops a service binary on disk, it is noisy.

```bash
wmiexec.py <domain>/<user>:<password>@<target-ip>
```

Execute commands on the target via WMI. Does not drop a binary or create a service. Leaves a lower forensic footprint than psexec.