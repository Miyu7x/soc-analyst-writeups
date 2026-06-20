---
title: Windows Threat Detection 1
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, initial-access, RDP, phishing, USB, sysmon, event-logs, MITRE, SOC]
status: in-progress
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

<p img=src

## Task 1 - Introduction

### Key Concepts

Windows Threat Detection involves a lot of Windows Logging
  - How do attackers gain Initial Access?
  - What discovery techniques can SOCs use while reading Windows event logs?

### Task Questions

**1. Let's begin!**

- **Answer:** N/A

---

## Task 2 - Intro to Initial Access

### Key Concepts

<!-- Initial Access = the moment a threat actor first successfully enters a target environment -->
<!-- Two main groups: exposed service attacks and user-driven attacks -->
<!-- Exposed services: RDP, VNC, SSH, web apps -- anything with a public-facing port -->
<!-- User-driven: phishing, USB drops, pirated software -- attacker relies on human error -->
<!-- MITRE T1133: External Remote Services (RDP/VNC/SSH brute-force or credential abuse) -->
<!-- MITRE T1190: Exploit Public-Facing Application (vulnerable/misconfigured web services) -->
<!-- MITRE T1566: Phishing (malicious attachments or links) -->
<!-- MITRE T1091: Replication Through Removable Media (infected USB) -->
<!-- Real-world ransomware groups (Medusa, Akira) use all described techniques across campaigns -->

### Task Questions

**1. Which MITRE technique ID describes Initial Access via a vulnerable mail server?**

- **Answer:**

---

**2. Which Initial Access method relies on a user opening a malicious email attachment?**

- **Answer:**

---

## Task 3 - Initial Access via RDP

### Key Concepts

<!-- RDP (port 3389) is a massively exposed attack surface -- 5M+ internet-facing machines per Censys -->
<!-- "Ransomware Deployment Protocol" -- RDP breach frequently precedes ransomware deployment -->
<!-- Brute-force detection: Event ID 4625 (failed logon), filter Logon Type 3 and 10, filter external source IPs -->
<!-- Successful RDP logon: Event ID 4624 (successful logon), Logon Type 10 = interactive RDP -->
<!-- Post-breach activity: copy Logon ID from 4624 event, search Sysmon logs by that Logon ID to see all threat actor processes -->
<!-- Logon Type 3 = network logon; Logon Type 10 = remote interactive (RDP) -->
<!-- Log file for this task: C:\Users\Administrator\Desktop\Practice\RDP Case\RDP-Security.evtx -->

#### RDP Attack Detection Chain

| Attack Step | Event ID | Filter | What You Find |
|---|---|---|---|
| Brute-force attempts | 4625 | Logon Type 3/10 + external source IP | Attacker IP flooding failed logins |
| Successful Initial Access | 4624 | Logon Type 3/10 + same source IP | Account used to breach the host |
| Interactive RDP session | 4624 | Logon Type 10 | Confirms hands-on attacker session |
| Post-breach process activity | Sysmon | Match Logon ID from 4624 | All processes spawned by threat actor |

### Task Questions

**1. Which user seems to be most actively brute-forced by botnets?**

- **Answer:**

---

**2. Which IP managed to breach the host via RDP (Logon Type 10)?**

- **Answer:**

---

**3. Can you get the real Workstation Name (hostname) of the threat actor?**

- **Answer:**

---

## Task 4 - Initial Access via Phishing

### Key Concepts

<!-- Phishing bypasses firewalls entirely -- user brings malware in from the inside -->
<!-- Phishing attacks up 41x since ChatGPT release per HoxHunt 2025 report -->
<!-- MITRE T1566: Phishing -->

<!-- Binary attachments: Windows hides known extensions by default -->
<!-- Dangerous extensions beyond .exe: .com, .scr, .cpl -- all can contain malware -->
<!-- Abuse pattern: double extension (invoice.pdf.exe) + matching icon tricks untrained users -->
<!-- Example: "tryhatme.com" looks like a website link, executes as a .COM binary -->

<!-- LNK attachments: used to avoid AV detection by wrapping PowerShell/VBS/BAT scripts -->
<!-- LNK "Target" field can contain any command; icon can be set to anything -->
<!-- Detection via LNK: right-click -> Properties -> Shortcut tab to see true target -->
<!-- RemcosRAT: common RAT delivered via LNK phishing in real-world campaigns against companies and government agencies -->
<!-- LNK PowerShell pattern: downloads encoded payload to C:\ProgramData\, then executes -->
<!-- Practice folder: C:\Users\Administrator\Desktop\Practice\Phishing Case 1-3 -->

#### Phishing Attachment Types

