## Penetration Testing Methodology — Where This Fits
```
Passive Recon (OSINT)
    ↓
Active Recon / Network Mapping  ← YOU ARE HERE
    ↓
Enumeration
    ↓
Vulnerability Analysis
    ↓
Exploitation
    ↓
Post-Exploitation
    ↓
Reporting
```

---

## Networking Fundamentals

### Network Protocols
- Rules that allow different systems (different hardware/software) to communicate
- Communication happens via **packets**
- Packets = streams of bits transferred over physical media (Ethernet, Wi-Fi)

### Packet Structure
Every packet has:
- **Header** — protocol-specific info (source/dest addresses, flags, TTL, etc.)
- **Payload** — the actual data being sent (email content, file data, etc.)

---

## OSI Model — MEMORIZE ALL 7 LAYERS

| # | Layer | Function | Examples |
|---|-------|----------|---------|
| 7 | **Application** | Services to end-users/apps | HTTP, FTP, SSH, DNS |
| 6 | **Presentation** | Data translation, encryption, compression | SSL/TLS, JPEG, GIF |
| 5 | **Session** | Manages connections between apps | APIs, NetBIOS, RPC |
| 4 | **Transport** | End-to-end comms, flow control | **TCP, UDP** |
| 3 | **Network** | Logical addressing + routing | **IP, ICMP**, IPSec |
| 2 | **Data Link** | Physical addressing, error detection | Ethernet, PPP, Switches |
| 1 | **Physical** | Physical connection | Cables, Hubs, USB, Wi-Fi |

> **Mnemonic (top → bottom):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing  
> **eJPT tip:** Exam questions reference OSI layers directly — know them cold

---

## Internet Protocol (IP) — Layer 3

### IPv4
- 32-bit address | 4 octets separated by dots → e.g. `192.168.0.1`
- Finite address space (led to IPv6 adoption)

### IPv6
- 128-bit address | hexadecimal notation → e.g. `2001:0db8:85a3::8a2e:0370:7334`
- Exponentially larger address space

### Reserved IPv4 Addresses

| Range | Purpose |
|-------|---------|
| `0.0.0.0 – 0.255.255.255` | "This" network |
| `127.0.0.0 – 127.255.255.255` | Localhost (loopback) |
| `10.0.0.0 – 10.255.255.255` | Private network |
| `172.16.0.0 – 172.31.255.255` | Private network |
| `192.168.0.0 – 192.168.255.255` | Private network |

### Key IP Header Fields

