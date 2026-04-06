---
title: Introduction to EDR
module: 02_Core-SOC-Solutions
platform: TryHackMe
path: SOC Level 1 > Core SOC Solutions > Introduction to EDR
tags:
  - soc-level-1
  - edr
  - endpoint-security
  - detection
  - response
  - telemetry
  - crowdstrike
  - mitre
status: in-progress
date_started: 2026-04-06
date_completed: 2026-04-06
---
# Introduction to EDR

---

## Task 1 - Introduction

### Key Concepts

<!-- What is EDR in one or two sentences in your own words? -->
Endpoint Detection and Response is a security tool that continuously monitors endpoints.
<!-- Why does a SOC analyst need to understand EDR specifically? -->
EDR solutions: Identify, Analyze and Responds to cyberthreats
<!-- What are the three things this room promises to cover? -->
What can EDR do?
How does  it detect?
How does it respond to threats?

### Task Questions

1. I am all set!
   **A:** No answer needed

---

## Task 2 - What is an EDR?

### Key Concepts

<!-- Why did the rise of remote work make endpoint protection more urgent? -->
Since the rise of remote work, company hardware and information is no longer safe and contained, it's out and about in the world. That's where EDR comes in, so these devices no matter their endpoint can stay protected.
<!-- What gap does EDR fill that perimeter-based security cannot? -->
Perimeter based means inside an area, contained, that means inside the company's walls or within it's range. Out of that perimeter is the world and thats where EDR comes in, these devices that are out of perimeter are monitored constantly by EDR.
<!-- Name the three pillars of EDR and explain each one briefly in your own words: -->
The 3 Pillars of EDR 
- Visibility
- Detection
- Response

**Visibility**
<!-- What does visibility mean in the context of EDR? What data does it collect? What can an analyst see that they couldn't see before? -->
EDR collects detailed information from it's endpoints and presents it to SOC analysts on a very detailed format. It shows the entire **process tree** with a complete timeline and sequence of events
- Process modifications
- Registry modification
- File and folder modification
- User actions and much more

![T2 Crowdstrike process tree visibility](screenshots/T2_crowdstike_visibility.png)

**Detection**
<!-- How does EDR detection go beyond signature matching? What makes it catch things AV misses? -->
One of the key features of EDR is its ability to detect any anomaly in baseline behavior and instantly alert to it. In other words: is this normal behavior for this user and this machine?
- Allows SOC to input custom **indicators of compromise** (IOCs) to improve threat detection
- **Tactic via Technique** provides a rich details of the detection following the MITRE framework
- **MITRE framework** is a structured knowledge base of **known attacker behaviors**, including tactics, techniques and procedures used in cyber attacks
![T2 Crowdstrike detection dashboard](screenshots/T2_crowdstrike_detection.png)

**Response**
<!-- What can an analyst actually do from the EDR console? List the response options and what situation each one fits. -->
EDR not only serves to detect threats but also remediate them 
SOCs have the ability to control the endpoint:
- Isolate it
- Terminate a process 
- Quarantine some files

**This screenshot shows the available actions on the response dashboard**
![T2 Crowdstrike response dashboard](screenshots/T2-crowdstrike_response.png)

<!-- Important limitation: what is the one thing EDR cannot see? -->
It is **important** to know that EDR is a host only security tool  and it does **not** detect network-level threats
- **EDR hots-based** security
- **Firewall network-based** security
### Task Questions

1. Which feature of EDR provides a complete context for all the detections?
   **A: Visibility**

2. Which process spawned sc.exe?
   **A: cmd.exe**


---

## Task 3 - Beyond the Antivirus

### Key Concepts

**The EDR and AV airport concept**

![T3 EDR and AV airport concept](screenshots/T3_edr_av_airport.png)

EDR & Antivirus why we need both
- If the threat has no "signature" it may bypass the antivirus easily
- Because EDR is 24/7 monitoring and behavioral based, anything that has bypassed the AV but is abnormal will get flagged by the EDR

![T3 How AV and EDR views processes](screenshots/T3_edr_and_av.png)