| Type | Disguise Method | Execution | Detection |
|---|---|---|---|
| Binary (.exe, .com, .scr, .cpl) | Double extension + fake icon | User double-clicks | Sysmon Event ID 1 (process create) from explorer.exe |
| LNK shortcut | Looks like legitimate shortcut, icon set to match | User opens shortcut | LNK properties Target field; Sysmon file creation in Downloads before exec |

### Task Questions

**1. Run the www.skype.com file from the Phishing Case 1 folder -- which flag do you get?**

- **Answer:**

---

**2. From which URL does the malicious LNK download the next stage malware?**

- **Answer:**

---

**3. What is the name of the double-extension file you see in Phishing Case 3?**

- **Answer:**

---

## Task 5 - Continuing Phishing Topic

### Key Concepts

<!-- Detecting malicious downloads: trace the full chain from browser launch to process execution -->
<!-- Archives (.zip, .rar) are more common than raw .exe attachments -- unarchiving adds a step -->

<!-- Sysmon Event ID 1: Process creation -- captures what ran and what spawned it (ParentImage) -->
<!-- Sysmon Event ID 11: File creation -- captures files written to disk (downloads, unarchived files) -->

<!-- Double-extension chain: browser opens -> archive drops to Downloads (ID 11) -> user extracts (ID 11, explorer.exe or 7zG.exe) -> user executes malware (ID 1, ParentImage = explorer.exe) -->

<!-- LNK execution trace: LNK files leave little direct execution trace -->
<!-- Windows Explorer reads LNK Target and launches it, making it appear as if explorer.exe spawned the payload directly -->
<!-- Key detection: look for Sysmon file creation events (ID 11) for the LNK in Downloads BEFORE execution -->

<!-- Sysmon log for this task: C:\Users\Administrator\Desktop\Practice\Phishing Case 3\Phishing-Sysmon.evtx -->

#### Sysmon Event Chain: Double-Extension Binary

| Step | Sysmon Event ID | Image | Key Field |
|---|---|---|---|
| Browser launched | 1 (process create) | msedge.exe / browser | ParentImage: Explorer.EXE |
| Archive downloaded | 11 (file create) | msedge.exe | TargetFilename: Downloads\*.zip |
| File extracted | 11 (file create) | Explorer.EXE or 7zG.exe | TargetFilename: malware path |
| Malware executed | 1 (process create) | malware binary | ParentImage: Explorer.EXE |

### Task Questions

**1. Which file did the user download via the web browser?**

- **Answer:**

---

**2. In which folder did the user unarchive the suspicious file?**

- **Answer:**

---

**3. What is the process ID of the launched phishing malware?**

- **Answer:**

---

**4. Which malicious domain did the malware try to connect to?**

- **Answer:**

---

## Task 6 - Initial Access via USB

### Key Concepts

<!-- USB attacks bypass firewalls and can spread without internet access or user interaction -->
<!-- MITRE T1091: Replication Through Removable Media -->
<!-- Real-world campaigns: Camaro Dragon, Raspberry Robin -- USB malware is not obsolete -->

<!-- Common USB malware tactics: -->
<!-- - Hide all legitimate files; create a malicious RECOVERY.lnk in their place -->
<!-- - Drop a Photos.exe with a folder icon -->
<!-- - Create double-extension copies of existing files (photo_2024_1_12.jpg.exe) -->

<!-- Detection looks similar to phishing: user launches malware via explorer.exe -->
<!-- Key differentiator: look for execution path from external drive (e.g., E:\malware.exe) -->
<!-- Sysmon Event ID 1 ParentImage = Explorer.EXE + Image path starting with removable drive letter = USB execution -->

<!-- Sysmon log for this task: C:\Users\Administrator\Desktop\Practice\USB Case\USB-Sysmon.evtx -->

#### USB vs Phishing Detection Comparison

| Factor | Phishing | USB |
|---|---|---|
| Launch point | Downloads folder (C:\Users\...\Downloads\) | External drive path (E:\, F:\, etc.) |
| ParentImage | Explorer.EXE | Explorer.EXE |
| Propagation | N/A | Malware copies itself to other removable drives |
| Pre-exec artifact | LNK or archive in Downloads | Malicious file visible on USB drive |

### Task Questions

**1. Which USB file was launched by the user?**

- **Answer:**

---

**2. Which suspicious file did the malware drop to the disk?**

- **Answer:**

---

**3. To which other USB did the malware propagate?**

- **Answer:**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Two most common Windows Initial Access methods: exposed services and user-driven attacks -->
<!-- Exposed services: detect via Security log Event IDs 4624/4625 (successful/failed auth) -->
<!-- User-driven attacks: best detected via Sysmon process execution events (Event ID 1, 11) -->
<!-- LNK phishing has unique trace pattern -- look for file creation events before execution -->
<!-- Each technique leaves distinct artifacts in logs -- pattern recognition is the skill to build -->

### Task Questions

**1. I am ready to move on!**

- **Answer:** N/A
