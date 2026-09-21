# 📝 Corp Website — TryHackMe Study Notes

> Personal learning notes from completing the **Corp Website (Romance & Co)** room on **TryHackMe**.
>
> These notes summarize the reconnaissance process, web exploitation methodology, privilege escalation path, defensive lessons, and key cybersecurity concepts learned during the lab.
>
> **Room Difficulty:** Medium  
> **Category:** Web Security • Linux Privilege Escalation

---

# 📚 Learning Summary

This room focused on identifying vulnerabilities in a web application built with **Next.js**, validating a publicly disclosed framework vulnerability, gaining initial access through remote code execution, establishing a reverse shell, and escalating privileges on a Linux host.

Rather than relying only on directory fuzzing, this challenge emphasized the importance of **technology fingerprinting** and **framework-specific vulnerability research** during penetration testing.

---

# 🎯 Skills Practiced

| Skill | Description |
|--------|-------------|
| Network Reconnaissance | Enumerating exposed services using Nmap. |
| Web Enumeration | Searching for hidden directories and subdomains. |
| Technology Fingerprinting | Identifying the application framework and attack surface. |
| Vulnerability Assessment | Detecting publicly known vulnerabilities with Nuclei. |
| Manual Validation | Verifying scanner findings using Burp Suite. |
| Remote Code Execution | Achieving command execution through a vulnerable endpoint. |
| Reverse Shell | Establishing an interactive Linux shell. |
| Linux Enumeration | Inspecting users, permissions, and sudo configuration. |
| Privilege Escalation | Exploiting insecure sudo permissions to gain root access. |
| Technical Documentation | Writing a professional penetration testing walkthrough. |

---

# 🛠️ Tools Used During the Lab

| Tool | Purpose |
|------|---------|
| **Nmap** | Host discovery, service detection, and OS fingerprinting. |
| **Gobuster** | Directory enumeration. |
| **Amass** | Active subdomain enumeration. |
| **Subfinder** | Passive subdomain discovery. |
| **Nuclei** | Vulnerability scanning against known templates. |
| **Burp Suite** | Intercepting, modifying, and replaying HTTP requests. |
| **Netcat** | Reverse shell listener. |
| **Linux CLI Utilities** | Local privilege enumeration and system exploration. |
| **Python 3** | Elevated command execution through sudo permissions. |

---

# 🔍 Enumeration Notes

## Network Enumeration

**Objective**

- Discover exposed services.
- Identify application entry points.
- Determine operating system and running technologies.

**Key Observations**

- Web service exposed on **TCP port 3000**.
- Service responded with a modern JavaScript web application.
- Enumeration provided the starting point for web analysis.

**Lesson Learned**

Always begin an engagement with service discovery before interacting with an application.

---

## Web Enumeration

### Directories

Attempted directory discovery using common wordlists.

**Result**

No valuable hidden directories were identified.

### Subdomains

Performed active and passive subdomain enumeration.

**Result**

No useful subdomains were discovered.

### Lesson Learned

Failure to discover directories does **not** mean the application is secure. Enumeration should continue through framework identification and application analysis.

---

# 🌐 Technology Fingerprinting Notes

The application was identified as being built using **Next.js**.

### Indicators

- Static asset naming convention.
- Framework-specific routing.
- JavaScript bundle structure.

### Why This Matters

Identifying the underlying framework narrows the attack surface and allows security testing to focus on known framework behaviors and vulnerabilities.

**Takeaway**

> Fingerprinting technologies is often more valuable than continuing blind fuzzing.

---

# 🚨 Vulnerability Assessment Notes

### Scanner Used

- Nuclei

### Purpose

- Detect known vulnerabilities affecting exposed technologies.
- Identify publicly documented weaknesses.

### Workflow

1. Scan target.
2. Review findings.
3. Research vulnerability.
4. Validate manually.

### Lesson Learned

Automated scanners produce **potential findings** that must always be manually validated.

---

# 🎯 Exploitation Notes

### Goal

Obtain remote command execution through the vulnerable application.

### Validation Process

- Intercept traffic.
- Modify request.
- Replay request.
- Observe server behavior.

### Result

The application executed attacker-controlled commands, providing an initial foothold.

### Important Concept

Manual request manipulation is essential for confirming vulnerability impact.

---

# 🐚 Reverse Shell Notes

