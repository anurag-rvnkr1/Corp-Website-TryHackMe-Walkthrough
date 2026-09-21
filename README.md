# Corp Website — TryHackMe Walkthrough

<p align="center">
  <img src="https://img.shields.io/badge/TryHackMe-Corp%20Website-212C42?style=for-the-badge&logo=tryhackme&logoColor=white" alt="TryHackMe">
  <img src="https://img.shields.io/badge/Category-Web%20Security-0A66C2?style=for-the-badge" alt="Web Security">
  <img src="https://img.shields.io/badge/Difficulty-Medium-F39C12?style=for-the-badge" alt="Medium Difficulty">
  <img src="https://img.shields.io/badge/OS-Linux-333333?style=for-the-badge&logo=linux&logoColor=white" alt="Linux">
  <img src="https://img.shields.io/badge/Focus-Next.js%20%7C%20RCE%20%7C%20Privesc-6A0DAD?style=for-the-badge" alt="Focus">
</p>

<p align="center">
  <strong>Professional documentation of a TryHackMe web exploitation lab covering reconnaissance, technology fingerprinting, vulnerability discovery, remote code execution, shell access, and Linux privilege escalation.</strong>
</p>

---

## 📌 Overview

**Corp Website (Romance & Co)** is a medium-difficulty TryHackMe web security challenge centered around identifying weaknesses in a web application, obtaining an initial foothold, and escalating privileges on the underlying Linux host.

This repository contains my **original technical documentation and methodology** for the completed lab.

The assessment demonstrates an important penetration-testing workflow:

```text
Reconnaissance
      ↓
Service Enumeration
      ↓
Web Application Analysis
      ↓
Technology Fingerprinting
      ↓
Vulnerability Discovery
      ↓
Manual Validation
      ↓
Remote Code Execution
      ↓
Interactive Shell
      ↓
Privilege Enumeration
      ↓
Privilege Escalation
      ↓
Root Access
```

> 🔐 **Flags are intentionally redacted throughout this repository.**
>
> The objective is to document the methodology, reasoning, tooling, and security concepts without publishing challenge flags.

---

## 🎯 Lab Information

| Attribute                  | Details                  |
| -------------------------- | ------------------------ |
| **Platform**               | TryHackMe                |
| **Room**                   | Corp Website             |
| **Challenge Theme**        | Web Application Security |
| **Difficulty**             | Medium                   |
| **Target Environment**     | Linux                    |
| **Primary Web Technology** | Next.js                  |
| **Initial Attack Surface** | Web application          |
| **Initial Access**         | Remote Code Execution    |
| **Post-Exploitation**      | Reverse Shell            |
| **Privilege Escalation**   | Sudo misconfiguration    |
| **Final Objective**        | Root access              |
| **Status**                 | ✅ Completed              |

---

## 🧠 Objectives

The main learning objectives of this room were:

* Perform systematic network and service reconnaissance.
* Identify the exposed web application and its technology stack.
* Understand why framework fingerprinting matters during web assessments.
* Use automated vulnerability detection to identify potential attack paths.
* Manually validate a suspected vulnerability using an intercepting proxy.
* Transition from command execution to an interactive shell.
* Enumerate Linux privilege boundaries after initial compromise.
* Identify and exploit an insecure `sudo` configuration.
* Document the attack chain in a professional, reproducible format.

---

## 🔎 Assessment Methodology

The engagement followed a structured penetration-testing workflow.

### 1. Reconnaissance

The target was first assessed for exposed TCP services and application technologies.

Primary tooling:

* `Nmap`

Example workflow:

```bash
nmap -sS -sV -A -v <TARGET>
```

The objective was to establish an initial understanding of the host, identify accessible services, and determine where deeper enumeration should begin.

---

### 2. Web Enumeration

After identifying the web application, application-layer enumeration was performed.

Tools considered during the assessment included:

* Gobuster
* Amass
* Subfinder

The enumeration phase was useful for determining whether hidden paths, subdomains, or additional application attack surfaces were exposed.

Traditional content discovery did not immediately reveal the main exploitation path, which made **technology identification** particularly important.

---

### 3. Technology Fingerprinting

The web application was identified as being built with **Next.js**.

This changed the direction of the assessment from generic web enumeration toward **framework-specific vulnerability research**.

The key lesson here is that identifying the underlying technology can significantly reduce the search space during security testing.

