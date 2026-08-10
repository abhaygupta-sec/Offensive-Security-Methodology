# Professional Penetration Testing Reporting

Professional reporting is the process of converting technical security-testing activity into a clear, reproducible, and actionable security assessment.

A strong penetration-test report should allow a technical reader to understand **what was discovered, why it mattered, how it was validated, what impact was demonstrated, and how the issue can be remediated**.

The methodology documented here focuses on clear attack-path documentation, reproducibility, technical reasoning, evidence handling, and actionable remediation.

---

## Reporting Philosophy

A penetration-test report should communicate:

```text
WHAT
What was discovered?

WHY
Why is it relevant?

HOW
How was it validated?

RESULT
What happened?

IMPACT
What can an attacker achieve?

REMEDIATION
How should the issue be fixed?

The goal is not to produce a command dump.

The goal is to produce a reproducible security narrative.

1. Report Structure

A professional penetration-test report can be structured as:

Executive Summary
        ↓
Scope
        ↓
Attack Path
        ↓
Methodology
        ↓
Findings Summary
        ↓
Detailed Technical Findings
        ↓
Exploitation
        ↓
Privilege Escalation
        ↓
Post-Exploitation / Impact
        ↓
Remediation
        ↓
Conclusion
        ↓
Appendix

Each section serves a different purpose.

2. Executive Summary

The executive summary should communicate the overall security outcome without requiring the reader to understand every technical detail.

Include:

Assessment objective
Scope
Overall result
Major security weaknesses
Highest-impact attack path
Business/security impact
High-level remediation direction

Avoid filling the executive summary with:

Tool commands
Long technical explanations
Raw terminal output
Excessive vulnerability details

The technical details belong later in the report.

3. Scope

Clearly define what was assessed.

Example:

Target:
<AUTHORIZED_TARGET>

Assessment Type:
Penetration Test

Testing Window:
<DATE>

Environment:
Authorized Laboratory / Client Environment

The scope should make it clear which systems were tested and under what authorization.

4. Attack Path

A short attack-path summary gives the reader an immediate understanding of how compromise occurred.

Example:

Initial Reconnaissance
        ↓
Service Enumeration
        ↓
Initial Access
        ↓
Credential Discovery
        ↓
Privilege Escalation
        ↓
Lateral Movement
        ↓
Administrative Access
        ↓
Impact Validation

For a specific assessment, replace the generic stages with the actual demonstrated attack chain.

5. Methodology

Explain how the assessment was performed.

A typical methodology includes:

Reconnaissance
      ↓
Enumeration
      ↓
Vulnerability Identification
      ↓
Validation
      ↓
Exploitation
      ↓
Privilege Escalation
      ↓
Lateral Movement
      ↓
Impact Validation
      ↓
Reporting

The methodology should describe the actual approach used during the assessment.

Do not claim testing activities that were not performed.

6. Findings Summary

Provide a high-level overview of confirmed findings.

Example:

ID	Finding	Severity	Status
F-01	Initial Access Vulnerability	Critical	Confirmed
F-02	Credential Exposure	High	Confirmed
F-03	Privilege Escalation	High	Confirmed
F-04	Excessive Permissions	Medium	Confirmed

Severity should reflect the demonstrated impact and risk.

Do not assign severity simply because a vulnerability has a particular CVE classification.

7. Finding Structure

Every technical finding should follow a consistent structure.

Recommended format:

Finding ID
Title
Severity
Affected Asset
Description
Technical Details
Evidence
Exploitation / Validation
Impact
Remediation
References

Example:

F-01 — Excessive Privileges

Severity:
High

Affected Asset:
<HOST>

Description:
The identified account possessed excessive privileges that allowed
unauthorized modification of a privileged resource.

Impact:
An attacker who obtains the account could leverage the permission
to obtain higher-privileged access.

Remediation:
Remove unnecessary privileges and apply least-privilege access controls.
8. Technical Step Format

For detailed attack steps, use:

WHAT
WHY
COMMAND / ACTION
RESULT
REASONING

Example:

WHAT:
SMB was exposed on the target.

WHY:
SMB can expose shares, credentials, scripts, and domain information.

COMMAND:
smbclient -L //<TARGET>

RESULT:
A readable share was identified.

REASONING:
The share was investigated because accessible files could contain
information relevant to further authentication or exploitation.

This format makes the report easier to follow and reproduce.

9. Evidence

Evidence should demonstrate that a finding actually exists.

Useful evidence may include:

Command output
HTTP requests/responses
Service configuration
File permissions
Authentication results
Relevant application behavior
Sanitized terminal output
Screenshots where appropriate

The evidence should support the claim being made.

Avoid including unnecessary output.

10. Evidence Handling

Public reports should never expose sensitive information.

Redact:

Passwords
NTLM hashes
API keys
Session tokens
Private keys
Access tokens
Flags
Personal information
Internal secrets
Sensitive configuration

Example:

Password:
[REDACTED]

NTLM Hash:
[REDACTED]

API Token:
[REDACTED]

The report should preserve enough information to demonstrate the finding without exposing reusable credentials.

11. Exploitation Documentation

When documenting exploitation, explain the complete chain.

Example:

Vulnerable Service
        ↓
Vulnerability Identification
        ↓
Exploit Validation
        ↓
Initial Access
        ↓
Current User
        ↓
Privilege Escalation
        ↓
Administrative Access

For each transition, explain why the next action was possible.

Avoid presenting an attack as:

Run command A
Run command B
Run command C
Root

Instead explain:

Finding
    ↓
Reasoning
    ↓
Action
    ↓
Result
    ↓
New Attack Path
12. Failed Attempts

Failed attempts can provide useful context when they demonstrate the testing methodology.

Document important failures when they help explain the final attack path.

Example:

Attempt:
Tested anonymous authentication.

Result:
Authentication was rejected.

Reasoning:
Anonymous access was therefore deprioritized and authenticated
enumeration was investigated instead.

This demonstrates decision-making rather than trial-and-error.

Do not include every irrelevant failed command.

13. Privilege Escalation Documentation

Privilege escalation should be documented as a separate attack stage.

Example:

Initial User
      ↓
Privilege Enumeration
      ↓
Misconfiguration Identified
      ↓
Validation
      ↓
Higher-Privilege Context
      ↓
Impact

Document:

Initial user
Initial privileges
Discovery
Vulnerability/misconfiguration
Validation
Resulting privilege
Impact
14. Active Directory Attack Chains

For Active Directory assessments, document relationships rather than isolated commands.

Example:

Low-Privilege Account
        ↓
User / Group Enumeration
        ↓
ACL Relationship
        ↓
Object Modification
        ↓
Higher Privilege
        ↓
Credential Discovery
        ↓
Remote Access
        ↓
Domain-Level Impact

For each relationship, explain:

Who controls what?
        ↓
Which permission enables it?
        ↓
What does the permission allow?
        ↓
What new access does it create?

This makes the report useful for both technical and security audiences.

15. Web Application Findings

Web findings should identify the exact affected functionality.

Include:

Application
Endpoint
HTTP Method
Parameter
Authentication Context
Vulnerability
Validation
Impact
Remediation

Example:

Endpoint:
POST /login

Parameter:
username

Issue:
Authentication behavior reveals whether a username exists.

Impact:
An attacker may enumerate valid accounts and use the information
to support further credential attacks.

Remediation:
Return consistent authentication responses and implement appropriate
rate limiting and monitoring.
16. Network Findings

Network findings should identify:

Host
Port
Protocol
Service
Version
Exposure
Vulnerability
Impact

Example:

Host:
<TARGET>

Port:
445/TCP

Service:
SMB

Observation:
A sensitive share was accessible to an unauthorized user.

Impact:
Information disclosure may enable further compromise.

Remediation:
Restrict share permissions and apply least privilege.
17. Remediation

Remediation should be specific to the finding.

Avoid vague recommendations such as:

"Improve security."
"Patch the system."
"Use stronger security."

Instead provide actionable recommendations.

Examples:

Remove unnecessary administrative privileges.

Restrict SMB shares to authorized users.

Disable anonymous access where it is not required.

Apply security patches to affected software.

Rotate exposed credentials.

Enforce least-privilege permissions.

Review Active Directory ACLs.

Remove unnecessary certificate enrollment permissions.

Implement appropriate authentication controls.

Validate authorization server-side.
18. Finding Severity

Severity should reflect the demonstrated risk.

A simple classification can be:

Severity	General Meaning
Critical	Direct path to severe compromise or domain/system-wide impact
High	Significant compromise or privilege escalation
Medium	Meaningful security weakness with limited or conditional impact
Low	Limited security impact
Informational	Security observation without direct demonstrated compromise

Severity should consider:

Exploitability
+
Privileges Required
+
Attack Complexity
+
Affected Assets
+
Confidentiality Impact
+
Integrity Impact
+
Availability Impact
=
Overall Risk
19. Attack Narrative

A strong report should tell a coherent story.

Example:

Reconnaissance
      ↓
Open Service Identified
      ↓
Version Enumeration
      ↓
Vulnerability Identified
      ↓
Initial Access
      ↓
Credential Discovery
      ↓
Privilege Escalation
      ↓
Administrative Access
      ↓
Impact Demonstrated

This is much easier to understand than a collection of unrelated findings.

20. Reproducibility

Another tester should be able to understand how the result was obtained.

Include:

Commands
Relevant parameters
Configuration
Preconditions
Expected behavior
Actual result
Reasoning

Avoid unexplained commands.

For example:

Command:
sudo nmap -p 80 -sC -sV <TARGET>

Why:
HTTP was identified during initial reconnaissance and required
deeper service enumeration.

Result:
The service version and web application were identified.

Next:
The application was manually enumerated.
21. Public Portfolio Reports

When publishing reports to GitHub:

Original Assessment
        ↓
Remove Sensitive Information
        ↓
Remove Credentials / Secrets
        ↓
Remove Flags
        ↓
Remove Private Infrastructure Details
        ↓
Review Screenshots / Evidence
        ↓
Publish Sanitized Report

The public version should demonstrate technical ability without exposing information that should remain private.

22. Professional Writing Style

Use:

Clear technical language
Short paragraphs
Consistent terminology
Reproducible steps
Evidence-based conclusions
Specific remediation
Clear attack paths

Avoid:

Excessive slang
Unnecessary storytelling
Unsupported assumptions
Large command dumps
Unverified scanner results
Unnecessary tool lists
Claims unsupported by evidence
23. Common Reporting Mistakes

Avoid:

Writing the report after forgetting the attack path
Reporting vulnerabilities without evidence
Including sensitive credentials
Claiming exploitation without validation
Copying scanner output directly into findings
Providing remediation that does not address the root cause
Mixing observations with assumptions
Omitting why a technique was selected
Failing to explain privilege transitions
Using inconsistent severity ratings
Publishing private client information
Turning a CTF walkthrough into a fake professional assessment
24. Reporting Workflow

My preferred workflow is:

During Assessment
       ↓
Record Important Findings
       ↓
Record Commands / Results
       ↓
Record Reasoning
       ↓
Build Attack Path
       ↓
Validate Impact
       ↓
Write Findings
       ↓
Add Remediation
       ↓
Sanitize Sensitive Information
       ↓
Technical Review
       ↓
Final Report

Reporting should not be an afterthought.

25. Report Quality Checklist

Before finalizing a report, verify:

Scope
 Scope is clearly defined
 Targets are identified
 Assessment type is documented
Technical Findings
 Every finding has evidence
 Findings are reproducible
 Impact is demonstrated
 Severity is justified
 Remediation is actionable
Attack Path
 Initial access is explained
 Privilege escalation is explained
 Lateral movement is explained where applicable
 Final impact is explained
Evidence
 Passwords removed
 Hashes removed
 Tokens removed
 Private keys removed
 Flags removed
 Sensitive infrastructure information removed
Quality
 No unsupported claims
 No unnecessary commands
 No unexplained technical steps
 Grammar reviewed
 Terminology consistent
 Attack chain is understandable
26. Reporting Philosophy

The objective is not:

Run Commands
    ↓
Copy Output
    ↓
Add Screenshots
    ↓
Submit Report

The objective is:

Observe
    ↓
Understand
    ↓
Validate
    ↓
Record
    ↓
Explain
    ↓
Demonstrate Impact
    ↓
Recommend Remediation

A strong penetration tester should be able to both find security weaknesses and communicate them professionally.

Related Projects
HTB Writeups

Practical machine assessments demonstrating technical exploitation and attack-path documentation.

HTB-Writeups

BlackField Professional Report

A sanitized professional penetration-test report demonstrating an end-to-end attack chain.

Offensive Security Methodology

See the repository root for the broader penetration-testing methodology.

Responsible Use

All reporting methodologies documented here are intended for:

Authorized penetration tests
Security laboratories
Hack The Box and other CTF environments
Security research
Systems owned or explicitly authorized for testing

Sensitive information obtained during assessments should never be published without authorization.
