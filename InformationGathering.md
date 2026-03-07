# eJPT Study Notes — Information Gathering

---

## What is Information Gathering?
- The **first step** of any penetration test
- Goal: collect as much info as possible about the target (individual, company, website, or system)
- More information = more success in later stages
- Two types: **Passive** and **Active**

---

## Passive vs Active Information Gathering

**Passive:**
- No direct interaction with the target
- Low risk of detection
- Always done **first**
- Examples of data collected: domain registration info, DNS records, public website content, search engine results, publicly available email addresses

**Active:**
- Direct interaction with the target system
- Higher visibility — target may detect you
- Done **after** passive recon
- Requires authorization
- Examples of data collected: live hosts, open ports, running services, network responses

---

## What Information Are We Looking For?

**Passive:**
- IP addresses & DNS info
- Domain names & ownership
- Email addresses & social media profiles
- Web technologies in use
- Subdomains

**Active:**
- Open ports on target systems
- Internal infrastructure details
- Enumerated information from target systems

---

## Target Scoping
- Defines exactly what systems, networks, or applications you are **authorized** to test
- Key question: *"What am I allowed to collect information about?"*

**Types of targets:**
- **Domain-based** — e.g. example.com, including subdomains like mail.example.com
- **IP-based** — single IP (192.168.1.10) or a range (192.168.1.0/24), common in internal/lab environments
- **Application-based** — specific web app, login portal, or API endpoint only

**In-Scope:** assets you can collect from, scan, and enumerate  
**Out-of-Scope:** third-party services, external domains not listed, systems owned by another org

---

## Recon Strategy (4 Steps)
1. **Define the Target** — identify domain/IP/range, confirm scope
2. **Perform Passive Recon** — gather public info, identify attack surfaces
3. **Perform Active Recon** — discover live hosts, open ports, exposed services
4. **Document & Organize Findings** — record domains, IPs, ports; prep for enumeration

---

## Common Beginner Mistakes
- Starting scans without defining scope
- Skipping passive recon
- Scanning everything instead of relevant targets
- Not documenting results
- Trusting tool output without verification

---

## Tools Covered

| Tool | Purpose |
|------|---------|
| **Whois** | Domain ownership/registration info |
| **Netcraft** | Website footprinting, hosting info, tech stack |
| **dnsrecon** | DNS record enumeration |
| **wafw00f** | WAF (Web Application Firewall) detection |
| **Sublist3r** | Subdomain enumeration |
| **theHarvester** | Email harvesting, subdomains, hosts |
| **HaveIBeenPwned** | Leaked password/credential databases |
| **Nmap** | Host discovery & port scanning |

---

## DNS Key Concepts

**What DNS does:** resolves domain names to IP addresses. A DNS server is like a phone directory for the internet.

**Public DNS servers:** Cloudflare (1.1.1.1), Google (8.8.8.8)

**DNS Record Types:**
- **A** — hostname/domain → IPv4 address
- **AAAA** — hostname/domain → IPv6 address
- **NS** — nameserver reference
- **MX** — mail server
- **CNAME** — domain alias
- **TXT** — text record
- **HINFO** — host information
- **SOA** — domain authority
- **SRV** — service records
- **PTR** — IP address → hostname (reverse lookup)

**DNS Interrogation:** querying a DNS server to enumerate records for a domain — reveals IPs, subdomains, mail servers, etc.

**DNS Zone Transfer:**
- Admins use zone transfers to copy zone files between DNS servers
- If **misconfigured**, attackers can pull the entire zone file
- Can reveal a full map of an organization's network
- May also expose internal network addresses
- Use `dig` or `dnsrecon` to test for this

---

## Nmap Essentials

**Host Discovery:** identify which hosts are live on a network  
**Port Scanning:** identify open ports on target systems  
**Service Identification:** detect what services are running on open ports

---

## Key Takeaways
- Always define scope before starting
- Always start with **passive recon** before active recon
- Reconnaissance = collecting **meaningful**, targeted data — not everything
- A structured approach improves accuracy and efficiency
- Document everything as you go