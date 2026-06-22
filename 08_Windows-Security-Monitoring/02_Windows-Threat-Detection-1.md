---
title: Windows Threat Detection 1
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, initial-access, RDP, phishing, USB, sysmon, event-logs, MITRE, SOC]
status: completed
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

<p align="center">
<img src=screenshots/windows_threatdetection1.png width="1500">
</p>

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
**Initial Access** can be described as the moment the attacker has successfully entered a target's environment
  - How do attackers get through the front door?
      - Attackers can use various methods to gain initial access; it can be through an exposed service, or they can rely on human interaction(user-driven)
          - brute forcing internet-exposed RDP
          - exploit a vulnerable mail server
          - enter the system via unprotected MS SQL
       
**Exposed Services**
<p align="center">
<img src=screenshots/windows_exposedservices.png width="700">
</p>

**Ports** offer a door to exposed services
  - RDP, HTTP, SMTP are all important ports that are regularly used by companies; leaving these unprotected can be disastrous

There are proven **MITRE** Techniques that attackers use to gain Initial Access
  - **T1133 External Remote Services**: Threat actors look to gain entry through RDP/VNC/SSH with weak passwords
      - use of legitimate external remote services thru weak passwords, stolen creds, valid accounts or brute-force
  - **T1190 Exploit Public-Facing Applications**: Threat actors scour the internet for misconfigured, vulnerable websites, applications and, servers
      - exploit a weakness; bug, CVE, misconfiguration

**User-Driven Methods**
<p align="center">
<img src=screenshots/windows_userdriven.png width="700">
</p>

Humans are often responsible for the breaches in security, whether we like it or not, humans  are the weakest link in security
  - clicking malicious links
  - launching phishing attachments
  - using pirated software
  - plugin in unknown USB devices

Common User-driven **MITRE** techiniques:
  - **T1566 Phishing**: Threat actors deploy various methods that trick the user into launching the malware
  - **T1091 Removable Media**: Threat actors will often leave infected USB devices in the parking lots of big companies, hoping an employee will plug them in
    


### Task Questions

**1. Which MITRE technique ID describes Initial Access via a vulnerable mail server?**

<p align="center">
<img src=screenshots/windows_t1190.png width="700">
</p>

**T1190 Exploit Public-Facing Application** Threat actors gain Initial Access on a website, application or server through weaknesses in the system.

- **Answer: T1190**

---

**2. Which Initial Access method relies on a user opening a malicious email attachment?**

<p align="center">
<img src=screenshots/windows_t1566.png width="700">
</p>

Threat Actors will attempt to gain Initial Access through various forms of Phishing that are electronically delivered via social engineering.

- **Answer: Phishing**

---

## Task 3 - Initial Access via RDP

### Key Concepts

It is common knowledge that if you set a weak password to your RDP, you will be breached quicker than you can see it coming.
  - Exposed RDP is such a problem that we started calling it Ransomware Deployment Protocol
  - Botnets are running 24/7 and automatically scanning the entire internet for exposed RDP services

**How to Detect an RDP Breach?**
  - Windows Event Viewer will be our main tool
  - C:\Users\Desktop\Administrator\Practice\RDP Case\RDP-Security.evtx file
  - An exposed RDP would show up in the Event Viewer as multiple Logon attempts Event ID**4625** 

  **Exposed RDP Example in Event Viewer**
  <p align="center">
<img src=screenshots/windows_exposedrdpexample.png width="700">
</p>


#### RDP Attack Detection Chain

| Attack Step | Event ID | Filter | What You Find |
|---|---|---|---|
| Brute-force attempts | 4625 | Logon Type 3/10 + external source IP | Attacker IP flooding failed logins |
| Successful Initial Access | 4624 | Logon Type 3/10 + same source IP | Account used to breach the host |
| Interactive RDP session | 4624 | Logon Type 10 | Confirms hands-on attacker session |
| Post-breach process activity | Sysmon | Match Logon ID from 4624 | All processes spawned by threat actor |

### Task Questions

**1. Which user seems to be most actively brute-forced by botnets?**

  <p align="center">
<img src=screenshots/windows_exposedrdpexample.png width="700">
</p>

Reviewing the log RDP-Security.evtx, we can observe multiple Brute-Force Logon attempts with Event ID 4625, Logon Type: 3, under General, and the Account Name ADMINISTRATOR is the account repeatedly probed.

