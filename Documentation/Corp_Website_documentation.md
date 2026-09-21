# Corp Website (Romance & Co) — TryHackMe Technical Walkthrough

<p align="center">
  <img src="../Assets/banner.png" alt="Corp Website TryHackMe Banner" width="100%">
</p>

<p align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-Corp%20Website-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20Security-blue?style=for-the-badge)
![Platform](https://img.shields.io/badge/OS-Linux-black?style=for-the-badge&logo=linux)

</p>

---

## Executive Summary

The **Corp Website (Romance & Co)** room is a medium-difficulty **TryHackMe** Capture The Flag challenge focused on **web application penetration testing** and **Linux privilege escalation**. The objective is to enumerate a target web application, identify the underlying technology stack, discover an exploitable vulnerability, obtain initial access through **Remote Code Execution (RCE)**, establish an interactive reverse shell, and escalate privileges to obtain root access.

Unlike many web challenges, this room demonstrates that traditional directory fuzzing alone is not always sufficient. Success depends on accurately fingerprinting the application's framework, researching framework-specific vulnerabilities, validating findings manually, and performing thorough post-exploitation enumeration.

> **Note:** Challenge flags have been intentionally **redacted** throughout this documentation to preserve the learning experience and comply with TryHackMe community guidelines.

---

# Table of Contents

- Executive Summary
- Lab Information
- Objectives
- Attack Path Overview
- Reconnaissance
- Web Enumeration
- Technology Fingerprinting
- Vulnerability Assessment
- Exploitation
- Reverse Shell
- Privilege Escalation
- Root Access
- MITRE ATT&CK Mapping
- OWASP Mapping
- Defensive Recommendations
- Lessons Learned
- Conclusion

---

# Lab Information

| Attribute | Details |
|-----------|---------|
| **Platform** | TryHackMe |
| **Room Name** | Corp Website (Romance & Co) |
| **Difficulty** | Medium |
| **Category** | Web Exploitation |
| **Target Environment** | Linux |
| **Primary Technology** | Next.js |
| **Initial Access** | Remote Code Execution |
| **Privilege Escalation** | Passwordless Sudo Misconfiguration |
| **Final Objective** | Root Access |

---

# Learning Objectives

During this lab, the following cybersecurity concepts were practiced:

- Network reconnaissance and service enumeration.
- Directory and subdomain enumeration.
- Web application fingerprinting.
- Framework identification.
- Vulnerability discovery using automated scanners.
- Manual vulnerability validation using Burp Suite.
- Remote Code Execution concepts.
- Reverse shell establishment.
- Linux privilege enumeration.
- Passwordless sudo privilege escalation.
- Professional penetration testing documentation.

---

# Attack Path Overview

The complete attack chain followed during this assessment is illustrated below.

<p align="center">
  <img src="../Assets/attack-chain.png" width="900">
</p>

<p align="center"><em>Figure 1 — High-level attack chain followed during the engagement.</em></p>

| Phase | Goal |
|-------|------|
| Reconnaissance | Discover exposed services. |
| Enumeration | Identify application attack surface. |
| Fingerprinting | Identify technologies powering the application. |
| Vulnerability Research | Detect known framework vulnerabilities. |
| Validation | Confirm vulnerability manually. |
| Exploitation | Achieve Remote Code Execution. |
| Initial Access | Obtain interactive shell access. |
| Privilege Escalation | Escalate to root privileges. |
| Objective Complete | Capture the root flag. |

---

# Reconnaissance

## Objective

The first step of every penetration test is understanding the target environment. Before interacting with the application itself, I performed network reconnaissance to identify exposed services, running technologies, and potential attack surfaces.

The room instructions indicated that the web application was hosted on **port 3000**, so I began by visiting the application in a browser.

### Initial Application View

The website appeared to be a simple corporate landing page with static content and no obvious login forms, search fields, or interactive inputs.

<p align="center">
  <img src="../Screenshots/01_room.png" width="900">
</p>

<p align="center"><em>Figure 2 — Initial view of the Corp Website application hosted on port 3000.</em></p>

### Initial Observations

- Static corporate-style landing page.
- Minimal visible functionality.
- No authentication portal.
- No user input fields.
- No immediately obvious attack vectors.

Although the page looked static, appearance alone does not determine the security posture of an application. Enumeration was required to understand what services were actually exposed.

---

## Service Enumeration with Nmap

To gather more information about the target host, I performed a comprehensive **Nmap** scan.

### Scan Command

```bash
nmap -sS -sV -A -v <TARGET_IP> -oN nmap.txt
```

### Why This Scan?

| Flag | Purpose |
|------|----------|
| `-sS` | TCP SYN scan. |
| `-sV` | Service version detection. |
| `-A` | OS detection, script scanning, and traceroute. |
| `-v` | Verbose output. |
| `-oN` | Save results for documentation. |

The scan provides information about exposed TCP ports, running services, operating system fingerprints, and additional service metadata useful for later exploitation.

### Nmap Scan Result

<p align="center">
  <img src="../Screenshots/02_nmap.png" width="900">
</p>

<p align="center"><em>Figure 3 — Nmap service enumeration results.</em></p>

### Findings

The scan revealed:

- The target exposed an HTTP service on **port 3000**.
- Service fingerprinting confirmed the application was reachable over HTTP.
- Additional version information provided context for later technology identification.

### Reconnaissance Outcome

| Finding | Security Value |
|---------|----------------|
| HTTP Service | Primary attack surface identified. |
| Port 3000 | Non-standard web application port. |
| Version Detection | Useful for technology fingerprinting. |

---

# Web Enumeration

After identifying the exposed web application, I moved into application-level enumeration.

The goal was to discover:

- Hidden directories.
- Administrative panels.
- API endpoints.
- Backup files.
- Additional content not visible from the homepage.

## Directory Enumeration

The first enumeration technique involved brute-forcing directories using **Gobuster**.

Example workflow:

```bash
gobuster dir -u http://<TARGET_IP>:3000 -w /path/to/wordlist.txt
```

The objective was to discover hidden application routes that could expose additional functionality.

### Result

No useful directories were discovered during enumeration.

---

## Subdomain Enumeration

To ensure no additional virtual hosts existed, I also performed subdomain enumeration.

Tools used during this phase included:

- Amass
- Subfinder

These tools help identify publicly available subdomains through passive and active reconnaissance techniques.

### Result

No additional subdomains or virtual hosts were identified.

---

## Enumeration Summary

<p align="center">
  <img src="../Screenshots/03_enum.png" width="900">
</p>

<p align="center"><em>Figure 4 — Directory and subdomain enumeration process.</em></p>

### What Was Attempted?

| Enumeration Technique | Result |
|----------------------|--------|
| Gobuster | No interesting directories discovered. |
| Amass | No useful subdomains identified. |
| Subfinder | No additional hosts discovered. |

### Key Takeaway

This stage is an important reminder that **negative enumeration results are still valuable findings**. Even though traditional content discovery did not expose an attack path, it narrowed the focus toward analyzing the application itself rather than searching for hidden endpoints.

---

# Technology Fingerprinting

## Identifying the Application Framework

After completing network and content enumeration, the next step was identifying the technologies powering the web application. Since directory fuzzing and subdomain enumeration did not reveal additional attack surfaces, understanding the framework became the primary focus.

Modern web applications often expose framework-specific artifacts through JavaScript bundles, routing behavior, static assets, and HTTP response headers. Inspecting these characteristics helps narrow the attack surface and identify vulnerabilities relevant to the underlying technology stack.

### Initial Analysis

Several indicators suggested that the application was built using **Next.js**, a React-based framework commonly used for server-side rendering and static web applications.

### Technology Fingerprinting Evidence

<p align="center">
  <img src="../Screenshots/04_nextjs.png" width="900">
</p>

<p align="center"><em>Figure 5 — Identifying the web application's underlying framework as Next.js.</em></p>

### Indicators Observed

| Indicator | Observation |
|-----------|-------------|
| Static asset structure | Framework-specific asset organization. |
| JavaScript bundles | Naming conventions consistent with Next.js builds. |
| Routing behavior | Client-side routing matched Next.js architecture. |
| Response behavior | Application responses aligned with a Next.js deployment. |

### Why Framework Fingerprinting Matters

Technology fingerprinting changes the direction of a penetration test. Instead of performing generic web testing indefinitely, identifying the framework allows testing to focus on publicly documented vulnerabilities, misconfigurations, and security advisories relevant to that technology.

> **Key Lesson:** Framework identification can reveal attack paths that traditional directory fuzzing may never expose.

---

# Vulnerability Assessment

## Automated Vulnerability Discovery

Once the framework was identified, I moved into vulnerability assessment using **Nuclei**. The objective was to compare the detected technology against publicly known vulnerability templates.

### Scanner Used

- **Nuclei**

### Purpose

- Detect known vulnerabilities affecting exposed technologies.
- Quickly identify potential attack vectors.
- Generate findings that can later be validated manually.

### Scan Workflow

```bash
nuclei -u http://<TARGET_IP>:3000
```

This scan evaluates the target against community-maintained vulnerability templates for web technologies and frameworks.

### Nuclei Scan Output

<p align="center">
  <img src="../Screenshots/05_nuclei.png" width="900">
</p>

<p align="center"><em>Figure 6 — Nuclei identifying a potential vulnerability affecting the detected framework.</em></p>

---

## Analysis of the Finding

During the scan, one result stood out because it was associated with a **publicly documented vulnerability** affecting the identified framework.

### Important Observation

The scanner suggested the application might be affected by a vulnerability capable of leading to **Remote Code Execution (RCE)** if certain conditions were met.

### Research Phase

Rather than immediately assuming the finding was exploitable, I reviewed publicly available security information related to the identified vulnerability.

The research focused on:

- Affected framework versions.
- Conditions required for exploitation.
- Public advisories and disclosures.
- Whether the lab environment intentionally simulated the vulnerable behavior.

### Why Manual Validation Was Necessary

Automated scanners can generate:

- False positives.
- Version-based matches.
- Informational findings.
- Potential attack paths.

A scanner result alone is **not proof** of exploitation.

The next phase was validating whether the application actually exhibited the vulnerable behavior.

---

# Manual Vulnerability Validation

## Using Burp Suite

To validate the suspected vulnerability, I intercepted application traffic using **Burp Suite**.

Burp Suite provides visibility into HTTP requests and responses, making it possible to understand how the application processes user input.

### Validation Workflow

1. Capture an HTTP request sent by the application.
2. Inspect request headers and body.
3. Modify the request method where appropriate.
4. Replay the request to the target.
5. Observe server behavior.

### Why Burp Suite?

| Feature | Purpose |
|--------|---------|
| Proxy | Capture application traffic. |
| Repeater | Replay and modify requests. |
| Inspector | Analyze headers, parameters, and body. |
| HTTP History | Compare request/response behavior. |

---

## Request Analysis

During inspection, I identified an endpoint that accepted structured request data. The request was modified for testing purposes inside **Burp Repeater**.

> **Security Note:** The exact payload used in the TryHackMe lab is intentionally omitted from this repository. This documentation focuses on the methodology rather than publishing exploit payloads.

### Burp Suite Validation

<p align="center">
  <img src="../Screenshots/06_burp_rce.png" width="900">
</p>

<p align="center"><em>Figure 7 — HTTP request interception and manual validation using Burp Suite.</em></p>

### Validation Outcome

After replaying the modified request, the application responded in a way that confirmed **remote command execution** within the authorized lab environment.

This established the first successful foothold on the target.

---

# Initial Access

## Confirming Remote Code Execution

Successful validation demonstrated that the application executed attacker-controlled commands.

This confirmed that the vulnerability was not merely detected by version matching—it was exploitable within the challenge environment.

### Initial Access Achieved

| Objective | Status |
|-----------|--------|
| Framework Identified | ✅ |
| Vulnerability Detected | ✅ |
| Manual Validation Completed | ✅ |
| Remote Code Execution Confirmed | ✅ |

### Security Impact

Remote Code Execution represents one of the highest-impact web application vulnerabilities because it allows an attacker to execute commands on the underlying server within the permissions of the affected service.

In this challenge, RCE served as the transition point from web application testing to operating system post-exploitation.

---

# Capturing the User Flag

After confirming command execution, I explored the accessible filesystem from the compromised user context.

The initial challenge objective was successfully completed.

### User Flag

> **Flag Redacted**

```text
THM{***********************}
```

The actual flag has been intentionally removed from this documentation to avoid spoilers and plagiarism while preserving the educational workflow.

---

# Phase Summary

The assessment reached a significant milestone during this stage.

| Phase | Outcome |
|-------|---------|
| Technology Fingerprinting | Next.js framework identified. |
| Vulnerability Assessment | Publicly documented framework issue discovered. |
| Manual Validation | Burp Suite confirmed application behavior. |
| Initial Exploitation | Remote Code Execution achieved. |
| Initial Objective | User flag captured (redacted). |

---

## Key Takeaways from This Phase

- Technology fingerprinting is often more valuable than continuing blind directory fuzzing.
- Vulnerability scanners should be treated as reconnaissance tools, not proof of exploitation.
- Manual request validation is essential for confirming security findings.
- Gaining command execution is only the beginning of an assessment; post-exploitation enumeration is the next critical step.

---

# Post-Exploitation

## Transitioning from RCE to an Interactive Shell

Successfully achieving **Remote Code Execution (RCE)** confirmed that commands could be executed on the target system. However, executing isolated commands through a web application is often inefficient for further enumeration.

The next objective was to establish a fully interactive shell, enabling easier navigation of the filesystem, privilege enumeration, and post-exploitation activities.

An interactive shell provides capabilities such as:

- Executing commands in real time.
- Browsing directories.
- Enumerating users and permissions.
- Running local enumeration utilities.
- Preparing privilege escalation techniques.

---

# Establishing a Reverse Shell

## Preparing the Listener

Before triggering the reverse shell from the target, a Netcat listener was started on the attacking machine to receive the incoming connection.

### Listener Command

```bash
nc -lnvp 1337
```

### Listener Breakdown

| Option | Purpose |
|--------|---------|
| `-l` | Listen for an incoming connection. |
| `-n` | Disable DNS resolution. |
| `-v` | Enable verbose output. |
| `-p` | Specify the listening port. |

Once the listener was ready, the reverse shell payload was executed through the previously validated command execution primitive.

---

## Receiving the Reverse Shell

After triggering the payload, the target initiated a connection back to the listener, providing an interactive shell.

<p align="center">
  <img src="../Screenshots/07_shell.png" width="900">
</p>

<p align="center"><em>Figure 8 — Interactive reverse shell successfully established from the target machine.</em></p>

### Verification Steps

After obtaining shell access, several basic verification commands were executed to understand the current execution context.

Typical verification included checking:

- Current user.
- Hostname.
- Working directory.
- Operating system.
- Available shell.

### Initial Access Summary

| Check | Purpose |
|-------|---------|
| Current User | Identify compromised account. |
| Hostname | Verify target environment. |
| Working Directory | Understand execution location. |
| Shell Type | Confirm interactive shell functionality. |

---

# Local Enumeration

## Objective

With interactive shell access established, the focus shifted to **local privilege enumeration**.

The purpose of this phase was to identify security misconfigurations that could allow escalation from the compromised user to root.

### Areas Investigated

- User permissions.
- Sudo privileges.
- Environment variables.
- Installed binaries.
- Writable files and directories.
- Scheduled tasks.
- System information.

### Enumeration Strategy

A structured enumeration process reduces the likelihood of overlooking privilege escalation opportunities.

| Enumeration Area | Reason |
|------------------|--------|
| User Identity | Determine current privilege level. |
| Groups | Identify additional permissions. |
| Sudo Rights | Discover executable privileged commands. |
| Filesystem | Search for writable sensitive locations. |
| Processes | Inspect running services. |
| Environment | Identify configuration weaknesses. |

---

# Checking Sudo Permissions

One of the highest-value enumeration commands on Linux systems is:

```bash
sudo -l
```

This command lists the binaries that the current user is allowed to execute using **sudo**.

### Why This Matters

Misconfigured sudo permissions frequently create privilege escalation opportunities by allowing execution of powerful binaries without requiring a password.

---

## Sudo Enumeration Output

<p align="center">
  <img src="../Screenshots/08_sudo.png" width="900">
</p>

<p align="center"><em>Figure 9 — Passwordless sudo permissions identified during local enumeration.</em></p>

### Important Discovery

The compromised user was permitted to execute:

```text
/usr/bin/python3
```

using `sudo` **without requiring a password**.

### Security Impact

Allowing unrestricted execution of scripting interpreters with elevated privileges violates the principle of least privilege and can enable complete system compromise.

### Why Python Is Dangerous Here

Python is not merely an application—it is a scripting interpreter capable of executing arbitrary operating system commands.

This makes unrestricted privileged execution particularly dangerous.

---

# Privilege Escalation

## Identifying the Escalation Path

The discovered sudo configuration created a straightforward privilege escalation opportunity.

The attack path became:

```text
Compromised User
        │
        ▼
Passwordless sudo Permission
        │
        ▼
Privileged Python Interpreter
        │
        ▼
Root Shell
```

---

## Exploiting the Misconfiguration

The privileged Python interpreter was used to spawn a shell with elevated privileges.

> **Note:** The exact escalation command used during the TryHackMe room has intentionally been omitted from this public repository. The purpose of this documentation is to explain the privilege escalation methodology without publishing exploit commands.

### Escalation Result

The shell immediately transitioned into the **root** security context.

### Verification

Privilege escalation was confirmed by validating the effective user identity and current privileges.

---

# Root Shell Verification

After successful privilege escalation, the shell was running with **UID 0**, indicating root privileges.

This provided unrestricted access to the system within the authorized TryHackMe lab environment.

<p align="center">
  <img src="../Screenshots/09_root.png" width="900">
</p>

<p align="center"><em>Figure 10 — Root shell obtained after exploiting the sudo misconfiguration.</em></p>

### Root Access Validation

| Validation | Result |
|-----------|--------|
| Effective User | Root |
| Privilege Level | Administrative |
| Objective | Successfully completed |

---

# Capturing the Root Flag

With administrative access confirmed, the final objective of the room was completed.

### Root Flag Location

The root flag was located within the root user's directory.

> **Flag Redacted**

```text
THM{********************************}
```

The flag has been intentionally removed from this documentation to preserve the integrity of the TryHackMe challenge.

---

# Privilege Escalation Analysis

## Root Cause

The privilege escalation was possible because of a **security misconfiguration** rather than a kernel vulnerability.

### Misconfiguration Summary

| Misconfiguration | Impact |
|------------------|--------|
| Passwordless sudo | Elevated execution without authentication. |
| Python Interpreter Allowed | Arbitrary command execution with elevated privileges. |
| Least Privilege Not Enforced | Complete system compromise. |

### Security Lesson

A scripting interpreter granted unrestricted sudo permissions effectively provides administrative command execution capabilities.

This demonstrates why organizations should carefully review `sudoers` configurations and avoid allowing interpreters unrestricted privileged execution.

---

# Post-Exploitation Summary

The post-exploitation phase progressed through the following stages:

| Stage | Outcome |
|-------|---------|
| Remote Code Execution | Initial foothold established. |
| Reverse Shell | Interactive Linux shell obtained. |
| Local Enumeration | User privileges inspected. |
| Sudo Enumeration | Passwordless Python execution identified. |
| Privilege Escalation | Root shell obtained successfully. |
| Final Objective | Root flag captured (redacted). |

---

## Key Takeaways from Post-Exploitation

- Interactive shells provide significantly better visibility than isolated command execution.
- Local enumeration should begin immediately after obtaining a shell.
- `sudo -l` is one of the most valuable privilege enumeration commands.
- Misconfigured sudo permissions remain a common privilege escalation vector.
- Security assessments should always validate privilege boundaries after gaining initial access.

At this point, the assessment shifted away from brute-force enumeration and toward **technology fingerprinting**, which ultimately revealed the real attack vector.

---

# MITRE ATT&CK Mapping

This room can be mapped to several MITRE ATT&CK techniques to better understand the tactics and techniques demonstrated during the engagement.

| MITRE ATT&CK Technique | Description |
|-------------------------|-------------|
| **T1046 — Network Service Discovery** | Enumerating exposed network services using Nmap. |
| **T1190 — Exploit Public-Facing Application** | Exploiting a vulnerability exposed through the web application. |
| **T1059 — Command and Scripting Interpreter** | Executing operating system commands after achieving RCE. |
| **T1059.004 — Unix Shell** | Obtaining and interacting with a Linux shell. |
| **T1548.003 — Sudo and Sudo Caching** | Abusing passwordless sudo permissions for privilege escalation. |
| **T1068 — Exploitation for Privilege Escalation** | Escalating from a compromised user account to root privileges. |

### MITRE ATT&CK Flow

```text
Reconnaissance
        │
        ▼
Network Service Discovery
        │
        ▼
Public-Facing Application Exploitation
        │
        ▼
Command Execution
        │
        ▼
Interactive Unix Shell
        │
        ▼
Privilege Escalation via sudo
        │
        ▼
Root Access
```

> These mappings are included for educational purposes and to connect the lab workflow with a commonly used adversary behavior framework.

---

# OWASP Top 10 Mapping

The techniques demonstrated during this room align with multiple OWASP Top 10 security concepts.

| OWASP Category | Relevance to the Lab |
|----------------|----------------------|
| **A01 — Broken Access Control** | Excessive privileges allowed escalation through sudo. |
| **A05 — Security Misconfiguration** | Improper sudo configuration created a privilege escalation path. |
| **A06 — Vulnerable and Outdated Components** | The web application relied on a vulnerable framework version. |

### Security Perspective

The challenge highlights how vulnerabilities can exist across different layers:

- Application Layer
- Framework Layer
- Operating System Layer
- Privilege Configuration Layer

A successful compromise required chaining weaknesses across these layers rather than exploiting a single issue.

---

# Attack Chain Summary

The complete penetration testing workflow followed throughout this assessment is summarized below.

| Phase | Objective | Outcome |
|-------|-----------|---------|
| **Reconnaissance** | Discover exposed services. | Web application identified on port 3000. |
| **Enumeration** | Search for hidden attack surface. | No useful directories or subdomains found. |
| **Technology Fingerprinting** | Identify framework. | Next.js identified. |
| **Vulnerability Assessment** | Detect known vulnerabilities. | Publicly documented framework issue identified. |
| **Manual Validation** | Confirm vulnerability. | Remote Code Execution achieved. |
| **Initial Access** | Gain shell access. | Reverse shell established. |
| **Privilege Enumeration** | Inspect permissions. | Passwordless sudo identified. |
| **Privilege Escalation** | Gain administrative access. | Root shell obtained. |
| **Objective Complete** | Capture final objective. | Root flag retrieved (redacted). |

---

# Security Recommendations

This room demonstrates several defensive practices that could prevent similar attack paths in production environments.

## Web Application Security

- Keep application frameworks and dependencies updated with security patches.
- Monitor public vulnerability disclosures affecting deployed technologies.
- Validate and sanitize server-side request processing.
- Restrict unnecessary endpoints and exposed functionality.
- Review application configurations before deployment.

## Linux System Hardening

- Apply the **Principle of Least Privilege**.
- Review `sudoers` configuration regularly.
- Avoid granting unrestricted sudo access to scripting interpreters.
- Audit privileged binaries periodically.
- Remove unnecessary administrative permissions.

## Detection & Monitoring

Security monitoring solutions should generate alerts for:

- Suspicious POST requests.
- Unexpected server-side command execution.
- Outbound reverse-shell connections.
- Privileged interpreter execution.
- Unusual sudo activity.

---

# Lessons Learned

This room provided practical experience across multiple phases of a penetration test.

## Key Technical Lessons

### 1. Technology Fingerprinting Matters

Directory fuzzing was not enough to identify the attack path. Understanding the underlying framework revealed where further research should focus.

### 2. Automated Tools Require Manual Validation

Nuclei provided a valuable lead, but Burp Suite was required to verify whether the vulnerability was genuinely exploitable.

### 3. Enumeration Never Stops

Enumeration continues after gaining initial access. Local privilege enumeration is just as important as web enumeration.

### 4. Sudo Misconfigurations Are High Risk

Passwordless sudo permissions for scripting interpreters can immediately lead to complete privilege escalation.

### 5. Documentation Is Part of Penetration Testing

Recording methodology, evidence, findings, and defensive recommendations creates a professional security assessment that is reproducible and useful for future reference.

---

# Skills Demonstrated

This challenge helped strengthen practical experience in the following areas.

| Domain | Skills Demonstrated |
|--------|----------------------|
| **Reconnaissance** | Nmap service enumeration and host discovery. |
| **Web Security** | Application fingerprinting and framework analysis. |
| **Vulnerability Assessment** | Nuclei scanning and vulnerability research. |
| **Manual Testing** | Burp Suite request interception and validation. |
| **Post-Exploitation** | Reverse shell establishment and Linux enumeration. |
| **Privilege Escalation** | Passwordless sudo exploitation methodology. |
| **Reporting** | Professional penetration testing documentation. |

---

# Blue Team Perspective

Understanding how an attack works helps defenders build better detections and mitigations.

### Defensive Improvements

- Maintain an inventory of application framework versions.
- Patch vulnerable software promptly.
- Harden Linux privilege configurations.
- Audit privileged command execution.
- Monitor unusual web server behavior.
- Restrict administrative interpreters from passwordless sudo.

### Detection Opportunities

Potential detection rules include:

- Abnormal HTTP POST requests.
- Web server spawning shell processes.
- Reverse-shell network connections.
- Passwordless sudo execution.
- Privileged Python execution.

---

# References

The following resources were useful for understanding the technologies involved in this lab.

- TryHackMe — Corp Website Room.
- Nmap Documentation.
- Burp Suite Documentation.
- Nuclei Documentation.
- Next.js Security Documentation.
- MITRE ATT&CK Framework.
- OWASP Web Security Testing Guide.

> References are included for educational purposes and do not contain challenge flags or proprietary lab solutions.

---

# Repository Structure

```text
Corp-Website-TryHackMe-Walkthrough/
│
├── README.md
├── LICENSE
├── SECURITY.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
│
├── Documentation/
│   ├── Corp_Website_Documentation.md
│   ├── Corp_Website_Documentation.docx
│   └── Tools_Used.md
│
├── Screenshots/
│   ├── 01_room.png
│   ├── 02_nmap.png
│   ├── 03_enum.png
│   ├── 04_nextjs.png
│   ├── 05_nuclei.png
│   ├── 06_burp_rce.png
│   ├── 07_shell.png
│   ├── 08_sudo.png
│   └── 09_root.png
│
├── Resources/
│   ├── notes.md
│   └── references.md
│
└── docs/
    ├── index.md
    ├── methodology.md
    └── architecture.md
```

---

# Conclusion

The **Corp Website (Romance & Co)** room demonstrates a complete web penetration testing workflow that combines reconnaissance, framework identification, vulnerability research, manual exploitation, post-exploitation enumeration, and Linux privilege escalation.

One of the biggest lessons from this challenge is that **understanding the technology stack is often more valuable than relying solely on automated enumeration tools**. Identifying the application as a **Next.js** deployment shifted the assessment toward framework-specific vulnerability research, ultimately leading to successful remote code execution.

After gaining an initial foothold, careful Linux enumeration revealed a passwordless `sudo` configuration that allowed escalation to root privileges. This reinforces an important defensive principle: **security misconfigurations can be just as impactful as software vulnerabilities.**

This walkthrough intentionally focuses on the methodology, evidence, and security concepts while **redacting all challenge flags** to preserve the educational value of the room.

---

# Author

## 👨‍💻 Anurag RVNKR

**Cybersecurity | Ethical Hacking | Penetration Testing | Web Security | TryHackMe**

This repository is part of my cybersecurity learning portfolio and documents hands-on labs completed through legal Capture The Flag environments.

### Connect

- GitHub: **anurag-rvnkr1**
- Platform: **TryHackMe**

---

# Educational Disclaimer

> This repository documents a Capture The Flag challenge completed inside an **authorized TryHackMe training environment**.
>
> All techniques discussed are intended solely for:
>
> - Cybersecurity education.
> - Ethical hacking practice.
> - Authorized penetration testing.
> - Defensive security research.
>
> Do **not** use these techniques against systems without explicit authorization.

---

<p align="center">
  <strong>Reconnaissance → Enumeration → Validation → Exploitation → Privilege Escalation → Documentation</strong>
</p>

<p align="center">
  ⭐ If you found this walkthrough useful, consider starring the repository.
</p>
