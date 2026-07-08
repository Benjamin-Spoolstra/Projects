# Projects

This repository is a collection of my hands-on cybersecurity home lab projects covering offensive security, defensive/blue team operations, and governance, risk, and compliance (GRC). Each project was built from scratch, documented in detail, and includes methodology, tooling, and screenshots.
---

### [Bug Bounty VPS](https://github.com/Benjamin-Spoolstra/Projects/blob/main/Bug%20Bounty%20VPS.md)
This is a hardened Kali Linux VPS built as a dedicated remote platform for bug bounty hunting and penetration testing. It covers OS hardening, a structured three-phase recon-to-exploitation methodology (passive recon → active enumeration → manual testing), and a full offensive security toolchain (Subfinder, Amass, httpx, Nuclei, Caido, and more).

**Skills:** Linux hardening, recon automation, web app vulnerability testing, secure remote access

---

### [Azure Cloud SOC Home Lab](https://github.com/Benjamin-Spoolstra/Projects/blob/main/SOC%20Lab.md)
This is a cloud-based Security Operations Center (SOC) built in Microsoft Azure with Terraform. It features an attacker VM, a misconfigured victim VM, and a Wazuh SIEM. It simulates a full attack-and-detect cycle with Nmap recon, Hydra credential brute-forcing, and custom Wazuh detection rules mapped to MITRE ATT&CK.

**Skills:** Infrastructure as Code, SIEM deployment, detection engineering, cloud network security

---

### [Azure Risk Management & DISA STIG Hardening Lab](https://github.com/Benjamin-Spoolstra/Projects/blob/main/Risk%20Management%20Lab.md)
This is a cloud-based GRC lab simulating the full NIST Risk Management Framework (RMF) lifecycle against an Active Directory environment in Azure. It includes a baseline vulnerability/compliance scan with Nessus and the SCAP Compliance Checker (SCC), an automated and manual DISA STIG remediation to the scan findings, and a control mapping to NIST SP 800-171 / CMMC 2.0 Level 2 regulations.

**Skills:** Risk management lifecycle, compliance auditing, PowerShell/DSC automation, control mapping

---

*All projects were conducted exclusively against infrastructure I own and control, in accordance with applicable laws, for educational purposes.*
