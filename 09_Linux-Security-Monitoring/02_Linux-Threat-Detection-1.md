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

Exposed SSH is a leading cause of Linux breaches, as it is a remote service widely used by IT teams.
  - Every internet-facing Linux machine has SSH enabled by default
  - Shodan reports over 40 million machines worldwide in 2025, with only some secured

<p align="center">
<img src=screenshots/linux_shodanreport.png width="700">
</p>

### Initial Access via SSH

Windows jas RDP and Linux SSH, both powerful remote tools that are often left unsecured.
  - MITRE has included both these protocols under their own technique. External Remote Services ID T1133
  - Tactics: Initial Access and Persistence

**Common Risks of Key-Based Authentication**
  - Attackers will steal keys and SSH credentials from GitHub repos or Ansible automation
  - Infect and Admins system with a data stealer to retrieve SSH keys

**Risks with Password-Based Authentication**
  - IT admin sets weak SSH password
  - IT sets up SSH for a contractor who sets a weak password
  - Accidental exposure of old insecure SSH server on the internet
---

**You can start from: cat /var/log/auth.log | grep "sshd"**

1. When did the ubuntu user log in via SSH for the first time? Answer Example: 2023-09-16.

<p align="center">
<img src=screenshots/linux_sshdlogin.png width="700">
</p>
In order to filter through the noise we can use the following keywords to nawrrow our log to find exactly what we need.
Command: cat /var/log/auth.log | grep "sshd" | grep "ubuntu" | grep "Accepted"

**Answer: 2024-10-22**

---

2. Did the ubuntu user use SSH keys instead of a password for the above found date? (Yea/Nay)

**Answer: Yea**

---

## Task 3 -- Detecting SSH Attacks

### SSH Breach Example

Real-World Scenario: An IT admin enables **public** SSH access to the server
  - Allows password-based authentication
  - Sets a weak password for one of the support users
      - These three actions, or lack thereof, will inevitably lead to an SSH breach, its just a matter of time
   
**Example: Log Sample of Brute Force Followed by Password Breach**
<p align="center">
<img src=screenshots/linux_sshbreach.png width="700">
</p>

### Detecting SSH Attacks

Detecting an SSH attack on Linux is more direct than Windows: No Logon Types and Event ID memorizing
  - You can just filter Linux logs by Accepted(successful) SSH logins

**Example: Three Successful SSH Logins**
ubuntu@thm-vm:~$ cat /var/log/auth.log | grep -E 'Accepted'
2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 10.14.105.255 port 18442 ssh2: [...]
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2

1. Ansible login through public key and automated account, at normal business hour of 14:00
2. JSmith logins are more interesting:
     - 2 password-based authentication logins
     - 1 login at 3am
         - Investigate who the user is?
         - IS this a normal login time for them?
         - Does this IP match the location they're supposed to be based in? Clean IP?
         - Login history for the user
         - After all those have been answered, you should determine if it requires further investigation

| Field | What to Check |
|---|---|
| Username | Who owns the user? Is the login time and IP expected? |
| Source IP | What do TI tools and asset lookups say? Trusted or malicious? |
| Login history | Was the login preceded by brute force or other suspicious events? |
| Next steps | Is the login suspicious? Should I analyze user actions following the login? |

---

1. When did the SSH password brute force start? Answer Format: 2023-09-15.

<p align="center">
<img src=screenshots/linux_sshbreach.png width="700">
</p>
Brute-Force attacks can be indicated by repeated failed login attempts in quick succession. This can be narrowed down by searching the logs for "Failed" sshd login attempts.
Command: cat /var/log/auth.log | grep "sshd" | grep "Failed"

**Answer: 2025-08-21**

---

2. Which four users did the botnet attempt to breach? Answer Format: Separate by a comma, in alphabetical order.

<p align="center">
<img src=screenshots/linux_users.png width="700">
</p>
Investigate the entire log, as the 4 different user names are spaced out throughout the log...
2025-08-21T16:34:04.201269+00:00 thm-vm sshd[18432]: Failed password for root from 197.39.195.136 port 47942 ssh2
2025-08-21T16:38:47.168117+00:00 thm-vm sshd[18479]: Failed password for invalid user sol from 193.32.162.145 port 46702 ssh2
2025-08-21T16:41:06.828985+00:00 thm-vm sshd[18495]: Failed password for invalid user roy from 80.94.95.112 port 63189 ssh2
2025-08-21T17:06:23.687748+00:00 thm-vm sshd[18544]: Failed password for invalid user user from 80.94.95.112 port 63350 ssh2

**Answer: root, roy, sol, user**

---

3. Finally, which IP managed to breach the root user?

<p align="center">
<img src=screenshots/linux_rootbreached.png width="700">
</p>
To find exactly what we're looking for in this scenario, we can narrow our search for the SSHD login to "Accepted," since we know it was successful, as well as the user "breached "root.
Command: cat /var/log/auth.log | grep "sshd" | grep "Accepted" | grep "root"

**Answer: 91.224.92.79**

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