---

### 4. Vulnerability Discovery

Automated vulnerability identification was performed using **Nuclei**.

Example:

```bash
nuclei -u http://<TARGET>:3000
```

The scan identified a potential security issue associated with the application's framework.

Automated scanner output was treated as a **lead rather than final proof**. The finding was subsequently examined and validated manually.

---

### 5. Manual Validation

**Burp Suite** was used to inspect and manipulate HTTP traffic.

The validation process involved:

1. Intercepting an application request.
2. Reviewing the request structure.
3. Changing the request method where required.
4. Supplying a crafted request body.
5. Replaying the request.
6. Observing the resulting server-side behavior.

The testing ultimately confirmed **remote command execution** against the lab target.

---

## 💻 Initial Access

Once remote command execution had been validated, the next objective was to obtain a more practical shell interface.

A listener was prepared on the attacking machine:

```bash
nc -lnvp 1337
```

The validated command-execution primitive was then used to establish a reverse shell.

This provided an interactive foothold for post-exploitation enumeration.

---

## 🐚 Post-Exploitation

With shell access established, local enumeration focused on understanding:

* Current user context
* Available privileges
* Sudo configuration
* Accessible executables
* Potential privilege boundaries

One of the most important checks performed was:

```bash
sudo -l
```

The resulting configuration showed that the compromised user could execute **Python 3 with elevated privileges without requiring a password**.

---

## ⬆️ Privilege Escalation

The `sudo` configuration represented a significant privilege boundary failure.

Because a scripting interpreter was permitted with elevated privileges, it could be used to spawn a shell with the same elevated execution context.

The resulting escalation path was:

```text
Compromised User
      ↓
sudo -l
      ↓
Passwordless Python Execution
      ↓
Privileged Command Execution
      ↓
Root Shell
```

Root access was successfully obtained within the authorized TryHackMe environment.

---

## 🚩 Flags

### Initial Flag

```text
THM{REDACTED}
```

### Root Flag

```text
THM{REDACTED}
```

> Flags are deliberately removed from this repository to avoid unnecessary spoilers and preserve the challenge experience.

---

## 🖼️ Evidence & Screenshots

Screenshots documenting the assessment are stored under:

```text
Screenshots/
```

Recommended evidence sequence:

|  # | Screenshot        | Purpose                           |
| -: | ----------------- | --------------------------------- |
| 01 | `01_room.png`     | Room / target context             |
| 02 | `02_nmap.png`     | Network and service enumeration   |
| 03 | `03_enum.png`     | Directory / subdomain enumeration |
| 04 | `04_nextjs.png`   | Technology fingerprinting         |
| 05 | `05_nuclei.png`   | Vulnerability identification      |
| 06 | `06_burp_rce.png` | Manual HTTP exploitation          |
| 07 | `07_shell.png`    | Reverse shell / initial access    |
| 08 | `08_sudo.png`     | Privilege enumeration             |
| 09 | `09_root.png`     | Root access confirmation          |

> Screenshots are included as technical evidence and referenced throughout the detailed documentation.

---

## 🛠️ Tools & Technologies

| Tool / Technology | Purpose                                 |
| ----------------- | --------------------------------------- |
| **Nmap**          | Port and service enumeration            |
| **Gobuster**      | Web content discovery                   |
| **Amass**         | Subdomain discovery                     |
| **Subfinder**     | Passive subdomain enumeration           |
| **Nuclei**        | Vulnerability identification            |
| **Burp Suite**    | HTTP interception and manual validation |
| **Netcat**        | Reverse-shell listener                  |
| **Linux CLI**     | Post-exploitation enumeration           |
| **sudo**          | Privilege-boundary analysis             |
| **Python 3**      | Privileged command execution analysis   |
| **Next.js**       | Application framework under assessment  |

---

## 📚 Repository Structure

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

## 🗺️ Attack Path

```text
                    ┌──────────────────────┐
                    │      Target Host     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Nmap Enumeration   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  Web App :3000       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Next.js Fingerprint  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Nuclei Detection   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Burp Suite Validation│
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Remote Code Execution│
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Reverse Shell     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      sudo -l         │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Privileged Python 3  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      Root Shell      │
                    └──────────────────────┘
```

---

## 🛡️ Security Lessons

This lab reinforced several practical security principles.

### Framework Awareness

