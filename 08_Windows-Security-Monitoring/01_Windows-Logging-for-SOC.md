---
title: Windows Logging for SOC
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, event-logs, sysmon, powershell, authentication, process-monitoring, soc-level-1]
status: completed
date: 2026-05-22
date_completed: 2026-06-17
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

SOCs spend most of their time triaging alerts and hunting threats using a SIEM and logs.
  - SOC analysts need to know how to read logs, interpret them, and identify malicious actions within them.

**1. I'm ready to move on!**

**Answer:** N/A

---

## Task 2 - What Is Logged

Every action performed on a computer is logged somewhere.
  - When a user logs in, the OS appends a line to a journal with the timestamp and the account that performed the action.
  - Proper logging ensures all user and system activities are recorded.
  - This means that if anything happens, SOC analysts can follow the log trail and investigate what the user and system did.

**Logs** are critical for investigations:
   - Incident Response: logs show when and how an attack occurred
   - Threat Hunting: logs allow SOC analysts to search for signs of malicious activity
   - Alerting and Triage: logs are the building blocks of any alert or detection rule

**Windows Logs**
  - Windows generates very powerful and detailed logs
  - Windows logs are stored in binary format, which requires specific tools to read and interpret
  - Windows logs are saved in EVTX file format (Windows XML Event Log)
  - Default location: `C:\Windows\System32\winevt\Logs`

Windows has a built-in tool called **Event Viewer**:
  - Used to view and manage event logs
  - Open with: _Win + R_, type _eventvwr_, press Enter

**Event Viewer Example**

<p align="center">
<img src=screenshots/windows_eventviewer.png width="500">
</p>

  1. **Log Sources**: All available EVTX files
  2. **Log List**: Each row is a single event, sortable by:
     - Keywords: indicates whether the action was successful (applies to some events, not all)
     - Date and Time: timestamp of when the event occurred (system time, not UTC)
     - Event ID: unique identifier for the event type (e.g., Event ID 4625 always indicates a **failed login**)
  3. **Log Details** (Tab): the actual event content in plain text or XML format
  4. **Filters Menu**: _Filter Current Log_ and _Find_ options for narrowing results

Everything has a log:
  - Over 500 Event IDs exist in **Security Logs** alone


**Logging Overview**

| Log Source | Purpose |
|---|---|
| Application Logs | Events from user-mode applications (IIS, MS SQL, etc.) |
| Security Logs | Logon attempts, process activity, user management |

**SOC Uses for Logs**

| Activity | How Logs Help |
|---|---|
| Incident Response | Show when and how an attack occurred |
| Threat Hunting | Search for signs of malicious activity |
| Alerting and Triage | Building block of any detection rule or alert |

**1. Looking at the last screenshot, which event ID describes a successful login? (Format: LogSource / ID)**

**Answer: Security/4624**

---

## Task 3 - Security Log: Authentication

**Windows Security Event Logs** provide critical information for triage:
  - Successful Logon: Event ID 4624
      - Logged on the target machine (the one being accessed)
      - **Noisy** -- hundreds of logon events per minute on loaded servers
  - Failed Logon: Event ID 4625
      - Logged on the target machine (the one being accessed)
      - **Inconsistent** -- logging behavior depends on many conditions, which can lead to misinterpretation

---

**Example: Structure of 4624 - Successful Logon**
<p align="center">
<img src=screenshots/windows_structure4624.png width="800">
</p>

**Example: Detecting an RDP Brute Force via 4625 - Failed Login Attempts**

  1. Open Security logs and filter for Event ID 4625 (Failed login attempts)
  2. Look for events with Logon Type 3 or 10 (Network and RDP logins):
      - Modern systems with NLA enabled will show Logon Type 3
      - Older or misconfigured systems without NLA will show Logon Type 10
  3. Red flags to watch for:
      - Multiple different usernames attempted (e.g., admin, helpdesk, cctv) -- indicates password spraying
      - High volume of failures against a single account, usually Administrator -- indicates brute force
      - Workstation Name does not match the corporate naming convention (e.g., "kali" instead of "THM-PC-06")
      - Unexpected source IP (e.g., a printer attempting to authenticate to a Windows Server)

---

  **Example: Detecting a Successful RDP Login via 4624**
  
