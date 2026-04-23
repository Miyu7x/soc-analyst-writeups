---
title: Snapped Phish-ing Line
module: Phishing Analysis
path: SOC Level 1
platform: TryHackMe
tags: [phishing, email-analysis, phishing-kit, virustotal, cyberchef, ctf]
status: in-progress
date: 2026-04-23
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - A Series of Suspicious Emails

<!-- scenario: SwiftSpend Financial IT dept, multi-user phishing incident, some credentials already compromised -->

### Key Concepts

<!-- email artifact analysis: sender address, attachment content, embedded URLs -->
<!-- phishing kit: what it is, how attackers host and reuse them -->
<!-- directory exposure: /data paths left open on attacker-controlled servers -->
<!-- sha256sum: command-line hash verification for file integrity/identification -->
<!-- VirusTotal: file hash lookup, threat categorization, archive file count -->
<!-- credential harvesting logs: what submit.php does and where it exfils data -->
<!-- CyberChef: decoding encoded flags or obfuscated strings -->

### Questions

**1. Which individual received the email regarding a Quote for Services Rendered?**

- **Answer:**

**2. What email address was used by the adversary to send the phishing emails?**

- **Answer:**

**3. What is the root domain of the redirection URL found within the attachment in the email addressed to Zoe Duncan?**

- **Answer:**

**4. Which company is the login page impersonating?**

- **Answer:**

**5. What is the name of the archive file found in the /data directory?**

- **Answer:**

**6. What is the SHA256 hash of the phishing kit archive?**

- **Answer:**

**7. Aside from phishing, what other threat category is assigned to the ZIP archive on VirusTotal?**

- **Answer:**

**8. How many files are contained within the archive?**

- **Answer:**

**9. What is the email address of the user who submitted their credentials more than once?**

- **Answer:**

**10. What email address is used by the adversary to collect compromised credentials?**

- **Answer:**

**11. What is the secret value decoded from flag.txt using CyberChef?**

- **Answer:**