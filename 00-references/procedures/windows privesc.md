## Enumeration

### Users and Groups

- List local users and local groups.
- Check membership of high-value groups:
    - Local Administrators
    - Remote Desktop Users
    - Remote Management Users.

### Operating System

- Identify the Windows version and build number. Cross-reference against the [Windows version list](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions).
- Identify the system architecture (32-bit or 64-bit).

### Network

- Enumerate IP addresses and network interfaces.
- List active connections and listening ports.

### Processes and Programs

- List all installed applications and services.
- Look for running processes not matching any installed application: may indicate manually dropped binaries.

### Security Features

- Enumerate active security features (AV, EDR, AppLocker, UAC level, LAPS, etc.).
- Check Windows Defender exclusion paths — a writable, scan-excluded directory is a good staging spot for payloads.

---

## Privilege Escalation

Run these regardless of which user account is held.

- Run WinPEAS and review output.
- Look for vulnerable services:
    - Services with weak executable permissions (check via `icacls`).
    - Services where the executable path can be modified (weak service config permissions, check via `icacls`/`sc sdshow`).
    - Unquoted service paths with spaces exploitable via binary planting.
    - Services prone to DLL hijacking.
- Search for DLL hijacking opportunities more broadly: writable directories in `PATH`, and missing DLLs reported by running processes (not just services).
- Check if `AlwaysInstallElevated` is enabled in both HKLM and HKCU.
- Search for CVEs in installed programs and their versions.
- Check for known exploits targeting the current Windows version and build number.
- Check for unattended install files (`unattend.xml`, `sysprep.inf`, `sysprep.xml`) for plaintext local admin credentials.
- Check for autologon credentials in the registry (`HKLM\...\Winlogon\DefaultPassword`).

---

## Privilege Escalation With A Specific User

Repeat for every newly compromised user account.

- Check the user's privileges (`whoami /priv`). Cross-reference SeImpersonate, SeBackup, SeDebug, SeTakeOwnership, and similar dangerous rights.
- Exploit dangerous privileges where present:
    - `SeImpersonate`/`SeAssignPrimaryToken`: exploit via the Potato family (PrintSpoofer, JuicyPotato, RoguePotato, GodPotato — pick based on OS build).
    - `SeBackup`/`SeRestore`: abuse to read/write protected files (e.g., copy the SAM/SYSTEM hives).
    - `SeTakeOwnership`/`SeDebug`: abuse to take ownership of protected files or dump arbitrary process memory.
- Check which groups the user belongs to.
- Look for secrets in shell environment variables.
- Hunt for credentials (config files, scripts, registry, browser-saved passwords, credential manager).
- Check Credential Manager and saved sessions: `cmdkey /list`, saved `.rdp` files, PuTTY saved session registry keys.
- Check PowerShell history and transcript files (`ConsoleHost_history.txt`, PS transcript logs) for plaintext credentials.
- Look for unquoted service paths exploitable by the current user.
- Look for exploitable scheduled tasks owned by or writable by the current user.
- Plant malicious SCF and `.lnk` files on writable SMB shares to capture NTLMv2 hashes.

---

## Local Administrator Privileges

- Dump credentials with LaZagne.
- Dump SAM and LSA secrets.
- Dump LSASS memory.