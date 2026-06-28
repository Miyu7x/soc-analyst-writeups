---
title: Linux Threat Detection 1
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, threat-detection, ssh, initial-access, auditd, process-tree, mitre]
status: in-progress
date: 2026-06-28
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/linux_introthreat.png width="1500">
</p>

---

## Task 1 -- Introduction

### Learning Objectives

- Understand the role and risk of SSH in Linux environments
- Learn how Internet-exposed services can lead to breaches
- Utilize process tree analysis to identify the origin of the attack
- Practice detecting Initial Access techniques in realistic labs

---

1. I'm ready to start!

**Answer:** N/A

---

## Task 2 -- Initial Access via SSH

### Popularity of SSH

Exposed SSH is the lead cause of Linux breaches, as it is a remote service that is popularly used by IT teams.
  - Every internet-facing Linux machine has SSH enabled by default
  - Shodan reports over 40 million machiones worldwide in 2025, with only some secured

### Initial Access via SSH

---

1. When did the ubuntu user log in via SSH for the first time? Answer Example: 2023-09-16.

**Answer:**

---

2. Did the ubuntu user use SSH keys instead of a password for the above found date? (Yea/Nay)

**Answer:**

---

## Task 3 -- Detecting SSH Attacks

### SSH Breach Example

### Detecting SSH Attacks

| Field | What to Check |
|---|---|
| Username | Who owns the user? Is the login time and IP expected? |
| Source IP | What do TI tools and asset lookups say? Trusted or malicious? |
| Login history | Was the login preceded by brute force or other suspicious events? |
| Next steps | Is the login suspicious? Should I analyze user actions following the login? |

---

1. When did the SSH password brute force start? Answer Format: 2023-09-15.

**Answer:**

---

2. Which four users did the botnet attempt to breach? Answer Format: Separate by a comma, in alphabetical order.

**Answer:**

---

3. Finally, which IP managed to breach the root user?

**Answer:**

---

## Task 4 -- Initial Access via Services

### Linux and Public Services

### Using Application Logs

### Web as Initial Access

---

1. What is the path to the Python file the attacker attempted to open?

**Answer:**

---

2. Looking inside the opened file, what's the flag you see there?

**Answer:**

---

## Task 5 -- Detecting Service Breach

### Building Process Tree

### Auditd and Process Tree

---

1. What is the PPID of the suspicious whoami command?

**Answer:**

---

2. Moving up the tree, what is the PID of the TryPingMe app?

**Answer:**

---

3. Which program did the attacker use to open a reverse shell?

**Answer:**

---

## Task 6 -- Advanced Initial Access

### Human-Led Attacks

| Scenario Example | Consequences |
|---|---|
| IT member runs unverified script from forum via curl pipe bash | Script was malware, silently infected the server |
| Developer mistypes a Python package name (e.g., fastpi vs fastapi) | Mistyped package was deliberately prepared malware (typosquatting) |

### Supply Chain Compromise

### Detecting the Attacks

---

1. Which Initial Access technique is likely used if a trusted app suddenly runs malicious commands?

**Answer:**

---

2. Which detection method can you use to detect a variety of Initial Access techniques?

**Answer:**

---

## Task 7 -- Conclusion

### Key Takeaways

- Attacks on SSH are widespread, but they are easy to detect via authentication logs
- Exposed services are always a risk since they can lead to a whole Linux compromise
- While phishing is not common on Linux, human-led and supply attacks are still possible
- Process tree analysis is your best approach in identifying Initial Access techniques

---

1. Let's continue!

**Answer:** N/A
