---
title: Windows Threat Detection 1
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, initial-access, rdp, phishing, usb, sysmon, mitre, soc-level-1]
status: in-progress
date: 2026-06-18
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

This room builds on Windows logging fundamentals from the Windows Logging for SOC room and applies that knowledge to real Initial Access detection scenarios using Windows event logs.

<!-- 
KEY CONCEPTS:
- What is the primary log source used for detection in this room?
- What two categories of Initial Access methods does this room cover?
- What prerequisites are needed before starting this room?
-->

**1. Let's begin!**

**Answer:** N/A

---

## Task 2 - Intro to Initial Access

**Initial Access** is the first stage of an attack -- the moment a threat actor successfully enters the target environment. No matter the final objective, every attack begins with getting through the front door.

**Two Categories of Initial Access:**

- **Exposed Services:** Internet-facing services attacked without user interaction
- **User-Driven Methods:** Attacks that rely on a user performing an action (clicking, plugging in a USB, etc.)

**Exposed Service Techniques**

| MITRE ID | Technique | Description |
|---|---|---|
| T1133 | External Remote Services | Exploit exposed RDP/VNC/SSH with weak or default credentials |
| T1190 | Exploit Public-Facing Application | Target misconfigured or unpatched web applications |

**User-Driven Techniques**

| MITRE ID | Technique | Description |
|---|---|---|
| T1566 | Phishing | Trick users into launching malware via email attachments or links |
| T1091 | Removable Media | Infect USB devices and rely on users to spread the malware |

<!-- 
KEY CONCEPTS:
- What is Initial Access in the context of the MITRE ATT&CK framework?
- What MITRE ID covers phishing as an Initial Access vector?
- What MITRE ID covers exploitation of a vulnerable public-facing service?
- What is the difference between exposed service attacks and user-driven attacks?
- Why are user-driven attacks harder to prevent than blocking a port?
-->

**1. Which MITRE technique ID describes Initial Access via a vulnerable mail server?**

**Answer:**

---

**2. Which Initial Access method relies on a user opening a malicious email attachment?**

**Answer:**

---

## Task 3 - Initial Access via RDP

<!-- 
KEY CONCEPTS:
- What Event ID logs a failed RDP logon attempt?
- What Event ID logs a successful logon?
- What Logon Type indicates an RDP session?
- What Logon Type indicates a Network (NLA) login?
- How does a Logon ID help correlate RDP session activity across logs?
- What is the detection approach for RDP brute force vs. credential-based access?
- Where in Sysmon logs can you trace post-RDP process activity using Logon ID?
-->

**Detection: RDP Brute Force (4625 Failed Logons)**

1. Open `RDP-Security.evtx` in Event Viewer
2. Filter for Event ID 4625 (Failed Logon)
3. Look for high-volume failures from a single IP in a short window -- this is the brute force signature
4. Note the target `Account Name` being attacked most frequently

**Detection: Successful RDP Login (4624 Logon Type 10)**

1. Filter for Event ID 4624 (Successful Logon) with Logon Type 10 (interactive RDP)
2. Identify the source IP and Workstation Name of the successful session
3. Record the Logon ID for post-access correlation
4. Open Sysmon logs and search events with the same Logon ID to trace all processes spawned during the attacker's session

**Log File:** `C:\Users\Administrator\Desktop\Practice\RDP Case\RDP-Security.evtx`

---

**1. Which user seems to be most actively brute-forced by botnets?**

**Answer:**

---

**2. Which IP managed to breach the host via RDP (Logon Type 10)?**

**Answer:**

---

**3. Can you get the real Workstation Name (hostname) of the threat actor?**

**Answer:**

---

## Task 4 - Initial Access via Phishing

**Phishing** cannot be mitigated simply by blocking a port. As long as users have internet access, malicious files can reach endpoints and bypass the perimeter firewall entirely.

**Two Common Phishing Attachment Types:**

**Binary Attachments**

- Windows hides known file extensions by default -- a file named `invoice.pdf.exe` appears as `invoice.pdf`
- Attackers exploit this by using extensions users do not recognize as executable: `.com`, `.scr`, `.cpl`
- Icons are spoofed to match the file type being impersonated

**LNK Attachments**

- LNK shortcut files can point to any command in their `Target` field
- Attackers embed PowerShell, VBScript, or BAT payloads inside LNK files disguised as PDFs or website links
- LNK execution leaves minimal trace -- Windows Explorer reads the `Target` field and launches the payload, making the parent process appear as `explorer.exe`
- Detection pivot: look for a file creation event (Event ID 11) showing the LNK file downloaded to the user's `Downloads` folder before execution

| Attachment Type | Detection Method | Parent Process |
|---|---|---|
| Binary (.exe, .com, .scr) | Sysmon Event ID 1 -- process creation with suspicious image path | explorer.exe |
| LNK Shortcut | Sysmon Event ID 11 (file created in Downloads) + Event ID 1 for spawned process | explorer.exe |

<!-- 
KEY CONCEPTS:
- Why does Windows hiding known file extensions aid phishing attacks?
- What extensions are commonly used by attackers besides .exe?
- What field in an LNK file contains the actual payload?
- Why does LNK execution show explorer.exe as the parent process?
- What Sysmon event confirms an LNK file was downloaded before execution?
- What is RemcosRAT and in what phishing campaigns has it been used?
-->

**Phishing Case 1 -- COM Binary**

