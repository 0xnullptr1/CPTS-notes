**Overview:**
- SSH provides encrypted, authenticated remote access over TCP port 22. Cross-platform: native on Linux and macOS, available on Windows via third-party clients.
- SSH-1 is vulnerable to MITM attacks and considered obsolete. SSH-2 is the current standard with stronger encryption, stability, and security. Seeing `Protocol 1` in a server config is an immediate flag.
- OpenSSH supports six authentication methods: password, public-key, host-based, keyboard, challenge-response, and GSSAPI. Public-key authentication is the most secure: the server challenges the client with a problem encrypted using the client's public key, which only the client's private key can solve.
- In public-key auth, the private key never leaves the client machine and is protected by a local passphrase. Compromise requires access to both the private key file and the passphrase.
---
## Dangerous Settings

|Setting|Description|
|---|---|
|`PasswordAuthentication yes`|Enables password-based login, allowing brute-force attempts|
|`PermitEmptyPasswords yes`|Allows accounts with no password to authenticate|
|`PermitRootLogin yes`|Allows direct root login over SSH|
|`Protocol 1`|Enables SSHv1, which is vulnerable to MITM attacks|
|`X11Forwarding yes`|Forwards GUI applications over SSH, expanding attack surface|
|`AllowTcpForwarding yes`|Permits TCP port forwarding, usable for tunneling|
|`PermitTunnel yes`|Allows full network tunneling through the SSH connection|
|`DebianBanner yes`|Displays OS/version banner on login, aiding fingerprinting|

---
## Enumeration

```bash
ssh <user>@<FQDN/IP>
```

Connect to SSH.

```bash
ssh -i private.key <user>@<FQDN/IP>
```

Connect to the SSH service using the private key.

```bash
ssh -v <user>@<target> -o PreferredAuthentications=password
```

Force password authentication regardless of server preference. Useful for testing if password auth is permitted and for brute-force staging.

```bash
./ssh-audit.py <target>
```

Audit SSH client and server configuration. Returns supported algorithms, protocol version, and known weaknesses.

## Brute Force Login

```bash
hydra -L user.list -P /usr/share/wordlists/rockyou.txt ssh://<target>
```

Utilizes the tool `Hydra` to brute force SSH.