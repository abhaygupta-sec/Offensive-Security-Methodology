# Reconnaissance Methodology

Reconnaissance is the first stage of a penetration test.

The objective is not simply to find open ports. The objective is to build an accurate model of the target's **attack surface**, identify interesting services, and determine where deeper enumeration should be performed.

---

## Methodology

My reconnaissance workflow generally follows:

```text
Target
  ↓
TCP Port Discovery
  ↓
Service / Version Enumeration
  ↓
Service-Specific Enumeration
  ↓
UDP Discovery
  ↓
Targeted Vulnerability / NSE Validation
  ↓
Attack Surface Map
  ↓
Initial Attack Hypotheses

The important principle is:

Enumerate first. Exploit second.

An open port is not automatically a vulnerability. Every discovered service should lead to a specific enumeration question.

1. Initial Nmap Scan

I start with a broad scan to identify exposed TCP services and their versions.

sudo nmap --top-ports 1000 -sC -sV -sS -T4 -Pn -n <TARGET>
What

Identify commonly exposed TCP ports, services, versions, and useful default-script information.

Why

This provides the initial attack-surface map and helps determine which services require deeper enumeration.

Important Information

Review:

Open ports
Service names
Service versions
Hostnames
OS indicators
Default NSE results
Authentication services
Potentially unusual ports
2. Full TCP Port Discovery

If the initial scan does not reveal enough information, expand the TCP port range.

sudo nmap -p- -sS -T4 -Pn -n <TARGET>

The purpose is to avoid relying only on commonly scanned ports.

A service running on a non-standard port can otherwise remain completely invisible.

3. Targeted Service Enumeration

After identifying open ports, perform deeper enumeration against those ports.

sudo nmap -p <PORTS> -sC -sV -Pn -n <TARGET>

Example:

sudo nmap -p 22,80,445 -sC -sV -Pn -n <TARGET>
Decision Process
Port discovered
      ↓
Identify service
      ↓
Identify version
      ↓
Understand functionality
      ↓
Enumerate service manually
      ↓
Look for credentials / misconfigurations / vulnerabilities

The goal is to transition from port discovery to service understanding.

4. Service-Specific Enumeration

Every interesting service should be investigated according to its functionality.

Service	Initial Questions
HTTP/HTTPS	What web application is running? What directories/endpoints exist?
SMB	Are shares exposed? Is anonymous access possible?
LDAP	Is anonymous enumeration possible? What domain information is exposed?
FTP	Is anonymous access enabled? Can files be read or written?
SSH	What authentication methods are available?
DNS	Can records or zone information be enumerated?
SNMP	Is community-string-based information disclosure possible?
WinRM	Is remote management exposed and are valid credentials available?
MSSQL	Is authentication required? What database functionality is exposed?

The exact enumeration path depends on the service discovered.

5. HTTP Reconnaissance

For web services, reconnaissance moves from service identification into application mapping.

Identify the Application

Review:

HTTP headers
Server information
Page titles
Technologies
Cookies
Authentication mechanisms
Redirects
Error messages
Content Discovery

Tools such as FFUF can be used to identify hidden resources.

Example:

ffuf -u http://<TARGET>/FUZZ -w <WORDLIST>

Interesting results should then be manually validated.

Look for:

/admin
/login
/api
/uploads
/backup
/.git
/config

The discovery result itself is not enough; determine what the endpoint does and whether it changes the attack surface.

6. SMB Reconnaissance

When SMB is exposed, determine whether anonymous or authenticated enumeration is possible.

smbclient -L //<TARGET>

If a share is accessible:

smbclient //<TARGET>/<SHARE>

Investigate:

Share names
Read permissions
Write permissions
Files
Scripts
Backups
Configuration files
Usernames
Credentials
SYSVOL / domain information

A readable share may provide information required for a completely different attack path.

7. LDAP / Active Directory Reconnaissance

When LDAP and other AD services are exposed, determine:

Domain
  ↓
Domain Controller
  ↓
Users
  ↓
Groups
  ↓
Computers
  ↓
Shares
  ↓
Authentication mechanisms
  ↓
Relationships / permissions

Where anonymous LDAP access is permitted, investigate the information exposed by the directory.

For authenticated environments, enumeration can be expanded using tools such as BloodHound and LDAP-specific tooling.

8. UDP Reconnaissance

TCP enumeration should not be treated as the complete network picture.

UDP services can expose additional attack surfaces.

A basic UDP discovery scan:

sudo nmap -sU --top-ports 100 -T4 -Pn -n <TARGET>

If UDP ports are discovered, perform targeted enumeration:

sudo nmap -sU -p <UDP_PORTS> -sC -sV -Pn -n <TARGET>

UDP scans are generally slower than TCP scans, so targeted follow-up is preferred when possible.

9. NSE / Vulnerability Validation

Nmap NSE scripts can help identify potential vulnerabilities and additional service information.

Example:

sudo nmap -p <PORTS> --script vuln -Pn -n <TARGET>

NSE output should be treated as a lead requiring validation, not automatic proof of vulnerability.

NSE Finding
    ↓
Understand Finding
    ↓
Manually Validate
    ↓
Determine Impact

Scanners suggest. Humans validate.

10. Six-Phase Automated Reconnaissance

My NmapTool project implements a structured six-phase reconnaissance workflow:

Phase 1
TCP Discovery
     ↓
Phase 2
TCP Deep Enumeration
     ↓
Phase 3
TCP Vulnerability Scan
     ↓
Phase 4
UDP Discovery
     ↓
Phase 5
UDP Deep Enumeration
     ↓
Phase 6
UDP Vulnerability Scan

The tool uses the results of earlier phases to drive later enumeration.

For example, discovered TCP ports are used for targeted TCP deep scanning, while discovered UDP ports are used for UDP deep enumeration.

11. Reconnaissance Decision Tree

When a new target is encountered:

              Target
                 |
                 v
          Port Discovery
                 |
        +--------+--------+
        |                 |
       TCP               UDP
        |                 |
        v                 v
 Service / Version    Service / Version
 Enumeration          Enumeration
        |                 |
        +--------+--------+
                 |
                 v
        Service-Specific
          Enumeration
                 |
                 v
       Identify Attack Surface
                 |
        +--------+--------+
        |        |        |
     Web       SMB/AD   Other
        |        |        |
        v        v        v
     Deeper   Identity  Protocol
     Testing  / Access  Testing
                 |
                 v
        Attack Hypothesis
12. What I Look For

During reconnaissance, I prioritize findings that can lead to:

Information Disclosure
Usernames
Hostnames
Domains
Software versions
Configuration files
Backups
Source code
Credentials
Authentication Weaknesses
Anonymous access
Default credentials
Weak authentication
Kerberos weaknesses
Exposed authentication endpoints
Access-Control Issues
Readable sensitive shares
Writable directories
Excessive permissions
Anonymous functionality
Potential Exploitation Paths
Outdated software
Known vulnerabilities
Dangerous service configurations
Web application weaknesses
Credential exposure
Privilege relationships
13. Reconnaissance Notes

For every important discovery, record:

WHAT
Why was this service / endpoint interesting?

WHY
What attack hypothesis does it create?

COMMAND
What was used to enumerate it?

RESULT
What was actually discovered?

NEXT STEP
What does the result tell me to investigate next?

This prevents reconnaissance from becoming a random collection of commands.

14. Handling Large Scans

Large scans can take significantly longer, particularly UDP scans.

For long-running authorized scans, the workflow can be split into stages:

Full TCP Discovery
        ↓
Review Results
        ↓
Deep Scan Interesting Services
        ↓
UDP Discovery
        ↓
Targeted UDP Enumeration

Long-running scans can also be executed inside screen or another persistent terminal session.

Example:

screen -S full_scan
sudo nmap -p- -sS -T4 -Pn -n <TARGET>

Detach:

Ctrl+A
D

Reattach:

screen -r full_scan
15. Output & Documentation

Reconnaissance results should be preserved so that findings can be reproduced later.

My NmapTool preserves:

Raw Nmap output
XML results
Phase-specific output
HTML reports
Consolidated reports
JSON summaries

The project separates:

TCP Discovery
TCP Deep Enumeration
TCP Vulnerability Scan
UDP Discovery
UDP Deep Enumeration
UDP Vulnerability Scan

This makes individual scan stages easier to review and reproduce.

16. Common Mistakes

Avoid:

Scanning only the top few ports and stopping
Ignoring UDP
Running vulnerability scanners without understanding the results
Failing to enumerate service versions
Ignoring non-standard ports
Treating every open port as exploitable
Skipping manual service enumeration
Forgetting SMB shares
Forgetting LDAP / AD enumeration
Ignoring HTTP history and hidden endpoints
Moving to exploitation without understanding the attack surface
17. Reconnaissance Philosophy

The objective is not:

Run Nmap
→ Find port
→ Run exploit

The objective is:

Discover
→ Understand
→ Enumerate
→ Correlate
→ Form Hypothesis
→ Validate
→ Exploit

Good reconnaissance reduces blind exploitation and makes the rest of the penetration test more deliberate.

Related Projects
NmapTool

Automated six-phase TCP/UDP reconnaissance and reporting:

NmapTool-Automated-Recon

HTB Writeups

Practical machine assessments demonstrating reconnaissance and exploitation:

HTB-Writeups

Responsible Use

All reconnaissance techniques documented here are intended for:

Authorized penetration tests
Security laboratories
CTF environments
Security research
Systems owned or explicitly authorized for testing

Never scan systems or networks without authorization.
