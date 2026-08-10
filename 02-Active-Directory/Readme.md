# Active Directory Penetration Testing Methodology

Active Directory (AD) penetration testing focuses on understanding identities, authentication mechanisms, permissions, trust relationships, and the attack paths that can lead from a low-privileged account to administrative or domain-level compromise.

The methodology documented here is based on hands-on lab practice and focuses on **enumeration first, relationship analysis second, and exploitation only after an attack path has been identified**.

---

## Methodology

The general Active Directory workflow is:

```text
Initial Enumeration
        ↓
Domain / Host Discovery
        ↓
Users / Groups / Computers
        ↓
SMB / LDAP / Kerberos Enumeration
        ↓
Credential & Authentication Analysis
        ↓
BloodHound / ACL Analysis
        ↓
Identify Attack Path
        ↓
Initial Access / Privilege Escalation
        ↓
Lateral Movement
        ↓
Domain-Level Impact Validation

The key question is:

What does this account control, and what does that control allow me to reach next?

1. Identify the Active Directory Environment

When services such as LDAP, Kerberos, SMB, DNS, and domain-related Windows services are exposed, the target should be treated as a potential Active Directory environment.

Important information to identify:

Domain name
Domain Controller
Hostname
Domain users
Domain groups
Domain computers
SMB shares
LDAP information
Kerberos services
Authentication mechanisms

A typical attack-path model is:

Domain
   ↓
Users
   ↓
Groups
   ↓
Permissions
   ↓
Credentials / Tickets
   ↓
Remote Access
   ↓
Privilege Escalation
   ↓
Domain Compromise
2. SMB Enumeration

SMB is one of the most useful initial enumeration points in an Active Directory environment.

Start with share enumeration:

smbclient -L //<TARGET>

If a share is accessible:

smbclient //<TARGET>/<SHARE>

Investigate:

Share names
Anonymous access
Read permissions
Write permissions
Scripts
Backups
Configuration files
Usernames
Credentials
Domain information
SYSVOL
NETLOGON

A readable SMB share may provide information that enables a completely different attack path.

For example:

SMB Share
    ↓
Username Discovery
    ↓
Kerberos Enumeration
    ↓
Valid Account
    ↓
Credential Attack
3. LDAP Enumeration

LDAP can expose valuable information about the Active Directory environment.

Where anonymous LDAP access is permitted, investigate the information exposed by the directory.

Example:

ldapsearch -x -H ldap://<DC-IP> -b "DC=domain,DC=local"

Information of interest includes:

Domain naming information
Users
Groups
Computers
Organizational Units
Service accounts
Descriptions
Group membership
Potentially sensitive attributes

The goal is to understand the directory structure before attempting attacks against individual accounts.

4. Domain User Enumeration

User enumeration is important because usernames become inputs for later authentication attacks.

Potential sources include:

SMB shares
LDAP
RPC
Kerberos
SYSVOL
Application files
Scripts
BloodHound

Once candidate usernames are collected, validate them against the domain where permitted.

Example:

kerbrute userenum -d <DOMAIN> --dc <DC-IP> users.txt

The result can distinguish likely valid accounts from invalid candidates.

5. Kerberos Enumeration

Kerberos is a major component of Active Directory authentication.

Important questions include:

Which users exist?
        ↓
Which accounts have SPNs?
        ↓
Which accounts do not require pre-authentication?
        ↓
Which accounts may be privileged?
        ↓
Can obtained tickets be attacked offline?

Two important attack paths covered in this methodology are:

AS-REP Roasting
Kerberoasting
6. AS-REP Roasting

AS-REP roasting targets accounts where Kerberos pre-authentication is not required.

The general workflow is:

Valid Username
      ↓
Check Kerberos Pre-Authentication
      ↓
Request AS-REP
      ↓
Obtain AS-REP Material
      ↓
Offline Password Analysis
      ↓
Validate Recovered Credentials

Example:

impacket-GetNPUsers <DOMAIN>/<USER> -dc-ip <DC-IP> -no-pass

If AS-REP material is obtained, it can be subjected to offline password cracking in the authorized environment.

The important observation is:

The attack does not require the user's plaintext password to request the AS-REP material.

7. Kerberoasting

Kerberoasting targets service accounts associated with Service Principal Names (SPNs).

The general workflow is:

Authenticated Domain User
        ↓
Enumerate SPNs
        ↓
Identify Service Accounts
        ↓
Request Service Ticket
        ↓
Extract Ticket Material
        ↓
Offline Password Analysis
        ↓
Validate Recovered Credentials

Example enumeration:

impacket-GetUserSPNs <DOMAIN>/<USER>:<PASSWORD> -dc-ip <DC-IP>

The important question is:

Which service accounts have SPNs, and what privileges do those accounts have?

A weak service-account password combined with high privileges can create a significant attack path.

8. BloodHound

BloodHound is used to understand relationships inside Active Directory.

Typical data collection:

bloodhound-python -u <USER> -p '<PASSWORD>' -d <DOMAIN> -ns <DC-IP> -c all

The objective is not simply to collect data.

The objective is to identify:

User
 ↓
Group
 ↓
Computer
 ↓
Session
 ↓
ACL
 ↓
Privilege
 ↓
Domain Admin

Prioritize relationships such as:

GenericAll
GenericWrite
WriteDACL
ForceChangePassword
AddMember
CanRDP
CanPSRemote
Local Administrator
Session relationships
Group nesting
Delegation relationships
9. ACL Abuse

Active Directory permissions can create attack paths even when no software vulnerability exists.

Important permissions include:

GenericAll
GenericWrite
WriteDACL
ForceChangePassword
AddMember

The correct analysis is:

Who?
 ↓
Controls what object?
 ↓
With which permission?
 ↓
What can that permission change?
 ↓
What new access does that create?

For example:

Low-Privilege User
       ↓
GenericAll on Group
       ↓
Add User to Group
       ↓
Inherited Privileges
       ↓
Higher-Level Access

ACL analysis should therefore focus on effective impact, not merely unusual permissions.

10. Password / Account Management Abuse

Some Active Directory attack paths involve excessive account-management permissions.

Examples include:

Password reset permissions
Group membership modification
Account modification
Delegated permissions

The workflow is:

Authenticated User
       ↓
Permission Enumeration
       ↓
Identify Account-Control Relationship
       ↓
Determine What Can Be Modified
       ↓
Validate Impact
       ↓
Use New Access Context

Always determine why the operation is permitted before documenting it as a finding.

11. Windows Remote Access

Once valid credentials are obtained, identify available remote-management paths.

Common services include:

WinRM
RDP
SMB
WMI
MSSQL
WinRM

Example:

evil-winrm -i <TARGET> -u <USER> -p '<PASSWORD>'
RDP

Where authorized and available:

xfreerdp /v:<TARGET> /u:<USER> /p:'<PASSWORD>'
SMB / PsExec

Where appropriate:

Authenticated SMB
        ↓
Administrative Access?
        ↓
Remote Execution

The available protocol determines the appropriate next step.

12. Pass-the-Hash

When an NTLM hash is obtained and the target service accepts NTLM authentication, the hash may be usable directly without recovering the plaintext password.

Example:

evil-winrm -i <TARGET> -u <USER> -H <NTLM_HASH>

The important distinction is:

Password Authentication
        vs.
Hash-Based Authentication

A recovered hash should therefore be treated as credential material.

13. DCSync

DCSync abuses Active Directory replication functionality when an account has the necessary replication permissions.

The conceptual attack path is:

Compromised Account
       ↓
Replication Rights
       ↓
DCSync
       ↓
Domain Credential Material
       ↓
Credential-Based Authentication
       ↓
Domain-Level Impact

The important permissions should be reviewed before attempting replication attacks.

Examples include replication-related rights such as:

DS-Replication-Get-Changes
DS-Replication-Get-Changes-All

DCSync should only be performed in an authorized environment.

14. AD CS Enumeration

Active Directory Certificate Services can introduce additional authentication attack paths.

The methodology includes investigating:

Certificate Authorities
Certificate Templates
Enrollment permissions
Template configuration
Client authentication
Certificate-based authentication

Certipy can be used for authorized enumeration:

certipy find -u <USER>@<DOMAIN> -p '<PASSWORD>' -dc-ip <DC-IP>

The notes cover certificate-service attack paths including:

ESC1
ESC2
ESC3
ESC4
ESC6
ESC7
ESC8
ESC9
ESC10
Shadow Credentials

The important workflow is:

Enumerate AD CS
      ↓
Identify Misconfigured Template / Permission
      ↓
Determine Who Can Enroll / Modify
      ↓
Request or Modify Certificate
      ↓
Certificate Authentication
      ↓
Validate Resulting Access
15. MachineAccountQuota (MAQ)

MachineAccountQuota can allow a domain user to create computer accounts when the domain configuration permits it.

The attack path documented in the lab material is:

Low-Privilege Domain Account
        ↓
Check MachineAccountQuota
        ↓
Create Machine Account
        ↓
Certificate / Authentication Abuse
        ↓
Obtain Higher-Value Identity
        ↓
RBCD
        ↓
Service Ticket
        ↓
DCSync
        ↓
Domain-Level Impact

Example tools used in the documented lab workflows include:

BloodyAD
Certipy
Impacket

The exact technique depends on the permissions and configuration discovered during enumeration.

16. Resource-Based Constrained Delegation

RBCD can create an attack path when an attacker can modify the appropriate computer object's delegation configuration.

Conceptually:

Control over Computer Object
        ↓
Modify msDS-AllowedToActOnBehalfOfOtherIdentity
        ↓
Configure RBCD
        ↓
Request Service Ticket
        ↓
Impersonate Target Identity
        ↓
Remote Access

The important question is:

Which principal can modify the target computer object's delegation configuration?

17. Shadow Credentials

Shadow Credentials involve modifying the msDS-KeyCredentialLink attribute when the attacker has sufficient rights over the target object.

Conceptually:

Object Control
      ↓
Modify KeyCredentialLink
      ↓
Add Attacker-Controlled Key
      ↓
Certificate-Based Authentication
      ↓
Obtain Access as Target

The technique should be evaluated as an ACL/identity-control problem rather than treated as an isolated exploit.

18. SYSVOL / Group Policy Enumeration

SYSVOL and NETLOGON can contain scripts and configuration information useful for enumeration.

Investigate:

Logon scripts
Group Policy files
Scheduled scripts
Configuration files
Historical credentials
GPP artifacts

The important question is:

Does a domain-controlled file expose credentials or information that can be reused elsewhere?

19. Attack Path Construction

The strongest AD findings usually come from chaining several smaller weaknesses.

Example:

Anonymous SMB
      ↓
Username Discovery
      ↓
Valid Domain Users
      ↓
AS-REP Roasting
      ↓
Credential Recovery
      ↓
Authenticated Enumeration
      ↓
ACL / Account Permission Abuse
      ↓
Sensitive Resource Access
      ↓
Credential Extraction
      ↓
Remote Access
      ↓
Privilege Escalation
      ↓
Domain-Level Impact

This is more valuable than simply listing individual vulnerabilities.

20. AD Enumeration Checklist

When approaching a new Active Directory environment, review:

Domain
Domain name
Domain Controller
DNS
LDAP
Kerberos
SMB
Users
Valid usernames
Privileged users
Service accounts
Password policy
Descriptions
Disabled accounts
AS-REP roastable users
Groups
Domain Admins
Enterprise Admins
Administrators
Nested groups
Custom privileged groups
Services
SPNs
Kerberoastable accounts
WinRM
RDP
SMB
MSSQL
WMI
Shares
SYSVOL
NETLOGON
User shares
Backups
Scripts
Configuration files
Permissions
GenericAll
GenericWrite
WriteDACL
ForceChangePassword
AddMember
RBCD-related permissions
Certificate-template permissions
AD CS
Certificate Authorities
Templates
Enrollment permissions
Misconfigured templates
Certificate authentication paths
Post-Compromise
Current user
Groups
Sessions
Tokens
Privileges
Credential material
Domain replication rights
21. Common Mistakes

Avoid:

Starting exploitation before understanding the domain
Enumerating only users and ignoring relationships
Ignoring SMB shares
Ignoring LDAP
Ignoring Kerberos
Running BloodHound without analyzing the resulting relationships
Looking only for Domain Admin membership
Ignoring ACLs
Ignoring service accounts
Ignoring SPNs
Ignoring AD CS
Ignoring machine-account permissions
Treating a recovered hash like ordinary text
Failing to identify how a privilege was obtained
Performing DCSync without first understanding replication permissions
Assuming every AD environment has the same attack path
22. AD Attack-Path Mindset

The objective is not:

Run BloodHound
→ Find Domain Admin
→ Done

The objective is:

Enumerate
   ↓
Understand Relationships
   ↓
Identify Controllable Object
   ↓
Determine Effective Permission
   ↓
Build Attack Path
   ↓
Validate Each Step
   ↓
Increase Access
   ↓
Measure Impact

Active Directory penetration testing is fundamentally a problem of identity, permissions, relationships, and trust.

23. Reporting AD Findings

For every important Active Directory finding, document:

WHAT
What object / permission / configuration was discovered?

WHY
Why does it matter?

HOW
How was it validated?

RESULT
What access or impact was demonstrated?

NEXT
What attack path did it enable?

Example:

User
  ↓
ACL Permission
  ↓
Object Modification
  ↓
New Privilege
  ↓
Remote Access
  ↓
Further Enumeration

This makes the attack chain understandable to both technical reviewers and interviewers.

Related Projects
HTB Writeups

Practical machine assessments demonstrating Active Directory enumeration, exploitation, and privilege escalation.

HTB-Writeups

BlackField Professional Report

A sanitized professional penetration-test report demonstrating an end-to-end Windows/Active Directory compromise.

Offensive Security Methodology

See the repository root for the broader penetration-testing methodology.

Responsible Use

All techniques documented here are intended for:

Authorized penetration tests
Active Directory laboratories
Hack The Box and other CTF environments
Security research
Systems owned or explicitly authorized for testing

Never use these techniques against systems without authorization.
