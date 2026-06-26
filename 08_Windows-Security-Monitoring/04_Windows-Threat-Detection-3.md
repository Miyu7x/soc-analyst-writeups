---
title: Windows Threat Detection 3
module: Windows Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [windows, threat-detection, c2, persistence, sysmon, event-logs, blue-team, soc-level-1]
status: completed
date: 06-26-2026
date_completed: 06-26-2026
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/windows_intro3.png width="1500">
</p>

---

## Task 1 - Introduction

This lab focuses on:

   - Command and Control (C2)

   - Persistence

   - Impact

   - Why threat actors maintain control of their victims

   - Windows logs to investigate persistence

---

## Task 2 - Command and Control

**MITRE Tactic: Command and Control (C2) TA0011**

  - How do threat actors send commands and maintain control of a victim host?

**Attacks Without C2**

  - Threat actors can type commands directly in an active RDP session
  - RDP becomes unavailable once the session is closed
      - Threat actors are known to set up C2 as soon as they breach
<p align="center">
<img src=screenshots/windows_attackwithoutc2.png width="700">
</p>

**Simplest C2**

  - RDP is not always available

  - Threat actors establish a Command and Control channel so the victim's system can "phone home"
      - In advanced cases, the attachment won't immediately connect back; instead, it downloads additional C2 malware
          - They'll hide it in a folder like C:\Temp and run it as a new stealthy process window
<p align="center">
<img src=screenshots/windows_simplestc2.png width="700">
</p>

---

**1. Which suspicious archive did the user download?**

<p align="center">
<img src=screenshots/windows_urgent.png width="700">
</p>
In Event Viewer at 7/2/2025 8:13:55 PM, Event ID 15 (File Stream Created), we observe TargetFilename: C:\Users\Administrator\Downloads\URGENT!.zip

- **Answer: URGENT!.zip**

---

**2. Where did the attackers hide the C2 malware file?**

<p align="center">
<img src=screenshots/windows_hiddenmalware.png width="700">
</p>
Following the timeline, we observe the attacker extract the zip with explorer.exe and the extraction of the .lnk file.
The .lnk file triggers PowerShell, which drops update.exe into C:\Users\Administrator\AppData\Roaming\

- **Answer: C:\Users\Administrator\AppData\Roaming\update.exe**

---

**3. What is the domain of the Command and Control server?**

<p align="center">
<img src=screenshots/windows_domain3.png width="700">
</p>
Continued following the timeline until a DNS query event appeared.

- **Answer: route.m365officesync.workers.dev**

---

## Task 3 - Persistence Overview

Most **infections** last only minutes -- attackers quickly:
  - breach the victim
  - collect the data
  - exfiltrate and exit

However, maintaining access to the victim for days or even months is crucial depending on the attacker's end goal.
  - Reliable access can survive reboots and password changes

**Persistence** MITRE ID: TA0003
<p align="center">
<img src=screenshots/windows_persistence.png width="700">
</p>

**Persisting via RDP**

Many Windows breaches happen due to exposed RDP. Once in, attackers can:
  - Create a backdoor or a web shell
  - Create a new user (MITRE ID T1136)
  - Elevate that user to Administrator (T1098)

**Example**
<p align="center">
<img src=screenshots/windows_userexample.png width="700">
</p>

**Detecting Backdoor Users**

Every **User Created** event is logged as Security Event ID **4720**.

Attackers are creative when naming backdoor accounts -- they won't call it "hacker". It's important to investigate:
  - Who created the account
  - What is the source IP and time of creation?
  - Which other suspicious events surround it?

Attackers will add these users to **Privileged Groups**. This change is tracked by Security Event ID **4732**:
  - Administrators
  - Remote Desktop Users

In more advanced cases, attackers reset passwords on old or unused accounts; Security Event ID **4724**

**Examples**
<p align="center">
<img src=screenshots/windows_persistenceexample.png width="700">
</p>

| Event ID | Log Source | What It Detects | Key Fields to Investigate |
|---|---|---|---|
| 4720 | Security | New user account created | Creator account, time of creation, source IP of creator's login session |
| 4732 | Security | User added to a privileged local group | Target group name (Administrators, Remote Desktop Users), who performed the action |
| 4724 | Security | Password reset on an existing account | Target account name (old/unused accounts are a red flag), who initiated the reset |

