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

<!--
ACTIVE RECALL
**1.** What are the four learning objectives of this room?
**2.** What four rooms are recommended as prerequisites?
**3.** What credentials are used to access the VM via RDP?
**4.** Where are Windows event logs stored on disk and in what format?
-->

**1. I'm ready to move on!**

**Answer:** N/A

---

## Task 2 - What Is Logged

<!--
ACTIVE RECALL
**1.** What three SOC activities do logs support?
**2.** Where are Windows event logs stored on disk and in what format?
**3.** What does each EVTX file correspond to?
**4.** What are the four main components of the Event Viewer interface?
**5.** What is an Event ID and why does it matter for detection?
**6.** Why does Windows require more knowledge than other OSes to read and interpret its logs?
-->

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

**Answer:**

---

## Task 3 - Security Log: Authentication

<!--
ACTIVE RECALL
**1.** What is Event ID 4624 and on which machine is it logged?
**2.** What is Event ID 4625 and what attacks does it help detect?
**3.** What are the key fields to review in a 4624 event?
**4.** What is Logon Type 10 and what does it indicate?
**5.** What is the Logon ID field used for and why is it important for correlation?
**6.** What makes 4624 noisy and 4625 inconsistent?
-->

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

**Answer:**

---

**2. Which user has been breached as a result of the attack?**

**Answer:**

---

**3. What was the Logon ID of the malicious RDP login? (Logon Type 10)**

**Answer:**

---

## Task 4 - Security Log: User Management

<!--
ACTIVE RECALL
**1.** What does Event ID 4720 indicate and how can attackers abuse it?
**2.** What do Event IDs 4732 and 4733 track and why are they critical to monitor?
**3.** What three parts make up the structure of user management events?
**4.** How do you use the Logon ID field to correlate a user management event with a login event?
**5.** What privileged group do attackers commonly add backdoor accounts to?
**6.** Why might a threat actor disable a privileged account (4725)?
-->

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

**Answer:**

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