- **Answer: Administrator**

---

**2. Which IP managed to breach the host via RDP (Logon Type 10)?**

  <p align="center">
<img src=screenshots/windows_ipaddresstask3.png width="700">
</p>

Events with the ID 4624 and Logon Type:10 indicate an active RDP connection, we can view the Details Tab and Friendly View: under IpAddress 203.205.34.107

- **Answer: 203.205.34.107**

---

**3. Can you get the real Workstation Name (hostname) of the threat actor?**

  <p align="center">
<img src=screenshots/windows_workstationname.png width="700">
</p>

Under the same Details for the Event in which we found the IpAddress we observe the  WorkstationName WIN-F89VT9IER10 and we keep inspecting earlier logs for further investigation we can observe the WIN-F89VT9IER10 which we saw earlier as a **TargetDomainName** and the **WorkstationName DESKTOP-QNBC4UU**

- **Answer: DESKTOP-QNBC4UU**

---

## Task 4 - Initial Access via Phishing

### Key Concepts

**Phishing Attacks still play a major role in how threat actors are able to get into systems**
  - As long as humans have access to the internet they will always be susceptible to phishing attacks

**Binary Attachments** While most of us know not to open random .exe files, people tend to be less cautious when it comes to .src files or .cpl even tyhought they can all contain malware
  - attackers can double up on the extension file.pdf.exe or cat.png.com

**LNK Attachments** Threat actors will hide malicious scripts behind LNK shortcuts
  - Example: File.zip that contains 2 files, a PDF and a shortcut to the website
      - PDF looks normal upon inspection
      - Since the PDF looks harmless, the user is inclined to click on the shortcut, which triggers a PowerShell command

**Example of the PDF file with a hidden LNK PowerShell**
  <p align="center">
<img src=screenshots/windows_lnkpowershell.png width="700">
</p>

#### Phishing Attachment Types

| Type | Disguise Method | Execution | Detection |
|---|---|---|---|
| Binary (.exe, .com, .scr, .cpl) | Double extension + fake icon | User double-clicks | Sysmon Event ID 1 (process create) from explorer.exe |
| LNK shortcut | Looks like legitimate shortcut, icon set to match | User opens shortcut | LNK properties Target field; Sysmon file creation in Downloads before exec |

### Task Questions

**1. Let's play the role of the untrained user and mindlessly open the COM file.
Run the www.skype.com file from the Phishing Case 1 folder, which flag do you get?**

  <p align="center">
<img src=screenshots/windows_phishingcase1.png width="700">
</p>

- **Answer: THM{misleading_extension}**

---

**2. Continue with the second attachment from the Phishing Case 2 folder.
From which URL does the malicious LNK download the next stage malware?**

Extracted the New PC Store!.zip, checked the Properties for the Official Website Shortcut

- **Answer: http://wp16.hqywlqpa.thm:8000/cgi-bin/f**

---

**3. Finally, move on to the Phishing Case 3 folder and review its content.
What is the name of the double-extension file you see there?**

 <p align="center">
<img src=screenshots/windows_bestcatwindow.png width="700">
</p>


- **Answer: best-cat.jpg.exe**

---

## Task 5 - Continuing Phishing Topic

### Key Concepts

**Detecting Malicious Download**

**Sysmon** is one the tools used to help you find a malware attachment and detect every **attack stage:**

**Example: Sysmon Event Chain for Double_extension Attachment**
 <p align="center">
<img src=screenshots/windows_sysmonexample.png width="700">
</p>

**Notes on LNK Attachments** LNK files leave execution traces
  - after the LNK attachment is run it can make it look like explorer.exe launches PowerShell directly
  - you can still find the preceding file creation events if the LNK was truly a phishing attachment, it would show up somewhere in the downloads

**Example**
 <p align="center">
<img src=screenshots/windows_lnknotes.png width="700">
</p>

#### Sysmon Event Chain: Double-Extension Binary

| Step | Sysmon Event ID | Image | Key Field |
|---|---|---|---|
| Browser launched | 1 (process create) | msedge.exe / browser | ParentImage: Explorer.EXE |
| Archive downloaded | 11 (file create) | msedge.exe | TargetFilename: Downloads\*.zip |
| File extracted | 11 (file create) | Explorer.EXE or 7zG.exe | TargetFilename: malware path |
| Malware executed | 1 (process create) | malware binary | ParentImage: Explorer.EXE |

