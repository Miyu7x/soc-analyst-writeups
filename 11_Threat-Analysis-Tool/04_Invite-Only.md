---
title: Invite Only
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [threat-intel, osint, malware-analysis, ioc-investigation]
status: in-progress
date: 2026-07-13
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/invite_intro.png width="1500">
</p>

---

## Task 1: Investigating Flagged Indicators

#### Scenario

**Reference**

You are an SOC analyst on the SOC team at Managed Server Provider TrySecureMe. Today, you are supporting an L3 analyst in investigating flagged IPs, hashes, URLs, or domains as part of IR activities. One of the L1 analysts flagged two suspicious findings early in the morning and escalated them. Your task is to analyse these findings further and distil the information into usable threat intelligence.

Flagged IP: `101.99.76.120`
Flagged SHA256 hash: `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`

We recently purchased a new threat intelligence search application called TryDetectThis2.0. You can use this application to gather information on the indicators above.

#### Connecting To The Machine

**Reference**

Just start the Lab Machine by clicking "Start Lab Machine." Once the VM is booted up, double-click the launcher on the desktop to start the TryDetectThis2.0 application.

#### Set up your virtual environment

**Reference**

To successfully complete this room, you'll need to set up your virtual environment. This involves starting the Target Machine, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.

---

**1. What is the name of the file identified with the flagged SHA256 hash?**

<p align="center">
<img src=screenshots/invite_file.png width="700">
</p>
As an SOC, finding a suspicious downloaded file called **syshelpers.exe** should set off alarm bells. Attackers will often name their files masquerading as legitimate tools and processes your computer might need; this is a prime example of such! Upon finding the file the SOC should begin immediate static analysis of the hash to collect more information on the potentially malicious file.

**Answer: syshelpers.exe**
 
---
 
**2. What is the file type associated with the flagged SHA256 hash?**

 <p align="center">
<img src=screenshots/invite_filetype.png width="700">
</p>

**Answer: Win32 EXE**
 
---
 
**3. What are the execution parents of the flagged hash? List the names chronologically, using a comma as a separator. Note down the hashes for later use.**

  <p align="center">
<img src=screenshots/invite_parentprocess.png width="700">
</p>
**Note:** When analyzing a file in VirusTotal it is best practice request VirusTotal to reanalyze thesuspicious file to pull new information or data that has not been populated!
Finding out what spawned our malicious file is important because it tells us how the infection breached the system, its is crucial for building a timeline of events to be documented in our **incident report**, future detection incidents could be flagged by the processes that ran which is more in-depth than just a hash which is a big deal for future prevention logic.

**361GJX7J:** 047c5eec0445746862710d20e50a5dd04510b7e625fa5c1f5d48ce078001c0de
**installer.exe:** fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942

**Answer: 	361GJX7J, 	installer.exe**
 
---
 
**4. What is the name of the file being dropped? Note down the hash value for later use.**

   <p align="center">
<img src=screenshots/invite_parentprocess.png width="700">
</p>
Searching for dropped files is a crtitical step for any SOC investigation as it now points to the attacker is moving forward in their attack, this dropped file is the attacker delivering the payload. It also signals they can be in the proceess of escalation, tracking these dropped files lets us build the chain of attacks.
**Hash AClient.exe:** dd02c105809e4ca41a5489e585ba025eddb89a91703b73a566c9903e6406a08c

**Answer: AClient.exe**
 
---
 
**5. Research the second hash in question 3 and list the four malicious dropped files in the order they appear (from up to down), separated by commas.**

 <p align="center">
<img src=screenshots/invite_dropped4.png width="700">
</p>

**Answer: syshelpers.exe, nat1.vbs, 3uhl34xj.0.cs, bitaddress.pdf**
 
---
 
**6. Analyse the files related to the flagged IP. What is the malware family that links these files?**

<p align="center">
<img src=screenshots/invite_dropped4.png width="700">
</p>
Researching the flagged IP we locate multiple COmmunicating Files that are worth investigating, upon checking the 8 Files individually we observe that they all have one family in common, they all have various Dynamic Analysis Sandbox Detection but there is one that i shared between all of them: **The sandbox C2AE flags this file as: RAT** along with the family label **AsyncRAT** all this information aquire on VirusTotal
 
**Answer:ASyncRAT**
 
---
 
**7. What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.**


 
**Answer:**
 
---
 
**8. Which tool did the attackers use to steal cookies from the Google Chrome browser?**
 
**Answer:**
 
---
 
**9. Which phishing technique did the attackers use? Use the report to answer the question.**
 
**Answer:**
 
---
 
**10. What is the name of the platform that was used to redirect a user to malicious servers?**
 
**Answer:**
 
---
