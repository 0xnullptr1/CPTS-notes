**Overview:**
- Kerberos tickets are scoped to a single resource, unlike NTLM hashes which are cached in memory and reusable across connections. This creates a "double hop" problem when authenticating over WinRM: the ticket grants access to the first hop host, but no credential material is cached to forward to a second hop (e.g., a Domain Controller).
- The issue manifests when running tools like PowerView from a WinRM shell that need to query the DC: the session has no ticket for the DC and no password to request one.
- Tools that authenticate over SMB or LDAP (e.g., PSExec) do cache the NTLM hash, so they are not affected. The double hop problem is specific to Kerberos-only flows like WinRM/PSRemoting.
- Two practical workarounds exist: passing a PSCredential object explicitly with each command, or registering a named PSSession configuration that runs under the credential persistently.
- The `Register-PSSessionConfiguration` method requires an interactive PowerShell console with GUI access and will not work from evil-winrm or a Linux-based PowerShell session due to Kerberos limitations on non-Windows hosts.
---
## PSCredential Object

```powershell
klist
```

Confirms only a ticket for the current host is cached, with no ticket for the DC.

```powershell
$SecPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential ('<domain>\<user>', $SecPassword)
```

Creates a PSCredential object to explicitly pass credentials with subsequent commands.

```powershell
get-domainuser -spn -credential $Cred | select samaccountname
```

Queries SPN accounts via PowerView by passing `$Cred` explicitly. Required for every command that needs to reach the DC from a WinRM shell.

---
## Register PSSession Configuration

> Requires a Windows attack host or jump host with RDP access and an elevated PowerShell console. Will not work from evil-winrm or PowerShell on Linux.

```powershell
Enter-PSSession -ComputerName <target-fqdn> -Credential <domain>\<user>
```

Establishes an initial WinRM session on the remote host.

```powershell
Register-PSSessionConfiguration -Name <session-name> -RunAsCredential <domain>\<user>
```

Registers a named PSSession configuration that will run as the specified credential, eliminating the need to pass `$Cred` per command. Triggers a credential popup.

```powershell
Restart-Service WinRM
```

Restarts WinRM to activate the new session configuration. Closes the current session.

```powershell
Enter-PSSession -ComputerName <target-fqdn> -Credential <domain>\<user> -ConfigurationName <session-name>
```

Reconnects using the registered named session. Commands issued from this session can reach the DC without additional credential objects.


