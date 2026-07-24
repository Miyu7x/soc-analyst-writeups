---
title: Tempest
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [incident-response, sysmon, windows-event-logs, packet-capture, malware, c2, privilege-escalation, persistence]
status: 
date: 
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/tempest_intro.png width="1500">
</p>

---

## TASK 1 - Introduction

This room walks through a full incident response investigation on the Tempest machine, tracing an attack chain from initial access via a malicious Word document, through a base64-encoded stage 2 payload and C2 traffic, internal reconnaissance, lateral movement via a reverse SOCKS proxy, privilege escalation, and finally persistence through newly created administrator accounts. The investigation draws on Sysmon logs, Windows Event Logs, and packet capture, using EvtxEcmd, Timeline Explorer, SysmonView, Wireshark, and Brim.

---

## TASK 2 - Preparation - Log Analysis

### Log Analysis

### Event Correlation

---

## TASK 3 - Preparation - Tools and Artifacts

### Compare by hash

### Toolset

### EvtxEcmd & Timeline Explorer

### SysmonView

---

**What is the SHA256 hash of the capture.pcapng file?**

**Answer:**

---

**What is the SHA256 hash of the sysmon.evtx file?**

**Answer:**

---

**What is the SHA256 hash of the windows.evtx file?**

**Answer:**

---

## TASK 4 - Initial Access - Malicious Document

### Tempest Incident

Acting as the Incident Responder on a CRITICAL-severity alert triaged by a SOC analyst, the intrusion traces back to a malicious `.doc` file, downloaded via `chrome.exe`, which then ran a chain of commands to achieve code execution.

### Investigation Guide

---

**The user of this machine was compromised by a malicious document. What is the file name of the document?**

**Answer:**

---

**What is the name of the compromised user and machine?**
Format: username-machine name

**Answer:**

---

**What is the PID of the Microsoft Word process that opened the malicious document?**

**Answer:**

---

**Based on Sysmon logs, what is the IPv4 address resolved by the malicious domain used in the previous question?**

**Answer:**

---

**What is the base64 encoded string in the malicious payload executed by the document?**

**Answer:**

---

**What is the CVE number of the exploit used by the attacker to achieve a remote code execution?**
Format: XXXX-XXXXX

**Answer:**

---

## TASK 5 - Initial Access - Stage 2 execution

### Malicious Document - Stage 2

The document successfully executed an encoded base64 command; decoding it reveals the exact command chain the document ran.

### Investigation Guide

---

**The malicious execution of the payload wrote a file on the system. What is the full target path of the payload?**

**Answer:**

---

**The implanted payload executes once the user logs into the machine. What is the executed command upon a successful login of the compromised user?**
Format: Remove the double quotes from the log.

**Answer:**

---

**Based on Sysmon logs, what is the SHA256 hash of the malicious binary downloaded for stage 2 execution?**

**Answer:**

---

**The stage 2 payload downloaded establishes a connection to a c2 server. What is the domain and port used by the attacker?**
Format: domain:port

**Answer:**

---

## TASK 6 - Initial Access - Malicious Document Traffic

### Malicious Document Traffic

The attacker fetched the stage 2 payload remotely. Sysmon logs show the domain and IP invoked by the malicious document, and a separate domain and IP used by the stage 2 payload.

### Investigation Guide

---

**What is the URL of the malicious payload embedded in the document?**

**Answer:**

---

**What is the encoding used by the attacker on the c2 connection?**

**Answer:**

---

**The malicious c2 binary sends a payload using a parameter that contains the executed command results. What is the parameter used by the binary?**

**Answer:**

---

**The malicious c2 binary connects to a specific URL to get the command to be executed. What is the URL used by the binary?**

**Answer:**

---

**What is the HTTP method used by the binary?**

**Answer:**

---

**Based on the user agent, what programming language was used by the attacker to compile the binary?**
Format: Answer in lowercase

**Answer:**

---

## TASK 7 - Discovery - Internal Reconnaissance

### Internal Reconnaissance

The malicious binary continuously uses the C2 traffic, which carries an encoded string of the commands the attacker ran and their output.

### Investigation Guide

---

**The attacker was able to discover a sensitive file inside the machine of the user. What is the password discovered on the aforementioned file?**

**Answer:**

---

**The attacker then enumerated the list of listening ports inside the machine. What is the listening port that could provide a remote shell inside the machine?**

**Answer:**

---

**The attacker then established a reverse socks proxy to access the internal services hosted inside the machine. What is the command executed by the attacker to establish the connection?**
Format: Remove the double quotes from the log.

**Answer:**

---

**What is the SHA256 hash of the binary used by the attacker to establish the reverse socks proxy connection?**

**Answer:**

---

**What is the name of the tool used by the attacker based on the SHA256 hash? Provide the answer in lowercase.**

**Answer:**

---

**The attacker then used the harvested credentials from the machine. Based on the succeeding process after the execution of the socks proxy, what service did the attacker use to authenticate?**
Format: Answer in lowercase

**Answer:**

---

## TASK 8 - Privilege Escalation - Exploiting Privileges

### Privilege Escalation

With a stable shell through the reverse SOCKS proxy, the attacker now moves to escalate privileges, having already established persistent low-privilege access.

### Investigation Guide

---

**After discovering the privileges of the current user, the attacker then downloaded another binary to be used for privilege escalation. What is the name and the SHA256 hash of the binary?**
Format: binary name,SHA256 hash

**Answer:**

---

**Based on the SHA256 hash of the binary, what is the name of the tool used?**
Format: Answer in lowercase

**Answer:**

---

**The tool exploits a specific privilege owned by the user. What is the name of the privilege?**

**Answer:**

---

**Then, the attacker executed the tool with another binary to establish a c2 connection. What is the name of the binary?**

**Answer:**

---

**The binary connects to a different port from the first c2 connection. What is the port used?**

**Answer:**

---

## TASK 9 - Actions on Objective - Fully-owned Machine

### Fully-Owned Machine

With administrative privileges secured, the goal now is to map out every persistence technique the attacker put in place, all tied to the same malicious C2 binary used during privilege escalation.

### Investigation Guide

---

**Upon achieving SYSTEM access, the attacker then created two users. What are the account names?**
Format: Answer in alphabetical order - comma delimited

**Answer:**

---

**Prior to the successful creation of the accounts, the attacker executed commands that failed in the creation attempt. What is the missing option that made the attempt fail?**

**Answer:**

---

**Based on windows event logs, the accounts were successfully created. What is the event ID that indicates the account creation activity?**

**Answer:**

---

**The attacker added one of the accounts in the local administrator's group. What is the command used by the attacker?**

**Answer:**

---

**Based on windows event logs, the account was successfully added to a sensitive group. What is the event ID that indicates the addition to a sensitive local group?**

**Answer:**

---

**After the account creation, the attacker executed a technique to establish persistent administrative access. What is the command executed by the attacker to achieve this?**
Format: Remove the double quotes from the log.

**Answer:**

---

## TASK 10 - Conclusion

This incident traces a complete attack chain: initial access through a malicious Word document, a base64-encoded stage 2 payload standing up C2 over an encoded channel, internal reconnaissance and credential harvesting, lateral movement via a reverse SOCKS proxy, privilege escalation exploiting a Windows privilege, and persistence through newly created accounts added to the local Administrators group. The case closes with a full timeline and the IOCs — domains, hashes, binaries — documented for containment and remediation.
