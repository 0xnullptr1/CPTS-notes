**Overview:**
- `LDAP` (Lightweight Directory Access Protocol) is a protocol used to access and manage directory information. A `directory` is a hierarchical data store containing information about network resources such as users, groups, computers, printers, and other devices.
- LDAP is commonly used to provide a central location for accessing and managing directory services, collections of information about the organisation, its users, and assets like usernames and passwords.
- `OpenLDAP` (open-source) and `Microsoft Active Directory` (Windows-based, integrates with Microsoft products).
- Although LDAP and AD are related, they serve different purposes. `LDAP` is a protocol that specifies how to access and modify directory services, whereas `AD` is a directory service that stores and manages user and computer data. LDAP works via a client-server architecture a client sends a request to a server, which searches the directory and returns a response.

---
## Enumeration

### Port scan

```bash
nmap -p- -sC -sV --open --min-rate=1000 <target>
```

Identify LDAP on port `389` (or `636` for LDAPS, `3268`/`3269` for Global Catalog).

### Basic ldapsearch query (template)

```bash
ldapsearch -H ldap://<target>:389 -D "cn=<user>,dc=<domain>,dc=<tld>" -w <password> -b "ou=<ou>,dc=<domain>,dc=<tld>" "(<filter>)"
```

`-H` target URI, `-D` bind DN (authenticate as), `-w` password, `-b` base DN to search under, last argument is the search filter (e.g. `(mail=john.doe@example.com)`).

example:

```bash
ldapsearch -H ldap://10.10.11.20:389 -D "cn=jdoe,dc=corp,dc=local" -w Passw0rd123 -b "ou=employees,dc=corp,dc=local" "(mail=jdoe@corp.local)"
```

---

## Anonymous bind

Many misconfigured LDAP/AD servers allow anonymous (unauthenticated) binds. Always test this first.

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local"
```

- `-x` simple authentication (no bind DN/password supplied = anonymous bind)

---

## Discover the naming context (base DN)


Query the RootDSE to find the base DN without any credentials.

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -s base -b "" "(objectclass=*)" namingContexts
```

---

## Dump the entire directory tree


```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=*)"
```

Authenticated version with discovered/valid credentials:

bash

```bash
ldapsearch -H ldap://10.10.11.20:389 -D "cn=jdoe,dc=corp,dc=local" -w Passw0rd123 -b "dc=corp,dc=local" "(objectclass=*)"
```

---

## Enumerate all users

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=user)"
```

Return only specific attributes (e.g. usernames and emails):


```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=user)" sAMAccountName mail
```

---

## Enumerate all groups


```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=group)" cn
```

## Enumerate group membership


```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(cn=Domain Admins)" member
```

---

## Enumerate computers

bash

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=computer)" cn dNSHostName
```

---

## Search for a specific attribute (e.g. description field password leaks)

bash

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(objectclass=user)" description
```

---

## Search for accounts with passwords never expiring / specific userAccountControl flags

bash

```bash
ldapsearch -H ldap://10.10.11.20:389 -x -b "dc=corp,dc=local" "(userAccountControl=66048)" sAMAccountName
```

---

## Using ldapwhoami to verify/test a bind

bash

```bash
ldapwhoami -H ldap://10.10.11.20:389 -D "cn=jdoe,dc=corp,dc=local" -w Passw0rd123
```

---

## LDAPS (LDAP over SSL/TLS, port 636)

bash

```bash
ldapsearch -H ldaps://10.10.11.20:636 -x -b "dc=corp,dc=local" "(objectclass=*)"
```

If certificate validation fails during testing:

bash

```bash
LDAPTLS_REQCERT=never ldapsearch -H ldaps://10.10.11.20:636 -x -b "dc=corp,dc=local" "(objectclass=*)"
```

---