**Note** Logon Types:
  - 2: Interactive (physical keyboard)
  - 10: RemoteInteractive (RDP session)

---

**1. How many times did the threat actor fail to log in to the Administrator account?**

<p align="center">
<img src=screenshots/windows_failedlogon.png width="700">
</p>
Inspected the logs for failed logon events (Event ID **4625**) and counted the occurrences.

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

Persistence through a backdoor requires an RDP login, so if an attack started via USB or phishing email, that is simply not possible.

 - Attackers need **actively running** malware that maintains a connection with their C2 server
    - This malware must stay active even after a system reboot
    - Threat actors use **Windows Services and Scheduled Tasks** to keep malware running
       - There are **hundreds of methods** to persist on a Windows machine

**Two Common Persistence Methods**

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| Create a Windows Service (runs after OS startup) | `sc create "BadService" binpath= "C:\malware.exe" start= auto` | Launch of sc.exe: Sysmon EID 1 / Service creation: Security EID 4697 |
| Create a Scheduled Task (runs after OS startup) | `schtasks /create /tn "BadTask" /tr "C:\malware.exe" /sc onstart /ru System` | Launch of schtasks.exe: Sysmon EID 1 / Task creation: Security EID 4698 |

**Detecting Services**

Critical Windows components include:
 - DNS Client
 - Security Center
    - View services: **services.msc**
    - Administrative privileges and the **sc.exe** command are required to create or modify a service

Threat actors will create their own services that run on startup. You can detect them a few different ways:

   1. Detect launch of the command: **sc.exe create**
   2. Service Creation: Security Event ID **4697** or System Event ID **7045**
   3. Processes are suspicious when their **Parent Process** is: **services.exe**

**Example**
<p align="center">
<img src=screenshots/windows_servicesexample.png width="700">
</p>

**Detecting Tasks**

Scheduled Tasks is a heavily used Windows feature -- the OS and external apps rely on it, which makes it a great place to hide malware.

 - Threat actors prefer scheduled tasks because they are easy to configure and blend in

 - Manage Tasks: **taskschd.msc** or **schtasks.exe**

How to detect suspicious scheduled tasks:

 1. Detect the launch of **schtasks.exe /create** (Sysmon Event ID 1)
 2. Analyze suspicious Task Creation events in Security Event ID **4698**
 3. Look for suspicious processes with **Parent Process**: **svchost.exe [...] -s Schedule**

**Example**
<p align="center">
<img src=screenshots/windows_scheduleexamples.png width="700">
</p>

---

**1. Which Windows service was created to persist the Nessie malware?**

<p align="center">
<img src=screenshots/windows_dps.png width="700">
</p>
Filtered for Event ID 4697 and reviewed the General information. We spot nessie.exe with Service Name: Data Protection Service.

- **Answer: Data Protection Service**

---

**2. Which scheduled task was created to persist the Troy malware?**

<p align="center">
<img src=screenshots/windows_amazonsync.png width="700">
</p>
Searched the logs for Event ID 4698 (Scheduled Task Created) and reviewed the General information for each event. Inside the task XML we spotted:
`<Command>"C:\Program Files\Common Files\troy.exe"</Command>`
This confirmed the task was created by the Troy malware.

- **Answer: AmazonSync**

---

**3. What flag do you get after finding and running the Troy malware?**

<p align="center">
<img src=screenshots/windows_c2schedule.png width="700">
</p>
Ran the command `& "C:\Program Files\Common Files\troy.exe"` and received the following prompt:

```
=========================================
Not so fast! What was my parent commandline?
Example: C:\Windows\System32\os.exe -run
=========================================
```

Inspected the post-reboot logs and found various svchost.exe processes. The suspicious one was:
`CommandLine: C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`

- **Answer: THM{c2_is_on_schedule!}**

---

## Task 5 - Persistence: Run Keys and Startup

**Run Keys and Startup**