### Task Questions

**1. Which file did the user download via the web browser?**

 <p align="center">
<img src=screenshots/windows_topcats.png width="700">
</p>

- **Answer: C:\Users\Administrator\Downloads\top-cats.zip**

---

**2. In which folder did the user unarchive the suspicious file?**

 <p align="center">
<img src=screenshots/windows_picturesfolder.png width="700">
</p>

Investigating the logs we follow the timeline pipe and see that the top-cats.zip file was extracted 20 seconds later a File Created Event ID 11 
TargetFilename: C:\Users\Administrator\Pictures\best-cat.jpg.exe

- **Answer: C:\Users\Administrator\Pictures**

---

**3. What is the process ID of the launched phishing malware?**

 <p align="center">
<img src=screenshots/windows_process5484.png width="700">
</p>

Invetigating down the timeline of the logs we search through the several Event ID 1 that are avaliable and we can see the CommandLine: "C:\Users\Administrator\Pictures\best-cat.jpg.exe" 

- **Answer: 5484**

---

**4. Which malicious domain did the malware try to connect to?**

 <p align="center">
<img src=screenshots/windows_rjjstore.png width="700">
</p>

Inspect the logs for Event ID 22 which is DNS query in order to find the domain

- **Answer: rjj.store**

---

## Task 6 - Initial Access via USB

### Key Concepts

USB attacks may seem like a thing of the past. But they're very much still active and used by threat actors
  - Initial Access over USB bypasses firewalls
  - Starts an attack chain without internet, and spreads  without user interaction

**Detecting an Infected USB**
  - Detecting Initial Access via USB looks a lot like detecting phishing attachments
  - Both methods rely on the user running a binary via a graphical interface (explorer.exe)
  - Some cases might offer evidence of external drive execution, such as: E:\malware.exe

**USB Detected Event Viewer Example**
 <p align="center">
<img src=screenshots/windows_usbexample.png width="700">
</p>

#### USB vs Phishing Detection Comparison

| Factor | Phishing | USB |
|---|---|---|
| Launch point | Downloads folder (C:\Users\...\Downloads\) | External drive path (E:\, F:\, etc.) |
| ParentImage | Explorer.EXE | Explorer.EXE |
| Propagation | N/A | Malware copies itself to other removable drives |
| Pre-exec artifact | LNK or archive in Downloads | Malicious file visible on USB drive |

### Task Questions

**1. Which USB file was launched by the user?**

 <p align="center">
<img src=screenshots/windows_externaldrive.png width="700">
</p>
Filtered by Event ID 1: Process Create and investigated the logs for an External Drive command. CommandLine: "E:\Open Sandisk 4GB USB.exe" 

- **Answer: E:\Open Sandisk 4GB USB.exe**

---

**2. Which suspicious file did the malware drop to the disk?**

<p align="center">
<img src=screenshots/windows_malwaredisk.png width="700">
</p>
Upon finding the USB file launched, I sorted the events by the time so I could follow the attack chain, the very next Event is Event ID 11 File Created, and we can observe TargetFilename: C:\Users\Public\Documents\winupdate.exe

- **Answer: C:\Users\Public\Documents\winupdate.exe**

---

**3. To which other USB did the malware propagate?**

<p align="center">
<img src=screenshots/windows_externaldrivecopy.png width="700">
</p>
Following the timeline, we can observe on Event ID 11: File Created that the USB malware can infect other drives and in this case we can observe it duplicated to another external drive TargetFilename: F:\Open Data Traveler 32 GB USB.exe

- **Answer:**

---

## Task 7 - Conclusion

### Key Concepts

  - Two most common Windows Initial Access methods: exposed services and user-driven attacks
  - Exposed services: detect via Security log Event IDs 4624/4625 (successful/failed auth)
  - User-driven attacks: best detected via Sysmon process execution events (Event ID 1, 11)
  - LNK phishing has a unique trace pattern, look for file creation events before execution
  - Each technique leaves distinct artifacts in logs, pattern recognition is the skill to build 

### Task Questions

**1. I am ready to move on!**

- **Answer:** N/A
