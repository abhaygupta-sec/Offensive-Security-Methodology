# Offensive Security Methodology


A structured collection of offensive-security methodologies, attack workflows, enumeration techniques, and penetration-testing practices developed through hands-on labs, Hack The Box, and structured offensive-security practice.


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