A web application should not be assessed purely from its visible pages. Identifying its framework and supporting technologies can expose attack paths that generic enumeration may miss.

### Automated Scanning Requires Validation

Tools such as Nuclei are highly useful for identifying candidate vulnerabilities, but scanner findings should be manually verified before being treated as confirmed vulnerabilities.

### Post-Exploitation Enumeration Matters

Obtaining an initial shell is not necessarily the end of an assessment. Local privilege enumeration can reveal configuration weaknesses that enable escalation.

### Least Privilege Is Critical

Allowing powerful interpreters or scripting environments to execute through `sudo` can undermine the entire privilege model if the configuration is not carefully restricted.

---

## 🛡️ Defensive Recommendations

### Web Application

* Keep the application framework and dependencies patched.
* Maintain an inventory of production dependencies and versions.
* Monitor suspicious HTTP methods and malformed request bodies.
* Restrict unnecessary application functionality exposed to the network.
* Use layered application and host-based monitoring.

### Linux Host

* Apply the principle of least privilege to `sudo`.
* Avoid granting unrestricted elevated access to scripting interpreters.
* Regularly audit `/etc/sudoers` and included configuration files.
* Monitor privileged command execution.
* Review local accounts and executable permissions periodically.

### Detection & Monitoring

Security teams should consider alerting on:

* Suspicious POST requests to application endpoints.
* Unexpected child-process creation by web services.
* Reverse-shell behavior.
* Unusual use of interpreters by privileged users.
* Passwordless execution of administrative binaries.

---

## 🧩 MITRE ATT&CK Alignment

The attack chain can be conceptually mapped to the following ATT&CK techniques:

| Technique                                         | Relevance                                                       |
| ------------------------------------------------- | --------------------------------------------------------------- |
| **T1046 — Network Service Scanning**              | Discovery of exposed network services                           |
| **T1190 — Exploit Public-Facing Application**     | Initial compromise through the exposed web application          |
| **T1059 — Command and Scripting Interpreter**     | Command execution after obtaining code execution                |
| **T1059.004 — Unix Shell**                        | Interactive Linux shell activity                                |
| **T1548.003 — Sudo and Sudo Caching**             | Abuse of elevated sudo execution                                |
| **T1068 — Exploitation for Privilege Escalation** | Transition from compromised user context to elevated privileges |

> ATT&CK mappings are provided as an analytical classification of the lab workflow rather than a claim about real-world threat-actor attribution.

---

## 🌐 OWASP Relevance

The exercise also demonstrates concepts related to several OWASP areas:

| OWASP Area                                   | Relevance                                           |
| -------------------------------------------- | --------------------------------------------------- |
| **A05 — Security Misconfiguration**          | Insecure privilege configuration on the host        |
| **A06 — Vulnerable and Outdated Components** | Framework-related vulnerability exposure            |
| **A01 — Broken Access Control**              | Security boundaries weakened by excessive privilege |

The exact OWASP classification can vary depending on how a real-world implementation and root cause are defined.

---

## 📖 Detailed Documentation

For the full technical walkthrough, evidence, commands, findings, and analysis, see:

**[`Documentation/Corp_Website_Documentation.md`](Documentation/Corp_Website_Documentation.md)**

Additional supporting material:

* **[`Documentation/Tools_Used.md`](Documentation/Tools_Used.md)**
* **[`Resources/notes.md`](Resources/notes.md)**
* **[`Resources/references.md`](Resources/references.md)**
* **[`docs/index.md`](docs/index.md)**

---

## ⚠️ Disclaimer

This repository documents activity performed within an **authorized TryHackMe training environment**.

The techniques described here are intended for:

* Cybersecurity education
* Capture The Flag practice
* Authorized penetration testing
* Security research in controlled environments
* Defensive security learning

Do **not** use these techniques against systems you do not own or do not have explicit permission to test.

---

## ✍️ Author

**Anurag Revankar**

Cybersecurity | Ethical Hacking | Web Security | Security Research

GitHub: [@anurag-rvnkr1](https://github.com/anurag-rvnkr1)

---

## ⭐ Acknowledgement

This repository is based on the **Corp Website** challenge hosted by **TryHackMe**.

The purpose of this repository is to document the technical learning process and create a structured cybersecurity portfolio artifact while keeping challenge flags redacted.

---

<p align="center">
  <strong>Recon → Enumerate → Validate → Exploit → Escalate → Document</strong>
</p>