1. Open Security logs and filter for Event ID 4624 (Successful logins)
2. Look for events with Logon Type 10 (RDP logins):
      - If NLA is enabled, every RDP logon is preceded by a 4624 with Logon Type 3
      - To get the real Workstation Name, check the preceding Logon Type 3 event

3. Red flags are either a preceding brute force or a suspicious source IP or hostname
4. If the login is determined to be malicious, note the Logon ID for correlation:
    - Windows assigns a unique Logon ID to every successful login (e.g., 0x5D6AC)
    - Save the Logon ID -- it links this session to all subsequent activity on the host

---

| Event ID | Purpose | Logging | Limitations |
|---|---|---|---|
| 4624 (Successful Logon) | Detect suspicious RDP/network logins and identify the attack entry point | Logged on the target machine | Noisy -- hundreds of events per minute on loaded servers |
| 4625 (Failed Logon) | Detect brute force, password spraying, or vulnerability scanning | Logged on the target machine | Inconsistent -- caveats may lead to misinterpretation |

**Key Fields in 4624**

| Field | Description |
|---|---|
| Logon ID | Correlates this event with other logs (process creation, user management) |
| Logon Type | Indicates how the login occurred (e.g., Type 10 = RDP) |
| Username | The account that logged in |
| Source IP / Hostname | Origin of the login attempt |

**1. Which IP performed a brute force of the THM-PC?**

<p align="center">
<img src=screenshots/windows_thmpcip.png width="800">
</p>

Opened Event Viewer and navigated to Saved Logs. Sorted events by time and scrolled through looking for clusters of Event ID 4625 (Failed Logon) events all targeting `Computer: THM-PC`. A high volume of repeated failures from a single source IP is the primary indicator of a brute force.

**Answer: 10.10.53.248**

---

**2. Which user has been breached as a result of the attack?**

<p align="center">
<img src=screenshots/windows_thmpcip.png width="800">
</p>

After identifying the brute force, I looked for a follow-up Event ID 4624 (Successful Logon) on the same machine with Logon Type 3 (Network). On modern systems with NLA enabled, successful RDP sessions appear as Logon Type 3 first. Found an Audit Success event for `Account Name: Administrator`, confirming the account was compromised.

**Answer: Administrator**

---

**3. What was the Logon ID of the malicious RDP login? (Logon Type 10)**

<p align="center">
<img src=screenshots/windows_logontype10.png width="800">
</p>

Filtered for Event ID 4624 and looked specifically for Logon Type 10, which indicates an interactive RDP session. This can also appear on older or misconfigured systems where NLA is not enforced. Located the event and recorded the Logon ID for use as a correlation pivot in later tasks.

**Answer: 0x183C36D**

---

## Task 4 - Security Log: User Management

Spotting suspicious Event IDs is the first step toward recognizing an attack in progress.

For example, Event ID 4720 (User Account Created) appearing shortly after a sequence of 4625s (Failed Logons) followed by a 4624 (Successful Logon) is a strong indicator of backdoor account creation.

Attackers often add compromised accounts to privileged groups to establish persistent access:
  - 4732: User was added to a security group
  - 4733: User was removed from a security group

Many real-world breaches involve some form of user manipulation: creating, deleting, or resetting accounts.

| Event ID | Description | Malicious Usage |
|---|---|---|
| 4720 / 4722 / 4738 | User account created / enabled / changed | Create backdoor accounts or enable dormant ones to avoid detection |
| 4725 / 4726 | User account disabled / deleted | Disable privileged SOC accounts to slow incident response |
| 4723 / 4724 | User changed password / password was reset | Reset credentials to hijack a required account |
| 4732 / 4733 | User added to / removed from a security group | Add backdoor accounts to privileged groups like Administrators |

**Event Structure**

| Part | Description |
|---|---|
| Subject | The account performing the action -- note the Logon ID for correlation with 4624 |
| Object | The target of the action (new account, group member, etc.) |
| Details | Specific changes made (target group, new user attributes, password settings) |

**1. Which user was created by the attacker soon after the RDP login?**

<p align="center">
<img src=screenshots/windows_usercreated.png width="800">
</p>

Filtered Event Viewer for Event ID 4720 (User Account Created). The Subject field showed the `Administrator` account as the actor, and the Object field identified the newly created account. Given the timing relative to the malicious RDP session, this is consistent with backdoor account creation.