| Field | Description |
|-------|-------------|
| Source IP (32 bits) | Sender's IP address |
| Destination IP (32 bits) | Recipient's IP address |
| TTL (8 bits) | Max hops before packet is discarded; decremented at each hop |
| Protocol (8 bits) | `6`=TCP, `17`=UDP, `1`=ICMP |
| Flags | DF (Don't Fragment), MF (More Fragments) |

> **eJPT tip:** TTL hints at OS — Windows starts at **128**, Linux starts at **64**, Cisco at **255**

---

## Transport Layer — TCP vs UDP

### TCP (Transmission Control Protocol)
- **Connection-oriented** — establishes connection before data transfer
- **Reliable** — guarantees delivery via ACKs and retransmission
- **Ordered** — reassembles out-of-order segments
- Used by: HTTP, HTTPS, FTP, SSH, SMTP, Telnet

### TCP 3-Way Handshake — MUST KNOW
```
Client            Server
  |---SYN--------->|    Client sends SYN with random ISN
  |<--SYN-ACK------|    Server responds SYN + ACK (ISN+1)
  |---ACK--------->|    Client acknowledges (server ISN+1)
  |    [DATA]      |    Connection established — data flows
```

### TCP Control Flags

| Flag | Meaning |
|------|---------|
| **SYN** | Initiate connection |
| **ACK** | Acknowledge data received |
| **FIN** | Terminate connection gracefully |
| **RST** | Reset / forcefully close connection |
| **PSH** | Push data immediately to application |
| **URG** | Urgent data |

### TCP Port Ranges

| Range | Type | Examples |
|-------|------|---------|
| **0–1023** | Well-Known Ports | 80=HTTP, 443=HTTPS, 22=SSH, 21=FTP, 25=SMTP, 110=POP3 |
| **1024–49151** | Registered Ports | 3389=RDP, 3306=MySQL, 8080=HTTP-alt, 27017=MongoDB |
| **49152–65535** | Dynamic/Ephemeral | OS-assigned client-side ports |

### UDP (User Datagram Protocol)
- **Connectionless** — no handshake, no persistent state
- **Unreliable** — no delivery guarantee, no retransmission
- **Fast** — low overhead, low latency
- Used by: DNS, DHCP, SNMP, VoIP, online gaming, streaming

### TCP vs UDP Summary

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | 3-Way Handshake | Connectionless |
| Reliability | Guaranteed | Not guaranteed |
| Order | Ordered delivery | No ordering |
| Speed | Slower | Faster |
| Header Size | Larger | Smaller |
| Use case | HTTP, Email, FTP, SSH | DNS, VoIP, Gaming, Streaming |

---

## ICMP (Internet Control Message Protocol)

- Error reporting and diagnostics — operates at Layer 3
- **Ping uses ICMP:**
  - Echo Request = **Type 8, Code 0**
  - Echo Reply = **Type 0, Code 0**
- If host is offline or blocks ICMP → no reply received
- **eJPT tip:** No ping response does NOT mean host is down — firewall may be blocking ICMP

---

## Network Mapping

### Why Map a Network?
- Identify what exists before you attack anything
- Example: scope = `200.200.0.0/16` → up to **65,536** possible hosts
- Goal: which are alive, what ports are open, what OS/services are running

### Network Mapping Objectives
1. **Live Host Discovery** — which IPs are active?
2. **Open Port Identification** — what's listening?
3. **Service Version Detection** — what version is running?
4. **OS Fingerprinting** — what OS is the target?
5. **Network Topology Mapping** — routers, switches, firewalls
6. **Security Measure Identification** — firewalls, IDS/IPS in place

---

## Host Discovery Techniques

### Ping Sweep (ICMP Echo)
- Sends ICMP Echo Request (Type 8) to IP range
- Live host replies with Echo Reply (Type 0)
- Quick but easily blocked by firewalls
- `nmap -sn <network>`

### ARP Scanning
- Uses ARP requests — works on **local network only (same subnet)**
- Very reliable, works even when ICMP is blocked
- `nmap -PR <network>` or use `arp-scan -l`

### TCP SYN Ping
- Sends SYN to a port — live host replies with SYN-ACK
- Stealthier than ICMP ping
- `nmap -PS80 <target>`

### TCP ACK Ping
- Sends ACK — expects RST if host is alive
- Can bypass firewalls blocking SYN
- `nmap -PA80 <target>`

### UDP Ping
- Useful when both ICMP and TCP are blocked
- `nmap -PU53 <target>`

---

## Nmap — Complete Reference

### Essential Nmap Commands
```bash
# Basic scan (top 1000 ports)
nmap <target>

# Ping sweep — find live hosts, no port scan
nmap -sn <network/range>

# SYN scan (stealth, requires root) — RECOMMENDED DEFAULT
nmap -sS <target>

# TCP Connect scan (no root needed)
nmap -sT <target>

# UDP scan
nmap -sU <target>

# Service version detection
nmap -sV <target>

# OS detection
nmap -O <target>

# Default script scan
nmap -sC <target>

# THE BEST COMBO FOR eJPT — use this on every target
nmap -sC -sV <target>

# Aggressive scan (OS + version + scripts + traceroute)
nmap -A <target>

# Scan ALL 65535 ports
nmap -p- <target>

# Scan specific ports
nmap -p 22,80,443,3389 <target>

# Scan a port range
nmap -p 1-1000 <target>

# Save all output formats
nmap -oA <filename> <target>

# Save normal text
nmap -oN <filename> <target>

# Save greppable
nmap -oG <filename> <target>

# Save XML
nmap -oX <filename> <target>

# Verbose
nmap -v <target>

# Very verbose
nmap -vv <target>

# Scan entire subnet
nmap 192.168.1.0/24
```

---

### Nmap Timing Templates

| Flag | Name | Speed | Use Case |
|------|------|-------|---------|
| `-T0` | Paranoid | Slowest | IDS evasion |
| `-T1` | Sneaky | Very slow | Evasion |
| `-T2` | Polite | Slow | Low network load |
| `-T3` | Normal | Default | Standard |
| `-T4` | Aggressive | Fast | Reliable network |
| `-T5` | Insane | Fastest | May miss results |

> **eJPT tip:** Use `-T4` in lab environments — fast and reliable

---

### Nmap Port States

| State | Meaning |
|-------|---------|
| **open** | Port is accepting connections |
| **closed** | Port accessible but no service listening |
| **filtered** | Firewall blocking — state unknown |
| **open\|filtered** | Can't tell if open or filtered (common in UDP) |
| **unfiltered** | Accessible but open/closed state unknown (ACK scan) |

---

### Nmap Scripting Engine (NSE)
```bash
# Run default scripts
nmap -sC <target>

# Run specific script
nmap --script <script-name> <target>

# Vulnerability scan
nmap --script vuln <target>

# Brute force
nmap --script brute <target>

# Discovery scripts
nmap --script discovery <target>

# Run all HTTP scripts
nmap --script "http-*" <target>

# List all available scripts
ls /usr/share/nmap/scripts/
```

> **eJPT tip:** `nmap --script vuln <target>` can automatically flag common CVEs — very useful in the exam

---

### Firewall Detection & Evasion
```bash
# Fragment packets (bypass simple packet filters)
nmap -f <target>

# Use random decoy IPs
nmap -D RND:10 <target>

# Use specific source port (e.g. mimic DNS traffic on port 53)
nmap --source-port 53 <target>

# Append random data to packets
nmap --data-length 25 <target>

# Slow scan to avoid IDS detection
nmap -T1 <target>

# Idle/Zombie scan (very stealthy)
nmap -sI <zombie-host> <target>
```

---

### OS Fingerprinting
```bash
# OS detection only
nmap -O <target>

# OS + version + scripts + traceroute
nmap -A <target>
```

- Nmap identifies OS by analyzing TCP/IP stack behavior
- Requires at least **one open** and **one closed** port to work accurately

**TTL-based OS guessing:**

| TTL Value | Likely OS |
|-----------|-----------|
| 64 | Linux / Unix |
| 128 | Windows |
| 255 | Cisco / Network devices |

---

## Common Ports — Must Know for eJPT

| Port | Protocol | Service |
|------|----------|---------|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 139 | TCP | NetBIOS |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | HTTP alternate |
| 8443 | TCP | HTTPS alternate |
| 27017 | TCP | MongoDB |

> **eJPT tip:** SMB (445) and RDP (3389) are among the most exploited — always note and enumerate these

---

## Full Recommended Scanning Workflow (eJPT Exam)
```bash
# Step 1: Discover live hosts in the subnet
nmap -sn 192.168.1.0/24

# Step 2: Full port scan + version + scripts on each live host
nmap -sC -sV -p- -T4 192.168.1.10

# Step 3: OS detection
nmap -O 192.168.1.10

# Step 4: Run vuln scripts
nmap --script vuln 192.168.1.10

# Step 5: Save everything
nmap -sC -sV -oA scan_results 192.168.1.10
```

---

## Key Takeaways for the Exam

- **`-sC -sV` is your best friend** — use it on every single target
- **Document per target:** IP, OS, open ports, services, versions, credentials
- **ICMP blocked ≠ host is down** — try TCP SYN ping if ping fails
- **Check robots.txt and HTTP headers** — frequently hide useful info
- **DNS zone transfer** = misconfiguration that reveals entire network layout
- **SMB (445) and RDP (3389)** = high-value targets, always enumerate
- **TTL tells you OS:** 64=Linux, 128=Windows, 255=Cisco
- **NSE `--script vuln`** automates vulnerability detection
- Use `-T4` in lab/exam environments for speed
- Save all scan output with `-oA` — you'll need to reference it later

---

## Quick Command Cheat Sheet — Part 2
```bash
# Host discovery
nmap -sn 192.168.1.0/24

# SYN scan
nmap -sS <target>

# Best all-round scan
nmap -sC -sV -T4 <target>

# All ports
nmap -p- <target>

# OS detection
nmap -O <target>

# Full aggressive
nmap -A <target>

# Vuln scripts
nmap --script vuln <target>

# Save output
nmap -sC -sV -oA output_name <target>

# Firewall evasion
nmap -f <target>
nmap -D RND:10 <target>
nmap --source-port 53 <target>
```

---