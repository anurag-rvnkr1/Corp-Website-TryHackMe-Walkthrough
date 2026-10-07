---
layout: default
title: "Corp Website — TryHackMe CTF Walkthrough"
description: "Professional documentation of the Corp Website TryHackMe CTF covering reconnaissance, Next.js security research, remote code execution, reverse shell access, and Linux privilege escalation."
category: "TryHackMe // Web Security"
tags:
  - TryHackMe
  - Web Security
  - Next.js
  - RCE
  - Linux
  - Privilege Escalation
---

<div class="ctf-hero">

# Corp Website — TryHackMe CTF Walkthrough

<p>
A structured penetration-testing assessment of a Linux-hosted web application, progressing from network reconnaissance and framework identification through validated remote code execution, reverse-shell access, and passwordless sudo-based privilege escalation.
</p>

<div class="ctf-badges">

<span class="ctf-badge">TRYHACKME</span>
<span class="ctf-badge">WEB SECURITY</span>
<span class="ctf-badge">NEXT.JS</span>
<span class="ctf-badge">RCE</span>
<span class="ctf-badge">LINUX</span>
<span class="ctf-badge">PRIVILEGE ESCALATION</span>

</div>

</div>

---

## Navigation

<div class="ctf-toc">

<div class="ctf-toc-title">Documentation Map</div>