**Answer: svc_sysrestore**

---

**2. Which two privileged groups was the backdoor user added to? (alphabetical order)**

<p align="center">
<img src=screenshots/windows_backupoperators.png width="500">
</p>

<p align="center">
<img src=screenshots/windows_remotedesktopusers.png width="500">
</p>

Filtered by Task Category for "Security Group Management" and looked for Event ID 4732 (Member Added to Security Group). Found two separate events. `Backup Operators` is a high-value target because members can dump credentials from the SAM database. `Remote Desktop Users` grants persistent RDP access, which is consistent with the attacker's initial entry method.

**Answer: Backup Operators, Remote Desktop Users**

---

**3. Does the Logon ID field match what you saw in the previous task? (Yea/Nay)**

<p align="center">
<img src=screenshots/windows_matchlogonid.png width="500">
</p>

Checked the Subject field of both 4732 events and confirmed the Logon ID matches the value recorded in Task 3 (`0x183C36D`). This correlation confirms all three actions -- the RDP login, the account creation, and the group additions -- belong to the same attacker session.

**Answer: Yea**

---

## Task 5 - Sysmon: Process Monitoring

There are two ways to monitor process creation on Windows:

  - **Security Log - Event ID 4688 (Process Creation):** logs every new process launch including its command line and parent process details
      - Disabled by default -- must be enabled through Group Policy or audit policy settings
  
  - **Sysmon - Event ID 1 (Process Creation):** extends 4688 with additional fields such as **process hash and binary signature**
      - Sysmon is a free external tool from Sysinternals and is not installed by default
      - The SOC standard for advanced process monitoring
      - Logs found at: `Applications & Services -> Microsoft -> Windows -> Sysmon -> Operational`
   
  **Comparison: Event ID 4688 vs Sysmon Event ID 1**
  
  <p align="center">
<img src=screenshots/windows_eventsexample.png width="700">
</p>


| Event Code | Purpose | Limitations |
|---|---|---|
| 4688 (Security Log) | Logs every new process launch including command line and parent process | Disabled by default -- must be enabled manually |
| 1 (Sysmon) | Replaces 4688 with advanced fields: process hash, signature, and more | External tool -- not installed by default |

**Sysmon Event ID 1 Key Fields**

| Field Group | Fields |
|---|---|
| Process Info | PID, image path, command line |
| Parent Info | Parent process context -- used to build the process tree and reconstruct the attack chain |
| Binary Info | Process hash, signature, PE metadata |
| User Context | User running the process and Logon ID (correlates with Security logs) |

**1. Which web browser does Sarah use to browse the web?**

  <p align="center">
<img src=screenshots/windows_googlechrome.png width="800">
</p>

Navigated to the Sysmon Operational log and filtered for Event ID 22 (DNS Query). Checked the `Image` field to identify what process was generating DNS traffic. The path `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe` confirmed the browser in use.

**Answer: Google Chrome**

---

**2. Which file did Sarah download from the browser?**

 <p align="center">
<img src=screenshots/windows_sarahdownload.png width="800">
</p>

Filtered Sysmon logs for Event ID 11 (File Created) and looked for events where the `Image` field pointed to Chrome and the `TargetFilename` was inside Sarah's `Downloads` folder. This identified the file dropped to disk by the browser.

**Answer: C:\Users\sarah.miller\Downloads\ckjg.exe**

---

**3. Which URL was the file downloaded from?**

 <p align="center">
<img src=screenshots/windows_hosturl.png width="800">
</p>

Filtered for Event ID 15 (File Stream Created), which captures the Alternate Data Stream (Zone.Identifier) written by Windows when a file is downloaded from the internet. The `Contents` field contained the zone transfer data including `HostUrl=http://gettsveriff.com/bgj3/ckjg.exe`, revealing the exact download source.

**Answer: http://gettsveriff.com/bgj3/ckjg.exe**

---

## Task 6 - Sysmon: Files and Network

**Sysmon** provides deep visibility beyond just process creation:
  - File and registry changes
  - Network connections
  - DNS queries
  - Many other events critical to threat detection

Sysmon is highly customizable -- you can filter what gets logged -- but for full attack reconstruction, a combination of log sources is required:
  - Network logs to identify exfiltration destinations
  - Registry change logs to detect persistence mechanisms


