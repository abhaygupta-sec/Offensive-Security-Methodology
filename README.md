# Offensive Security Methodology

A structured collection of offensive-security methodologies, attack workflows, enumeration techniques, and penetration-testing practices developed through hands-on labs, Hack The Box, and OSCP preparation.

The goal of this repository is to document **how I approach a penetration test** — from initial reconnaissance and attack-surface mapping through exploitation, privilege escalation, pivoting, and professional reporting.

---

## Areas Covered

| Area | Focus |
|------|-------|
| 🔎 Reconnaissance | Network discovery, service enumeration, attack-surface mapping |
| 🌐 Web Application Security | HTTP testing, Burp Suite, FFUF, SQLi, XSS, SSTI, command injection, authentication testing |
| 🏢 Active Directory | LDAP, SMB, Kerberos, BloodHound, ACL abuse, AD CS, RBCD, credential attacks |
| 🐧 Linux Privilege Escalation | SUID, sudo, capabilities, cron, services, credentials |
| 🪟 Windows Privilege Escalation | Token privileges, services, scheduled tasks, registry, credentials |
| 🔑 Credential Attacks | Password attacks, Kerberoasting, AS-REP roasting, credential extraction |
| 🔀 Pivoting | SSH forwarding, SOCKS, Meterpreter routing, tunneling, port forwarding |
| 💥 Exploitation | Vulnerability validation, exploit selection, Metasploit, shell handling |
| 📝 Reporting | Attack paths, technical findings, evidence, impact, remediation |

---

## Methodology

My general penetration-testing workflow follows:

```text
Information Gathering
        ↓
Network / Application Enumeration
        ↓
Attack Surface Mapping
        ↓
Vulnerability & Misconfiguration Identification
        ↓
Initial Access
        ↓
Privilege Escalation
        ↓
Credential / Access Discovery
        ↓
Lateral Movement / Pivoting
        ↓
Impact Validation
        ↓
Professional Reporting
