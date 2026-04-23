---
title: Phishing Analysis - Challenge
module: Phishing Analysis
path: SOC Level 1
platform: TryHackMe
tags:
  - phishing
  - email-analysis
  - SMTP
  - SPF
  - DMARC
  - VirusTotal
  - blue-team
  - challenge
status: completed
date: 2026-04-23
date_completed: 2026-04-23
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - A Message That Doesn't Add Up

### Key Concepts

**Analyzing an Email**
	- Identify and extract key artifacts
	- Investigate the message for source origin and authenticity
	- Use tools to analyze email

### Task Questions

**1. What is the `Transfer Reference Number` listed in the email's Subject line?**
- **Answer:**

**2. What is the display name of the sender?**
- **Answer:**

**3. Continue investigating the email headers. What is the sender's email address?**
- **Answer:**

**4. What email address will receive a reply to this email?**
- **Answer:**

**5. Begin analyzing the message source. What is the originating IP address of this email?**
- **Answer:**

**6. Investigate the IP address from the previous question. Who is the owner of the originating IP?**
- **Answer:**

**7. Run an SPF record check on the Return-Path domain identified in the email headers. What is the full SPF record for this domain?**
- **Answer:**

**8. Perform a DMARC lookup for the Return-Path domain found in the email headers. What is the complete DMARC record for this domain?**
- **Answer:**

**9. What is the file name of the attachment found in the email?**
- **Answer:**

**10. Download the attachment to your virtual environment. Using the `sha256sum` command, what is the `SHA256` hash of the file?**
- **Answer:**

**11. Investigate the file hash from the previous question using VirusTotal. What is the attachment's file size in `KB` (e.g., `122.31 KB`)?**
- **Answer:**

**12. Continue your research on the file. What is the actual file type of the attachment?**
- **Answer:**