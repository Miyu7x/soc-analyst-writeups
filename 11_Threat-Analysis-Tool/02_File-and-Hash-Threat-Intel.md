---
title: File and Hash Threat Intel
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [cti, file-hash-analysis, virustotal, malwarebazaar, sandbox-analysis, mitre-attack, soc-l1]
status: completed
date: 2026-07-10
date_completed: 2026-07-10
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/threat2_intro.png width="1500">
</p>

---

## Task 1: Introduction

Security Operations teams live inside alert queues and triaging. Every entry follows three essential steps: verify, enrich, and decide. File and hash intelligence sits squarely in the enrich phase, transforming a lone path or hash value into contextual knowledge within your organization about malicious artifacts that would be identified.

### Learning Objectives

By completing this room, you will be able to:

* Interpret suspicious filepaths and filenames using heuristics.
* Generate and validate file hashes.
* Leverage VirusTotal and MalwareBazaar to enrich newly observed binaries.
* Extract behavior from sandbox telemetry and map it to MITRE ATT&CK.

### Prerequisites

Before embarking on the journey, ensure that you have covered the following concepts and rooms:

* Familiarity with basic Windows and Linux shell utilities.
* [Intro to Cyber Threat Intelligence](https://tryhackme.com/room/cyberthreatintel)

### Scenario

It is a Monday in April. Try Daily is preparing a significant release. The EDR tool flags multiple binaries on various workstations during a routine alert sweep. You, the L1 analyst (shadowing your L2 mentor), receive a curated triage package containing those samples. Within 60 minutes, you must provide evidence to showcase whether these files are bait, benign, or malicious.

### Lab Access

Before proceeding, start the lab by clicking the Start Machine button below. The VM will open in split view and take about 2 minutes to load fully. If it is not visible, click the Show Split View button at the top of the page. Once the VM loads, you will find:

* Case files on the Desktop, ready for analysis.
* TryDetectThis — accessible via the Desktop shortcut or by navigating to `http://TryDetectThis.thm:8080` in the browser. Use this platform to search for hashes, review vendor detections, inspect file properties, and analyze sandbox behavior.
* Note: Due to the changing nature of threat intelligence, we have included all necessary data in our offline Threat Intel tool TryDetectThis. Please only use the TryDetectThis tool to solve the various questions in the tasks ahead.

**Answer the questions below**

---

**1. Dive into file threat intelligence.**

**Answer:** N/A

---

## Task 2: Filenames and Paths

You spot a suspicious .exe; your first reliable course of action is to read the strings, such as filepaths.

<p align="center">
<img src=screenshots/threat2_filepath.png width="700">
</p>

### Filepath Analysis

Think of a crime scene: file paths and names are like clues left by the attacker.
  - Attackers may attempt to hide evidence by using different disk locations to hide their actions
      - C:\ disk is a common target for persistence techniques
      - C:\Users\Public, the **public** profile, can be used since files can be shared to all other users through this account
      - C:\Users\Public\Public Downloads is a high traffic directory that is often overlooked in monitoring
      - C:\Windows\Temp\ attackers will put files in here because they are created, used, and deleted temporarily
      - C:\ProgramData\ attackers will often use this directory for stealthy persistence

### Filename Heuristic Indicators

Heuristic indicators basically mean little clues that could point to an attacker trick:
  - doubling up on extension: **file.pdf.exe**
  - changing the file name to a regularly used process, such as **scvhost.exe**
  - a file name like **jh8F21.exe** suggests packing or polymorphic generation, known as high-entropy strings
  - disguising filenames such as backup-2300.exe can blend with routine files

**Tip**: These should be in the starter book of any SOC analyst to learn and recognize!

---

**1. One of the files included in the `CTI Files` folder on the Desktop shows one of the indicators mentioned. Can you identify the file and the indicator? (Answer: file, property)**

Looking inside the directory, we observed four different files. The one that jumped out right away as it could represent CTI is the obvious double extension **payroll.pdf.exe**

<p align="center">
<img src=screenshots/threat2_doubleexe.png width="700">
</p>

**Answer:** payroll.pdf, Double extensions

---

## Task 3: File Hash Lookup

Even after spotting the suspicious filename, we still don't have concrete evidence of a malicious file. Attackers are constantly changing file names; however, the file's cryptographic fingerprint remains the same. By analyzing the SHA256 and MD5 we can identify files and malware.

Tips on dealing with hashes:
  - Store hashes in lowercase to avoid nuisances
  - .zip files should be hashed as well as the extracted file
  - Plain strings without any context of where you got them could confuse your team
  - **Note:** if one byte of the file is changed, the hash will be completely different, so hashes should not be considered foolproof evidence

We can generate and verify our file hash using the following commands:

**Windows Commands**

Command prompt:
```cmd
certutil -hashfile bl0gger.exe SHA256
```

PowerShell:
```powershell
Get-Filehash -Algorithm SHA256 bl0gger.exe
```

**Linux Commands**
```bash
ubuntu@tryhackme$ sha256sum bl0gger.exe
```

### Analysis With VirusTotal

Hash in hand, the SOC can move to the next step in the investigation. https://www.virustotal.com/ is a tool widely used by security analysts; it provides us with dozens of antivirus reports from a single search.
  - Simply submit a hash and the website will return a comprehensive report with various fields of useful information:
      - **Detection Score**: the more dangerous a hash/file/website is, the higher the number
      - **Detection rules**: Technical signatures used by antivirus software, YARA rules, heuristic indicators, and behavioral triggers
      - **Properties**: Core metadata for the file, including type, size, and compilation timestamp
      - **Contained domains and IPs**: The malware's network infrastructure
      - **Contained files**: Details on files embedded or dropped during malware execution

| Section | Key Question | Red Flags | Analyst Considerations |
|---|---|---|---|
| Detection Score and Threat Labels | How many vendors detect this file as malicious? | Five or more solid vendors flag it. Conflicting classifications (e.g., "Trojan" vs "PUA"). No consensus among the top 3 vendors | New malware often has low initial detection. Recheck after 24h for updated results. Look at the classification of the malware family name or capability name |
| Upload Time | When was the file first submitted? | Uploaded seven days ago with more than 10 detections. Sudden detection spike after days/weeks | Vendors need 48-72 hours for full analysis. Historical detection growth indicates malware ageing |
| Signatures | Is the file properly signed? | Invalid/missing certificate. Certificate issued to an unrelated entity | Even valid certs can be stolen/abused. Check cert chain expiration dates |
| Properties | Are there anomalies in the file data? | Compile timestamp at odd hours (e.g., 3 AM). High entropy (>7.5) in non-media files | Some legitimate packers (UPX) increase entropy. Compare with known-good versions |
| Relations | What infrastructure does the malware connect to? | Known-bad IPs in VirusTotal's graph. DGA-like domains (e.g., xk8f92.xyz) | Legitimate CDNs may host malware. Check IPs in Shodan for open ports |
| Behavioral | What post-execution actions occur? | Modifies critical registry keys. Attempts process injection | Some admin tools modify registries legitimately. Correlate with endpoint logs |

### Cross-Reference With MalwareBazaar

https://bazaar.abuse.ch/ is an all-in-one database for malware collection and analysis:
  - Upload malware samples for analysis
  - Malware hunting through: malware family tagging, YARA rule integration search, campaign attribution search through tags such as #TA551
  - Malware samples available for download and research

---

In a production environment, you would search MalwareBazaar using the syntax `is sha256:<file_hash>`. For our case, TryDetectThis has integrated the vendor intelligence from VirusTotal and MalwareBazaar.

---

**1. What is the SHA256 hash of the file bl0gger?**

<p align="center">
<img src=screenshots/threat2_doubleexe.png width="700">
</p>

**Answer:** 2672B6688D7B32A90F9153D2FF607D6801E6CBDE61F509ED36D0450745998D58

---

**2. What is the threat classification label used to identify the malicious file?**

<p align="center">
<img src=screenshots/threat2_graftor.png width="700">
</p>

**Answer:** trojan.graftor/flystudio

---

**3. When was the file first submitted for analysis? (Answer format: YYYY-MM-DD HH:MM:SS)**

<p align="center">
<img src=screenshots/threat2_submitted.png width="700">
</p>

**Answer:** 2025-05-15 12:03:49

---

**4. Which vendor classified the Morse-Code-Analyzer file as non-malicious?**

<p align="center">
<img src=screenshots/threat2_morse.png width="700">
</p>

**Answer:** CyberFortress

---

**5. What MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?**

<p align="center">
<img src=screenshots/threat2_sideloading.png width="700">
</p>

**Answer:** DLL Side-Loading

---

## Task 4: Sandbox Analysis

Safely running malware might sound contradictory, but often it's the only way to find its secrets and inner workings.
  - Sandboxes provide us that privilege: running malware in a contained environment for research purposes, with the ability to revert the state of the machine to a clean one.

### Dynamic Analysis for SOC

Looking up a hash is just static investigation. In order to decipher malware further, an L2 analyst will sandbox the file and run it to discover its inner workings.

### Sandboxing Tools

**Hybrid Analysis** focuses on process trees and the MITRE ATT&CK heatmap, perfect for a quick summary of the file.
**Joe Sandbox** focuses on deep analysis, system calls, strings, and memory dumps, great for reverse engineering and detection engineering.

### Analysing Hybrid Analysis Report

When we search using the file hash and analyze the generated report from Hybrid Analysis, we can confirm that the file is malicious, with a threat score of 100/100. Additionally, TryDetectThis has all the details ingested for the remainder of the room.

We are also provided with information about the file's risk behavior and the ATT&CK techniques the threat actor was observed using.

Additionally, we can gather all file details and information to inform our threat intelligence report. These details are vital for threat reporting, both internally and externally, including file compositions, imports, processes called, and network connections.

### Sandboxing Limitations

Attackers know their malware will probably end up in a sandbox, so they have devised various methods to evade sandbox investigation.
  - Environment Awareness Checks
  - Anti-Debugging & Anti-Sandboxing Tricks
  - Limited Execution Time & Coverage
  - Encrypted & Obfuscated Traffic
  - Fileless & Living-off-the-Land (LotL) Malware

---

Now it is your turn. Use the TryDetectThis tool to answer the questions below.

---

**1. What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis? (Answer: Tag1, Tag2, Tag3)**

<p align="center">
<img src=screenshots/threat2_tags.png width="700">
</p>

We can observe several tags for the bl0gger.exe file: #Blackmoon, #Discovery, #windows-server-utility, #flystudio, #trojan.

**Answer:** BlackMoon, Discovery, windows-server-utility

---

**2. What was the stealth command line executed from the file?**

<p align="center">
<img src=screenshots/threat2_werfault.png width="700">
</p>

On the TryDetectThis dashboard, the Sandbox Analysis section shows the **Observed Command:** `regsvr32 %WINDIR%\Media\ActiveX.ocx /s`

**Answer:** regsvr32 %WINDIR%\Media\ActiveX.ocx /s

---

**3. Which other process was spawned according to the process tree?**

<p align="center">
<img src=screenshots/threat2_werfault.png width="700">
</p>

On the Hybrid Analysis website, the **Hybrid Analysis** tab lists the processes run in order by the malware. **werfault.exe** is a sneaky, legitimate Windows Error Reporting process.

Seeing werfault.exe spawn could mean a couple of things: the malware crashed and triggered a real Windows error report, or the real reason is that certain **malware families** will deliberately abuse **werfault.exe**, since it's normal to see it running on a process list, making it easy to hide behind.

**Answer:** werfault.exe

---

**4. Analyze the payroll.pdf file located in the CTI Files folder and answer the questions below. The payroll.pdf application seems to be masquerading as which known Windows file?**

<p align="center">
<img src=screenshots/threat2_dll.png width="700">
</p>

TryDetectThis Sandbox Analysis: Impersonation Behavior

**Answer:** svchost.dll

---

**5. What associated URL is linked to the file?**

<p align="center">
<img src=screenshots/threat2_network.png width="700">
</p>

TryDetectThis, Network Indicators: hxxp://121.182.174.27:3000/server.exe

**Answer:** hxxp://121.182.174.27:3000/server.exe

---

**6. How many extracted strings were identified from the sandbox analysis of the file?**

<p align="center">
<img src=screenshots/threat2_strings.png width="700">
</p>

TryDetectThis, Extracted Strings: 454

**Answer:** 454

---

## Task 5: Threat Intelligence Challenge

Now that we have looked at gathering threat intelligence through looking at file attributes, you will now have the opportunity to investigate a suspected file, Challenge.bin.sample.

<p align="center">
<img src=screenshots/threat2_ioc.png width="700">
</p>

---

**1. What is the SHA256 hash of the file?**

<p align="center">
<img src=screenshots/threat2_challenge.png width="700">
</p>

**Answer:** 43b0ac119ff957bb209d86ec206ea1ec3c51dd87bebf7b4a649c7e6c7f3756e7

---

**2. What family labels are assigned to the file on VirusTotal?**

<p align="center">
<img src=screenshots/threat2_akira.png width="700">
</p>

**Answer:** akira, filecryptor

---

**3. When was the first time the file was recorded in the wild? (Answer Format: YYYY-MM-DD HH:MM:SS UTC)**

<p align="center">
<img src=screenshots/threat2_itw.png width="700">
</p>

**Answer:** 2024-10-30 17:17:24 UTC

---

**4. Name the text file dropped during the execution of the malicious file.**

<p align="center">
<img src=screenshots/threat2_dropped.png width="700">
</p>

**Answer:** akira_readme.txt

---

**5. What PowerShell command is observed to be executed?**

<p align="center">
<img src=screenshots/threat2_powershell.png width="700">
</p>

**Answer:** Get-WmiObject Win32_Shadowcopy | Remove-WmiObject

---

**6. What MITRE ATT&CK ID is associated with the actions of the command?**

<p align="center">
<img src=screenshots/threat2_t1490.png width="700">
</p>

**Answer:** T1490

---

## Task 6: Conclusion

Congratulations on completing the room! Threat intelligence is integral to an SOC workflow that enriches unknown binaries encountered by threat alerts.

**Key Takeaways**

* Validate evidence before analysis: confirm you have the exact binaries and hashes.
* Analyze filepaths and filenames for quick context: Unusual storage locations, trusted software directories, and naming tricks highlight items that warrant deeper review.
* Generate hashes early to pivot into external or internal knowledge bases.
* Observe runtime behavior in a controlled environment to confirm intent, extract network and persistence indicators, and map activity to recognized attack techniques.
* Translate findings into a brief that lists indicators by type, summarizes behavior, and provides proportionate recommendations supported by evidence.

**Answer the questions below**

---

**1. I am ready to continue with more threat intelligence!**

**Answer:** N/A

---
