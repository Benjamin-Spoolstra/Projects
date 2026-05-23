# Bug Bounty VPS Build

> This is a hardened Kali Linux VPS built from scratch as a dedicated, remote platform for bug bounty hunting, penetration testing, and CTF challenges.

![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-blueviolet?style=flat-square&logo=kalilinux)
![Stack](https://img.shields.io/badge/Stack-VPS%20%2B%20TigerVNC%20%2B%20Caido-informational?style=flat-square)
![Phase](https://img.shields.io/badge/Completed-Phase%201%3A%20Environment%20Setup-success?style=flat-square)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat-square)

---

## Table of Contents

- [Motivation](#motivation)
- [Environment](#environment)
- [Hardening & Infrastructure](#hardening--infrastructure)
- [Toolset](#toolset)
- [Methodology](#methodology)
- [Skills Gained](#skills-gained)

---

## Motivation

I wanted to create a dedicated Linux server for my bug hunting and penetration testing hobby projects that I do in my free time. This VPS allows me to run engagements remotely and with a flexibility I need to effectively hunt even with limited time. I created a structured file system and methodology so that I can navigate through the phases of bug testing efficiently. This helps me stay on track and use the time I have to make meaningful progress on my engagements.

---

## Environment

| Component | Details |
|-----------|---------|
| **VPS** | Hostinger KVM VPS 2 — Kali Linux |
| **Local OS** | Windows 11 |
| **Proxy** | Caido |
| **GUI Access** | TigerVNC via SSH tunnel |
| **Local Test Target** | DVWA in Docker |
| **Platforms** | HackerOne |

---

## Hardening & Infrastructure

Before I setup my VPS for vulnerability testing I hardened it according to security best practices. The core theme behind my hardening decision revolved around reducing the attack surface. I disabled every service, open port, and default behavior that wasn't strictly necessary for my activities.

Key controls applied:

- **SSH** — key-only authentication (ed25519 elliptical curve cryptography) with root login disabled and idle timeout enforced
- **Firewall** — UFW with default-deny inbound; only strictly necessary ports are left open
- **Fail2ban + PAM lockout** — brute-force protection at the network and OS level
- **Service cleanup** — stopped and disabled unnecessary desktop services like `polkit`, `accounts-daemon`, and `colord`
- **Kernel hardening** — ASLR, ptrace restrictions, SYN flood protection, IP forwarding disabled, kernel pointer exposure restricted
- **Filesystem** — `/tmp` and `/dev/shm` mounted `noexec`; restrictive umask; home directory permissions tightened
- **IPv6 disabled** — not needed for this use case; removes an entire network stack from the surface

**TigerVNC** is used for GUI access when needed and is bound exclusively to `127.0.0.1:5901`. The VNC is connected through an SSH tunnel which encrypts communication and reduces the attack surface with more open ports.

**DVWA** (Damn Vulnerable Web Application) runs in a Docker container on loopback as a safe local target for practicing the full tool pipeline and manual testing techniques before touching any real program. This local target allows me to test tool functionality without damaging any internet-facing infrastructure.

---

## Toolset

| Tool | Category | Purpose |
|------|----------|---------|
| **Subfinder** | Passive Recon | Multi-source passive subdomain enumeration |
| **Assetfinder** | Passive Recon | Lightweight passive subdomain discovery |
| **Amass** | Passive Recon | Comprehensive OSINT-based subdomain enumeration |
| **Chaos Client** | Passive Recon | ProjectDiscovery bug bounty subdomain dataset |
| **dnsx** | DNS | Fast DNS resolution and validation |
| **httpx** | HTTP Probing | Live host detection, tech fingerprinting, header analysis |
| **Katana** | Crawling | Live endpoint crawling (static and JS-rendered) |
| **gau** | URL Discovery | Historical URL collection (Wayback, Common Crawl, URLScan) |
| **waybackurls** | URL Discovery | Wayback Machine URL extraction |
| **gowitness** | Screenshots | Automated visual screenshots of live hosts |
| **ffuf** | Fuzzing | Fast directory and parameter fuzzing |
| **Nuclei** | Scanning | Template-based passive/safe vulnerability checks |
| **gf** | Analysis | Parameter categorization using grep patterns |
| **unfurl** | Analysis | URL decomposition and parameter extraction |
| **anew** | Deduplication | Unique-line append for pipeline deduplication |
| **LinkFinder** | JS Analysis | API endpoint extraction from JavaScript files |
| **SecretFinder** | JS Analysis | Secret and credential detection in JavaScript files |
| **interactsh** | OOB Detection | Out-of-band callback listener for blind SSRF testing |
| **Shodan CLI** | OSINT | Passive IP and port intelligence gathering |
| **Caido** | Proxy | HTTP proxy for manual testing and request replay |
| **tmux** | Workflow | Terminal multiplexing for organized engagement sessions |

---

## Methodology

My engagements follow a three-phase approach:

**Phase 1 — Passive Recon** This phase runs entirely against third-party databases (certificate transparency logs, Wayback Machine, Shodan, GitHub) with zero traffic sent to the target. Subdomain discovery, historical URL collection, DNS infrastructure mapping, and cloud storage enumeration all happen here. The output is a complete attack surface map before the target has seen a single request.

**Phase 2 — Active Enumeration** This phase is where controlled traffic begins. DNS resolution, HTTP probing, live crawling with Katana, JavaScript analysis, response header inspection, and Nuclei template checks all run here at a strict 5 req/s. Every tool command includes a rate limit flag and an identifying User-Agent string so security teams can recognize the traffic.

**Phase 3 — Manual Testing** This phase uses Caido to test the most interesting endpoints identified during enumeration. Vulnerability coverage includes XSS, IDOR, SSRF, open redirects, CORS misconfigurations, authentication logic flaws, SQL injection (error-based detection), GraphQL enumeration, and HTTP verb tampering.

Each engagement runs inside an auto-initialized workspace (`new-engagement.sh`) that creates a consistent directory structure, a pre-structured engagement log, and a named tmux session with dedicated windows for recon, testing, monitoring, and shell access. This persistent setup allows me to seamlessly move between phases keeping track of interesting findings. Documentation is also much easier when I take step by step notes to refer to later.

---

## Skills Gained

**Infrastructure & hardening** — This project exposed me to Linux server hardening best practices such as SSH connection managemetn, host-based firewalling, kernel parameter tuning via sysctl, PAM authentication layering, and systematic service enumeration. The process of filtering out unnecessary services and components taught me a practical way to examine and secure digital assets.

**Recon toolchain** — Developing my own testing methodology showed me a practical way to divide up the work performed during real bug bounty hunting. I learned the primary differences between passive and active target reconnaissance, and that there are times for both types of testing. I developed a structured pipeline that converts wide attack surfaces into meanginful targets that have the highest chance of exposing vulnerabilities. 

The pipeline is:

Subfinder/amass/chaos feed a merged passive subdomain list 
↓
Dnsx resolves it
↓
Httpx probes live hosts
↓
Gowitness visually triages them
↓
Gau/waybackurls + katana build the URL pool
↓
Gf categorizes parameters by vulnerability class
↓
Caido testing is performed on parameterized endpoints 

Following the same, structured bug hunting workflow allows me to adapt it to any unique target or program where it's a bunch of wildcard domains or a single primary domain.

**Secure remote access** — Configuring SSH tunneling for both VNC and local service access (DVWA, gowitness report server) made port forwarding more intuitive to me. No longer is SSH just a key phrase I heard on the Security+ exam. I now see configuring remote connections and managing port forwarding as an essential security skill that not only reduces the attack surface, but efficiently uses the available bandwidth for multiple purposes beyond just command line access.

**Methodical documentation** — Building the engagement log system and finding templates helped me streamline my bug hunting workflow and save time in the reporting phase. With my documentation setup I can easily log all meaningful findings and refer back to them as needed without worrying about missing key details or events.

---

<div align="center">

[![HackerOne](https://img.shields.io/badge/Platform-HackerOne-ef4444?style=flat-square)](https://hackerone.com)

</div>
