
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
<img width="1033" height="291" alt="Screenshot 2026-04-23 at 1 55 40 PM" src="https://github.com/user-attachments/assets/6bf83a93-ef31-480a-b2ca-80c8750da6e2" />

- **Answer: 09674321**

**2. What is the display name of the sender?**
- **Answer: Mr. James Jackson**

**3. Continue investigating the email headers. What is the sender's email address?**
<img width="1275" height="419" alt="Screenshot 2026-04-23 at 2 06 26 PM" src="https://github.com/user-attachments/assets/b04c23af-5209-45f2-8424-0b991f7f5915" />

- **Answer: info@mutawamarine.com**

**4. What email address will receive a reply to this email?**
<img width="1097" height="962" alt="Screenshot 2026-04-23 at 2 14 16 PM" src="https://github.com/user-attachments/assets/e60b34a7-00bb-451e-a02c-c714f0a93d08" />

- **Answer: info.mutawamarine@mail.com**

**5. Begin analyzing the message source. What is the originating IP address of this email?**
<img width="858" height="173" alt="Screenshot 2026-04-23 at 2 19 25 PM" src="https://github.com/user-attachments/assets/a9c15f9a-86dd-4e90-99e9-d2d85f7ed9ee" />

- **Answer: 192.119.71.157**

**6. Investigate the IP address from the previous question. Who is the owner of the originating IP?**
<img width="807" height="698" alt="Screenshot 2026-04-23 at 2 22 01 PM" src="https://github.com/user-attachments/assets/1ec9f642-2cb3-401a-9812-2253795bd347" />

- **Answer: Hostwinds LLC**

**7. Run an SPF record check on the Return-Path domain identified in the email headers. What is the full SPF record for this domain?**
<img width="1565" height="524" alt="Screenshot 2026-04-23 at 2 25 27 PM" src="https://github.com/user-attachments/assets/2917f23e-cd76-4402-92a6-e36edcbe1e9b" />

- **Answer: v=spf1 include:spf.protection.outlook.com -all**

**8. Perform a DMARC lookup for the Return-Path domain found in the email headers. What is the complete DMARC record for this domain?**
<img width="2307" height="552" alt="Screenshot 2026-04-23 at 2 28 11 PM" src="https://github.com/user-attachments/assets/9ffc2787-d668-4682-acb0-9d870cace3a7" />

- **Answer: v=DMARC1; p=quarantine; fo=1**

**9. What is the file name of the attachment found in the email?**
<img width="701" height="93" alt="Screenshot 2026-04-23 at 2 29 33 PM" src="https://github.com/user-attachments/assets/91b0f1ba-cb84-4366-beb2-47fa0d84e747" />

- **Answer: SWT_#09674321____PDF__.CAB
**

**10. Download the attachment to your virtual environment. Using the `sha256sum` command, what is the `SHA256` hash of the file?**
<img width="1014" height="247" alt="Screenshot 2026-04-23 at 2 32 01 PM" src="https://github.com/user-attachments/assets/da17b438-8b73-4573-a2bc-26469ca36d06" />

- **Answer: 2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f**

**11. Investigate the file hash from the previous question using VirusTotal. What is the attachment's file size in `KB` (e.g., `122.31 KB`)?**
<img width="2609" height="343" alt="Screenshot 2026-04-23 at 2 33 43 PM" src="https://github.com/user-attachments/assets/487b3704-bbf9-4c55-b5fa-717202a2facd" />

- **Answer: 400.26 KB**

**12. Continue your research on the file. What is the actual file type of the attachment?**
<img width="1248" height="749" alt="Screenshot 2026-04-23 at 3 13 53 PM" src="https://github.com/user-attachments/assets/76b3021e-d4b9-4883-8f2f-492e39ad3076" />

- **Answer: RAR**
