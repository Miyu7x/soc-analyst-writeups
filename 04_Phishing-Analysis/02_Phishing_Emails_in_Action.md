---
title: Phishing Emails in Action
module: Phishing Analysis
path: SOC Level 1
platform: TryHackMe
tags: [phishing, email-analysis, social-engineering, credential-harvesting, url-obfuscation, tracking-pixels, attachment-analysis]
status: in-progress
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

Attackers use various techniques when it comes to crafting a **phishing email**
	- spoofed email address
	- url shortnening
	- branded html
	- attention grabbing subject
	  
**Raw** view of the email gives SOC a deep look into the contents of the email
	- attackers hide malicious links inside html buttons "click here"
	  -url shortening obfuscates the actual destination
	- www.wheregoes.com is a URL redirect checker tool

### Task Questions

![Email body example](screenshots/email_body_analysis.png)

1. I am ready to analyze phishing emails!
   - **Answer: Amazing Stuff**

---

## Task 2 - Cancel Your Order

### Key Concepts

<!-- Phishing technique: spoofed email address - how display name differs from actual sender domain -->
<!-- Phishing technique: URL shortening services - why shortened URLs are dangerous and how they obfuscate destination -->
<!-- Phishing technique: branded HTML - use of corporate imagery to build false credibility -->
<!-- Red flag checklist for this sample: subject line urgency, From address mismatch, unusual To address -->
<!-- Tool introduced: WhereGoes - for investigating shortened URLs without visiting the destination -->

| Technique | Description |
|---|---|
| Spoofed email address | Display name mimics trusted service; actual sender domain is unrelated |
| URL shortening | Redirection service hides true link destination |
| Branded HTML | Corporate imagery used to impersonate legitimate service |

### Task Questions

1. Who is listed as the Merchant in the email body?
   - **Answer: Amazing Stuff**

---

## Task 3 - Track Your Package

### Key Concepts


| Technique | Description |
|---|---|
| Spoofed email address | Display name does not match actual sender domain |
| Pixel tracking | Invisible images embedded to detect when email is opened |
| Link manipulation | Fraudulent tracking number masks malicious URL destination |

![Example of raw email](screenshots/task3_email_example.png)

### Task Questions

1. What root domain does the hyperlink in the above example point to? Be sure to defang the URL.
   - **Answer: devret[.]xyz**

---

## Task 4 - Download Document Here

### Key Concepts

**Phishing Campaign**

![Phishing Campaign explained](screenshots/phishing_campaign.png)


| Redirection Stage | Impersonated Service |
|---|---|
| Stage 1 | OneDrive share landing page |
| Stage 2 | Adobe document portal |
| Stage 3 | Credential harvesting login portal |

### Task Questions

1. The attacker deployed a fake portal to capture and exfiltrate user credentials. What is this type of attack called?
   - **Answer: Credential Harvesting**

---

## Task 5 - Your Account Is on Hold

### Key Concepts

<!-- Phishing technique: spoofed email address - display name "Netllx billing" vs actual sender -->
<!-- Phishing technique: sense of urgency - suspended account notification pressures immediate action -->
<!-- Phishing technique: brand impersonation - HTML templates and Netflix logos -->
<!-- Phishing technique: poor grammar and typos - "Netllx" misspelling as a red flag -->
<!-- Phishing technique: attachment-based delivery - PDF used to hide malicious URL from email filters -->
<!-- Red flag: atypical phone number format -->
<!-- Red flag: use of legitimate Netflix help center domain to build false trust -->

| Technique | Description |
|---|---|
| Spoofed email address | Display name set to "Netllx billing"; does not match actual sender |
| Urgency | Suspended account notification demands immediate action |
| Brand impersonation | HTML and logos mimic Netflix billing communications |
| Attachment | PDF contains embedded link to non-Netflix domain |

### Task Questions

1. What is the actual sender email address hidden behind the `Netllx billing` display name?
   
   ![Spoofed Netflix email](netflix_phishingemail.png)
   - **Answer: z99@musacombi.online**

---

## Task 6 - Your Recent Purchase

### Key Concepts

![Phishing email observation explained](screenshots/task6_phishing_observations.png)

| Technique | Description |
|---|---|
| Spoofed email address | Display name "Apple Support" does not match actual sender domain |
| BCC | Victim is blind carbon copied; not a direct recipient |
| Attachment | .dot file (Word Template) contains redirect to phishing site |
| Blank body | No body text; all content is inside the attachment |


### Task Questions


1. What does the acronym BCC stand for?
   - **Answer: Blind Carbon Copy**

2. What is the file extension of the attachment?
   - **Answer: .dot**

---

## Task 7 - Scheduled Shipment

### Key Concepts

![Screenshot from thm](screenshots/task7email_attachment.png)

| Post-Execution Risk | Description |
|---|---|
| Persistence | Backdoor or scheduled task survives reboot |
| Data exfiltration | Credentials, files, and browser-stored passwords stolen |
| Ransomware | System encrypted; payment demanded for recovery |

### Task Questions

1. What is the name of the executable that the Excel attachment attempts to run?
   
   ![regasms.exe](screenshots/task7email_q1.png)
   - **Answer:**

---

## Task 8 - Conclusion

### Key Concepts

Recap of phishing and learning to pay attention to detail to the header, misspelling was always a dead giveaway but now with AI phishing emaisl have gotten a lot more legit looking!

### Task Questions

1. Complete the room and continue on your cyber learning journey!
   - **Answer: check**