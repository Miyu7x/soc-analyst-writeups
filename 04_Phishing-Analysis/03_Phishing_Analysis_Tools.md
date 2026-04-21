---
title: Phishing Analysis Tools
module: Phishing Analysis
path: SOC Level 1
platform: TryHackMe
tags: [phishing, email-analysis, threat-intelligence, url-analysis, malware-sandbox, virustotal, anyrun, phishtool, sha256, credential-harvesting]
status: in-progress
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts


### Task Questions

1. I am ready to learn about phishing analysis tools!
   - **Answer: Check**

---

## Task 2 - Identifying Artifacts

### Key Concepts

**Email Header Artifacts**

![THM screenshot explaining header artifacts and body analysis](screenshots/email_header_artifacts.png)

| Artifact Category | Key Items to Collect |
|---|---|
| Header | Sender email, sender IP, subject line, recipient (To/CC/BCC), Reply-To, date/time |
| Body | URLs/hyperlinks, attachment names, attachment SHA256 hash |

### Task Questions

1. I understand the key indicators to look for when analyzing an email.
   - **Answer: Check**

---

## Task 3 - Email Header Analysis

### Key Concepts

**Message Header Analysis**
	- **Messageheader** is a Google Admin Toolbox that helps analyze email headers
	- https://mha.azurewebsites.net/ is another header analyzer tool

**IPs** can be a great tool to validate if the sender is legit
	- https://ipinfo.io/ isa great tool that gathers information on IP addresses
		- geo location
		- organization

**Phishing emails** can also contain malicious website links
	- https://urlscan.io/ allows SOCs to safely investigate a website without vising it directly

| Tool | Purpose |
|---|---|
| Messageheader | Parse full email headers; extract routing and sender IP |
| Message Header Analyzer | Alternative header parsing tool |
| IPinfo | IP geolocation and organization lookup |
| URLScan.io | Safe URL inspection via simulated browsing session |
| Talos Reputation Center | IP, domain, and network reputation and classification |

### Task Questions

1. Use Talos Reputation Center to look up `malware-test.com`. What is the web reputation assigned to this domain?
   
   ![Reputation for wbesite](screenshots/malwaretest_com.png)
   
   **Important: It says Questionable now on the website upon checking but the answer is actually Neutral...**
   - **Answer: Neutral**

---

## Task 4 - Email Body Analysis

### Key Concepts

**The raw body** of an email is where attackers hide their malicious payloads
	- malicious urls
	- https://www.convertcsv.com/url-extractor.htm is a tool used to find all embeded links in an email
	- check attachments by pulling their hash with the command **sha256sum attachment.pdf**
	- https://www.virustotal.com/gui/home/upload is a great tool to check file reputation, URLs and IP addresses
	  
	  ![](virustotal_example.png)


| Tool | Use Case |
|---|---|
| URL extraction tool | Auto-parse all links from raw email content |
| CyberChef | URL extraction and other data transformations |
| sha256sum (Linux) | Generate SHA256 hash of a file |
| Talos Reputation Center | Hash and domain reputation lookup |
| VirusTotal | Multi-vendor file, URL, IP, and domain reputation |

### Task Questions

1. What command can you use in a Linux environment to obtain the `SHA256` hash value of an attachment?
   - **Answer: sha256sum**

---

## Task 5 - Malware Sandboxes

### Key Concepts

An important tool for any SOC is a **Sandbox environment** which allows us to safely analyze files in a controlled manner
	- identifies urls 
	- are there any payloads to download
	- other IOCs
	- https://app.any.run/ is an online tool used for sandboxing
	- https://hybrid-analysis.com/ another online  sandbox tool
	- https://www.joesandbox.com/#windows used for **advanced malware analysis**


| Sandbox | Key Feature |
|---|---|
| ANY.RUN | Interactive real-time execution; hands-on environment |
| Hybrid Analysis | Free; detailed behavior and IOC reporting |
| JOESandbox | Advanced static and dynamic analysis; comprehensive reports |

### Task Questions

1. I understand the available sandbox environments for safely analyzing files and URLs.
   - **Answer: Check**

---

## Task 6 - Using PhishTool

### Key Concepts

A powerful tool for security analysts is PhishTool https://www.phishtool.com/
	- Combines threat intelligence, email metadata, automated analysis workflow
	- OSINT 

