**Overview:**
- FTP uses two separate TCP channels: a persistent control channel on port 21 for commands and status codes, and a data channel on port 20 opened only during transfers.
- In active mode the server initiates the data connection back to the client, which is almost always blocked by firewalls protecting the client side. Passive mode flips this: the server advertises a high port and the client connects to it, so the firewall sees only outbound traffic and lets it through
- TFTP is a stripped-down alternative using UDP instead of TCP, it has no authentication, no directory listing, and no session state, relying on the application layer to handle retransmission. 
- Both FTP and TFTP transmit everything in cleartext, including credentials. If attackers can position on the same network segment or intercept traffic, credentials are recoverable with a packet capture.
- vsFTPd is the default FTP server on most Linux distributions. its config at `/etc/vsftpd.conf` controls all behavior including anonymous access, write permissions, and chroot jails, making it the primary target for misconfiguration review.
---
## Dangerous Settings

| Setting                        | Description                                                   |
| ------------------------------ | ------------------------------------------------------------- |
| `anonymous_enable=YES`         | Anonymous login allowed                                       |
| `anon_upload_enable=YES`       | Anonymous users can upload files                              |
| `anon_mkdir_write_enable=YES`  | Anonymous users can create directories                        |
| `no_anon_password=YES`         | No password prompt for anonymous                              |
| `anon_root=/home/username/ftp` | Root directory served to anonymous users                      |
| `write_enable=YES`             | Enables STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, SITE commands |

---
## Enumeration

```bash
sudo nmap -sV -p21 --script ftp-anon <target>
```

Check for anonymous authentication on the target FTP service.

```bash
ftp <target>
nc -nv <target> 21
telnet <target> 21
```

Different ways to connect to the FTP service.

```bash
openssl s_client -connect <target>:21 -starttls ftp
```

Interact with FTP over an encrypted connection using STARTTLS.

```bash
wget -m --no-passive ftp://<user>:<password>@<target>
```

Recursively download all accessible files from the target FTP server. `--no-passive` forces active mode.