| Attack Step | What Happened                                        | AV Response                                                                                           | EDR Response                                                                                                                           |
| ----------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Step 1      | Phishing email with malicious Word doc received      | Does the downloaded file have a previous signature in the database? <br><br>Yes: Alert<br>NO: Nothing | Logs the downloaded file and monitors it's activity                                                                                     |
| Step 2      | User opens the document                              | Does **nothing** since winword.exe is a legitimate utility process                                    | Records the process execution of winword.exe and **continues** monitoring it                                                           |
| Step 3      | Macro silently spawns PowerShell                     | Does **nothing** if the executed macro has no previous signature                                      | **Detects and flags** the macro execution due to the unusual parent-child relationship of **winword.exe and PowerShell.exe** processes |
| Step 4      | Obfuscated PowerShell downloads second-stage payload | AV does **not** usually detects obfuscated PowerShell scripts                                         | **Flags** the obfuscated script execution                                                                                              |
| Step 5      | Payload injected into legitimate svchost.exe         | Does **not** monitor memory injections                                                                | **Detects Process Injection** in svchost.exe                                                                                           |
| Step 6      | Attacker gains remote access                         | **No** network visbility                                                                              | **Flags unexpected behavior** of the svchost.exe making an outbound connection                                                         |
| Final       | Alert generated or missed                            | May be marked as clean                                                                                | **Full Alert** is generated complete with the entire **attack chain** and enables the SOC analyst to take immediate action             |

<!-- What does this scenario teach you about why process relationships matter so much in detection? -->

### Task Questions

1. In the given analogy, what presents an AV?
   **A: Immigration Check**

2. Which legitimate process was hijacked by the attacker in the scenario?
   **A: svchost.exe**

3. Which security solution might mark this activity as clean?
   **A: Antivirus**

---

## Task 4 - How an EDR Works

### Key Concepts

<!-- Two components make up an EDR solution. Describe each one and what it is responsible for: --> Multiple endpoints can be managed through a centralized console. 
- Agents("sensors") are deployed inside those endpoints and they monitor all activity
- The information about these activities is then sent to the EDR central console in real-time
- EDR agents can perform basic signature-based and behavior-based detection 

![T4 EDR agents watch the EDR console](screenshots/T4_edr_agents_console.png)

**Console:**
<!-- What does it do with the data the agent sends? What does "correlating" data mean in this context? -->
All data is sent to the EDR console which analyzes it through **complex logic and machine learning algorithms**
- EDR is much like a brain connecting all the information together like dots
- When these dots connect to form a **detection that becomes an Alert**
- **EDR Agents** collect different data form their endpoints and push it to **EDR console**

A SOC analysts first job is to **acknowledge and prioritize the alert**
- EDR console makes this easier by sorting the alerts by **severity**
- The alert will contain **all** the details of the detection

The EDR is one of the many security tools that form a bigger **security ecosystem** which includes:
- Firewalls
- DLPs
- Email Security Gateways
- IAMs
- EDRs

All these security tools come-together in the **Security Information and Event Management**(SIEM) system

### Task Questions

1. Which component of the EDR is responsible for collecting telemetry from the endpoints?
   **A: Agent**

2. An EDR agent is also known as a?
   **A: Sensor**

---

## Task 5 - EDR Telemetry

### Key Concepts

<!-- What is telemetry? Why does more telemetry = better detection? -->
The agents collect data and send it to the EDR central console. This data is  called **telemetry**
- Telemetry is the **black box** of an endpoint

| Telemetry Type                      | What It Monitors                           | Why It Matters for Detection                                                                     |
| ----------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Process Executions and Terminations | All running and idle **proccesses**        | Key for identifying **child-parent process** relationships                                       |
| Network Connections                 | All endpoint **network connections**       | - Connection to a C2 server<br>- unusual port usage<br>- data exfiltration<br>- lateral movement |
| Command Line Activity               | All **executed commands** on the endpoints | Identify malicious command execution<br>- obfuscated PowerShell script executions                |
| Files and Folders Modifications     | Malicious file movements                   | - data staging<br>- malicious file dropping<br>- ransomware executions                           |
| Registry Modifications              | Windows system registry                    | It contains vital configuration information                                                      |