### Objective

Convert command execution into an interactive shell.

### Steps

1. Prepare listener.
2. Trigger payload.
3. Receive callback.
4. Verify shell access.

### Why Reverse Shells Matter

Interactive shells allow:

- File exploration.
- Privilege enumeration.
- Process inspection.
- Local exploitation.

---

# 🧑‍💻 Linux Enumeration Notes

After gaining shell access, local enumeration focused on:

- Current user.
- Host information.
- Installed binaries.
- Environment variables.
- Sudo configuration.

### Most Valuable Enumeration Command

```bash
sudo -l
```

### Why It Is Important

`sudo -l` reveals commands executable with elevated privileges and often identifies privilege escalation opportunities.

---

# ⬆️ Privilege Escalation Notes

### Finding

The compromised user could execute **Python 3** using `sudo` without requiring a password.

### Security Impact

Allowing unrestricted execution of scripting interpreters can lead to complete privilege escalation.

### Lesson Learned

Always inspect:

- sudo permissions
- SUID binaries
- capabilities
- writable cron jobs
- scheduled tasks
- PATH configuration

---

# 🏁 Root Access Notes

### Objective

Verify privileged access and retrieve the final objective.

### Validation

- Confirm effective user is root.
- Inspect root-owned resources.
- Complete the room objective.

> **Challenge flags are intentionally omitted from these notes.**

---

# 🧠 Cybersecurity Concepts Learned

## Web Security

- Technology fingerprinting.
- Framework-specific attack surfaces.
- HTTP request manipulation.
- Known vulnerability validation.
- Remote code execution concepts.

## Linux Security

- User privilege model.
- Passwordless sudo risks.
- Command interpreter abuse.
- Local privilege escalation methodology.

## Penetration Testing

- Reconnaissance before exploitation.
- Validate scanner findings manually.
- Enumerate after every successful foothold.
- Document evidence during each phase.

---

# 🛡️ Defensive Security Notes

## Web Application Hardening

- Keep framework dependencies updated.
- Monitor unusual POST requests.
- Restrict unnecessary API routes.
- Apply security updates promptly.

## Linux Hardening

- Review sudoers configuration regularly.
- Apply least privilege principles.
- Avoid unrestricted interpreters in sudo.
- Enable auditing of privileged commands.

## Detection Opportunities

Security teams should monitor:

- Unexpected web server child processes.
- Interactive shells spawned from web services.
- Privileged interpreter execution.
- Abnormal outbound connections.
- Unauthorized command execution patterns.

---

# 📖 MITRE ATT&CK Concepts Practiced

| ATT&CK Technique | Concept Learned |
|------------------|-----------------|
| Network Service Scanning | Reconnaissance against exposed services. |
| Exploit Public-Facing Application | Initial compromise through the web application. |
| Command and Scripting Interpreter | Executing commands after gaining RCE. |
| Unix Shell | Interactive shell activity on Linux. |
| Sudo Abuse | Privilege escalation through sudo permissions. |
| Exploitation for Privilege Escalation | Escalating from user privileges to root. |

> These mappings are provided for learning purposes within the lab context.

---

# 🧩 OWASP Concepts Practiced

| OWASP Area | Relevance |
|------------|-----------|
| Security Misconfiguration | Insecure sudo configuration. |
| Vulnerable Components | Framework vulnerability research. |
| Broken Access Control | Improper privilege boundaries. |

---

# 💡 Key Takeaways

- Enumeration should include **technology fingerprinting**, not just directory fuzzing.
- Framework identification can reveal attack vectors that traditional reconnaissance misses.
- Automated vulnerability scanners accelerate discovery but require manual verification.
- Reverse shells provide a practical environment for post-exploitation enumeration.
- Misconfigured sudo permissions remain one of the most common Linux privilege escalation vectors.
- Careful documentation is an essential part of every penetration test and CTF walkthrough.

---

# 📌 Personal Takeaway

This room reinforced the importance of combining **reconnaissance, framework analysis, manual validation, and privilege enumeration** into a structured penetration-testing workflow instead of relying on a single tool or technique.

The biggest lesson from this lab was:

> **Understand the technology first, validate findings manually, enumerate thoroughly, and document every step professionally.**

---

**Author:** Anurag Revankar

*Cybersecurity Portfolio • TryHackMe Walkthrough Notes*
