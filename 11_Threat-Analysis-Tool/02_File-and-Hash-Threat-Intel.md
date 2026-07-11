---
title: File and Hash Threat Intel
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [cti, file-hash-analysis, virustotal, malwarebazaar, sandbox-analysis, mitre-attack, soc-l1]
status: in-progress
date: 2026-07-10
date_completed: 
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
* Extract behaviour from sandbox telemetry and map it to MITRE ATT&CK.

### Prerequisites

Before embarking on the journey, ensure that you have covered the following concepts and rooms:

* Familiarity with basic Windows and Linux shell utilities.
* [Intro to Cyber Threat Intelligence](https://tryhackme.com/room/cyberthreatintel)

### Scenario

It is a Monday in April. Try Daily is preparing a significant release. The EDR tool flags multiple binaries on various workstations during a routine alert sweep. You, the L1 analyst (shadowing your L2 mentor), receive a curated triage package containing those samples. Within 60 minutes, you must provide evidence to showcase whether these files are bait, benign, or malicious.

### Lab Access

Before proceeding, start the lab by clicking the Start Machine button below. The VM will open in split view and take about 2 minutes to load fully. If it is not visible, click the Show Split View button at the top of the page. Once the VM loads, you will find:

* Case files on the Desktop, ready for analysis.
* TryDetectThis — accessible via the Desktop shortcut or by navigating to `http://TryDetectThis.thm:8080` in the browser. Use this platform to search for hashes, review vendor detections, inspect file properties, and analyse sandbox behaviour.
* Note: Due to the changing nature of threat intelligence, we have included all necessary data in our offline Threat Intel tool TryDetectThis. Please only use the TryDetectThis tool to solve the various questions in the tasks ahead.

**Answer the questions below**

---

**1. Dive into file threat intelligence.**

**Answer:** N/A

---

## Task 2: Filenames and Paths

You spot a suspicious.exe; your first reliable course of action is to read the strings, such as filepaths.

<p align="center">
<img src=screenshots/threat2_filepath.png width="700">
</p>

### Filepath Analysis

Think of a crime scene: file paths and names are like clues left by the attacker.
  - Attackers may attempt to hide evidence by using different disk locations to hide their actions
      - C:\ disk is a common target for persistence tachiniques
      - C:\Users\Public the **public** profile can enable be used as files can be shared to all other users through this account
      - C:\Users\Public\Public Downloads is a high traffic directory that is often over looked in monitoring
      - C: \Windows\Temp\ attackers will put files in here because they are created, used, and deleted temporarily.
      - C:\ProgramData\ attackers will often use this directory for stealthy persistence
    
### Filename Heuristic Indicators

Heuristic Indicators basically means little clues that could point to an attacker trick; 
  - doubling up on extension **file.pdf.exe**
  - changing the file name to a regularly used process such as **scvhost.exe**
  - a file name **jh8F21.exe** suggests packing or polyphormic generation known as high-entropy strings
  - dishguising filenames such as backup-2300.exe can blend with routine files

**Tip**: These should be in the starter book of any SOC to learn and recognize!


---

**1. One of the files included in the `CTI Files` folder on the Desktop shows one of the indicators mentioned. Can you identify the file and the indicator? (Answer: file, property)**

Looking inside the directory, we observed four different files, the one that jumped out right away that it could represent CTI is the obvious double exenstion **payroll.pdf.exe**
<p align="center">
<img src=screenshots/threat2_doubleexe.png width="700">
</p>

**Answer: payroll.pdf, Double extensions**

---

## Task 3: File Hash Lookup

Even after spotting the suspicious filename, we still don't have concrete evidence of a malicious file. Attackers are constantly changing file names; however, the file's cryptographic fingerprint remains the same. By analyzing the SHA256 and MD5 we can identify files and malware.

Tips on dealing with hashes:
  - Store hashes in lowercase to avoid nuisances
  - .zip files should be hashed as well as the extracted file
  - Plain strings without any context where you got them could confuse your team
  - **Note:** one byte of the file is changed, and the hash will be completely different, so hashes should not be considered foolproof as evidence.

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

Hash in hand the SOC can move to the next step in his investigation. https://www.virustotal.com/ is a tool widely used by security analysts, it provides us with dozens of antivirus reports from a single serach.
  - Simply submit a hash and the website will return a comprehensive report with various fields of useful information
      - **Detection Score**: the more dangerous a hash/file/website is, the higher the number
      - **Detection rules**: Technical signatures used by antivirus software, YARA rules, heuristic indicators, and behavioral triggers
      - **Properties**: Core metadata can for the file, type, size, compilation, timestamp
      - **Contained domains and IPs**: Malwares network infrastructure
      - **Contained files**: Details files embedded or dorpped during malware execution 



| Section | Key Question | Red Flags | Analyst Considerations |
|---|---|---|---|
| Detection Score and Threat Labels | How many vendors detect this file as malicious? | Five or more solid vendors flag it. Conflicting classifications (e.g., "Trojan" vs "PUA"). No consensus among the top 3 vendors | New malware often has low initial detection. Recheck after 24h for updated results. Look at the classification of the malware family name or capability name |
| Upload Time | When was the file first submitted? | Uploaded seven days ago with more than 10 detections. Sudden detection spike after days/weeks | Vendors need 48-72 hours for full analysis. Historical detection growth indicates malware ageing |
| Signatures | Is the file properly signed? | Invalid/missing certificate. Certificate issued to an unrelated entity | Even valid certs can be stolen/abused. Check cert chain expiration dates |
| Properties | Are there anomalies in the file data? | Compile timestamp at odd hours (e.g., 3 AM). High entropy (>7.5) in non-media files | Some legitimate packers (UPX) increase entropy. Compare with known-good versions |
| Relations | What infrastructure does the malware connect to? | Known-bad IPs in VirusTotal's graph. DGA-like domains (e.g., xk8f92.xyz) | Legitimate CDNs may host malware. Check IPs in Shodan for open ports |
| Behavioral | What post-execution actions occur? | Modifies critical registry keys. Attempts process injection | Some admin tools modify registries legitimately. Correlate with endpoint logs |


### Cross-Reference With MalwareBazaar

https://bazaar.abuse.ch/ All-in-one database for malware collection and analysis 
  - Upload malware samples for analysis
  - Malware hunting through: malware family tagging, YARA rule integration search, campaign contribution search through tags such as #TA551
  - Malware samples for download and research


**Answer the questions below**

---

**1. What is the SHA256 hash of the file bl0gger?**

**Answer:**

---

**2. What is the threat classification label used to identify the malicious file?**

**Answer:**

---

**3. When was the file first submitted for analysis? (Answer format: YYYY-MM-DD HH:MM:SS)**

**Answer:**

---

**4. Which vendor classified the Morse-Code-Analyzer file as non-malicious?**

**Answer:**

---

**5. What MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?**

**Answer:**

---

## Task 4: Sandbox Analysis
*[quick definition]*


### Dynamic Analysis for SOC
*[quick definition]*


### Sandboxing Tools
*[quick definition]*


### Analysing Hybrid Analysis Report
*[reference]*

When we search using the file hash and analyse the generated report from Hybrid Analysis, we can confirm that the file is malicious, with a threat score of 100/100. Additionally, TryDetectThis has all the details ingested for the remainder of the room.

We are also provided with information about the file's risk behaviour and the ATT&CK Techniques the threat actor observed using.

Additionally, we can gather all file details and information to inform our threat intelligence report. These details are vital for threat reporting, both internally and externally, including file compositions, imports, processes called, and network connections.

### Sandboxing Limitations
*[decision logic]*


**Answer the questions below**

---

**1. What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis? (Answer: Tag1, Tag2, Tag3)**

**Answer:**

---

**2. What was the stealth command line executed from the file?**

**Answer:**

---

**3. Which other process was spawned according to the process tree?**

**Answer:**

---

**4. Analyze the payroll.pdf file located in the CTI Files folder and answer the questions below. The payroll.pdf application seems to be masquerading as which known Windows file?**

**Answer:**

---

**5. What associated URL is linked to the file?**

**Answer:**

---

**6. How many extracted strings were identified from the sandbox analysis of the file?**

**Answer:**

---

## Task 5: Threat Intelligence Challenge
*[reference]*

Now that we have looked at gathering threat intelligence through looking at file attributes, you will now have the opportunity to investigate a suspected file Challenge.bin.sample.

**Answer the questions below**

---

**1. What is the SHA256 hash of the file?**

**Answer:**

---

**2. What family labels are assigned to the file on VirusTotal?**

**Answer:**

---

**3. When was the first time the file was recorded in the wild? (Answer Format: YYYY-MM-DD HH:MM:SS UTC)**

**Answer:**

---

**4. Name the text file dropped during the execution of the malicious file.**

**Answer:**

---

**5. What PowerShell command is observed to be executed?**

**Answer:**

---

**6. What MITRE ATT&CK ID is associated with the actions of the command?**

**Answer:**

---

## Task 6: Conclusion
*[reference]*

Congratulations on completing the room! Threat intelligence is integral to an SOC workflow that enriches unknown binaries encountered by threat alerts.

**Key Takeaways**

* Validate evidence before analysis: confirm you have the exact binaries and hashes.
* Analyse filepaths and filenames for quick context: Unusual storage locations, trusted software directories, and naming tricks highlight items that warrant deeper review.
* Generate hashes early to pivot into external or internal knowledge bases.
* Observe runtime behaviour in a controlled environment to confirm intent, extract network and persistence indicators, and map activity to recognised attack techniques.
* Translate findings into a brief that lists indicators by type, summarises behaviour and provides proportionate recommendations supported by evidence.

**Answer the questions below**

---

**1. I am ready to continue with more threat intelligence!**

**Answer:** N/A

---