**Log File:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 1\`

**1. Run the www.skype.com file from the Phishing Case 1 folder -- which flag do you get?**

**Answer:**

---

**Phishing Case 2 -- LNK Attachment**

**Log File:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 2\`

**2. From which URL does the malicious LNK download the next stage malware?**

To find the download URL, right-click the LNK file, open Properties, and navigate to the Shortcut tab. The `Target` field contains the full PowerShell command including the download URL embedded in the payload.

**Answer:**

---

**Phishing Case 3 -- Double-Extension File**

**Log File:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 3\`

**3. What is the name of the double-extension file you see there?**

Navigated to the Phishing Case 3 folder and enabled "Show file extensions" in File Explorer options. A double-extension file abuses Windows' default behavior of hiding known extensions -- the file appears to be one type while actually being an executable.

**Answer:**

---

## Task 5 - Phishing Detection via Sysmon

**LNK Detection Logic:**

Since LNK files make `explorer.exe` appear as the parent of any spawned process, the key to identifying LNK phishing over other attack vectors is correlating process execution (Event ID 1) with a preceding file creation event (Event ID 11) showing the LNK file arriving in the user's `Downloads` folder.

```yaml
ParentImage: C:\Windows\Explorer.EXE
```

**Detection Workflow:**

1. Filter Sysmon logs for Event ID 1 (Process Created) with `ParentImage: explorer.exe` and a suspicious `CommandLine` (e.g., PowerShell with encoded commands)
2. Note the `ProcessId` and timestamp
3. Filter for Event ID 11 (File Created) in the same timeframe -- look for the LNK file in `Downloads` or a temp folder
4. Correlate the two events to confirm LNK as the delivery mechanism
5. Follow the `ProcessId` to Event ID 3 (Network Connection) or Event ID 22 (DNS Query) to identify C2 activity

**Log File:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 3\Phishing-Sysmon.evtx`

<!-- 
KEY CONCEPTS:
- Why does LNK execution always show explorer.exe as the parent?
- What two Sysmon event IDs are used together to confirm LNK-based phishing?
- What field is used to pivot from a process creation event to its network activity?
- How does a double-extension file evade user detection?
-->

**1. Which file did the user download via the web browser?**

**Answer:**

---

**2. In which folder did the user unarchive the suspicious file?**

**Answer:**

---

**3. What is the process ID of the launched phishing malware?**

**Answer:**

---

**4. Which malicious domain did the malware try to connect to?**

**Answer:**

---

## Task 6 - Initial Access via USB

**Removable Media** as an Initial Access vector bypasses firewalls entirely and can spread without internet access. Threat actors use USB delivery in targeted attacks and worm-style campaigns.

**Common USB Malware Techniques:**

- Hide all legitimate files on the USB and replace them with a malicious `RECOVERY.lnk`
- Create a binary (e.g., `Photos.exe`) with a folder icon to trick users into launching it
- Create double-extension copies of legitimate files (e.g., `photo_2024_1_12.jpg.exe`)

**Detection Note:** USB-based Initial Access looks nearly identical to phishing in Sysmon logs because both vectors result in a user launching a binary through `explorer.exe`. The distinguishing indicator is the `Image` path in Event ID 1 -- if execution originates from a removable drive (e.g., `E:\malware.exe`), that confirms USB delivery.

**Detection Workflow:**

1. Filter Sysmon Event ID 1 for processes with `Image` paths on a non-system drive letter (D:, E:, F:, etc.)
2. Note the `ProcessId` and parent process -- should be `explorer.exe`
3. Filter Event ID 11 (File Created) to find files the malware dropped to the host disk
4. Check for additional drive letters in `TargetFilename` paths to identify propagation to other USB devices
5. Filter Event ID 3 or 22 for C2 activity using the same `ProcessId`

| Sysmon Event ID | Purpose in USB Investigation |
|---|---|
| 1 (Process Created) | Identify the USB binary executed by the user -- note the drive letter in the Image path |
| 11 (File Created) | Find files dropped to the host or propagated to other USB drives |
| 3 (Network Connection) | Detect C2 beaconing after execution |

<!-- 
KEY CONCEPTS:
- What distinguishes USB Initial Access from phishing in Sysmon logs?
- What field in Event ID 1 reveals execution from a removable drive?
- What Sysmon event shows malware propagating to a second USB device?
- How does malware hide itself on a USB to trick users into launching it?
- What real-world threat actor groups have used USB-based Initial Access?
-->

**Log File:** `C:\Users\Administrator\Desktop\Practice\USB Case\USB-Sysmon.evtx`

**1. Which USB file was launched by the user?**

**Answer:**

---

**2. Which suspicious file did the malware drop to the disk? (Format: full path)**

**Answer:**

---

**3. To which other USB did the malware propagate? (Format: just the drive letter)**

**Answer:**

---

## Task 7 - Conclusion

**Key Takeaways**

- The two primary Windows Initial Access categories are exposed services and user-driven attacks
- RDP brute force is detectable using default Security logs (Event IDs 4624/4625) -- Logon ID is the key correlation pivot
- User-driven attacks (phishing, USB) are best detected through Sysmon process execution events (Event ID 1)
- LNK phishing is identified by correlating Event ID 11 (file creation in Downloads) with Event ID 1 (process launched by explorer.exe)
- USB delivery is distinguished from phishing by the drive letter in the `Image` path of Event ID 1

**1. I am ready to move on!**

**Answer:** N/A

---
