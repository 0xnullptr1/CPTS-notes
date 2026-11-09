**Overview:**
- ICMP tunneling encapsulates traffic within ICMP packets (echo requests and responses). It only works when ping responses are permitted within the firewalled network.
- When a host inside a firewalled network is allowed to ping an external server, it can encapsulate traffic inside ping echo requests and send it outbound. The external server validates the traffic and responds accordingly, enabling data exfiltration and pivot tunnels.
- `ptunnel-ng` implements this by running a server on the pivot host and a client on the attack host, establishing an ICMP-encapsulated channel that can carry SSH traffic.
---
## Installation

```bash
git clone https://github.com/utoni/ptunnel-ng.git
```

Clone the repository on the attack host, then run `autogen.sh` from the root of the ptunnel-ng directory to build the binary.

```bash
scp -r ptunnel-ng ubuntu@<target>:~/
```

Transfer the full repository to the pivot host. `-r` transfers the directory recursively.

---
## Server (Pivot Host)

```bash
sudo ./ptunnel-ng -r<target> -R22
```

Start the ptunnel-ng server on the pivot host. `-r` IP address the server will accept connections on (the pivot host's IP), `-R` port to forward to on that host (`22` for SSH).

---
## Client (Attack Host)

```bash
sudo ./ptunnel-ng -p<target> -l2222 -r<target> -R22
```

Connect to the ptunnel-ng server from the attack host. `-p` pivot host IP (server address), `-l` local port to listen on, `-r` remote host to forward traffic to, `-R` remote port.

---
## SSH Through the Tunnel

```bash
ssh -p2222 -l ubuntu 127.0.0.1
```

Connect via SSH through the ICMP tunnel. Traffic sent to local port `2222` is encapsulated in ICMP and forwarded to port `22` on the pivot host.

```bash
ssh -D 9050 -p2222 -l ubuntu 127.0.0.1
```

Enable dynamic port forwarding through the established ICMP tunnel. `-D` creates a SOCKS proxy on local port `9050`, allowing proxychains to route traffic further into the internal network.