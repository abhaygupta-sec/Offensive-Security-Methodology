# Privilege Escalation Methodology

Privilege escalation is the process of moving from an existing low-privileged foothold to a higher-privileged account or execution context.

The objective is not to immediately search for a single exploit. The objective is to understand the current privilege level, enumerate the host, identify misconfigurations or accessible privileged functionality, and determine which path provides meaningful privilege escalation.

---

## Methodology

My general privilege-escalation workflow follows:

```text
Initial Shell
    ↓
Identify Current User
    ↓
Enumerate Privileges
    ↓
Enumerate Host Configuration
    ↓
Identify Interesting Files / Services / Processes
    ↓
Identify Credentials / Secrets
    ↓
Identify Misconfigurations
    ↓
Select Escalation Path
    ↓
Validate Privilege Escalation
    ↓
Confirm New Privilege Level

The core principle is:

Enumerate the environment before choosing the escalation technique.

1. Identify the Current Context

After obtaining a shell, first determine who you are and what context the shell is running under.

Linux
id
whoami
groups
hostname
uname -a

Review:

Current user
Group membership
Hostname
Operating system
Kernel version
Available privileges
Windows
whoami
whoami /groups
whoami /priv
hostname
systeminfo

Review:

Current user
Domain
Group membership
Enabled privileges
Operating system version
Host configuration

The current context determines which escalation paths are relevant.

2. Linux Privilege Escalation

The Linux workflow can be represented as:

Current User
     ↓
sudo
     ↓
SUID / SGID
     ↓
Capabilities
     ↓
Cron / Timers
     ↓
Services
     ↓
Writable Files
     ↓
Credentials
     ↓
Kernel / Application Weaknesses

No single category should be checked in isolation.

3. Linux sudo Enumeration

Check whether the current user can execute commands with elevated privileges.

sudo -l

Review:

Commands that can be executed
Whether a password is required
Allowed arguments
Environment restrictions
Whether the command can execute another program

The important question is:

What does the allowed command actually permit me to execute or control?

A sudo entry is not automatically exploitable.

4. SUID / SGID Enumeration

SUID binaries can execute with the permissions of their owner.

Search for SUID binaries:

find / -perm -4000 -type f 2>/dev/null

Search for SGID binaries:

find / -perm -2000 -type f 2>/dev/null

For each unusual binary, determine:

What is the binary?
        ↓
Who owns it?
        ↓
What version?
        ↓
Can input be controlled?
        ↓
Can it execute another program?
        ↓
Is there a known abuse path?

Do not assume that every SUID binary provides privilege escalation.

5. Linux Capabilities

Capabilities can provide powerful privileges without requiring a traditional SUID configuration.

Enumerate capabilities:

getcap -r / 2>/dev/null

Interesting capabilities should be investigated in context.

The workflow is:

Capability Found
      ↓
Identify Binary
      ↓
Understand Capability
      ↓
Determine How Input Is Controlled
      ↓
Validate Privilege Impact
6. Cron Jobs and Scheduled Tasks

Scheduled jobs can create escalation opportunities when privileged processes execute attacker-controlled files.

Review scheduled tasks:

cat /etc/crontab
ls -la /etc/cron.*

Look for:

Writable scripts
Writable directories
Relative paths
Weak permissions
Jobs executing as root
User-controlled files referenced by privileged tasks

The important relationship is:

Privileged Scheduled Task
        ↓
Attacker-Controlled File / Command
        ↓
Privileged Execution
7. Linux Services

Enumerate running services and their configuration.

Useful commands include:

systemctl list-units --type=service

And:

ps aux

Investigate:

Service binaries
Service configuration
File permissions
Environment variables
Credentials
Writable service files
Custom services

The goal is to identify whether a privileged service depends on something the current user can modify.

8. Writable Files and Directories

Search for writable locations that are used by privileged processes.

Review:

/etc
Service directories
Scheduled-task scripts
Application directories
Configuration files
Startup scripts
Custom binaries

A useful reasoning model is:

Privileged Process
       ↓
Uses File / Script
       ↓
Current User Can Modify File
       ↓
Influence Privileged Execution
9. Credential Discovery on Linux

Credentials may be present in:

Configuration files
Environment variables
Shell history
Application files
Backup files
SSH keys
Database configuration
Web application configuration

Examples:

env
history
find /home -name ".*history" 2>/dev/null

Search application directories carefully for configuration and credential material.

Credentials found during authorized testing should be treated as sensitive.

10. Linux Kernel / Application Weaknesses

Kernel or application vulnerabilities may provide another escalation path.

Before considering a kernel exploit:

Identify OS
     ↓
Identify Kernel
     ↓
Identify Architecture
     ↓
Identify Relevant Vulnerability
     ↓
Determine Exploit Compatibility
     ↓
Validate in Authorized Environment

Do not blindly run public exploits.

A working exploit must match the target's actual configuration.

11. Windows Privilege Escalation

The Windows workflow is:

Current User
     ↓
Groups
     ↓
Privileges
     ↓
Services
     ↓
Scheduled Tasks
     ↓
Registry
     ↓
Credentials
     ↓
Applications
     ↓
Writable Paths
     ↓
Token / Privilege Abuse
     ↓
Administrative Context
12. Windows User and Group Enumeration

Start with:

whoami
whoami /groups

Review:

Local groups
Domain groups
Administrators membership
Remote-management groups
Special privileges

Additional enumeration:

net user
net localgroup
net localgroup administrators

The objective is to understand the current user's effective permissions.

13. Windows Token Privileges

Enumerate enabled privileges:

whoami /priv

Important privileges covered in the methodology include:

SeImpersonatePrivilege
SeBackupPrivilege
SeRestorePrivilege

The correct workflow is:

Privilege Identified
       ↓
Is It Enabled?
       ↓
What Does It Permit?
       ↓
What Resources Can It Access?
       ↓
Can That Access Be Abused?
       ↓
Validate Privilege Escalation

A privilege being listed does not automatically mean exploitation is possible.

14. SeImpersonatePrivilege

When SeImpersonatePrivilege is enabled, investigate whether the current context can abuse Windows impersonation mechanisms.

The general reasoning is:

Current Account
      ↓
SeImpersonatePrivilege
      ↓
Privileged Token / Service Interaction
      ↓
Token Impersonation
      ↓
Higher-Privilege Context

The exact technique depends on the target configuration and available services.

15. SeBackupPrivilege

SeBackupPrivilege can provide access to protected data when appropriately enabled.

The documented workflow includes:

Check Privilege
      ↓
Identify SeBackupPrivilege
      ↓
Access Protected Registry Hives
      ↓
Save SAM / SYSTEM
      ↓
Extract Credential Material
      ↓
Validate Higher-Privilege Access

Example commands used in authorized lab environments:

whoami /priv
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM

The resulting files can then be analyzed offline.

16. SeRestorePrivilege

SeRestorePrivilege should be investigated when it is enabled because it can allow privileged file and registry operations that ordinary users cannot perform.

The workflow is:

Identify Privilege
      ↓
Understand Restore Semantics
      ↓
Identify Writable / Replaceable Resource
      ↓
Determine Privileged Execution Context
      ↓
Validate Impact

The exact escalation path depends on the services, files, and permissions present on the target.

17. Windows Services

Services are a common privilege-escalation category.

Enumerate services:

sc query

Inspect a specific service:

sc qc <SERVICE>

Review:

Service account
Binary path
Start type
Service permissions
Binary permissions
Configuration permissions

Potential attack path:

Privileged Service
      ↓
Writable Service Configuration
      ↓
Controlled Binary / Command
      ↓
Service Restart
      ↓
Privileged Execution
18. Scheduled Tasks

Enumerate scheduled tasks:

schtasks /query /fo LIST /v

Look for:

Tasks running as SYSTEM
Executables in writable directories
Scripts controlled by low-privileged users
Weak file permissions
Interesting arguments
Unusual scheduled execution

The important question is:

Can the current user influence something executed by a privileged scheduled task?

19. Registry Enumeration

Review registry configuration where relevant.

Potential areas include:

Service configuration
Application configuration
Stored credentials
Auto-run locations
Startup configuration

Example:

reg query HKLM

Do not blindly dump the entire registry.

Target enumeration toward a specific hypothesis.

20. Windows Credential Discovery

Credential material can appear in:

Application configuration
PowerShell history
Registry
Scheduled tasks
Scripts
Configuration files
User profiles
LSASS
SAM / SYSTEM

The methodology includes tools such as:

Mimikatz
pypykatz
secretsdump
PowerShell-based enumeration

Sensitive credential material should never be committed to a public repository.

21. LSASS Credential Extraction

When authorized post-exploitation conditions permit access to an LSASS dump, analyze it offline.

Example:

pypykatz lsa minidump lsass.dmp

Conceptual workflow:

LSASS Dump
    ↓
Offline Analysis
    ↓
Credential Material
    ↓
Identify Account
    ↓
Validate Credential
    ↓
Determine Access

A credential obtained from LSASS can become a lateral-movement or privilege-escalation path depending on the account's permissions.

22. SAM / SYSTEM Extraction

When protected registry hives are legitimately obtained during an authorized assessment:

SAM
+
SYSTEM
    ↓
Offline Credential Extraction
    ↓
NTLM Credential Material
    ↓
Authentication Testing

Example:

secretsdump.py -sam SAM -system SYSTEM LOCAL

Treat extracted hashes as highly sensitive credentials.

23. Pass-the-Hash

If an NTLM hash is recovered, some Windows services may allow authentication using the hash directly.

Example:

evil-winrm -i <TARGET> -u <USER> -H <NTLM_HASH>

The workflow is:

Hash Obtained
     ↓
Identify Account
     ↓
Determine Account Privileges
     ↓
Identify Compatible Service
     ↓
Authenticate
     ↓
Enumerate New Context

A hash should therefore be considered credential material even when the plaintext password is unknown.

24. Windows Post-Exploitation Enumeration

After obtaining a new shell, restart enumeration.

whoami
whoami /groups
whoami /priv
hostname
ipconfig

Then determine:

Current privileges
Network interfaces
Domain membership
Local administrators
Available sessions
Accessible shares
Other users
Security products
Remote-management services

Privilege escalation is iterative.

A new account or context can expose a completely different attack surface.

25. Automated Enumeration

Automated enumeration tools can accelerate discovery.

Examples include:

WinPEAS
PowerUp
LinPEAS
BloodHound
PowerView

The correct workflow is:

Run Enumeration Tool
       ↓
Review Findings
       ↓
Prioritize Interesting Results
       ↓
Manually Validate
       ↓
Build Attack Path

Do not copy every scanner result into a report.

26. Privilege Escalation Decision Tree
Initial Shell
     |
     v
Who Am I?
     |
     v
What Privileges Do I Have?
     |
     +---------------------------+
     |                           |
   Linux                       Windows
     |                           |
     v                           v
 sudo                       whoami /priv
 SUID                       Services
 Capabilities               Scheduled Tasks
 Cron                       Registry
 Services                   Credentials
 Credentials               Writable Paths
     |                           |
     +-------------+-------------+
                   |
                   v
          Identify Misconfiguration
                   |
                   v
            Validate Exploit Path
                   |
                   v
          Higher-Privilege Shell
                   |
                   v
             Re-enumerate
27. Common Mistakes

Avoid:

Running an exploit immediately after obtaining a shell
Ignoring the current user's groups
Ignoring enabled privileges
Checking only one privilege-escalation category
Running automated tools without understanding the results
Ignoring writable files
Ignoring scheduled tasks
Ignoring services
Ignoring credentials
Ignoring application configuration
Blindly using kernel exploits
Treating every SUID binary as exploitable
Treating every Windows privilege as automatically exploitable
Failing to re-enumerate after gaining new credentials
Publishing recovered credentials or hashes
28. Privilege Escalation Mindset

The objective is not:

Get Shell
    ↓
Run PrivEsc Tool
    ↓
Find Exploit
    ↓
Root / Administrator

The objective is:

Get Shell
    ↓
Understand Context
    ↓
Enumerate
    ↓
Identify Weakness
    ↓
Understand Why It Works
    ↓
Validate
    ↓
Escalate
    ↓
Re-enumerate

The most important question is:

What can my current security context control that a privileged process also trusts?

29. Reporting Privilege Escalation

Every escalation step should document:

WHAT
What privilege or misconfiguration was discovered?

WHY
Why does it matter?

COMMAND / ACTION
How was it validated?

RESULT
What changed?

IMPACT
What privilege was obtained?

NEXT STEP
What access became available?

Example:

Low-Privilege Account
        ↓
Enabled Privilege
        ↓
Protected Resource Access
        ↓
Credential Material
        ↓
Higher-Privilege Authentication
        ↓
Administrative Access

This makes the escalation path reproducible and understandable.

Related Projects
HTB Writeups

Practical machine assessments demonstrating Linux and Windows privilege escalation.

HTB-Writeups

BlackField Professional Report

A sanitized professional penetration-test report demonstrating Windows credential extraction, SeBackupPrivilege abuse, and administrative compromise.

Offensive Security Methodology

See the repository root for the broader penetration-testing methodology.

Responsible Use

All techniques documented here are intended for:

Authorized penetration tests
Security laboratories
Hack The Box and other CTF environments
Security research
Systems owned or explicitly authorized for testing

Never use privilege-escalation techniques against systems without authorization.
