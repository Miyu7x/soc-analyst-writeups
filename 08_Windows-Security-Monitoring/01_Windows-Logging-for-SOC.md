---
title: Windows Logging for SOC
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, event-logs, sysmon, powershell, authentication, process-monitoring, soc-level-1]
status: in-progress
date: 2026-05-22
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

SOCs spend most of their time traiging alerts and hunting threats using a SIEM and logs
  - SOCs should know how to read logs, interpret them and be able to spot malicious actions within them

**1. I'm ready to move on!**

**Answer:** N/A

---

## Task 2 - What Is Logged

Every action performed ona computer is logged somewhere
  - user logs in, the OS appends a line to some journal, with the time of the action and thw user who perfomed it
  - propper logging ensures all user and system activities are recorded
  - this means if anything happens, SOCs can follow the trail of the log and investigate all actions the user and system performed

**Logs** are critical for investigations:
   - Incident Response: logs will show when and how the attack occurred
   - Threat Hunting: logs allow SOCs to search for signs of malicious activities
   - Alert and Triage: logs are the building blocks of any alert or detection rules

**Windows Logs**
  - Windows generates very powerful logs
  - Windows logs are stored in binary format which will require extensive knowledge to understand and read
  - Windows logs are saved in EVTX file format; Windows XML event log file
  - C:\Windows\System32\winevt\Logs

Windows have a built-in tool called **Event Viewer**
  - views and manages event logs
  - open event viewer with _Win + R_ type in _eventvwr_ press enter

**Event Viewer Example**

<p align="center">
<img src=screenshots/windows_eventviewer.png width="500">
</p>

  1. **Log Sources**: All EVTX files
  2. **Log List**: Each row is a single event that you can sort by:
     - Keywords: this shows if the action performed was successful or not(for some events, not all)
     - Date and Time: timestamp when the event occurred(system time, not UTC!)
     - Event ID: unique number for the event name(an example is 4625 will always be a **failed login**)
  3. **Log Details**(Tab): actual content of the log in plain text or XML format
  4. **Filters Menu**: _Filter Current Log_ and _Find_ options to filter the logs

Everything has a Log
  - over 500 event IDs just for **Security Logs**


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

**Windows Security Event Logs**
  - provides critical information for triage
  - Successful Logon: Event ID 4624
      - log appears on the traget machine(the one you are trying to access)
      - **noisy** meaning hundreds of logon events per minute on loaded servers
  - Failed Logon: Event ID 4625
      - log appears on the traget machine(the one you are trying to access)
      - **inconsistent** logs depend on many different conditions, this may trick SOCs into wrongfully reading the event
---

**Example: Structure of 4624 - Successful Logon**
<p align="center">
<img src=screenshots/windows_structure4624.png width="800">
</p>

**Example: Detecting a Remote Desktop Protocol Brute Force - 4625 Failed Login Attempts**

  1. Open Security logs and filter for 4625 event ID (Failed login attempts)
  2. Look for events with Logon Type 3 and 10 (Network and RDP logins)
      - For most modern systems, the logon type will be 3 (since NLA(opens in new tab) is enabled by default)
      - For older or misconfigured systems, the logon type will be 10 (since NLA is not used)
  3. Every event is now worth your attention, but the main red flags are:
      - Many attempted users like admin, helpdesk,  and cctv (Indicates password spraying)
      - Many login failures on a single account, usually Administrator (Indicates brute force)
      - Workstation Name does not match a corporate pattern (e.g. kali instead of THM-PC-06)
      - Source IP is not expected (e.g. your printer trying to connect to your Windows Server)

---
   
  **Example: Remote Desktop Protocol - 4624 Successful Logins**
  
1. Open Security logs and filter for 4624 event ID (Successful logins)
2. Look for events with Logon Type 10 (RDP logins)
      - If NLA(opens in new tab) is enabled, every RDP logon event is preceded by another 4624 with logon type 3
      - To get a real Workstation Name, you need to check the preceding logon type 3 event

3. Your red flags are either a preceding brute force or a suspicious source IP / hostname
4. If you assume that the login was indeed malicious, find out what happened next:
    - Windows assigns a Logon ID to every successful login (e.g. 0x5D6AC)
    - Logon ID is a unique session identifier. Save it for future analysis!

---

Even experienced IT admins rely on security experts to distinguish between **good** and **bad** events



| Event ID | Purpose | Logging | Limitations |
|---|---|---|---|
| 4624 (Successful Logon) | Detect suspicious RDP/network logins and identify attack starting point | Logged on the target machine | Noisy -- hundreds of events per minute on loaded servers |
| 4625 (Failed Logon) | Detect brute force, password spraying, or vulnerability scanning | Logged on the target machine | Inconsistent -- caveats may lead to misinterpretation |

**Key Fields in 4624**

| Field | Description |
|---|---|
| Logon ID | Correlates this event with other logs (process creation, user management) |
| Logon Type | Indicates how the login occurred (e.g. Type 10 = RDP) |
| Username | The account that logged in |
| Source IP / Hostname | Origin of the login attempt |

**1. Which IP performed a brute force of the THM-PC?**

<p align="center">
<img src=screenshots/windows_thmpcip.png width="800">
</p>

Opened the Event Viewer, under Saved Logs sorted the logs by time, I scrolled to see if i could find multiple Failed Logons Type 3 in a row under the Computer: THM-PC.

**Answer: 10.10.53.248**

---

**2. Which user has been breached as a result of the attack?**

<p align="center">
<img src=screenshots/windows_thmpcip.png width="800">
</p>

Kept scroling down looking for a Successful Logon 4624 under the Computer: THM-PC with the Logon Type: 3 which is a Network login, and we found Audit Success under Account Name: Administrator. Logon Type 3 will be for most modern systems, the logon type will be 3 since Network Level Authentication is always enabled by default.

