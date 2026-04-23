---
title: Snapped Phish-ing Line
module: Phishing Analysis
path: SOC Level 1
platform: TryHackMe
tags: [phishing, email-analysis, phishing-kit, virustotal, cyberchef, ctf]
status: completed
date: 2026-04-23
date_completed: 2026-04-23
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - A Series of Suspicious Emails

![snapped intro](screenshots/intro_snapped.png)

A multi-user phishing incident at SwiftSpend Financial. Multiple employees across departments received suspicious emails, some had already submitted credentials and lost account access by the time it was escalated. The task: analyze the email samples, trace the phishing infrastructure, and identify how the adversary operated.

### Key Concepts

<!-- email artifact analysis: sender address, attachment content, embedded URLs -->
<!-- phishing kit: what it is, how attackers host and reuse them -->
<!-- directory exposure: /data paths left open on attacker-controlled servers -->
<!-- sha256sum: command-line hash verification for file integrity/identification -->
<!-- VirusTotal: file hash lookup, threat categorization, archive file count -->
<!-- credential harvesting logs: what submit.php does and where it exfils data -->
<!-- CyberChef: decoding encoded flags or obfuscated strings -->

### Task Questions

**1. Which individual received the email regarding a Quote for Services Rendered?**

![snapped q1](screenshots/snapped_q1.png)

- **Answer:** William McClean

**2. What email address was used by the adversary to send the phishing emails?**

- **Answer:** Accounts.Payable@groupmarketingonline.icu

**3. What is the root domain of the redirection URL found within the attachment in the email addressed to Zoe Duncan?**

![snapped q3](screenshots/snapped_q3.png)

- **Answer:** kennaroads.buzz

**4. Which company is the login page impersonating?**

![snapped q4](screenshots/snapped_q4.png)

- **Answer:** Microsoft

**5. What is the name of the archive file found in the /data directory?**

![snapped q5](screenshots/snapped_q5.png)

- **Answer:** Update365.zip

**6. What is the SHA256 hash of the phishing kit archive?**

![snapped q6](screenshots/snapped_q6.png)

- **Answer:** ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686

**7. Aside from phishing, what other threat category is assigned to the ZIP archive on VirusTotal?**

![snapped q7](screenshots/snapped_q7.png)

- **Answer:** Trojan

**8. How many files are contained within the archive?**

![snapped q8](screenshots/snapped_q8.png)

- **Answer:** 49

**9. What is the email address of the user who submitted their credentials more than once?**

![snapped q9](screenshots/snapped_q9.png)

- **Answer:** michael.ascot@swiftspend.finance

**10. What email address is used by the adversary to collect compromised credentials?**

![snapped q10](screenshots/snapped_q10.png)

- **Answer:** m3npat@yandex.com

**11. What is the secret value decoded from flag.txt using CyberChef?**

![snapped q11](screenshots/snapped_q11.png)

- **Answer:** THM{pL4y_w1Th_tH3_URL}
