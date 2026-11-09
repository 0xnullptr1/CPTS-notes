**Overview:**
- MySQL is an open-source relational database management system developed by Oracle. Data is organized into schemas containing tables of rows and columns, queried and manipulated using SQL.
- Follows a client-server model. The MySQL server handles storage and query execution. Clients connect via TCP port 3306, either over an internal network or the public internet.
- Commonly deployed as part of LAMP (Linux, Apache, MySQL, PHP) or LEMP (Linux, Nginx, MySQL, PHP) stacks, making it a frequent target in web application assessments.
- Errors returned by misconfigured web applications interacting with MySQL often leak query structure or schema details, providing a foothold for SQL injection.
---
## Dangerous Settings

|Setting|Description|
|---|---|
|`user`|OS user the MySQL service runs as|
|`password`|Password for the MySQL service user|
|`admin_address`|IP address the admin network interface listens on|
|`debug`|Current debug logging settings|
|`sql_warnings`|Controls whether INSERT statements return warnings|
|`secure_file_priv`|Restricts directories for file import/export operations|

---
## Enumeration

```bash
sudo nmap -sV -sC -p3306 --script mysql* <target>
```

Run all MySQL NSE scripts against the target. Returns version, auth methods, and exposed configuration details.

---
## Connection

```bash
mysql -u <user> -p<password> -h <target>
```

```bash
mysql -u admin -pPassword123! -h 127.0.0.1 customer_db
```

Connect to a remote MySQL server. No space between `-p` and the password.

---
## Commands Reference

|Command|Description|
|---|---|
|`show databases;`|List all databases on the server|
|`use <database>;`|Select a database|
|`show tables;`|List all tables in the selected database|
|`show columns from <table>;`|List all columns in a table|
|`select * from <table>;`|Dump all rows from a table|
|`select * from <table> where <column> = "<string>";`|Query a table for a specific value|