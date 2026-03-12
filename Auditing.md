# eJPT v2 – Auditing Fundamentals Notes

These notes summarize the **Auditing Fundamentals module from INE’s eJPTv2 training** and include practical exam tips from people who have taken the certification.

---

# 1. Introduction to Security Auditing

Security auditing is the process of **systematically evaluating an organization's security posture** to ensure that systems, policies, and procedures meet security standards and compliance requirements.

The goal is to identify weaknesses before attackers do.

### Objectives of Security Auditing

- Identify vulnerabilities in systems, networks, and applications.
- Verify compliance with regulatory frameworks.
- Evaluate effectiveness of security controls.
- Improve existing security policies and procedures.
- Provide recommendations to improve security posture.

Security auditing is part of the **assessment phase of penetration testing**.

---

# 2. Security Auditing vs Penetration Testing

| Feature | Security Audit | Penetration Test |
|------|------|------|
| Goal | Evaluate compliance and security posture | Simulate real attacks |
| Focus | Policies, configurations, controls | Exploitation of vulnerabilities |
| Approach | Review documentation and configurations | Active attacks and exploitation |
| Outcome | Compliance gaps and recommendations | Compromised systems and attack paths |

### Important eJPT Concept

**Audit → Identify weaknesses  
Pentest → Exploit weaknesses**

Many companies perform audits **before penetration tests** to identify configuration problems.

---

# 3. Key Auditing Terminology

### Security Policy
A formal document defining rules, procedures, and guidelines for protecting organizational assets.

Example:
- Password policies
- Access control rules
- Network usage policies

---

### Compliance
Following regulatory standards or legal requirements.

Examples:
- GDPR
- HIPAA
- PCI DSS
- ISO 27001

---

### Vulnerability
A weakness in a system that attackers can exploit.

Examples:
- Weak passwords
- Outdated software
- Misconfigured services

---

### Control
A safeguard designed to reduce risk.

Types of Controls:

| Control Type | Description |
|------|------|
| Preventive | Prevent attacks (firewalls, authentication) |
| Detective | Detect attacks (IDS, logging) |
| Corrective | Fix issues after incidents |

---

### Risk Assessment
Process of identifying and evaluating risks to an organization's assets.

Risk is determined by:
Risk = Likelihood × Impact


---

### Audit Trail
A chronological record of system activities and events.

Examples:

- Login logs
- File access logs
- Configuration changes

Audit trails help during **incident investigations**.

---

### Audit Report
A document that includes:

- Identified vulnerabilities
- Security weaknesses
- Risk severity
- Recommended fixes

---

# 4. Security Auditing Lifecycle

Security auditing follows a structured process.

## 1. Planning and Preparation

Define:

- Scope of the audit
- Objectives
- Systems being evaluated
- Required documentation

Examples of collected documents:

- Security policies
- Network architecture diagrams
- System configuration files

---

## 2. Information Gathering

Collect relevant data from systems.

Methods include:

- Documentation review
- Interviews with staff
- System configuration analysis
- Network scanning

---

## 3. Risk Assessment

Identify:

- Critical assets
- Threats
- Vulnerabilities

Then determine the **risk level of each issue**.

---

## 4. Audit Execution

Perform technical evaluations such as:

- Configuration review
- Vulnerability scanning
- Compliance checks
- Security control validation

---

## 5. Analysis and Evaluation

Analyze collected data.

Compare findings against:

- Industry standards
- Organizational policies
- Security frameworks

---

## 6. Reporting

Create an audit report containing:

- Identified vulnerabilities
- Severity ratings
- Risk analysis
- Remediation recommendations

---

## 7. Remediation

Organization implements fixes such as:

- Updating software
- Strengthening password policies
- Reconfiguring services
- Hardening systems

Follow-up audits verify that fixes are effective.

---

# 5. Types of Security Audits

### Internal Audit

Conducted by the organization's own security team.

Purpose:
- Evaluate internal controls
- Improve security posture

Example:
Reviewing access permissions.

---

### External Audit

Conducted by independent third-party auditors.

Purpose:
- Provide unbiased evaluation
- Validate compliance

Example:
PCI DSS audit by external auditors.

---

### Compliance Audit

Focuses on verifying adherence to regulations.

Examples:

- HIPAA compliance
- GDPR compliance
- PCI DSS compliance

---

### Technical Audit

Examines the technical security of infrastructure.

Includes:

- Server configurations
- Firewall rules
- Network architecture
- System hardening

---

### Network Audit

Focuses on network security.

Examples:

- Open ports
- Insecure protocols
- Router configurations

---

### Application Audit

Focuses on application security.

Examples:

- Input validation
- Authentication mechanisms
- SQL injection risks
- Cross-site scripting (XSS)

---

# 6. Security Frameworks and Standards

Understanding these frameworks is important for security audits.

---

## NIST Cybersecurity Framework (CSF)

NIST defines five core functions:

- Identify
- Protect
- Detect
- Respond
- Recover


This framework helps organizations manage cybersecurity risks.

---

## ISO/IEC 27001

International standard for **Information Security Management Systems (ISMS)**.

Focuses on:

- Risk management
- Security policies
- Continuous improvement

---

## PCI DSS

Security standard for organizations that process **credit card payments**.

Requirements include:

- Secure network architecture
- Encryption of cardholder data
- Strong access control

---

## NIST SP 800-53

Catalog of security controls for information systems.

Common control categories:

- Access control
- Incident response
- System integrity
- Risk management

---

## CIS Critical Security Controls

A prioritized set of **best practices for cybersecurity defense**.

Examples:

- Inventory of assets
- Secure configurations
- Continuous vulnerability management

---

# 7. Practical Security Auditing Tool – Lynis

### Lynis

Lynis is an open-source security auditing tool used to analyze **Linux and Unix systems**.

It checks for:

- System vulnerabilities
- Misconfigurations
- Weak security policies
- Compliance issues

Supported platforms:

- Linux
- macOS
- BSD systems

---

### Installing Lynis

git clone https://github.com/CISOfy/lynis

cd lynis

### Running a Scan

sudo lynis audit system


The scan generates a report showing:

- security warnings
- hardening suggestions
- configuration issues

---

### Example Remediation Steps

- Update outdated packages
- Enforce strong password policies
- Disable insecure services
- Improve system logging

After remediation, organizations usually perform a **penetration test** to verify that the vulnerabilities are fixed.

---

# 8. eJPT Exam Tips (From Candidates)

### Tip 1 – Understand the Audit → Pentest Workflow

The exam sometimes simulates a **real penetration testing engagement workflow**:

1. Perform audit / enumeration
2. Identify vulnerabilities
3. Exploit them during the pentest phase

---

### Tip 2 – Focus on Misconfigurations

In the labs, vulnerabilities are often caused by:

- weak passwords
- outdated software
- open services
- misconfigured permissions

---

### Tip 3 – Use Enumeration Tools

Common tools used in the exam environment:

- Nmap
- Dirb
- Nikto
- Hydra
- Metasploit

---

### Tip 4 – Read Questions Carefully

Some exam questions give hints about:

- the target machine
- the service to attack
- the vulnerability type

---

### Tip 5 – Document Findings

Think like a real pentester.

Always ask:

- What vulnerability exists?
- What caused it?
- How can it be fixed?

---

# Quick Summary

Security auditing focuses on evaluating security posture through:

- policy review
- configuration analysis
- compliance verification
- vulnerability identification

Key components include:

- auditing lifecycle
- frameworks and standards
- vulnerability assessment
- reporting and remediation

Tools like **Lynis** help automate system auditing and improve security hardening.