<!-- Advanced threats try to stay stealthy by using legitimate tools. Why does detailed telemetry expose them even when individual actions look innocent? -->

### Task Questions

1. Which telemetry data helps in detecting C2 communications?
   **A: Network Connections**

2. Where are the configuration settings of a Windows system primarily stored?
   **A: Registry**

---

## Task 6 - Detection and Response Capabilities

### Key Concepts

Once the **telemetry data** is gathered the EDR applies some advanced techniques to this data

![159](screenshots/T5_edr_detection_response.png)


**Behavioral Detection:**
It does not just observe file signatures it completely analyzes the behavior of a file. If something is acting out of the normal for example a word document spawning a PowerShell;

winword.exe ➜ PowerShell.exe

**Anomaly Detection:**
With use EDR learns it's device baseline behavior, any activity that is out of the **normal or expected** usage will be flagged
- It can pick up "noise" or false positives but since it offers the analyst the **full context** of the detection, analysts are able to rule it out
- An anomaly example: A process modifies an auto-registry key

**IOC Matching:**
EDR flags any activity matching a known **Indicator of Compromise** or IOC
- EDR uses intelligence field integration which uses indicators that are published in the threat intelligence feeds
- Example: User downloaded file contains a hash of a file commonly used in attacks, it's in the database the EDR will flag it immediately
- If it were a **zero day attack** the EDR would not have any intelligence on it

**MITRE ATT&CK Mapping:**
EDR not only detects the malicious activity it also maps it to it's corresponding **MITRE ATT&CK tactic and techinique**. This provides the analyst with:
- The **stage** of the attack
**Example:** If an EDR detects the creation of a scheduled task, it may map the activity as:
- **Tactic:** Persistence
- **Technique:** Scheduled Task/Job

**Machine Learning:**
Modern EDRs have machine learning models which are trained by large datasets of normal and malicious behavior
- Fliless attacks and multi stage intrusion which may appear harmless can be detected through machine learning 

<!-- What artefacts can be pulled without physically touching the device? List them. -->

### MITRE ATT&CK Mapping

| Technique Observed | Tactic | Technique ID | Notes |
|---|---|---|---|
| Scheduled task creation | Persistence | T1053 | Example from room |
| | | | |
| | | | |

### Task Questions

1. Which feature of the EDR helps you identify threats based on known malicious behaviours?
   **A: IOC Matching**

---

## Task 7 - Investigate an Alert on EDR

### Key Concepts

Triage Alerts of varying severity on the EDR console

![T7 EDR Dashbnoard](screenshots/T7_edr_dashboard_analysis.png)

### Task Questions

1. Which tool was launched by CMD.exe to download the payload on DESKTOP-HR01?

![T7 Q1 Tool Launch Process](screenshots/T7_q1_initial_access.png)
 **A: cURL.EXE**

2. What is the absolute path to the downloaded malware on the DESKTOP-HR01 machine?

![T7 Q2 File Path](screenshots/T7_q2_absolute_path.png)
 **A: C:\Users\Public\install.exe**

3. What is the absolute path to the suspicious syncsvc.exe on the WIN-ENG-LAPTOP03 machine?

![T7 Q3 Second suspicious Path](screenshots/T7_q3_second_path.png)
   **A: C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe**

4. On which URL was the exfiltration attempt being made on WIN-ENG-LAPTOP03?
![T7 Q4 Which URL is attempting Exfil](screenshots/T7_q4_exfil_url.png)
   **A: https://files-wetransfer.com/upload/session/ab12cd34ef56/dump_2025.dmp **

5. What was UpdateAgent.exe labelled by Threat Intel on DESKTOP-DEV01?
![T7 Q5 Threat Intel Dashboard](T7_q5_threat_intel.png)
   **A: Known internal IT utility tool**

---
## What I Learned

I learned how EDR makes just one part of LARGE security ecosystem, and that various tools used together are needed to keep attackers out.

---

## What Confused Me and How I Resolved It

That EDR is host based, meaning machine or device based. A Firewall is network based its watching traffic thats coming and going. EDR wont know that an attacker might be scanning for open ports.

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*