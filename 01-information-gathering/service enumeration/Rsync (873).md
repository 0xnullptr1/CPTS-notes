**Overview:**
- Rsync is a fast file transfer tool for local and remote copying, commonly used for backups and mirroring. Operates on TCP port 873 by default.
- Uses a delta-transfer algorithm: only sends the differences between source and destination files, minimizing data transmitted over the network.
- Can be configured to tunnel over SSH using the `-e ssh` flag, inheriting SSH's encryption and authentication. If SSH runs on a non-standard port, pass it as `-e "ssh -p<port>"`.
---
## Enumeration

```bash
nc -nv <target> 873
```

Connect to the Rsync service and grab the banner.

```bash
rsync -av --list-only rsync://<target>/<share>
```

List all files in an open Rsync share without transferring anything. `-a` preserves attributes, `-v` enables verbose output.

---
## File Transfer

```bash
rsync -av rsync://<target>/<share>
```

Sync all files from the remote share to the current directory on the attack host.

```bash
rsync -av -e ssh rsync://<target>/<share>
rsync -av -e "ssh -p<port>" rsync://<target>/<share>
```

Sync files over an SSH-tunneled Rsync connection. Use the second form when SSH runs on a non-standard port.
