---
title: Invite Only
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [threat-intel, osint, malware-analysis, ioc-investigation]
status: completed
date: 2026-07-13
date_completed: 2026-07-16
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

As an SOC analyst, finding a suspicious downloaded file called **syshelpers.exe** should set off alarm bells. Attackers often name their files to masquerade as legitimate tools or processes a computer might need, and this is a prime example. Upon finding the file, the SOC should begin immediate static analysis of the hash to collect more information on the potentially malicious file.

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

**Note:** When analyzing a file in VirusTotal, it is best practice to request VirusTotal reanalyze the suspicious file to pull new information or data that has not been populated.

Finding out what spawned our malicious file is important because it tells us how the infection breached the system in the first place. That's crucial for building a timeline of events to document in our **incident report**. It also matters for future prevention, since a process chain is a lot harder for an attacker to change than a single file hash. Flagging on parent/child process behavior instead of just a hash means detection logic that actually survives the attacker swapping their payload.

**361GJX7J:** 047c5eec0445746862710d20e50a5dd04510b7e625fa5c1f5d48ce078001c0de
**installer.exe:** fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942

**Answer: 361GJX7J, installer.exe**

---

**4. What is the name of the file being dropped? Note down the hash value for later use.**

<p align="center">
<img src=screenshots/invite_parentprocess.png width="700">
</p>

Searching for dropped files is a critical step in any SOC investigation, since a dropped file is the attacker delivering their payload. That's a sign the attacker is moving forward, potentially into escalation, and tracking each dropped file lets us build out the full chain of attack.

**Hash AClient.exe:** dd02c105809e4ca41a5489e585ba025eddb89a91703b73a566c9903e6406a08c

**Answer: AClient.exe**

---

**5. Research the second hash in question 3 and list the four malicious dropped files in the order they appear (from up to down), separated by commas.**

<p align="center">
<img src=screenshots/invite_dropped4.png width="700">
</p>

**Answer: searchhost.exe, syshelpers.exe, nat.vbs, runsys.vbs**

---

**6. Analyse the files related to the flagged IP. What is the malware family that links these files?**

<p align="center">
<img src=screenshots/invite_dropped4.png width="700">
</p>

Researching the flagged IP on VirusTotal surfaces multiple Communicating Files worth investigating. Checking all 8 files individually, each one carries its own mix of Dynamic Analysis Sandbox detections, but one classification is shared across every single file: the sandbox flags each of them as a **RAT**, with the family label **AsyncRAT**. That consistency across 8 separate files is what confirms the malware family tying this whole flagged IP together.

**Answer: AsyncRAT**

---

**7. What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.**

<p align="center">
<img src=screenshots/invite_report.png width="700">
</p>

An SOC analyst has the entire web for research, so don't limit yourself to just the bookmarked tools. Sometimes finding a report requires digging on Google, so your OSINT instincts should kick in here. For this question, I searched: Report for 5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f

**Answer: From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery**

---

**8. Which tool did the attackers use to steal cookies from the Google Chrome browser?**

<p align="center">
<img src=screenshots/invite_cookies.png width="700">
</p>

The full report gives the SOC various layers of context, including how the attack unfolds, the techniques involved, how the exploitation works, and example scripts used along the way.

**Answer: ChromeKatz**

---

**9. Which phishing technique did the attackers use? Use the report to answer the question.**

<p align="center">
<img src=screenshots/invite_phishing.png width="700">
</p>

**Answer: ClickFix**

---

**10. What is the name of the platform that was used to redirect a user to malicious servers?**

<p align="center">
<img src=screenshots/invite_redirect.png width="700">
</p>

**Answer: Discord**

---
