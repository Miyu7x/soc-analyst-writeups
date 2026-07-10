---
title: Intro to Cyber Threat Intel
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [cti, threat-intelligence, mitre-attack, cyber-kill-chain, stix-taxii, soc-l1]
status: in-progress
date: 2026-07-10
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

## Task 1: Introduction

### Cyber Threat Intelligence for SOC L1 Analysts


### Prerequisites


**Answer the questions below**

---

**1. Ready to get started!**

**Answer:** N/A

---

## Task 2: Cyber Threat Intelligence

### From Raw Data to Usable Intelligence


| Layer | Definition | Alert-queue example | SOC L1 action |
|---|---|---|---|
| Data | An unprocessed observable | 45.155.205.3 :443 | Capture the artefact. |
| Information | Data plus factual annotation | IP registered to Hetzner, first seen 2023-07-14 | Record attributes. |
| Intelligence | Analysed information that answers so-what | IP belongs to the current BumbleBee C2; block immediately | Escalate or suppress. |

### Indicator Types Essential to First-Line Triage


| Indicator | Example | First Resources | Associated IOA or TTP Examples |
|---|---|---|---|
| IPv4 / IPv6 | 45.155.205.3 | WHOIS (ASN, allocation date), VirusTotal Relations, Shodan banner scan | IOA: Repeated SSH failures. TTP: T1110.003 Password Guessing |
| Domain / FQDN | malicious-updates[.]net | WHOIS age, RiskIQ or SecurityTrails passive-DNS, urlscan.io | IOA: surge of DNS queries to a 24-hour-old domain |
| URL | hxxp://malicious-updates[.]net/login | URLhaus reputation, urlscan.io behaviour graph, Any.Run dynamic run (network off) | IOA: Browser POST to /gateway.php with payload |
| File hash | e99a18c428cb38d5… | VirusTotal static & dynamic, Hybrid-Analysis, MalShare corpus | TTP: T1055 Process Injection into regsvr32.exe |
| E-mail address | billing@evil-corp.com | MXToolbox header analysis, Have I Been Pwned | IOA: SPF failure plus recent domain registration |
| Local artefact | HKCU\Software\Run\updater.exe | Sigma rules, EDR prevalence query, Vendor knowledge base | TTP: T1060.001 Registry Run Keys |

### Feeds, Platforms, and Why the Distinction Matters


### Sources of Cyber Threat Intelligence


### Threat Intelligence Classifications


**Answer the questions below**

---

**1. What does CTI stand for?**

**Answer:**

---

**2. IP addresses, Hashes and other threat artefacts would be found under which Threat Intelligence classification?**

**Answer:**

---

## Task 3: CTI Lifecycle


### Traffic Light Protocol (TLP)—A Primer for Proper Sharing


| TLP label | Sharing boundary | Typical SOC L1 behaviour |
|---|---|---|
| TLP: CLEAR | No restriction | Post to the internal wiki or platform. |
| TLP: GREEN | Share with peer community but not publicly | Upload to MISP/Slack workspace restricted to partner SOCs. |
| TLP: AMBER | Organisation-wide, external sharing only with need-to-know clients | Keep within the company CTI platform; reference, do not copy, in tickets. |
| TLP: RED | Named recipients only | Store in an encrypted note; do not post to the ticketing system without clearance. |

### Intel Formats


### Scenario Premise


### Step 1: Direction – Defining the Mission


### Step 2: Collection – Assembling the Raw Material


| Source | Rationale | Example artefacts collected |
|---|---|---|
| Commercial feed from the NGFW vendor | Tailored to the same firewall models, high-fidelity | 37 IPv4 addresses flagged as database-exfil C2 in the last 24 h |
| Open-source project AbuseIPDB filtered by tag PostgreSQL-brute-force | Rapid, community-driven updates | 15 IP addresses, four domains |
| The Malware Information Sharing Platform (MISP) internal to the company | Historical view of incidents | 2 SHA-256 hashes of PgSQL credential stealers |
| Vendor threat report released this week | Strategic insight distilled into technical IOCs | 1 new malware hash, 3 C2 domains |

### Step 3: Processing – Normalising and Correlating the Data


### Step 4: Analysis – Turning Information Into Judgement


| Confidence | Source agreement | Local sightings | Action |
|---|---|---|---|
| High | Same IOC in ≥2 sources | ≥1 local attempt | Immediate block |
| Medium | Single trusted source | No local hits | Alert-only |
| Low | OSINT only | No context | Monitor for 14 days |

### Step 5: Dissemination – Getting Intelligence to the Right Consumers


| Stakeholder | Format | Why they need it |
|---|---|---|
| Firewall team | CSV upload + change-ticket | They own the block rules; the ticket documents risk and indicator TLP |
| Endpoint team | YARA rule set in the EDR console | To load rules into the policy |
| CTI platform | Indicator objects with full tags | Keeps history of artefacts, supports future correlations, honours TLP |
| Management | 200-word summary in weekly cyber-risk memo | The report shows the return on investment for the process |

### Step 6: Feedback – Measuring and Refining the Cycle


| KPI | Baseline | After the first cycle |
|---|---|---|
| Median dwell time of PgSQL brute-force IPs | 48 h | 0 h (pre-emptive block) |
| False-positive rate on new blocks | — | 0% (none revoked) |

**Answer the questions below**

---

**1. At which phase of the CTI lifecycle is data converted into usable formats through sorting, organising, correlation and presentation?**

**Answer:**

---

**2. During which phase do security analysts get the chance to define the questions to investigate incidents?**

**Answer:**

---

## Task 4: CTI Standards & Frameworks

### MITRE ATT&CK


### MITRE D3FEND


### Cyber Kill Chain


| Technique | Purpose | Examples |
|---|---|---|
| Reconnaissance | Obtain information about the victim and the tactics used for the attack. | Harvesting emails, OSINT, and social media, network scans |
| Weaponisation | Malware is engineered based on the needs and intentions of the attack. | Exploit with a backdoor, a malicious Office document |
| Delivery | Covers how the malware would be delivered to the victim's system. | Email, weblinks, USB |
| Exploitation | Breach the victim's system vulnerabilities to execute code and create scheduled jobs to establish persistence. | EternalBlue, Zero-Logon, etc. |
| Installation | Install malware and other tools to gain access to the victim's system. | Password dumping, backdoors, and remote access trojans |
| Command & Control | Remotely control the compromised system, deliver additional malware, move across valuable assets and elevate privileges. | Empire, Cobalt Strike, etc. |
| Actions on Objectives | Fulfil the intended goals for the attack: financial gain, corporate espionage, and data exfiltration. | Data encryption, ransomware, and public defacement |

### CVEs, CVSS, and the NVD


### Sharing and Processing Intel


**Answer the questions below**

---

**1. What sharing models are supported by TAXII?**

**Answer:**

---

**2. When an adversary has obtained access to a network and is extracting data, what phase of the kill chain are they on?**

**Answer:**

---

## Task 5: Practical Analysis


**Answer the questions below**

---

**1. What was the source email address?**

**Answer:**

---

**2. What was the name of the file downloaded?**

**Answer:**

---

**3. After building the threat profile, what message do you receive?**

**Answer:**

---
