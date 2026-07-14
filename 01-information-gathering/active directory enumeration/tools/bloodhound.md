**Overview:**
- BloodHound ingests AD data (users, groups, computers, GPOs, ACLs, trusts, sessions, local admin rights, RDP/WinRM access) and maps attack paths using graph theory.
- Two components: a collector (BloodHound.py for Linux, SharpHound for Windows) that outputs JSON files, and the BloodHound GUI backed by a Neo4j graph database.
- Collected data is uploaded to the GUI and queried via built-in path-finding queries or custom Cypher queries to identify privilege escalation paths to Domain Admin.
- Even indirect or multi-hop paths (user → group → ACL → host → DA) are surfaced automatically, making it the fastest way to identify non-obvious escalation routes.
---
## Start Docker

```
sudo systemctl start docker
```

```
cd ~/bloodhound-ce
sudo docker compose up -d
```

Start the containers.

```
sudo docker logs bloodhound-ce-bloodhound-1 2>&1 | grep "Initial Password Set To:"
```

Checks initial password for loggin in at http://localhost:8080

```
sudo docker compose down -v
```

Stops the containers, when starting again use the new password with the command above.
## Collection - Linux

```
sudo bloodhound-python -u <user> -p <password> -ns <dc-ip> -d <domain> -c all
```

Run the BloodHound ingestor from a Linux attack host with valid domain credentials. `-ns` sets the nameserver to the domain controller, `-d` specifies the target domain, `-c all` runs all collection methods. Outputs JSON files in the current directory named by date and object type.

```
zip -r <out-file>.zip *.json
```

Compress all JSON output files into a single archive for upload to the BloodHound GUI.

---
## Collection - Windows

```powershell
.\SharpHound.exe -c All --zipfilename <out-file>
```

Run SharpHound from a Windows attack host or domain-joined machine. `-c All` runs all collection methods including Group, LocalAdmin, GPOLocalGroup, Session, ACL, Trusts, RDP, DCOM, SPNTargets, and PSRemote. `--zipfilename` sets the output archive name. Does not require the host to be domain-joined if valid credentials are available.

**SharpHound flags:**

|Flag|Description|
|---|---|
|`-c, --collectionmethods`|Collection methods to run. Default runs a standard subset. `All` runs everything.|
|`-d, --domain`|Specify the target domain.|
|`-s, --searchforest`|Search all domains in the forest.|
|`--stealth`|Prefer DC-only collection to reduce noise on endpoints.|
|`-f`|Append a custom LDAP filter to the pre-generated query.|
|`--distinguishedname`|Set a base DN to start the LDAP search from.|
|`--computerfile`|Path to a file containing computer names to enumerate.|

---
## GUI Setup

```
sudo neo4j start
```

Start the Neo4j database service before launching BloodHound. Required for the GUI to function.

```
bloodhound
```

Launch the BloodHound GUI. Log in with Neo4j credentials, upload the zip file via the `Upload Data` button, then use the `Analysis` tab for queries.

---
## Built-in Queries

| Query                                               | Use                                                                                                                                                                         |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Find Shortest Paths To Domain Admins`              | Maps all logical paths to DA via users, groups, ACLs, GPOs, and hosts. Primary escalation planning query.                                                                   |
| `Find Computers with Unsupported Operating Systems` | Identifies legacy hosts (Windows 7, Server 2008, etc.) susceptible to older RCE exploits like MS08-067. Validate they are live before attacking.                            |
| `Find Computers where Domain Users are Local Admin` | Identifies hosts where any domain account has local admin rights. Any controlled account can be used to access these hosts and potentially harvest credentials from memory. |