![Phishtool example](screenshots/phishtool_example.png)

![Second example of Phishtool in action](screenshotsphishtool_example2.png)

![Resolving the case with Phishtool](screenshots/phishtool_resolvescase.png)

### Task Questions

1. According to the VirusTotal analysis from above, which vendor categorized the URLs as phishing?
   - **Answer: SafeToOpen**

---

## Task 7 - Your Account Is on Hold

### Key Concepts

**Phishing Scenario: Your Account is on Hold**

![Scenario for Phishing case](screenshots/task7_phishing_scenario.png)

### Task Questions

1. What reputable brand is this email tailored to impersonate?
   
   ![Phishing email analysis](screenshots/spoofed_netflixemail.png)
   - **Answer: Netflix**

2. Based on the email headers, who is the intended recipient of this message?
   - **Answer: redacted@yahoo.com**

1. In Thunderbird mail use `View` > `Message Source`. What is the `Received: from` IP address?
   
   ![Find the IP under message source](screenshots/netflix_email_source.png)
   - **Answer: 10.197.37.234**

4. Check out the `Return-Path` field in the message source. What would you consider a domain of interest based on this field?
   - **Answer: etekno.xyz**

1. Investigate the `UPDATE ACCOUNT NOW` button. What is the shortened URL?
*Right-click on update button to copy link*
   - **Answer: https://t.co/yuxfZm8KPg?amp=1**

---

## Task 8 - Update Payment Details

We use **ANYRUN** to investigate the malicious attachment from the previous task.

### Key Concepts

**Important Self Note: On ANYRUN click Text Report, this will make viewing the malware much easier**

![Make life easier by using Text report on ANYRUN](getreport_anyrun.png)

### Task Questions

1. How does ANYRUN classify this suspected phishing email?
   
   ![Suspicious Activity on netflix email using ANYRUN](anyrun_netflixemail.png)
   - **Answer: Supicious Activity**

2. What is the name of the `PDF` attachment?
   - **Answer: Payment-updateid.pdf**

3. Investigate the email attachment. What is the SHA256 hash of the `PDF` file?
   - **Answer: CC6F1A04B10BCB168AEEC8D870B97BD7C20FC161E8310B5BCE1AF8ED420E2C24**

1. Check out the ANYRUN text report on the phishing email. Which IP address associated with the process `AcroRd32.exe` is flagged as malicious?
   
![Malicious IP for Netflix email](screenshots/netflixemail_connections.png)
   - **Answer: 2.16.107.24**

1. Continue investigating the text report. Which Windows process is classed as `Potentially Bad Traffic`?
   
   ![Malicious windows process for netflix email](screenshots/netflixemail_threat.png)
   - **Answer: svchost.exe**

---

## Task 9 - Excel Executable

### Key Concepts

![Preview of task 9 THM screennshot](screenshots/excel_task9preview.png)

**Important Self Note: On ANYRUN click Text Report, this will make viewing the malware much easier**

![Make life easier by using Text report on ANYRUN](getreport_anyrun.png)

### Task Questions

![Text report of malicious file](screenshots/task9_anyrun_secondexample.png)

1. How does ANYRUN classify the `.xlsx` attachment?
   - **Answer: Malicious Activity**

2. What is the file name of the Excel attachment?
   - **Answer: CBJ200620039539.xlsx**

3. Investigate the Excel attachment. What is the `SHA256` hash value?
   - **Answer: 5F94A66E0CE78D17AFC2DD27FC17B44B3FFC13AC5F42D3AD6A5DCFB36715F3EB**

1. Check out the ANYRUN text report. What IP address is associated with the malicious domain `biz9holdings.com`?
   
   ![Network activity text report](screenshots/anyrun_biz9holdings.png)
   - **Answer: findresults.site**

5. Which other domain is classified as malicious?
   - **Answer: findresults.site**

1. What vulnerability does this malicious attachment attempt to exploit?
   ![The answer was lloking for the CVE number not a name!](phshing_emailcve.png)
   - **Answer:  CVE-2017-11882**

---

## Task 10 - Conclusion

### Key Concepts

This lab was great in the sense that i was able to use various different tools to analyze malicious, file or URL...Its nice that these sandboxes are all available online meaning you never have to open anything in your OS without checking first!

### Task Questions

1. Complete the room and continue on your cyber learning journey!
   - **Answer: Check**