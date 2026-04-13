---
title: Introduction to SIEM
module: SOC Level 1
platform: TryHackMe
tags:
  - siem
  - log-analysis
  - blue-team
  - soc
  - thm
  - soc-level-1
status: in-progress
date: 2026-04-07
date completed: 2026-04-07
room_url: https://tryhackme.com/room/introtosiem
---
# Introduction to SIEM

## Task 1 - Introduction

### Key Concepts

**SIEM** stands for Security Information and Event Management. It is the core security solution used by SOC analysts inside a security operations center. At its most fundamental level, SIEM is a centralized platform that collects logs from across a network, normalizes them into a consistent format, and correlates them to detect malicious activity.

**Learning Objectives**
- Understand the different types of log sources and where they originate
- Identify the limitations of analyzing isolated, decentralized logs
- Recognize why a centralized solution is necessary
- Explore the core features a SIEM provides
- Understand how logs are ingested and how alerting works

### Task Questions

1. What does SIEM stand for?

   **Answer: Security Information and Event Management**

---

## Task 2 - Logs Everywhere, Answers Nowhere

### Key Concepts

Every device in a network generates logs continuously. Think of any home or enterprise network: laptops, servers, firewalls, routers all running at the same time, each producing a record of every action. That volume alone creates a problem. Now multiply it across an enterprise with hundreds of endpoints, and the challenge becomes clear.

These logs fall into two categories based on where the activity originates.

| Log Source Category | Example Devices | Example Log Events |
|---|---|---|
| Host-Centric | Windows endpoints, Linux servers, workstations | File access, user authentication, process execution, registry edits, PowerShell activity |
| Network-Centric | Firewalls, IDS/IPS, routers, VPN gateways | SSH connections, FTP file transfers, web traffic, VPN access, network file sharing |

The distinction matters because host-centric logs tell you what happened on a machine, while network-centric logs tell you how machines were communicating. Both are needed to reconstruct an attack.

**The five core challenges of working with raw, decentralized logs:**

| Challenge | Why It Matters to a SOC Analyst |
|---|---|
| Numerous Log Sources | Hundreds of events per second across many devices; manually reviewing all of them is not realistic |
| No Centralization | Logs live on the device that generated them; accessing them requires connecting to each machine individually via SSH, RDP, etc. |
| Limited Context | A single log event rarely tells the full story; attackers use lateral movement, so the picture only becomes clear when logs are combined |
| Limited Analysis | The sheer volume of logs makes it impossible for a human analyst to catch everything manually |
| Format Issues | Windows logs, Linux logs, and firewall logs all look different; an analyst needs to know every format to work across sources |

### Task Questions

1. Is Registry-related activity host-centric or network-centric?

   **Answer: Host-Centric**

2. Is VPN-related activity host-centric or network-centric?

   **Answer: Network-Centric**

---

## Task 3 - Why SIEM?

### Key Concepts

SIEM is the direct answer to every challenge from Task 2. It collects logs from all sources, standardizes their format, correlates them to surface patterns, and alerts analysts when detection rules are triggered.

![SIEM data flow diagram](screenshots/T3_siem_flow.png)

Two terms worth distinguishing: **parsing** is the process of breaking a raw log down into individual fields. **Normalization** is converting all those parsed logs from different sources into one consistent schema. Normalization is what makes correlation possible; you cannot correlate a Windows event log against a firewall log if they are structured completely differently.

**The Haris example from the room is worth sitting with.** Taken one at a time, each of these events looks unremarkable: a VPN login from an unfamiliar IP, access to files on a shared drive, a PowerShell script execution, an outbound network connection. Correlated across a five-minute window, that sequence is a textbook data exfiltration pattern. The individual logs were not the signal. The relationship between them was. That is the entire value proposition of correlation.

| SIEM Feature | Problem It Solves | How an Analyst Uses It |
|---|---|---|
| Centralized Log Collection | Eliminates the need to SSH or RDP into each machine individually | All logs flow into one platform; analysts work from a single interface |
| Normalization | Raw logs from different sources use different formats and field names | Parsed and standardized logs allow consistent querying and rule-writing across all sources |
| Correlation | Individual log events lack context; attacks span multiple systems | SIEM links related events across sources and timeframes to surface attack patterns |
| Real-time Alerting | Threats need to be caught as they happen, not in a post-mortem | Detection rules fire alerts the moment conditions are met; analysts investigate from inside the SIEM |
| Dashboards and Reporting | High log volume makes it hard to see what matters | Dashboards surface alert highlights, failed login counts, ingestion health, top domains visited, and rule trigger summaries |

### Task Questions

1. Read the task above.

   **Answer:** No answer needed.

---

## Task 4 - Log Sources and Ingestion

### Key Concepts

**Windows logs** are viewed through Event Viewer and organized by unique Event IDs. Each ID corresponds to a specific type of activity, which makes writing detection rules against Windows logs precise and repeatable. All Windows endpoint logs are forwarded to the SIEM for centralized monitoring.

**Linux logs** are stored as flat files in `/var/log/`. Each file captures a specific category of activity:

| Linux Log Path | What It Captures |
|---|---|
| /var/log/httpd | HTTP requests, responses, and web server errors |
| /var/log/cron | Scheduled job execution logs |
| /var/log/auth.log | Authentication events: logins, logoffs, sudo usage |
| /var/log/secure | Authentication logs on Red Hat/CentOS-based systems |
| /var/log/kern | Kernel-level events: boot sequences, driver interactions, hardware errors |

**Web server logs** (Apache) record every request hitting the server. Each line includes the source IP, timestamp, HTTP method, requested path, status code, response size, and user agent. These are essential for detecting web attacks.

![Log ingestion methods in Splunk](screenshots/T4_various_ways_logs.png)

**Log ingestion methods** vary by SIEM and deployment context:

| Ingestion Method | How It Works | When You Would Use It |
|---|---|---|
| Agent / Forwarder | Lightweight agent installed on the endpoint captures and ships logs to the SIEM | Standard deployment for managed endpoints |
| Syslog | Industry-wide protocol for forwarding log data from network devices and servers in real time | Devices that support syslog natively: routers, firewalls, switches |
| Manual Upload | Offline log files are uploaded directly into the SIEM | Quick analysis of logs from an isolated or offline system |
| Port-Forwarding | SIEM listens on a configured port; endpoints forward data directly to that port | Lightweight integration without an agent install |

### Task Questions

1. In which location within a Linux environment are HTTP logs stored?

   **Answer: /var/log/httpd**

---

## Task 5 - Alerting Process and Analysis

### Key Concepts

Detection rules are the engine behind SIEM alerting. A rule is a logical expression: when specific field values appear in ingested logs and a defined condition is met, the SIEM triggers an alert. This is why normalization matters so much. Rules are written against field names and values; if logs are not consistently parsed, rules break or miss events entirely.

| Use Case | Threat Behavior | Key Log Fields | Rule Logic |
|---|---|---|---|
| Event Log Cleared (ID 104) | Attacker clears Windows event logs to remove evidence of their activity | Log source: WinEventLog, EventID: 104 | If Log Source = WinEventLog AND EventID = 104, trigger alert: Event Log Cleared |
| whoami Execution (ID 4688) | Attacker runs whoami after exploitation to confirm their privilege level | Log source: WinEventLog, EventID: 4688, NewProcessName | If Log Source = WinEventLog AND EventCode = 4688 AND NewProcessName contains whoami, trigger alert: whoami Execution Detected |

After an alert fires, analysts work through a standard triage workflow: examine the events tied to the alert, review which rule conditions were met, determine whether the activity is genuine or a false positive, and take action accordingly.

| Alert Outcome | What It Means | Follow-up Actions |
|---|---|---|
| False Positive | The rule fired, but the activity is benign | Tune the rule to tighten the condition and reduce noise |
| True Positive | The rule fired and the activity is genuinely malicious | Investigate further, contact the asset owner, isolate the host, block the suspicious IP as needed |

### Task Questions

1. Which Event ID is generated when event logs are removed?

   **Answer: 104**

2. What type of alert may require tuning?

   **Answer: False Positive**

---

## Task 6 - Lab Work

### Key Concepts

This task puts everything together in a static SIEM dashboard simulation. The goal is to trigger a suspicious activity event, trace it through the SIEM, identify who did what and from where, evaluate the alert, and take the correct response action.

![SIEM dashboard baseline](screenshots/T6_siem_dashboard.png)

### Task Questions

1. After clicking on the Start Suspicious Activity button, which process caused the alert?

   ![cudominer.exe alert triggered](screenshots/T6_siem_cudominer.png)

   **Answer: cudominer.exe**

2. Find the event that caused the alert and identify the user responsible for the process execution.

   ![Event detail showing user Chris](screenshots/T6_chris_cudominer.png)

   **Answer: Chris**

3. What is the hostname of the suspect user?

   **Answer: HR_02**

4. Examine the rule and the suspicious process; which term matched the rule that caused the alert?

   ![Detection rule that matched](screenshots/T6_chris_rule_alert.png)

   **Answer: Miner**

5. Which option best represents the event? (False Positive / True Positive)

   ![True Positive classification](screenshots/T6_chris_tp.png)

   **Answer: True Positive**

6. Selecting the right ACTION will display the FLAG. What is the FLAG?

   **Answer: THM{000_SIEM_INTRO}**

---

## Task 7 - Conclusion

### Key Concepts

Before this room, SIEM was a vague concept: a dashboard somewhere that received alerts. Now it has structure. SIEM is a centralized log collection and analysis platform that makes large-scale security monitoring possible by solving the problems that raw, isolated logs cannot: volume, format inconsistency, lack of context, and the impossibility of manual analysis at scale.

The lab made the abstract concrete. A process ran on an endpoint, a detection rule matched a keyword in the process name, an alert fired, and the correct response action confirmed the activity was malicious. That end-to-end flow is the daily reality of SOC work.

### Task Questions

1. Complete this room.

   **Answer:** No answer needed.

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*