**Answer: Administrator**

---

**3. What was the Logon ID of the malicious RDP login? (Logon Type 10)**

<p align="center">
<img src=screenshots/windows_logontype10.png width="800">
</p>

This was also a 4624 Event ID with a Logon Type 10 which indicates a RDP login. Logon Type 10 could also indicate older or misconfigured systems.

**Answer: 0x183C36D**

---

## Task 4 - Security Log: User Management

Being able to spot suspicious Event IDs on logs is the first step to recognizing a possible attack!

For Example: Event ID 4720 indicates: User Created an Account. This coming after multiple 4625 which are failed logins followed by a successful logon 4624 could indicate a backdoor creation.

Attackers will often add a user that they exploited to a priviliged group like "Administrator" in order to establish a backdoor connection.
  - 4732 User was added
  - 4733 User removed from a security group

**Structure of User Management Events**
  - Subject
  - Object
  - Details

Many real world breaches involve some sort of user manipulation: create, delete, reset accounts. 

| Event ID | Description | Malicious Usage |
|---|---|---|
| 4720 / 4722 / 4738 | User account created / enabled / changed | Create backdoor accounts or enable dormant ones to avoid detection |
| 4725 / 4726 | User account disabled / deleted | Disable privileged SOC accounts to slow down response |
| 4723 / 4724 | User changed password / password was reset | Reset credentials to gain access to a required account |
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

The Event ID for created accounts is 4720, I narrowed it down by such and we can note the Administrator account created a new user account.

**Answer: svc_sysrestore**

---

**2. Which two privileged groups was the backdoor user added to? (alphabetical order)**

**Answer:**

---

**3. Does the Logon ID field match what you saw in the previous task? (Yea/Nay)**

**Answer:**

---

## Task 5 - Sysmon: Process Monitoring

<!--
ACTIVE RECALL
**1.** What is the difference between Event ID 4688 and Sysmon Event ID 1?
**2.** Why is Sysmon preferred over 4688 for process monitoring?
**3.** Where in Event Viewer do Sysmon logs appear?
**4.** What are the four field groups in Sysmon Event ID 1 and what does each contain?
**5.** What is the Logon ID field used for in Sysmon Event ID 1?
**6.** Why is process monitoring considered the most important log source for SOC teams?
-->

| Event Code | Purpose | Limitations |
|---|---|---|
| 4688 (Security Log) | Logs every new process launch including command line and parent process | Disabled by default -- must be enabled manually |
| 1 (Sysmon) | Replaces 4688 with advanced fields: process hash, signature, and more | External tool -- not installed by default |

**Sysmon Event ID 1 Key Fields**

| Field Group | Fields |
|---|---|
| Process Info | PID, image path, command line |
| Parent Info | Parent process context -- used to build process tree and attack chain |
| Binary Info | Process hash, signature, PE metadata |
| User Context | User running the process and Logon ID (correlates with Security logs) |

**1. Which web browser does Sarah use to browse the web?**

**Answer:**

---

**2. Which file did Sarah download from the browser?**

**Answer:**

---

**3. Which URL was the file downloaded from?**

**Answer:**

---

## Task 6 - Sysmon: Files and Network

<!--
ACTIVE RECALL
**1.** What do Sysmon Event IDs 11 and 13 detect and what are their Security log alternatives?
**2.** What do Sysmon Event IDs 3 and 22 detect and why is there no direct Security log alternative?
**3.** What field do all Sysmon events share that allows correlation back to Event ID 1?
**4.** Why are network and registry logs important beyond just process creation events?
**5.** What is Florian's config and why is it commonly used?
**6.** How would you use Event ID 3 to identify C2 communication?
-->

| Event ID | Security Log Alternative | Purpose |
|---|---|---|
| 11 (File Create) | 4656 -- disabled by default | Detect files dropped by malware |
| 13 (Registry Value Set) | 4657 -- disabled by default | Detect registry changes used for persistence |
| 3 (Network Connection) | No direct alternative | Detect traffic from untrusted processes |
| 22 (DNS Query) | No direct alternative | Detect connections to known malicious destinations |

**Correlation Note:** All Sysmon events share `ProcessId` -- use it to find the corresponding Event ID 1 to get full process context including parent info and Logon ID.

**1. Which file was created by the downloaded malware to persist on the host?**

**Answer:**

---

**2. What is the Command and Control server the malware connected to? (Format: IP:Port)**

**Answer:**

---

**3. Which domain does the malicious IP correspond to?**

**Answer:**

---

## Task 7 - PowerShell: Logging Commands

<!--
ACTIVE RECALL
**1.** Why does Sysmon Event ID 1 fail to capture PowerShell activity effectively?
**2.** Where is the PowerShell history file located and what does it record?
**3.** What are the key limitations of the PowerShell history file?
**4.** Does the history file survive reboots? Does it log command outputs?
**5.** How many history files can exist on a single system and why?
**6.** What other PowerShell logging methods exist beyond the history file?
-->

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

**Answer:**

---

**2. When did the Administrator run the first PS command? (Format: April 18, 2025)**

**Answer:**

---

**3. Can you find the flag stored in the PowerShell history? (Format: THM{...})**

**Answer:**

---

## Task 8 - Conclusion

<!--
ACTIVE RECALL
**1.** What are the four key takeaways from this room?
**2.** How do Logon ID and Process ID help you reconstruct an attack chain?
**3.** Which log source gives you the most context about endpoint activity?
**4.** Why is PowerShell logging increasingly important in modern SOC work?
-->

**1. Hope you enjoyed the room!**

**Answer:** N/A

---
