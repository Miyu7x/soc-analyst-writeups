---
title: Boogeyman 2
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [phishing-analysis, eml, maldoc, macro-analysis, olevba, vba, memory-forensics, volatility, volatility3, dfir, c2, persistence, scheduled-task, capstone]
status: in-progress
date: 2026-08-03
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/boogeyman2.png width="1500">
</p>

---

## TASK 1 - [Introduction] The Boogeyman is back

- Second capstone in the Boogeyman series. Quick Logistics LLC hardened after the first intrusion, and the group returned with new TTPs.
- Two artefacts this time instead of three, in `/home/ubuntu/Desktop/Artefacts`:
  - a copy of the phishing email
  - a memory dump of the victim's workstation
- Shift from Boogeyman 1: no pcap, no PowerShell logs. Everything post-delivery has to come out of RAM.
- Toolkit on the VM: Volatility 3, olevba (part of oletools), plus the usual grep / strings / file
- Prerequisite rooms: Phishing Analysis Fundamentals, Phishing Analysis Tools, Boogeyman 1, Volatility

### Mental model for this room

- The email and the attachment are **static** analysis. What was delivered.
- The memory dump is **dynamic** analysis. What actually ran.
- Questions 1 to 5 come from the email and the maldoc. Questions 6 onward come from memory.
- Do not jump to Volatility until the macro has given you the stage 2 URL. The macro tells you what to look for in RAM.

---

### Volatility 3 cheatsheet

Set a variable once so the path is not retyped every time:

```
cd ~/Desktop/Artefacts
```

List every available plugin:

```
vol -f memorydump.raw -h
```

**Process tree, the first thing to run**

```
vol -f memorydump.raw windows.pstree
```

- Shows parent and child relationships, which answers "what launched what" directly
- `windows.pslist` gives a flat list instead, useful when you only need a PID

**Full command lines**

```
vol -f memorydump.raw windows.cmdline
```

- Arguments are where payload paths and encoded blobs live

**Network connections**

```
vol -f memorydump.raw windows.netscan
```

- Gives local and foreign address, port, state, owning PID
- Cross reference the PID back to `pstree` to name the process

**Files referenced in memory**

```
vol -f memorydump.raw windows.filescan
```

- Very long output. Pipe it and filter rather than reading it raw:

```
vol -f memorydump.raw windows.filescan > filescan.txt
grep -i "keyword" filescan.txt
```

**Console history and typed commands**

```
vol -f memorydump.raw windows.cmdscan
vol -f memorydump.raw windows.consoles
```

**Registry, for persistence hunting**

```
vol -f memorydump.raw windows.registry.printkey --key "Software\\Microsoft\\Windows\\CurrentVersion\\Run"
```

Notes:

- Volatility takes several minutes per plugin on this dump. Redirect output to a file the first time so it never has to be re-run:

```
vol -f memorydump.raw windows.pstree > pstree.txt
```

- Then grep the saved file instead of waiting again.

---

### Olevba cheatsheet

```
olevba document.doc
```

- Dumps the VBA source and flags suspicious keywords in a summary table at the bottom
- `--decode` attempts to decode obfuscated strings
- `--reveal` substitutes decoded strings back into the source, which makes the logic readable

Hashing an attachment:

```
md5sum document.doc
```

---

## TASK 2 - [Spear Phishing Human Resources]

### Scenario

- Maxine, a Human Resource Specialist at Quick Logistics LLC, received an application for an open position
- The attached resume was malicious and compromised her workstation
- The security team flagged suspicious commands on Maxine's workstation, which triggered the investigation
- Task is to analyse and assess the impact of the compromise

---

### 1. What email was used to send the phishing email?

*Method:*

<p align="center">
<img src=screenshots/bm2-sender.png width="700">
</p>

**Answer:**

---

### 2. What is the email of the victim employee?

*Method:*

<p align="center">
<img src=screenshots/bm2-victim.png width="700">
</p>

**Answer:**

---

### 3. What is the name of the attached malicious document?

