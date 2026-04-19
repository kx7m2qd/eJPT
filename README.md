<div align="center">

<img src="https://img.shields.io/badge/eLearnSecurity-eJPT-red?style=for-the-badge&logo=target&logoColor=white" />
<img src="https://img.shields.io/badge/Status-COMPLETED%20✓-brightgreen?style=for-the-badge" />
<img src="https://img.shields.io/badge/Platform-INE-0052CC?style=for-the-badge&logo=internet-explorer&logoColor=white" />
<img src="https://img.shields.io/badge/Level-Junior%20Pentester-orange?style=for-the-badge&logo=hackthebox&logoColor=white" />

# 🔐 eJPT — Junior Penetration Tester

**Personal study notes, command references, and cheatsheets for the eJPT certification by eLearnSecurity / INE.**

</div>

---

## 📋 Exam At a Glance

| Property | Details |
|:---|:---|
| 🏷️ **Certification** | eJPT – Junior Penetration Tester |
| 🏢 **Provider** | eLearnSecurity / INE |
| 📝 **Format** | 35 Questions + Practical Pentest Lab |
| ⏱️ **Duration** | 48 Hours |
| ✅ **Passing Score** | 70% |
| 📖 **Open Book** | Yes |
| 🧩 **Domains** | Networking · Web Apps · Host & Network Pentesting |

---

## 🗂️ Notes Index

> Click any topic below to jump directly to its notes.

### 🕵️ Reconnaissance & Information Gathering

| # | File | Description |
|:---:|:---|:---|
| 01 | 📄 [Information Gathering](./InformationGathering.md) | OSINT, passive & active recon techniques |
| 02 | 📄 [Footprinting & Scanning](./Footprinting&Scanning.md) | Network mapping, Nmap, host discovery |

---

### 🔬 Assessment & Enumeration

| # | File | Description |
|:---:|:---|:---|
| 03 | 📄 [Vulnerability Assessment](./VulnerabilityAssessment.md) | Identifying and analyzing vulnerabilities |
| 04 | 📄 [Enumeration](./Enumeration.md) | SMB, FTP, HTTP, DNS, SNMP enumeration |
| 05 | 📄 [Auditing](./Auditing.md) | Security auditing concepts and checklists |

---

### ⚔️ Attack Techniques

| # | File | Description |
|:---:|:---|:---|
| 06 | 📄 [Network-Based Attacks](./Network-BasedAttacks.md) | MitM, ARP spoofing, sniffing, MITM tools |
| 07 | 📄 [System & Host Based Attacks](./System&HostBasedAttacks.md) | Windows & Linux local exploitation |
| 08 | 📄 [Web Application Pentest](./WebApplicationPentest.md) | SQLi, XSS, file inclusion, Burp Suite |
| 09 | 📄 [The Metasploit Framework](./TheMetasploitFramework.md) | msfconsole, exploits, payloads, sessions |

---

### 🚀 Exploitation & Post-Exploitation

| # | File | Description |
|:---:|:---|:---|
| 10 | 📄 [Exploitation](./Exploitation.md) | Exploiting services, CVEs, manual attacks |
| 11 | 📄 [Post Exploitation](./PostExploitation.md) | Privilege escalation, pivoting, persistence |

---

### 📋 Quick Reference

| # | File | Description |
|:---:|:---|:---|
| 12 | 📄 [CheatSheet](./CheatSheet.md) | All key commands in one place |

---

## 🧠 Exam Topic Coverage

```
Information Gathering      ████████████████████  100%
Footprinting & Scanning    ████████████████████  100%
Vulnerability Assessment   ████████████████████  100%
Enumeration                ████████████████████  100%
Exploitation               ████████████████████  100%
Web Application Attacks    ████████████████████  100%
Post Exploitation          ████████████████████  100%
Metasploit Framework       ████████████████████  100%
```

---

## 🗺️ Pentesting Methodology

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. Information Gathering  ──►  2. Scanning         │
│         │                              │            │
│         ▼                              ▼            │
│  3. Enumeration           ──►  4. Vuln Assessment   │
│         │                              │            │
│         ▼                              ▼            │
│  5. Exploitation          ──►  6. Post Exploitation │
│                                        │            │
│                               7. Reporting          │
└─────────────────────────────────────────────────────┘
```

---

## ⚡ Key Tools Quick Reference

| Tool | Use Case |
|:---|:---|
| `nmap` | Port scanning & service detection |
| `netdiscover` | Host discovery on local networks |
| `gobuster` / `dirb` | Web directory brute-forcing |
| `nikto` | Web server vulnerability scanning |
| `hydra` | Online brute-force (SSH, FTP, HTTP) |
| `john` / `hashcat` | Offline password / hash cracking |
| `metasploit` | Exploitation framework |
| `msfvenom` | Custom payload generation |
| `sqlmap` | Automated SQL injection |
| `wireshark` / `tcpdump` | Packet capture & traffic analysis |
| `enum4linux` | SMB/Samba enumeration |
| `netcat` | Bind/reverse shells, port listening |
| `burpsuite` | Web app interception & testing |

---

## 📚 Official Resources

- 🌐 [INE Learning Platform](https://ine.com)
- 📖 [eJPT Exam Page](https://elearnsecurity.com/product/ejpt-certification/)
- 💬 [INE Community Discord](https://discord.gg/ine)
- 🗂️ [PTS Course (Penetration Testing Student)](https://ine.com/learning/courses/penetration-testing-student)

---

<div align="center">

*Made with 💻 for eJPT exam prep*

</div>
