**Overview:**
- Plink (PuTTY Link) is a Windows command-line SSH tool included with the PuTTY package. Like SSH, it can create dynamic port forwards and SOCKS proxies.
- `netsh.exe` is a Windows built-in tool for network configuration, usable for pivoting by creating port forwarding rules that redirect traffic from one host/port to another.
- `Netsh` is used for:
    - Finding routes
    - Viewing the firewall configuration
    - Adding proxies
    - Creating port forwarding rules

---
## Plink

```bash
plink -ssh -D 9050 ubuntu@<target>
```

Start a dynamic port forward via SSH. `-ssh` forces SSH protocol, `-D` creates a SOCKS proxy on local port `9050`, routing all traffic through the tunnel.

---
## Netsh

```powershell
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=<lhost> connectport=3389 connectaddress=<target>
```

Create a port forwarding rule. `v4tov4` forwards IPv4 traffic, `listenport` is the local port to listen on, `listenaddress` is the local interface to bind, `connectport` is the destination port, `connectaddress` is the destination host.

```powershell
netsh.exe interface portproxy show v4tov4
```

Verify active port forwarding rules.
