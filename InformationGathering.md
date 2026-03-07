## What is Information Gathering?
- The **first step** of any penetration test
- Collecting information about an individual, company, website, or system
- More info = more attack surface = more success later
- Broken into two types: **Passive** and **Active**

---

## Passive vs Active — At a Glance

| | Passive | Active |
|---|---|---|
| Target interaction | None | Direct |
| Detection risk | Low | Higher |
| When | First | After passive |
| Authorization needed | No | Yes |
| Examples | WHOIS, DNS records, Google Dorks | Nmap, DNS zone transfers |

> **Rule:** Always passive first, active second. Never skip passive recon.

---

## Passive Information Gathering

**What you're looking for:**
- IP addresses & DNS records
- Domain names & ownership (registrar, registrant)
- Email addresses & social media profiles
- Web technologies in use (CMS, frameworks, server type)
- Subdomains

---

### WHOIS
- Queries domain registration databases
- Reveals: registrar, registrant name/org, creation date, nameservers, contact info
- Command: `whois <domain>`
- Online: whois.domaintools.com

---

### Website Recon / Footprinting
What to look for manually:
- IP addresses
- Hidden directories (not indexed by search engines)
- Names, email addresses, phone numbers, physical addresses
- Web technologies in use

---

### Netcraft
- Website: netcraft.com
- Shows: hosting history, IP ranges, web server type, OS, SSL cert info
- Useful for **technology fingerprinting** without touching the target

---

### DNS Recon
- Tool: `dnsrecon` or `dig`
- Enumerates DNS records for a domain
- Reveals: IPs, subdomains, mail servers
- Command: `dnsrecon -d <domain>`

---

### WAF Detection — wafw00f
- Detects Web Application Firewalls before scanning
- Important: knowing a WAF exists changes your approach
- Command: `wafw00f <target>`

---

### Subdomain Enumeration — Sublist3r
- Finds subdomains using OSINT sources (search engines, DNSdumpster, etc.)
- Command: `python sublist3r.py -d <domain>`
- Why it matters: subdomains = more attack surface

---

### Email Harvesting — theHarvester
- Collects emails, subdomains, hosts, employee names from public sources
- Command: `theHarvester -d <domain> -b google`
- Sources: Google, Bing, LinkedIn, Hunter.io, etc.
- **eJPT tip:** This tool is frequently tested — know the `-b` (source) flag

---

### Leaked Password Databases — HaveIBeenPwned
- Check if an email or domain has appeared in known breaches
- Site: haveibeenpwned.com
- Useful for finding credential exposure during passive recon

---

## Active Information Gathering

**What you're looking for:**
- Open ports on target systems
- Internal infrastructure details
- Enumerated information from target systems (OS, services, versions)

---

## DNS Zone Transfers

### DNS Basics
- DNS (Domain Name System) resolves domain names → IP addresses
- DNS server = like a phone directory
- Public DNS: Cloudflare `1.1.1.1`, Google `8.8.8.8`

### DNS Record Types — MEMORIZE THESE

| Record | Purpose |
|--------|---------|
| **A** | Domain/hostname → IPv4 address |
| **AAAA** | Domain/hostname → IPv6 address |
| **NS** | Nameserver for the domain |
| **MX** | Mail server |
| **CNAME** | Domain alias (canonical name) |
| **TXT** | Text records (SPF, DKIM, verification) |
| **HINFO** | Host information (OS/CPU type) |
| **SOA** | Start of Authority — domain authority info |
| **SRV** | Service location records |
| **PTR** | IP address → hostname (reverse lookup) |

### DNS Interrogation
- Querying a DNS server for all available records on a domain
- Tools: `dnsrecon`, `dig`, `nslookup`

### DNS Zone Transfer
- Admins use zone transfers to replicate zone files between DNS servers
- If **misconfigured**, anyone can request a full copy of the zone file
- Exposes: all subdomains, internal IPs, full network layout
- Test with: `dig axfr @<nameserver> <domain>` or `dnsrecon -d <domain> -t axfr`
- **eJPT tip:** Zone transfer misconfiguration = classic exam scenario

---

## Target Scoping

- Defines what you are **authorized** to test — always confirm before starting
- Key question: *"What am I allowed to collect information about?"*

**Types of targets:**
- **Domain-based:** example.com + its subdomains
- **IP-based:** single IP or CIDR range (e.g. 192.168.1.0/24) — common in internal/lab environments
- **Application-based:** specific web app, login portal, API endpoint only

**In-Scope:** collect, scan, enumerate  
**Out-of-Scope:** third-party services, unlisted domains, systems owned by others

---

## 4-Step Recon Strategy (Use Every Engagement)
```
Step 1 → Define Target       (scope, domain/IP/range)
Step 2 → Passive Recon       (WHOIS, DNS, Netcraft, theHarvester, Sublist3r, Google Dorks)
Step 3 → Active Recon        (Nmap: host discovery, port scan, service detection)
Step 4 → Document Everything (IP, ports, services, OS — per target)
```

---

## Google Dorks — OSINT Search Operators

| Dork | Use |
|------|-----|
| `site:example.com` | All indexed pages of a domain |
| `site:example.com filetype:pdf` | Find exposed documents |
| `intitle:"index of"` | Open directory listings |
| `inurl:admin` | Admin panel pages |
| `site:example.com inurl:login` | Login pages |
| `"@example.com"` | Find email addresses |
| `cache:example.com` | Cached version of a site |

> Resource: exploit-db.com/google-hacking-database (Google Hacking Database)

---

## Common Mistakes to Avoid
- Starting scans without defining scope
- Skipping passive recon entirely
- Scanning everything instead of in-scope targets
- Not documenting findings as you go
- Trusting tool output without manual verification

---

## Quick Command Cheat Sheet — Part 1
```bash
# WHOIS
whois <domain>

# DNS recon
dnsrecon -d <domain>
dnsrecon -d <domain> -t axfr        # Zone transfer test
dig axfr @<nameserver> <domain>      # Zone transfer with dig

# Subdomain enumeration
python sublist3r.py -d <domain>

# Email harvesting
theHarvester -d <domain> -b google
theHarvester -d <domain> -b all      # All sources

# WAF detection
wafw00f <target>
wafw00f -a <target>                  # Test all WAFs
```

---
