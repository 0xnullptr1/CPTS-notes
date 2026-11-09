**Overview:**
- SocksOverRDP uses Dynamic Virtual Channels (DVC) from Windows Remote Desktop Services to tunnel arbitrary packets over an RDP connection. This makes it useful for pivoting in environments where SSH is unavailable.
- The tool requires two components: `SocksOverRDP-Plugin.dll` (loaded on the foothold host) and `SocksOverRDP-Server.exe` (loaded on the next pivot target). Proxifier is used as the proxy client to route traffic through the established SOCKS tunnel.
- Required binaries: `SocksOverRDP x64` (from the SocksOverRDP GitHub releases) and `ProxifierPE.zip` (Proxifier Portable).
---
## Workflow

1. Transfer `SocksOverRDP-Plugin.dll` to the foothold Windows host (reachable via RDP from the attack host).
2. Load the plugin on the foothold host, then RDP into the next target.
3. Transfer `SocksOverRDP-Server.exe` to the next target and start it with admin privileges.
4. Configure Proxifier on the foothold host to forward traffic to `127.0.0.1:1080`.
5. Use `mstsc.exe` through Proxifier to reach deeper network segments.

---
## Foothold Host

```powershell
regsvr32.exe SocksOverRDP-Plugin.dll
```

Load the SocksOverRDP plugin on the foothold host. On success, the plugin confirms it is enabled and listening on `127.0.0.1:1080`. Connect to the next target via `mstsc.exe` to activate the tunnel.

```powershell
netstat -antb | findstr 1080
```

Verify the SOCKS listener is active on port `1080`.

---
## Next Pivot Target

Transfer `SocksOverRDP-Server.exe` to the next target via the RDP session. Start it with admin privileges. This establishes the server-side endpoint of the tunnel, allowing traffic forwarded from the foothold host to be routed further into the internal network.

---
## Proxifier Configuration

Transfer `ProxifierPE.exe` to the foothold host. Configure a proxy profile pointing to `127.0.0.1:1080` (SOCKS5). Once active, all traffic from tools launched through Proxifier (including `mstsc.exe`) will be tunneled over the RDP connection to the next pivot target and routed onward by `SocksOverRDP-Server.exe`.