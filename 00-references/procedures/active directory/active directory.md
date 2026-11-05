## 1 - Live Host Discovery

- Conduct a ping sweep on the IP range.
- Use NetExec on the IP range for richer output (hostname, OS, domain, SMB signing status).
- Run Responder in analyze mode (`-A`) to passively collect IP addresses and NetBIOS/LLMNR announcements without poisoning.
- Map the role of each live host (DC, file server, workstation) via targeted service enumeration before moving on.

---
## 2 - User Enumeration (No Foothold)

Attempt all four passive methods. No single method is guaranteed to return all users.

- Attempt user enumeration via SMB null authentication.
- Attempt user enumeration via LDAP anonymous bind.
(Whitebox / Grey-box)- Attempt user enumeration via RPCClient null session.
- Attempt user enumeration via RID brute-forcing.
- Attempt username validation via Kerbruting. Requires no credentials and generates minimal noise. Useful when null session methods are locked down.

---
## 3 - Initial Foothold

Work through these in order. Password spraying is the last resort and can lock accounts. Always determine the lockout policy first.

- Find ASREPRoastable users from the user list.
- Find Kerberoastable users from the user list.
- Run Responder to catch NTLMv2 hashes during active hours.
- Use SMB null authentication to spider shares for credentials.
- Compromise a domain-connected host to `SYSTEM`/`root` to obtain a machine account for further enumeration.
- As a last resort, password spray with the user list using common patterns and previously found passwords.

---
## 4 - Attacks

Run SharpHound on every newly compromised host immediately. Review outbound BloodHound attack paths from each compromised principal before proceeding.

- Check command execution for each compromised user via SMB, WinRM, and RDP.
- Kerberoast all accounts with SPNs set.
- ASREPRoast all accounts with pre-authentication disabled.
- Check GPOs for credentials via `gpp_password` and `gpp_autologin`.
- Spider all readable SMB shares for each compromised user.
- Conduct SMB hash theft attacks on writable shares for each compromised user. Pair with Responder to capture resulting hashes.
- Check user description fields for embedded passwords.
- Check the DC's SYSVOL share for logon scripts and scheduled task XML files containing hardcoded credentials.
- Enumerate ADCS for vulnerable certificate templates (ESC1–ESC8).
- Test compromised local administrator hashes against other hosts via pass-the-hash.
- Re-run Responder from newly compromised hosts on different network segments.
- Identify accounts with the `PASSWD_NOTREQD` flag set.
- Password spray using previously found passwords across all known users.

---
## Exploitation Vectors

Evaluate once BloodHound paths and initial enumeration are complete. Confirm prerequisites before attempting.

- **NoPac (CVE-2021-42278 / CVE-2021-42287)**: Requires `MachineAccountQuota > 0`. Grants Domain Admin via S4U2self abuse.
- **PrintNightmare (CVE-2021-34527)**: Requires Print Spooler running on the DC. Achieves SYSTEM via malicious DLL.
- **PetitPotam (CVE-2021-36942)**: Requires ADCS with Web Enrollment enabled and no EPA/signing enforced. Coerces DC authentication for NTLM relay to ADCS.
- **DCSync**: Requires `DS-Replication-Get-Changes-All` rights. Dumps all domain hashes without touching LSASS.