*Method:*

<p align="center">
<img src=screenshots/bm2-attachment.png width="700">
</p>

**Answer:**

---

### 4. What is the MD5 hash of the malicious attachment?

*Method:*

```
md5sum <attachment>
```

<p align="center">
<img src=screenshots/bm2-md5.png width="700">
</p>

**Answer:**

---

### 5. What URL is used to download the stage 2 payload based on the document's macro?

*Method:*

```
olevba <attachment>
```

*What the macro is doing:*

<p align="center">
<img src=screenshots/bm2-macro.png width="700">
</p>

**Answer:**

---

### 6. What is the name of the process that executed the newly downloaded stage 2 payload?

*Method:*

```
vol -f memorydump.raw windows.pstree > pstree.txt
```

<p align="center">
<img src=screenshots/bm2-pstree.png width="700">
</p>

**Answer:**

---

### 7. What is the full file path of the malicious stage 2 payload?

*Method:*

```
vol -f memorydump.raw windows.cmdline > cmdline.txt
```

<p align="center">
<img src=screenshots/bm2-stage2-path.png width="700">
</p>

**Answer:**

---

### 8. What is the PID of the process that executed the stage 2 payload?

*Method:*

<p align="center">
<img src=screenshots/bm2-pid.png width="700">
</p>

**Answer:**

---

### 9. What is the parent PID of the process that executed the stage 2 payload?

*Method:*

*Why the parent matters:* the PPID proves the execution chain back to the Office application, which is what turns "a script ran" into "the maldoc ran it".

<p align="center">
<img src=screenshots/bm2-ppid.png width="700">
</p>

**Answer:**

---

### 10. What URL is used to download the malicious binary executed by the stage 2 payload?

*Method:*

<p align="center">
<img src=screenshots/bm2-stage3-url.png width="700">
</p>

**Answer:**

---

### 11. What is the PID of the malicious process used to establish the C2 connection?

*Method:*

```
vol -f memorydump.raw windows.netscan > netscan.txt
```

<p align="center">
<img src=screenshots/bm2-netscan.png width="700">
</p>

**Answer:**

---

### 12. What is the full file path of the malicious process used to establish the C2 connection?

*Method:*

<p align="center">
<img src=screenshots/bm2-c2-path.png width="700">
</p>

**Answer:**

---

### 13. What is the IP address and port of the C2 connection initiated by the malicious binary?

*Format: IP address:port*

*Method:*

<p align="center">
<img src=screenshots/bm2-c2-conn.png width="700">
</p>

**Answer:**

---

### 14. What is the full file path of the malicious email attachment based on the memory dump?

*Method:*

```
vol -f memorydump.raw windows.filescan > filescan.txt
grep -i "<attachment name>" filescan.txt
```

*Note:* the path in memory will be where Outlook cached or where Maxine saved it, not where it appeared in the mail client.

<p align="center">
<img src=screenshots/bm2-attachment-path.png width="700">
</p>

**Answer:**

---

### 15. The attacker implanted a scheduled task right after establishing the C2 callback. What is the full command used by the attacker to maintain persistent access?

*Method:*

```
vol -f memorydump.raw windows.cmdline | grep -i schtasks
```

*If cmdline does not carry it, try console history:*

```
vol -f memorydump.raw windows.consoles
```

<p align="center">
<img src=screenshots/bm2-persistence.png width="700">
</p>

**Answer:**

---

## TASK 3 - [Conclusion]

*Chain summary:*

- Initial access:
- Execution:
- Command and control:
- Persistence:

### Techniques worth keeping

-
-
-

### MITRE ATT&CK mapping

| Tactic | Technique | Evidence |
| --- | --- | --- |
| Initial Access | T1566.001 Spearphishing Attachment | |
| Execution | T1204.002 User Execution: Malicious File | |
| Execution | T1059.001 Command and Scripting Interpreter: PowerShell | |
| Command and Control | T1105 Ingress Tool Transfer | |
| Persistence | T1053.005 Scheduled Task | |
