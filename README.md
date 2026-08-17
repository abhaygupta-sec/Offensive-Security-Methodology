# Offensive Security Methodology

A structured collection of offensive-security methodologies, attack workflows, enumeration techniques, and penetration-testing practices developed through hands-on labs, Hack The Box, and PEN-200 preparation.

The purpose of this repository is to document **how I approach a penetration test** — from reconnaissance and attack-surface mapping through exploitation, privilege escalation, pivoting, and professional reporting.

> **Important:** This repository documents learning and authorized lab practice. It is not a record of professional client engagements.

---

## 🧭 Methodology at a Glance

```text
Information Gathering
        ↓
Network / Application Enumeration
        ↓
Attack-Surface Mapping
        ↓
Vulnerability & Misconfiguration Identification
        ↓
Initial Access
        ↓
Post-Compromise Enumeration
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
```

The emphasis is on **decision-making and attack-chain development**, not simply collecting commands.

## 📚 Areas Covered

| Section | Focus |
| --- | --- |
| 🔎 [01 — Reconnaissance](01-Reconnaissance) | Network discovery, service enumeration, and attack-surface mapping |
| 🏢 [02 — Active Directory](02-Active-Directory) | LDAP, SMB, Kerberos, BloodHound, ACL abuse, AD CS, RBCD, and credential attacks |
| 🌐 [03 — Web Application Security](03-Web-Application%20Security) | HTTP testing, Burp Suite, FFUF, SQLi, XSS, SSTI, command injection, and authentication testing |
| 🐧 [04 — Privilege Escalation](04-Privilege-Escalation) | Linux and Windows privilege-escalation methodology |
| 🔀 [05 — Pivoting](05-Pivoting) | SSH forwarding, SOCKS, tunneling, routing, and port forwarding |
| 💥 [06 — Exploitation Workflow](06-Exploitation-Workflow) | Vulnerability validation, exploit selection, Metasploit, and shell handling |
| 📝 [07 — Reporting](07-Reporting) | Attack paths, technical findings, evidence, impact, and remediation |

## 🔍 How I Use the Methodology

A typical lab workflow is:

1. Establish the attack surface.
2. Enumerate services and applications systematically.
3. Identify vulnerabilities and misconfigurations.
4. Form a testable exploitation hypothesis.
5. Validate the hypothesis safely in the authorized lab.
6. Re-enumerate after gaining access.
7. Identify privilege-escalation and credential opportunities.
8. Continue the attack chain where applicable.
9. Document evidence, impact, and the path taken.

## 🎯 Portfolio Context

This repository complements my other security work:

- **HTB Writeups:** detailed machine-level attack walkthroughs.
- **NmapTool-Automated-Recon:** Python-based reconnaissance automation and reporting.
- **This repository:** reusable methodology, workflows, and structured security notes.

## ⚠️ Responsible Use

The material is intended for authorized penetration testing, security research, CTFs, and laboratory environments. Only test systems where you have explicit permission.

## 🔗 Related Work

- [GitHub profile](https://github.com/abhaygupta-sec)
- [HTB Writeups](https://github.com/abhaygupta-sec/HTB-Writeups)
- [NmapTool-Automated-Recon](https://github.com/abhaygupta-sec/NmapTool-Automated-Recon)
- [LinkedIn](https://www.linkedin.com/in/abhay-gupta-sec)