| Event ID | Security Log Alternative | Purpose |
|---|---|---|
| 11 (File Create) | 4656 -- disabled by default | Detect files dropped by malware |
| 13 (Registry Value Set) | 4657 -- disabled by default | Detect registry changes used for persistence |
| 3 (Network Connection) | No direct alternative | Detect outbound traffic from untrusted processes |
| 22 (DNS Query) | No direct alternative | Detect connections to known malicious destinations |

**Correlation Note:** All Sysmon events share `ProcessId` -- use it to find the corresponding Event ID 1 to get full process context including parent info and Logon ID.

**1. Which file was created by the downloaded malware to persist on the host?**

 <p align="center">
<img src=screenshots/windows_hosturl.png width="800">
</p>

Returned to Event ID 11 (File Created) and filtered for events where `Image` pointed to `ckjg.exe`. The `TargetFilename` field showed a file written to the Startup folder (`C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`). Writing to Startup is a classic persistence technique -- the file executes automatically on every user logon.

**Answer: C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url**

---

**2. What is the Command and Control server the malware connected to? (Format: IP:Port)**

 <p align="center">
<img src=screenshots/windows_hostip.png width="800">
</p>

Filtered for Event ID 3 (Network Connection) with `Image` set to `ckjg.exe`. The `DestinationIp` and `DestinationPort` fields directly identify where the malware beaconed out to. Non-standard high ports like 7777 are a common indicator of C2 traffic.

**Answer: 193.46.217.4:7777**

---

**3. Which domain does the malicious IP correspond to?**

<p align="center">
<img src=screenshots/windows_hostdomain.png width="800">
</p>

Filtered for Event ID 22 (DNS Query) with `Image` pointing to `ckjg.exe`. The `QueryResults` field returned `::ffff:193.46.217.4`, which matched the C2 IP from the previous question. The `QueryName` field revealed the domain the malware resolved before establishing the connection.

**Answer: hkfasfsafg.click**

---

## Task 7 - PowerShell: Logging Commands

**PowerShell** is a favored attacker tool because it is trusted by Windows and highly capable:
  - Downloading malware
  - System discovery
  - Data exfiltration
  - Advanced techniques like process injection

To capture the **exact commands** executed, analysts can check the PowerShell history file:
  - Location: `C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`
  - Records every command typed interactively into a PowerShell session
  - A separate history file exists for each user on the system
  - Persists across reboots unless manually deleted
  - Does **not** log command outputs or script block content

**PowerShell History File Path:**

```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

| Property | Detail |
|---|---|
| Format | Plain text, one command per line |
| Created for | Every active user on the system |
| Persistence | Survives reboots unless manually deleted |
| Logs | Commands typed -- does not log outputs or script content |

**1. Which PowerShell command was executed first?**

<p align="center">
<img src=screenshots/windows_consolehistory.png width="800">
</p>

Navigated to the Administrator's PSReadline history file at the path above. The file lists commands in chronological order, one per line. The first entry was the first command executed in that PowerShell session.

**Answer: Get-ComputerInfo**

---

**2. When did the Administrator run the first PS command? (Format: April 18, 2025)**

<p align="center">
<img src=screenshots/windows_txtfiledate.png width="800">
</p>

Checked the file metadata (last modified / created timestamp) of the `ConsoleHost_history.txt` file to determine when the session began. The date on the file corresponds to when the first command was recorded.

**Answer: May 18, 2025**

---

**3. Can you find the flag stored in the PowerShell history? (Format: THM{...})**

<p align="center">
<img src=screenshots/windows_thmbobflag.png width="800">
</p>

There were two user accounts on the machine. I checked the PSReadline history file for both users. The flag was not in the history file itself -- it was stored in a separate file found by following a path referenced in the history. Located under `thm.bob\Documents\flag`.

**Answer: THM{it_was_me!}**

---

## Task 8 - Conclusion

**Takeaways**
  - Identified and read Event IDs commonly seen in SOC work
  - Grouped logs by Logon ID and Process ID to reconstruct attacker sessions
  - Learned the advantages of Sysmon over default Security Logging
  - Explored PowerShell logging through the PSReadline history file

**1. Hope you enjoyed the room!**

**Answer:** N/A

---