- [Mission](#mission)
- [Challenge Profile](#challenge-profile)
- [Assessment Workflow](#assessment-workflow)
- [Skills Demonstrated](#skills-demonstrated)
- [Attack Surface](#attack-surface)
- [Reconnaissance](#reconnaissance)
- [Network and Service Enumeration](#network-and-service-enumeration)
- [Directory and Subdomain Enumeration](#directory-and-subdomain-enumeration)
- [Technology Fingerprinting](#technology-fingerprinting)
- [Vulnerability Discovery](#vulnerability-discovery)
- [Vulnerability Validation](#vulnerability-validation)
- [Exploitation and Initial Access](#exploitation-and-initial-access)
- [Reverse Shell](#reverse-shell)
- [Local Privilege Enumeration](#local-privilege-enumeration)
- [Privilege Escalation](#privilege-escalation)
- [Root Access](#root-access)
- [Complete Attack Timeline](#complete-attack-timeline)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [OWASP Security Concepts](#owasp-security-concepts)
- [Key Findings](#key-findings)
- [Security Recommendations](#security-recommendations)
- [Lessons Learned](#lessons-learned)
- [Evidence Gallery](#evidence-gallery)
- [Tools Used](#tools-used)
- [Portfolio Repository](#portfolio-repository)
- [Portfolio Value](#portfolio-value)
- [References](#references)
- [Responsible Use](#responsible-use)
- [Final Summary](#final-summary)

</div>

---

# Mission

The **Corp Website** challenge demonstrates a complete offensive-security workflow against a Linux-hosted web application.

The assessment began with network and application reconnaissance, progressed through technology fingerprinting and automated vulnerability discovery, and then used manual HTTP analysis to validate remote command execution.

After obtaining host-level access, local privilege enumeration revealed an overly permissive `sudo` configuration that allowed passwordless execution of `/usr/bin/python3`. This configuration was then leveraged to obtain root access within the authorized TryHackMe environment.

<div class="key-finding">

<div class="key-finding-title">Assessment Objective</div>

Identify the exposed application, determine its technology stack, discover and validate the attack path, obtain host-level access, enumerate local privileges, and complete the challenge with root-level access.

</div>

---

# Challenge Profile

<div class="ctf-card-grid">

<div class="ctf-card">
<div class="ctf-card-title">Platform</div>
<div class="ctf-card-value">TryHackMe</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Room</div>
<div class="ctf-card-value">Corp Website</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Theme</div>
<div class="ctf-card-value">Romance &amp; Co</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Difficulty</div>
<div class="ctf-card-value">Medium</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Category</div>
<div class="ctf-card-value">Web Security</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Target OS</div>
<div class="ctf-card-value">Linux</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Web Port</div>
<div class="ctf-card-value">3000/tcp</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Framework</div>
<div class="ctf-card-value">Next.js</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Initial Access</div>
<div class="ctf-card-value">Remote Code Execution</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Post-Exploitation</div>
<div class="ctf-card-value">Reverse Shell</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Privilege Escalation</div>
<div class="ctf-card-value">Passwordless sudo</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Privileged Binary</div>
<div class="ctf-card-value">/usr/bin/python3</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Final Objective</div>
<div class="ctf-card-value">Root Access</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Status</div>
<div class="ctf-card-value">Completed</div>
</div>

</div>

---

# Assessment Workflow

The assessment followed an adaptive methodology rather than relying on a single enumeration technique.

<div class="attack-chain">

<div class="attack-step">
<strong>01</strong>
<span>Web Application</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>02</strong>
<span>Technology Fingerprinting</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>03</strong>
<span>Vulnerability Discovery</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>04</strong>
<span>Remote Code Execution</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>05</strong>
<span>Reverse Shell</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>06</strong>
<span>Local Enumeration</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>07</strong>
<span>Sudo Misconfiguration</span>
</div>

<div class="attack-arrow">↓</div>

<div class="attack-step">
<strong>08</strong>
<span>Root Access</span>
</div>

</div>

The resulting attack path can be summarized as:

```text
Port 3000
    ↓
Next.js
    ↓
Framework Vulnerability
    ↓
Remote Code Execution
    ↓
Reverse Shell
    ↓
sudo -l
    ↓
Passwordless Python
    ↓
Root
```

---

# Skills Demonstrated

<div class="tool-list">

<span class="tool-tag">Network Reconnaissance</span>
<span class="tool-tag">Service Enumeration</span>
<span class="tool-tag">Web Application Enumeration</span>
<span class="tool-tag">Directory Discovery</span>
<span class="tool-tag">Subdomain Discovery</span>
<span class="tool-tag">Technology Fingerprinting</span>
<span class="tool-tag">Next.js Security Research</span>
<span class="tool-tag">Vulnerability Scanning</span>
<span class="tool-tag">HTTP Request Analysis</span>
<span class="tool-tag">RCE Validation</span>
<span class="tool-tag">Reverse Shells</span>
<span class="tool-tag">Linux Privilege Enumeration</span>
<span class="tool-tag">Sudo Analysis</span>
<span class="tool-tag">Privilege Escalation</span>
<span class="tool-tag">Security Documentation</span>

</div>

---

# Attack Surface

The initial assessment identified a web application exposed on:

```text
3000/tcp
```

The exposed application became the primary attack surface.

The assessment then progressed through four major security layers:

| Layer | Assessment Focus | Result |
|---|---|---|
| Network | Port and service discovery | Web application identified |
| Application | Directory, subdomain, and framework analysis | Next.js identified |
| Exploitation | Automated discovery and manual validation | RCE confirmed |
| Host | Shell and privilege enumeration | Root access obtained |

This progression demonstrates how application-layer reconnaissance can transition into host-level post-exploitation when vulnerabilities are successfully chained.

---

# Reconnaissance

## Room and Target Context

The initial room context established the target environment and provided the starting point for the assessment.

![Corp Website TryHackMe room context](assets/01_room.png)

*Figure 1 — TryHackMe Corp Website room context.*

The assessment then moved from contextual information to direct network reconnaissance.

---

# Network and Service Enumeration

## Nmap Service Enumeration

A comprehensive Nmap scan was performed to identify open ports, services, versions, operating-system information, and additional reconnaissance data.

### Command

```bash
nmap -sS -sV -A -v <TARGET_IP> -oN nmap.txt
```

### What It Does

The scan combines:

- SYN-based port scanning.
- Service and version detection.
- Operating-system and platform detection.
- Aggressive reconnaissance.
- Verbose output.
- Local output storage in `nmap.txt`.

### Why Nmap Was Used

The objective was to establish:

- Available network services.
- Service versions.
- Application technologies.
- Operating-system characteristics.
- Additional reconnaissance information.

Saving the output to `nmap.txt` also provided a local record of the reconnaissance results.

### Key Finding

The scan identified the primary web application on:

```text
3000/tcp
```

The service fingerprint also provided an important technology clue associated with the application's JavaScript framework.

### Evidence

![Nmap scan results](assets/02_nmap.png)

*Figure 2 — Nmap reconnaissance identifying the web service exposed on TCP port 3000.*

<div class="key-finding">

<div class="key-finding-title">Reconnaissance Finding</div>

The exposed web service on `3000/tcp` became the primary application attack surface for the remainder of the assessment.

</div>

---

# Directory and Subdomain Enumeration

With the HTTP service identified, the next step was to determine whether additional web content, routes, virtual hosts, or subdomains were available.

## Tools Used

<div class="tool-list">

<span class="tool-tag">Gobuster</span>
<span class="tool-tag">Amass</span>
<span class="tool-tag">Subfinder</span>

</div>

## Enumeration Goals

The enumeration phase focused on discovering:

- Hidden directories.
- Administrative routes.
- Backup files.
- API paths.
- Additional virtual hosts.
- Subdomains.

### Evidence

![Directory and subdomain enumeration](assets/03_enum.png)

*Figure 3 — Directory and subdomain enumeration using multiple reconnaissance tools.*

## Result

No additional useful directories or subdomains were identified.

This result was significant because it indicated that continuing with blind content discovery was unlikely to reveal the primary attack vector.

The assessment therefore pivoted toward **application and framework analysis**.

<div class="key-finding">

<div class="key-finding-title">Enumeration Finding</div>

Generic directory and subdomain enumeration did not expose a useful additional attack surface. The methodology therefore shifted toward identifying and researching the underlying application framework.

</div>

---

# Technology Fingerprinting

## Identifying Next.js

Inspection of the web application revealed that it was built using **Next.js**.

This represented an important change in the assessment strategy.

Rather than continuing exclusively with generic fuzzing, identifying the underlying framework provided a more focused direction for security research and manual testing.

### Evidence

![Next.js technology fingerprint](assets/04_nextjs.png)

*Figure 4 — Evidence of the application's Next.js technology stack.*

## Why Technology Fingerprinting Matters

Technology fingerprinting can:

- Narrow the vulnerability research space.
- Reveal framework-specific attack surfaces.
- Enable version-specific security research where applicable.
- Guide subsequent manual testing.
- Help distinguish application-specific behavior from generic HTTP behavior.

## Assessment Decision

The methodology changed from:

```text
Generic Enumeration
```

to:

```text
Framework-Specific Security Research
```

This adaptive decision ultimately led toward the successful attack path.

---

# Vulnerability Discovery

## Nuclei Assessment

After identifying Next.js, Nuclei was used to compare the exposed application against known vulnerability templates.

### Command

```bash
nuclei -u http://<TARGET_IP>:3000
```

### Objective

The objective was to identify known vulnerabilities relevant to the detected application and technology stack.

Automated scanning was treated as a **vulnerability-discovery aid**, rather than as proof that a finding was exploitable.

### Evidence

![Nuclei vulnerability scan](assets/05_nuclei.png)

*Figure 5 — Nuclei identifying a potential framework-related remote code execution issue.*

## Finding

The scan identified a potential **remote code execution** issue associated with the Next.js application.

The result was treated as a lead requiring manual validation.

<div class="key-finding">

<div class="key-finding-title">Vulnerability Discovery Finding</div>

Nuclei provided a framework-related RCE lead. The finding was not treated as confirmed until the application behavior was manually validated through HTTP request analysis.

</div>

---

# Vulnerability Validation

## Manual Verification with Burp Suite

Automated detection was followed by manual validation using **Burp Suite**.

The purpose of this stage was to inspect the application's HTTP request structure and reproduce the suspected vulnerable behavior.

## Validation Process

```text
Intercept Request
      ↓
Inspect Request Structure
      ↓
Modify HTTP Method / Body
      ↓
Replay Request
      ↓
Observe Server Response
      ↓
Confirm Command Execution
```

### Evidence

![Burp Suite RCE validation](assets/06_burp_rce.png)

*Figure 6 — Burp Suite request manipulation used to validate remote command execution.*

## Validation Result

Manual validation confirmed that attacker-controlled input could result in command execution on the target application.

This established the initial compromise path.

<div class="key-finding">

<div class="key-finding-title">Validation Finding</div>

The suspected RCE condition was manually validated. The assessment therefore progressed from vulnerability discovery into controlled exploitation and host-level access.

</div>

## Payload Handling

Exact exploit payloads are intentionally omitted from this portfolio page.

The documented methodology preserves the vulnerability discovery and validation process while keeping the write-up focused on assessment reasoning and responsible presentation.

---

# Exploitation and Initial Access

## Remote Code Execution

With command execution confirmed, the assessment transitioned from web application testing into post-exploitation.

The validated RCE primitive provided the ability to interact with the underlying Linux environment.

The next objective was to establish a more usable interactive shell.

<div class="attack-chain">

<div class="attack-step">
<strong>01</strong>
<span>Next.js Application</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>02</strong>
<span>Validated RCE</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>03</strong>
<span>Command Execution</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>04</strong>
<span>Host-Level Access</span>
</div>

</div>

The successful transition from the application layer to the underlying host represented the initial foothold.

---

# Reverse Shell

## Establishing Interactive Access

A Netcat listener was prepared on the attacking machine.

### Listener

```bash
nc -lnvp 1337
```

The validated command-execution primitive was then used to trigger a connection back to the listener.

### Evidence

![Reverse shell](assets/07_shell.png)

*Figure 7 — Reverse shell established on the target Linux host.*

## Shell Verification

Once the shell was received, the execution context was verified.

The post-exploitation phase focused on determining:

- Current user.
- Host identity.
- Working directory.
- Application files.
- Available privileges.

This confirmed that the web-layer compromise had successfully transitioned into host-level access.

<div class="key-finding">

<div class="key-finding-title">Initial Access Result</div>

The validated RCE condition was successfully converted into an interactive reverse shell, providing host-level access to the Linux environment.

</div>

---

# Local Privilege Enumeration

## Checking Sudo Permissions

After gaining an interactive shell, local privilege enumeration was performed.

One of the first high-value checks was:

### Command

```bash
sudo -l
```

### What the Command Does

`sudo -l` displays the commands the current user is permitted to execute through `sudo`.

### Why It Was Used

The command was used to determine whether the compromised account had access to privileged executables.

### Why the Result Matters

A permissive `sudoers` configuration can create a path from a low-privilege account to elevated execution.

### Evidence

![Sudo privilege enumeration](assets/08_sudo.png)

*Figure 8 — Sudo enumeration revealing passwordless execution of Python 3.*

## Finding

The compromised user was permitted to execute:

```text
/usr/bin/python3
```

with elevated privileges without requiring a password.

<div class="key-finding">

<div class="key-finding-title">Privilege Boundary Weakness</div>

The documented sudo configuration allowed passwordless elevated execution of `/usr/bin/python3`. Because Python is a general-purpose scripting interpreter capable of executing operating-system commands, unrestricted privileged execution creates a direct path toward administrative command execution.

</div>

---

# Privilege Escalation

## Sudo Misconfiguration

The discovered configuration represented a significant privilege-boundary weakness.

Python is a general-purpose scripting interpreter capable of executing operating-system commands. Allowing unrestricted privileged execution of such an interpreter can effectively provide a path to administrative command execution.

## Documented Escalation Path

<div class="attack-chain">

<div class="attack-step">
<strong>01</strong>
<span>Low-Privilege User</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>02</strong>
<span>sudo -l</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>03</strong>
<span>NOPASSWD Python 3</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>04</strong>
<span>Privileged Interpreter</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>05</strong>
<span>Root Shell</span>
</div>

</div>

The same escalation path can be represented textually as:

```text
Low-Privilege User
       ↓
sudo -l
       ↓
NOPASSWD Python 3
       ↓
Privileged Interpreter
       ↓
Root Shell
```

## Result

The misconfiguration was successfully leveraged within the authorized TryHackMe environment to obtain root privileges.

---

# Root Access

## Final Privilege Verification

Once privilege escalation succeeded, the target was operating in the root security context.

### Evidence

![Root shell](assets/09_root.png)

*Figure 9 — Root access successfully obtained on the target system.*

## Final Objective

The root flag was successfully retrieved from:

```text
/root/root.txt
```

### Flag Status

```text
THM{REDACTED}
```

> **The actual flag is intentionally hidden.**

The documented objective was therefore completed with root-level access.

<div class="key-finding">

<div class="key-finding-title">Final Result</div>

The complete attack chain progressed from an exposed web application to validated RCE, an interactive Linux shell, passwordless privileged Python execution, and ultimately root access.

</div>

---

# Complete Attack Timeline

| Phase | Activity | Result |
|---|---|---|
| 01 | Room analysis | Target application identified |
| 02 | Nmap | Web service discovered on port 3000 |
| 03 | Gobuster / Amass / Subfinder | No useful additional attack surface |
| 04 | Technology fingerprinting | Next.js identified |
| 05 | Nuclei | Potential RCE discovered |
| 06 | Burp Suite | RCE manually validated |
| 07 | Command execution | Initial foothold obtained |
| 08 | Netcat | Reverse shell established |
| 09 | `sudo -l` | Passwordless Python privilege discovered |
| 10 | Privilege escalation | Root shell obtained |
| 11 | Post-exploitation | Root objective completed |

---

# MITRE ATT&CK Mapping

The documented workflow can be associated with the following ATT&CK techniques.

| Technique | ID | Evidence |
|---|---|---|
| Network Service Scanning | **T1046** | Nmap reconnaissance of the target |
| Exploit Public-Facing Application | **T1190** | Exploitation of the exposed web application |
| Command and Scripting Interpreter | **T1059** | Command execution following RCE |
| Unix Shell | **T1059.004** | Linux shell interaction after obtaining host access |
| Sudo and Sudo Caching | **T1548.003** | Sudo-based privilege escalation through the documented privileged Python configuration |

> These mappings are used as a learning aid for understanding how the observed lab workflow corresponds to ATT&CK techniques.

---

# OWASP Security Concepts

The documented workflow also demonstrates several application and infrastructure security concepts.

| Security Concept | Demonstrated Through |
|---|---|
| **Vulnerable Components** | Framework vulnerability research involving the identified Next.js application |
| **Security Misconfiguration** | Insecure sudo privilege configuration |
| **Access Control / Privilege Boundaries** | Excessive privileged execution rights |

The challenge demonstrates how weaknesses at different security layers can be chained:

```text
Application Weakness
        ↓
Remote Command Execution
        ↓
Host Access
        ↓
Privilege Configuration Weakness
        ↓
Root Access
```

---

# Key Findings

## Finding 01 — Exposed Web Application

The target exposed a web application on TCP port `3000`, making the application the primary initial attack surface.

**Security significance:** An externally reachable application represents the first boundary that must be assessed for exposed functionality, technology, configuration, and vulnerabilities.

---

## Finding 02 — Next.js Technology Stack

Technology fingerprinting identified Next.js, allowing the assessment to move from generic enumeration toward framework-specific security research.

**Security significance:** Accurate technology identification can substantially narrow the security research space and improve assessment efficiency.

---

## Finding 03 — Potential RCE

Nuclei identified a potential framework-related remote code execution issue, which was subsequently manually validated through Burp Suite.

**Security significance:** Automated scanner output should be treated as a lead until the underlying behavior has been independently verified.

---

## Finding 04 — Command Execution to Host Access

The validated RCE condition enabled command execution and was used to establish an interactive reverse shell on the Linux target.

**Security significance:** Successful application-layer RCE can cross the application-to-operating-system security boundary.

---

## Finding 05 — Excessive Sudo Privilege

The compromised account could execute `/usr/bin/python3` through `sudo` without a password, creating the documented privilege-escalation path to root.

**Security significance:** Unrestricted privileged execution of a general-purpose interpreter can undermine the intended separation between low-privilege and administrative execution.

---

# Security Recommendations

The following recommendations are derived directly from the weaknesses documented in the assessment.

## Web Application

### Finding

The exposed application presented a framework-specific attack surface and was associated with a validated remote code execution condition.

### Risk

Remote code execution can allow an attacker to move from application-layer interaction to operating-system command execution.

### Recommended Security Controls

- Keep Next.js and application dependencies updated.
- Track security advisories affecting deployed frameworks.
- Review exposed HTTP endpoints regularly.
- Monitor anomalous HTTP requests.
- Validate and constrain server-side input processing.

---

## Linux Privilege Configuration

### Finding

The compromised user had passwordless elevated execution permission for:

```text
/usr/bin/python3
```

### Risk

A general-purpose scripting interpreter with unrestricted privileged execution can undermine the intended operating-system privilege boundary.

### Recommended Security Controls

- Apply least-privilege principles.
- Audit `sudoers` configuration regularly.
- Avoid unrestricted sudo access to scripting interpreters.
- Monitor privileged process execution.
- Review administrative permissions periodically.

---

## Detection Opportunities

The documented attack chain also provides several useful defensive detection points:

```text
Unexpected Web Server → Shell Process

Unexpected Outbound Reverse Connection

Suspicious POST Requests

Privileged Python Execution

Unusual Sudo Activity
```

These events can provide useful signals when investigating web-server compromise followed by host-level privilege escalation.

---

# Lessons Learned

## 1. Enumeration Must Be Adaptive

Directory and subdomain enumeration did not expose a useful additional attack surface.

Continuing blindly with the same methodology would have added limited value. Identifying the underlying framework provided a stronger direction for the assessment.

---

## 2. Framework Fingerprinting Is Valuable

Identifying Next.js significantly narrowed the vulnerability research scope.

Technology identification should therefore be treated as an important part of web application reconnaissance rather than merely an informational step.

---

## 3. Automated Findings Must Be Validated

Nuclei identified a potential issue, but the result was treated as a lead.

Burp Suite was then used to manually inspect and reproduce the suspected behavior. This provided the validation necessary to establish that command execution was possible.

---

## 4. Initial Access Is Not the End

Obtaining command execution and a reverse shell represented only the beginning of the host-level assessment.

Local privilege enumeration was required to determine whether the compromised account could move to a higher privilege level.

---

## 5. Sudo Configuration Requires Care

A single overly permissive sudo rule can undermine the intended operating-system privilege model.

Allowing unrestricted privileged execution of a general-purpose interpreter such as Python can effectively eliminate the intended separation between low-privilege and administrative execution.

---

# Evidence Gallery

All evidence captured during the assessment is maintained under:

```text
docs/assets/
```

## Room Context

![Room context](assets/01_room.png)

**Figure 1 — TryHackMe Corp Website room context.**

---

## Network Reconnaissance

![Nmap reconnaissance](assets/02_nmap.png)

**Figure 2 — Nmap reconnaissance identifying the web service exposed on TCP port 3000.**

---

## Directory and Subdomain Enumeration

![Enumeration](assets/03_enum.png)

**Figure 3 — Directory and subdomain enumeration using Gobuster, Amass, and Subfinder.**

---

## Next.js Fingerprinting

![Next.js fingerprinting](assets/04_nextjs.png)

**Figure 4 — Evidence of the application's Next.js technology stack.**

---

## Nuclei Vulnerability Discovery

![Nuclei scan](assets/05_nuclei.png)

**Figure 5 — Nuclei identifying a potential framework-related remote code execution issue.**

---

## Burp Suite RCE Validation

![Burp Suite RCE validation](assets/06_burp_rce.png)

**Figure 6 — Burp Suite request manipulation used to validate remote command execution.**

---

## Reverse Shell

![Reverse shell](assets/07_shell.png)

**Figure 7 — Reverse shell established on the target Linux host.**

---

## Sudo Enumeration

![Sudo enumeration](assets/08_sudo.png)

**Figure 8 — Sudo enumeration revealing passwordless execution of Python 3.**

---

## Root Access

![Root access](assets/09_root.png)

**Figure 9 — Root access successfully obtained on the target system.**

---

# Tools Used

<div class="ctf-card-grid">

<div class="ctf-card">
<div class="ctf-card-title">Nmap</div>
<div class="ctf-card-value">Network &amp; Service Enumeration</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Gobuster</div>
<div class="ctf-card-value">Directory Discovery</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Amass</div>
<div class="ctf-card-value">Subdomain Enumeration</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Subfinder</div>
<div class="ctf-card-value">Passive Subdomain Enumeration</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Nuclei</div>
<div class="ctf-card-value">Vulnerability Discovery</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Burp Suite</div>
<div class="ctf-card-value">HTTP Analysis &amp; Validation</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Netcat</div>
<div class="ctf-card-value">Reverse Shell Listener</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Linux CLI</div>
<div class="ctf-card-value">Post-Exploitation Enumeration</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">sudo</div>
<div class="ctf-card-value">Privilege Analysis</div>
</div>

<div class="ctf-card">
<div class="ctf-card-title">Python 3</div>
<div class="ctf-card-value">Privileged Execution Analysis</div>
</div>

</div>

---

# Portfolio Repository

The documented project contains the walkthrough, supporting notes, evidence, and portfolio documentation.

```text
Corp-Website-TryHackMe-Walkthrough/
│
├── README.md
│
├── Documentation/
│   ├── Corp_Website_Documentation.md
│   ├── Corp_Website_Documentation.docx
│   └── Tools_Used.md
│
├── Resources/
│   ├── notes.md
│   └── references.md
│
└── docs/
    ├── index.md
    └── assets/
        ├── 01_room.png
        ├── 02_nmap.png
        ├── 03_enum.png
        ├── 04_nextjs.png
        ├── 05_nuclei.png
        ├── 06_burp_rce.png
        ├── 07_shell.png
        ├── 08_sudo.png
        └── 09_root.png
```

---

# Portfolio Value

This CTF demonstrates practical exposure to a complete offensive-security workflow rather than an isolated exploit.

## Demonstrated Workflow

<div class="attack-chain">

<div class="attack-step">
<strong>01</strong>
<span>Reconnaissance</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>02</strong>
<span>Enumeration</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>03</strong>
<span>Technology Analysis</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>04</strong>
<span>Vulnerability Research</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>05</strong>
<span>Manual Validation</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>06</strong>
<span>Initial Access</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>07</strong>
<span>Post-Exploitation</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>08</strong>
<span>Privilege Escalation</span>
</div>

<div class="attack-arrow">→</div>

<div class="attack-step">
<strong>09</strong>
<span>Security Reporting</span>
</div>

</div>

The assessment demonstrates practical exposure to:

<div class="tool-list">

<span class="tool-tag">Web Application Security</span>
<span class="tool-tag">Linux Security</span>
<span class="tool-tag">Vulnerability Assessment</span>
<span class="tool-tag">Penetration Testing</span>
<span class="tool-tag">Post-Exploitation</span>
<span class="tool-tag">Privilege Escalation</span>
<span class="tool-tag">Security Documentation</span>

</div>

The strongest portfolio value comes from the demonstrated ability to **adapt the assessment methodology based on reconnaissance results**, validate automated findings manually, transition from application-layer compromise to host-level access, and identify a local privilege-boundary weakness.

---

# References

The original documentation does not provide a separate external-reference list beyond the tools, platform, and resources represented within the walkthrough.

No additional external URLs, CVEs, advisories, or research references have been introduced into this transformation.

---

# Responsible Use

> This documentation describes activity performed inside an **authorized TryHackMe CTF environment**.
>
> The techniques presented here are intended for cybersecurity education, Capture The Flag training, authorized penetration testing, controlled security research, and defensive security learning.
>
> Do not apply these techniques to systems without explicit authorization.

---

# Final Summary

The **Corp Website** challenge demonstrates how a seemingly simple web application can become the entry point to a full host compromise when multiple weaknesses are chained together.

The documented attack path was:

```text
Port 3000
    ↓
Next.js
    ↓
Framework Vulnerability
    ↓
Remote Code Execution
    ↓
Reverse Shell
    ↓
sudo -l
    ↓
Passwordless Python
    ↓
Root
```

The central lesson from the engagement was the importance of **adaptive enumeration**.

When conventional directory and subdomain enumeration produced limited results, framework identification provided a more focused direction. Nuclei then surfaced a potential RCE condition, while Burp Suite was used to manually validate the behavior.

After obtaining host-level access, `sudo -l` revealed a passwordless privileged Python configuration that enabled the final escalation to root.

<div class="key-finding">

<div class="key-finding-title">Assessment Takeaway</div>

**Identify the technology. Validate the weakness. Enumerate the host. Understand the privilege boundary. Document the evidence.**

</div>

---

<div class="ctf-footer">

**RECON → ENUMERATE → VALIDATE → EXPLOIT → ESCALATE → DOCUMENT**

**TryHackMe • Corp Website • Portfolio Documentation**

</div>
