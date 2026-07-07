# Azure Cloud SOC Home Lab

> This is a cloud-based Security Operations Center home lab built from scratch in Microsoft Azure as a hands-on platform for practicing threat detection, SIEM administration, and attack simulation.

![Platform](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?style=flat-square&logo=microsoftazure)
![Stack](https://img.shields.io/badge/Stack-Wazuh%20%2B%20Terraform%20%2B%20Ubuntu-informational?style=flat-square)

---

### Ethical and Legal Disclaimer
> [!IMPORTANT]
> The following project covers tools and methodologies that are for authorized, ethical use in controlled lab environments only. I do not condone the usage of such tools and methods for malicious or unethical purposes. Follow all applicable international and local laws accordingly. I am not liable for any damage done fully or in part from the information covered in this project writeup. The below information is for educational purposes only. All attack simulations were conducted exclusively against infrastructure I own and control.

## Table of Contents

- [Motivation](#motivation)
- [Environment](#environment)
- [Infrastructure & Automation](#infrastructure--automation)
- [Toolset](#toolset)
- [Methodology](#methodology)
- [Skills Gained](#skills-gained)
- [Screenshots](#screenshots)

---

## Motivation

I wanted to build a hands-on SOC environment to develop practical blue team skills beyond what certifications and theory alone can offer. Rather than using a local VM setup, I deployed this lab entirely in Microsoft Azure to gain real experience working with cloud infrastructure while simultaneously learning SIEM administration and detection engineering.

The project was designed around a realistic attack-and-detect scenario. A dedicated attacker machine runs live credential attacks and network reconnaissance against an intentionally misconfigured target, while a central SIEM ingests logs from both machines and surfaces the activity as mapped alerts. Every component was deployed through Infrastructure as Code (IaC) so the environment is fully standardized, reproducible, and predictable.

---

## Environment

| Component | Details |
|-----------|---------|
| **Cloud** | Microsoft Azure |
| **IaC** | Terraform (azurerm provider0 v4.0) |
| **SIEM** | Wazuh 4.14 (Manager + Indexer + Dashboard) |
| **SIEM Server** | Ubuntu 22.04 LTS - Standard_D4s_v3 (4 vCPU / 16 GB RAM) |
| **Attacker VM** | Ubuntu 22.04 LTS - Standard_D2s_v3 (2 vCPU / 8 GB RAM) |
| **Victim VM** | Ubuntu 22.04 LTS - Standard_D2s_v3 (2 vCPU / 8 GB RAM) |
| **Network** | Azure Virtual Network with 3 segmented subnets |
| **Access** | SSH key authentication (ed25519) + NSG IP restrictions |

---

## Infrastructure & Automation

All infrastructure was provisioned using Terraform with a modular layout. A root module calls separate child modules for networking, the SIEM, the attacker, and the victim. Each VM receives its full software configuration automatically at first boot via cloud-init, meaning a fully operational three-VM SOC environment is created with zero manual server configuration.

Key design decisions:

- **Network segmentation** - three dedicated Azure subnets (SOC / attacker / victim) each with their own Network Security Group (NSG) enforcing least-privilege traffic rules. Attack traffic flows only from the attacker subnet to the victim subnet and cannot go elsewhere. Wazuh agent traffic flows from both agent VMs inward to the SOC subnet only so that activity can be monitored. 
- **Static private IP for SIEM** - the SIEM is pinned a private `10.0.0.0/24` address so both agent cloud-init scripts can reference a known address at boot time, eliminating any dependency on Terraform output values
- **NSG home IP restriction** - SSH access and Wazuh Dashboard access (HTTPS port 443) are restricted to a single source IP, preventing the management interfaces from being exposed to the public internet
- **cloud-init automation** - the SIEM cloud-init runs the Wazuh 4.14 all-in-one quickstart script which ensures all agents and logs are set up correctly the first time. The attacker cloud-init installs Hydra, Nmap, and the Wazuh agent, and the victim cloud-init installs vsftpd with intentionally weak credentials, enables SSH password authentication, and enrolls the Wazuh agent.

**Wazuh Dashboard** is accessed through HTTPS on port 443 directly from the local workstation. The stack uses Wazuh's bundled OpenSearch-based indexer and OpenSearch Dashboards-based frontend rather than a standalone ELK deployment. All three components are installed by the Wazuh quickstart installer in a single automated step at the beginning.

---

## Toolset

| Tool | Category | Purpose |
|------|----------|---------|
| **Terraform** | Infrastructure as Code | Declarative provisioning of all Azure resources across modular configuration files |
| **cloud-init** | Automation | First-boot VM configuration which installs and configures all software without manual SSH |
| **Wazuh Manager** | SIEM | Central log processing engine; parses agent events, runs rules, generates alerts |
| **Wazuh Indexer** | Log Storage | OpenSearch-based backend that indexes and stores all alerts for querying |
| **Wazuh Dashboard** | Visualization | OpenSearch Dashboards-based UI for alert investigation, MITRE ATT&CK mapping, and custom dashboards |
| **Nmap** | Reconnaissance | Network service discovery and version fingerprinting against the victim VM |
| **Hydra** | Credential Attack | Automated FTP and SSH brute-force using dictionary wordlist (rockyou.txt) |
| **vsftpd** | Target Service | Intentionally misconfigured FTP server on the victim VM which is the primary brute-force target |
| **Azure NSG** | Network Control | Stateful traffic rules enforcing subnet isolation and restricting management access |

---

## Methodology

The lab runs in three sequential phases that mirror a real-world attack-and-detect workflow:

**Phase 1 - Infrastructure Deployment** provisions the full environment from a single Terraform deployment operation. It creates the resource group, virtual network, three subnets, NSGs, public IPs, and all three VMs in the correct dependency order. The `cloud-init` module handles the Wazuh installation on the SIEM VM, attack tools installation on the attacker VM, and the vsftpd configuration with a weak credential set on the victim. After approximately 20 minutes of automated first-boot configuration, the Wazuh Dashboard is accessible and both agents appear as active.

**Phase 2 - Attack Simulation** runs entirely from the attacker VM. An Nmap service version scan maps the victim's open ports and identifies the running services (vsftpd on port 21, OpenSSH on port 22). Hydra then executes a dictionary attack against the FTP service using the rockyou.txt wordlist, generating a sustained stream of authentication failures before finding the correct credential. Because the attacker VM also runs a Wazuh agent, process execution telemetry from the attack tools is captured alongside the victim's authentication failure logs, which gives the SIEM dual-perspective visibility into the same event.

**Phase 3 - Detection & Investigation** is conducted from the Wazuh Dashboard on the local workstation. Custom XML detection rules (rule IDs 100001–100003) fire on the authentication failure patterns and escalate them to severity level 12 with MITRE ATT&CK technique tags (T1046, T1110, T1110.001, T1110.003). The events view shows the full alert timeline, the MITRE ATT&CK Framework view maps the techniques to their tactics, and a custom dashboard built on the `wazuh-alerts-*` index visualizes alert volume by rule description and source IP over time.

```mermaid
flowchart TD
    A["Terraform + cloud-init\nInfrastructure Deployment"] -->|3 VMs provisioned + configured| B
    B["Nmap\nNetwork Service Discovery (T1046)"] -->|Open ports identified: 21, 22| C
    C["Hydra\nCredential Brute Force (T1110)"] -->|Auth failures + successful login generated| D
    D["Wazuh Agents\nLog Collection on Attacker + Victim"] -->|Events forwarded to 10.0.1.4:1514| E
    E["Wazuh Manager\nRule Processing & Custom Rule Matching"] -->|Alerts indexed at level 10–12| F
    F["Wazuh Dashboard\nInvestigation + MITRE ATT&CK Mapping"]
```

---

## Skills Gained

**Infrastructure as Code** - Building the lab entirely in Terraform exposed me to the practical realities of IaC. Some being modular design patterns, managing resource dependencies across modules, handling state after partial deployment failures, and using `terraform plan` as a safety gate before every apply. The experience of debugging a real deployment failure (wrong resource name in a module, missing password on a VM) and resolving it without destroying already-provisioned resources demonstrated exactly how Terraform's state model works in practice, not just in theory.

**Cloud network security** - Designing the NSG ruleset from scratch required thinking carefully about the minimum necessary traffic flows for each subnet. The SOC subnet only accepts inbound traffic on ports 22, 443, 1514, and 1515 from controlled sources. The victim subnet accepts all traffic from the attacker subnet (by design) but is fully blocked from the internet. Working through the Azure quota system gave me practical familiarity with how cloud subscription limits work and how to query them before deployment.

**SIEM deployment and administration** - Standing up Wazuh 4.14 from scratch, enrolling agents across multiple VMs, diagnosing why an agent showed as disconnected, and manually registering agents via `agent-auth` all gave me operational experience that reading documentation alone doesn't provide. Understanding that the Wazuh Indexer is an OpenSearch fork, that the dashboard runs on port 443 rather than the 5601 I expected from a traditional ELK setup, and that the all-in-one installer manages the entire stack as a unit deepened my understanding of how modern SIEM products are packaged. This also made the deployment process much simpler as I only had to use one product rather than multiple, which drops the configuration complexity by a sizeable margin.

**Detection engineering** - Writing custom Wazuh rules in XML taught me the parent-child rule model: how to reference a parent rule's ID with `if_matched_sid`, how `frequency` and `timeframe` work together to create composite detection logic, and how `same_source_ip` prevents multiple different hosts from falsely triggering a brute-force rule. Attaching MITRE ATT&CK technique IDs via the `<mitre>` tag and validating rules with `wazuh-logtest` before deployment showed me how detection rules connect to a broader threat intelligence framework used in production SOCs rather than existing as isolated signatures.

**Attack simulation and offensive awareness** - Running Nmap and Hydra against a live target gave me a defender's intuition about what attacks actually look like in logs. Seeing that a single Hydra FTP session against a 4-thread limit generates hundreds of individual PAM authentication failure events in under two minutes helped me understand why frequency-based detection rules need tight timeframes to be actionable.

---

## Screenshots

### Phase 1 - Infrastructure Deployment

<img width="1917" height="862" alt="Wazuh_home" src="https://github.com/user-attachments/assets/975e5d2f-ff2b-4c55-acd5-464dfa58f1b4" />

Both Wazuh agents `attacker-vm` and `victim-vm` shown as active, with 1 high severity alert, 502 medium severity alerts, and 1,675 low severity alerts generated during the attack simulation window. 

The Wazuh home screen confirms that both agents enrolled successfully after the Terraform deployment completed and `cloud-init` finished running on all three VMs.

---

### Phase 2 - Attack Simulation: Nmap Service Scan (T1046)

> <img width="1918" height="988" alt="Nmap_scan" src="https://github.com/user-attachments/assets/9387556b-4c0b-4383-957c-7e49fceb57a4" />

Two Nmap commands run against the victim (`10.0.2.4`): a service version scan (`-sV -T4`) identifying vsftpd 3.0.5 on port 21 and OpenSSH 8.9p1 on port 22, followed by a full port scan (`-p-`) confirming only those two ports are open across all 65535.

---

### Phase 2 - Attack Simulation: Hydra FTP Brute Force (T1110)

> <img width="1918" height="988" alt="Hydra_brute_force" src="https://github.com/user-attachments/assets/c607db81-8560-4127-b3d1-dbe433ca302e" />

Hydra targeting `ftp://10.0.2.4` with user `ftpuser` and a 201-entry wordlist slice from rockyou.txt. The successful credential hit (`ftpuser:Password123`) is confirmed at line `[21][ftp] host: 10.0.2.4 login: ftpuser password: Password123`.

---

### Phase 3 - Detection: Wazuh Default Dashboard

> <img width="1918" height="867" alt="Wazuh_default_dashboard" src="https://github.com/user-attachments/assets/78c58a5f-5a53-4569-8a06-8786d115d807" />

The Threat Hunting dashboard filtered to `agent.name: victim-vm`, `rule.level: 12 to 14`, showing 1 level-12 alert generated during the attack window. The Top 10 MITRE ATT&CKs pie chart shows Brute Force and Valid Accounts as the triggered technique categories.

---

### Phase 3 - Detection: Events View

> <img width="1918" height="870" alt="wazuh_events_view" src="https://github.com/user-attachments/assets/320d7b29-d98a-4193-9d7f-01698d6b599f" />

The MITRE ATT&CK events view filtered to the attack timeframe showing 147 hits. The alert types include rule 40112 (multiple authentication failures followed by success, T1078 + T1110, level 12), rule 11452 (vsftpd multiple FTP connection attempts, T1110, level 10), and rule 5503 (PAM user login failed, T1110.001, level 5). All of these are sourced from `victim-vm`.

---

### Phase 3 - Detection: MITRE ATT&CK Dashboard

> <img width="1918" height="870" alt="Wazuh_mitre_attack_dashboard" src="https://github.com/user-attachments/assets/9f9037d9-4cda-4047-b5e3-8c0f8fc678ac" />

The MITRE ATT&CK Dashboard shows the full attack session across four panels: alert evolution over time (spike at 21:35 during the Hydra run), attacks by technique (Credential Access dominant), top tactics by agent, and MITRE techniques broken out per agent. All of these are attributed to `victim-vm`.

---

### Phase 3 - Detection: MITRE ATT&CK Framework - T1110.001 Drill-Down

> <img width="1918" height="865" alt="wazuh_mitre_attack_ttps" src="https://github.com/user-attachments/assets/7f272875-7c24-4492-950a-441f1c8915a7" />

The Framework tab showing Credential Access with 147 events, T1110.001 (Password Guessing) as the primary technique with 112 hits. The event table confirms individual events mapped to `victim-vm` agent with rule ID 5503 (PAM: User login failed) at level 5.

---

### Phase 3 - Detection: Custom Dashboard

> <img width="1918" height="857" alt="wazuh_custom_dashboard" src="https://github.com/user-attachments/assets/c5dd4300-f0c1-4b2e-8187-c7b8c09fb59b" />

A custom Wazuh Dashboard built on the `wazuh-alerts-*` index showing alert volume by rule description split by source IP (`10.0.3.4` = the attacker). The alert categories include vsftpd login failures, PAM multiple failed logins, vsftpd FTP brute force, and vsftpd multiple FTP connection attempts which are all generated during the Hydra session.
