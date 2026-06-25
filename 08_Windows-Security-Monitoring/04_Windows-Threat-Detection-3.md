---
title: Windows Threat Detection 3
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, threat-detection, c2, persistence, sysmon, event-logs, blue-team, soc-level-1]
status: in-progress
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/windows_intro3.png width="1500">
</p>

---

## Task 1 - Introduction

 This lab focuses on:

   - Command and Control C2

   - Persistance

   - Impact

   - Why threat actors maintain control of their victims

   - Windows logs to investigate persistence


---

## Task 2 - Command and Control

**MITRE tactic Command and Control C2 TA0011**

  - How do threat actors send commands and maintain control of a victim host

**Attacks Without C2**

  - Threat actors can type commands directly in an active RDP session
  - RDP becomes unavailable once the session is closed
      - Threat actors are known to set up C2 as soon as they breach
<p align="center">
<img src=screenshots/windows_intro3.png width="700">
</p>

**Simplest C2**

  - RDP is not available every time

  - Threat actors establish a Command and Control channel so the victim's system can "phone home"
      - In advanced cases, the attachment won't immediately connect back; instead, it downloads additional C2 malware
          - They'll hide it in a folder like C:\Temp and run it as new stealthy process window
<p align="center">
<img src=screenshots/windows_simplestc2.png width="700">
</p>

---

**1. Which suspicious archive did the user download?**

<p align="center">
<img src=screenshots/windows_urgent.png width="700">
</p>
In the Event Viewer at 7/2/2025 8:13::55 PM, Event ID 15 File Stream Created. We observe TargetFilename: C:\Users\Administrator\Downloads\URGENT!.zip

- **Answer: URGENT!.zip**

---

**2. Where did the attackers hide the C2 malware file?**

<p align="center">
<img src=screenshots/windows_hiddenmalware.png width="700">
</p>
Following the timeline we observe the attacker extract the zip with explorer.exe we observe the extraction of the .lnk file. 
The .lnk file triggers Powershell, Powershell drops the update.exe into C:\Users\Administrator\AppData\Roaming\

- **Answer: C:\Users\Administrator\AppData\Roaming\update.exe**

---

**3. What is the domain of the Command and Control server?**

<p align="center">
<img src=screenshots/windows_domain3.png width="700">
</p>
Continued following the timeline till we can see a DNS query.

- **Answer: route.m365officesync.workers.dev**

---

## Task 3 - Persistence Overview

Most **Infections** lasts several minutes, attackers quickly;
  - breach the victim
  - collect the data
  - exfiltrate and EXIT

However, maintaining access to the victim for days or even months is crucial, depending on the attacker's end GOAL.
  - reliable access can survive reboots and password changes

**Persistence** MITRE ID: TA0003
<p align="center">
<img src=screenshots/windows_persistence.png width="700">
</p>

**Persisting via RDP**

Many Windows breaches happen due to exposed RDP; once in attackers can:
  - create a backdoor or a webshell
  - create a new user MITRE ID T1136
  - make it an Administrator T1098

**Example**
<p align="center">
<img src=screenshots/windows_userexample.png width="700">
</p>

**Detecting Backdoor Users**

Every **User Created** event is logged as Security event ID **4720**

Attackers are creative when creating **user names**, they're not going to be naming it: hacker. Its important to investigate:
  - Who created the account
  - What is the Source IP, and time created?
  - Which other suspicious events can you see?

Attackers will add these users to **Privileged Groups**. This change is tracked by Security Event ID **4732**
  - Administrators
  - Remote Desktop Users

More advanced cases, attackers will reset passwords to old or unused accounts; Security Event ID **4724**

**Examples**
<p align="center">
<img src=screenshots/windows_persistenceexample.png width="700">
</p>


| Event ID | Log Source | What It Detects | Key Fields to Investigate |
|---|---|---|---|
| 4720 | Security | New user account created | Creator account, time of creation, source IP of creator's login session |
| 4732 | Security | User added to a privileged local group | Target group name (Administrators, Remote Desktop Users), who performed the action |
| 4724 | Security | Password reset on an existing account | Target account name (old/unused accounts are a red flag), who initiated the reset |

**Note** Logon Type:
  - 2 Interactive, physical keyboard
  - 10 RemoteInteractive, whos RPD'd in

---

**1. How many times did the threat actor fail to log in to the Administrator?**

<p align="center">
<img src=screenshots/windows_failedlogon.png width="700">
</p>
Inpected the logs for failed logons Event ID **4625** and counted how many were present

- **Answer: 6**

---

**2. After the successful login, which backdoor user did the attacker create?**

<p align="center">
<img src=screenshots/windows_support.png width="700">
</p>

- **Answer: support**

---

**3. Which privileged group was the backdoor user added to?**

- **Answer: Administrator**

---

## Task 4 - Persistence: Tasks and Services

**Malware Persistence**

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| Create a Windows Service (runs after OS startup) | `sc create "BadService" binpath= "C:\malware.exe" start= auto` | Launch of sc.exe: Sysmon EID 1 / Service creation: Security EID 4697 |
| Create a Scheduled Task (runs after OS startup) | `schtasks /create /tn "BadTask" /tr "C:\malware.exe" /sc onstart /ru System` | Launch of schtasks.exe: Sysmon EID 1 / Task creation: Security EID 4698 |

**1. Which Windows service was created to persist the Nessie malware?**

- **Answer:**

---

**2. Which scheduled task was created to persist the Troy malware?**

- **Answer:**

---

**3. What flag do you get after finding and running the Troy malware?**

- **Answer:**

---

## Task 5 - Persistence: Run Keys and Startup

<!-- What is the functional difference between Startup Folder and Run Key persistence? What triggers each one -- boot vs. user login? -->

<!-- Why does Startup Folder persistence produce an explorer.exe parent, and why does that make it harder to differentiate from legitimate user activity? -->

<!-- What is the shared MITRE technique ID for both methods, and why are they grouped together? -->

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| Add malware to Startup Folder (runs upon user login) | `copy C:\malware.exe "%AppData%\Microsoft\Windows\Start Menu\Programs\Startup\malware.exe"` | New startup item: Sysmon EID 11 |
| Add malware to "Run" keys (runs upon user login) | `reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v BadKey /t REG_SZ /d "C:\malware.exe"` | New registry value: Sysmon EID 13 |

**1. What is the parent process image of the "Odin" malware?**

- **Answer:**

---

**2. What is the last line that the "Odin" malware outputs?**

- **Answer:**

---

**3. What flag do you get after finding and running the "Kitten" malware?**

- **Answer:**

---

## Task 6 - Impact and Threat Detection Recap

<!-- What are the three main reasons threat actors use Persistence -- botnet, espionage, lateral movement? Summarize each with the real-world example given in the room. -->

<!-- What is ransomware and why is it the biggest threat to corporate Windows networks? What does it do beyond encrypting files? -->

<!-- At which attack stage is detection most valuable and why? What is the relationship between early detection and preventing ransomware Impact? -->

**1. What is the biggest threat to most corporate Windows networks?**

- **Answer:**

---

**2. At which stage is it best to detect and stop the attack (e.g. Exfiltration)?**

- **Answer:**

---

## Task 7 - Conclusion

<!-- What three MITRE tactics did this room cover? Write a one-sentence summary of each in your own words. -->

<!-- How does this room fit into the full Windows Threat Detection arc (rooms 1, 2, 3)? What attack chain stages does each room address? -->

**1. Complete the room!**

- **Answer:** N/A