When a system boots, **Services and Scheduled Tasks** typically run -- but they require **Administrative Privileges** to configure.

What happens when malware needs to run only when a specific user logs in?

**Examples of Per-User Persistence**

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| Add malware to Startup Folder (runs upon user login) | `copy C:\malware.exe "%AppData%\Microsoft\Windows\Start Menu\Programs\Startup\malware.exe"` | New startup item: Sysmon EID 11 |
| Add malware to "Run" keys (runs upon user login) | `reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v BadKey /t REG_SZ /d "C:\malware.exe"` | New registry value: Sysmon EID 13 |

**Detecting Startup**

The Startup folder lets users launch applications automatically on login -- simply drop the executable in and it runs.

**Startup Folder Paths**
```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
Or for all users: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

Programs launched from Startup will have a **Parent Process** of: **explorer.exe**
  - This makes it harder to differentiate from legitimate user activity
  - Startup items are logged as Sysmon Event ID 11 (File Created)

**Example**
<p align="center">
<img src=screenshots/windows_startup_example.png width="700">
</p>

**Detecting Run Keys**

Unlike Startup, Run Keys require a new value to be written to the Windows Registry pointing to the malware path.

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
Or for all users: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
```

**Detecting Run Key Entries**

 - Check **regedit.exe**
 - Registry change events: Sysmon Event ID 13

**Example**
<p align="center">
<img src=screenshots/windows_runkey.png width="700">
</p>

---

**1. What is the parent process image of the "Odin" malware?**

<p align="center">
<img src=screenshots/windows_odin.png width="700">
</p>
Filtered the logs for Sysmon Event ID 11 (File Created) and located the new startup item:
`TargetFilename: C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\odin.cmd`

Since malware launched from the Startup folder always has explorer.exe as its parent, the answer follows directly.

- **Answer: C:\Windows\explorer.exe**

---

**2. What is the last line that the "Odin" malware outputs?**

<p align="center">
<img src=screenshots/windows_badstuff.png width="700">
</p>
Ran the target file: `& "C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\odin.cmd"`

- **Answer: Done doing bad stuff!**

---

**3. What flag do you get after finding and running the "Kitten" malware?**

<p align="center">
<img src=screenshots/windows_basket.png width="700">
</p>
Rather than searching logs individually, I opted to search the C:\ drive directly for the file, bypassing permission errors on protected folders:
`Get-ChildItem C:\ -Recurse -Filter "*kitten*" -ErrorAction SilentlyContinue`

Located the file at `C:\Users\Public\kitten.exe` and attempted to run it:

```
=========================================
Not so fast! How is my Run key named?
Example: WinUpdate
=========================================
```

Since this is a Run Key-launched malware, I checked the Registry Editor at:
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`

- **Answer: THM{persisting_in_basket!}**

---

## Task 6 - Impact and Threat Detection Recap

**Need for Persistence**

Why don't attackers just breach a system, grab what they need, and leave? There are several possible reasons:

 - Add the breached host to a botnet and use it for future attacks
 - Spy on the host as part of a state-sponsored espionage campaign
 - Use the host as an entry point into a much larger network

**Active Directory and Ransomware**

Large networks are prime targets because they run Active Directory, giving attackers access to every connected system.
 - These attacks are often ransomware-motivated
 - Hospitals are high-value targets due to the volume of sensitive patient data they hold

**Example of Ransom Letter Sent to a Health Organization**
<p align="center">
<img src=screenshots/windows_ransomware.png width="700">
</p>

**MITRE Threat Detection Recap**
<p align="center">
<img src=screenshots/windows_mitrerecap.png width="700">
</p>

---

**1. What is the biggest threat to most corporate Windows networks?**

- **Answer: Ransomware**

---

**2. At which stage is it best to detect and stop the attack (e.g. Exfiltration)?**

- **Answer: Initial Access**

---

## Task 7 - Conclusion

<!-- What three MITRE tactics did this room cover? Write a one-sentence summary of each in your own words. -->

<!-- How does this room fit into the full Windows Threat Detection arc (rooms 1, 2, 3)? What attack chain stages does each room address? -->

**1. Complete the room!**

- **Answer:** N/A
