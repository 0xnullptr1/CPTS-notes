**Overview:**
- IPMI is a hardware-based management interface that operates independently of the OS, allowing remote control of systems even when powered off or unresponsive. Communicates over UDP port 623.
- Managed through Baseboard Management Controllers (BMCs), typically embedded ARM systems running Linux, directly connected to the motherboard. Common BMCs encountered in internal pentests: HP iLO, Dell DRAC, Supermicro IPMI.
- BMC access is equivalent to physical access. It allows monitoring, rebooting, powering off, reinstalling the OS, and modifying BIOS settings. Most BMCs also expose a web management console and remote access via Telnet or SSH.
- IPMI 2.0 has a flaw in the RAKP protocol: the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication completes. This allows hash retrieval for any valid user account without knowing the password, crackable offline with Hashcat mode `7300`.
---
## BMC Default Credentials

| Product         | Username        | Password                                            |
| --------------- | --------------- | --------------------------------------------------- |
| Dell iDRAC      | `root`          | `calvin`                                            |
| HP iLO          | `Administrator` | Randomized 8-character string (uppercase + numbers) |
| Supermicro IPMI | `ADMIN`         | `ADMIN`                                             |

---
## Hash Dump

```bash
msf > use auxiliary/scanner/ipmi/ipmi_dumphashes
```

Exploit the IPMI 2.0 RAKP flaw to retrieve SHA1 password hashes for valid BMC user accounts. Hashes can be cracked offline with Hashcat mode `7300`.