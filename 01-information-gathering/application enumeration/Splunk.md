- Splunk is a log analytics tool used to gather, analyze and visualize data. Though not originally intended to be a SIEM tool, Splunk is often used for security monitoring and business analytics.
- Splunk deployments are often used to house sensitive data and could provide a wealth of information for an attacker if compromised.
- Splunk is prevalent in internal networks and often runs as root on Linux or SYSTEM on Windows systems. 
- The biggest focus of Splunk during an assessment would be weak or null authentication because admin access to Splunk gives  the ability to deploy custom applications that can be used to compromise a Splunk server and possibly other hosts in the network.
---
## Discovery

Splunk web runs on port `8000` by default. Port `8089` is the management port for the Splunk REST API.

```bash
sudo nmap -sV <target>
```

Identify Splunk via `Splunkd httpd` on ports 8000 and 8089.

---
## Default Credentials

Older versions use `admin:changeme` displayed on the login page. The free version (auto-converted after 60-day trial) requires no authentication at all. Try common weak passwords if defaults fail: `admin`, `Welcome`, `Welcome1`, `Password123`.