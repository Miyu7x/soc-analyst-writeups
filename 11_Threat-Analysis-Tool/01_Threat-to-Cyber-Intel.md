---
title: Intro to Cyber Threat Intel
module: Threat Analysis Tools
path: SOC Level 1
platform: TryHackMe
tags: [cti, threat-intelligence, mitre-attack, cyber-kill-chain, stix-taxii, soc-l1]
status: completed
date: 2026-07-10
date_completed: 2026-07-10
---
 
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/threat_intro.png width="1500">
</p>
---
 
## Task 1: Introduction
 
### Cyber Threat Intelligence for SOC L1 Analysts
 
To say that a Security Operations Center is lively is an understatement. Blinkers and alerts are always pinging and popping.
  - At the front line of it all is your **Level 1 analyst**
How do we achieve best practices of Cyber Threat Intelligence? We can start by elevating how we enrich indicators, triage alerts, and turn raw data into a written story.
  - The alert will tell the SOC that something may or may not be wrong. If something is wrong, can you explain why and prove that it is malicious?
---
 
**1. Ready to get started!**
 
**Answer:** N/A
 
---
 
## Task 2: Cyber Threat Intelligence
 
<p align="center">
<img src=screenshots/threat_intel.png width="700">
</p>
Any regular shift includes hundreds of alerts, from benign network scans to a suspicious PowerShell request.
This is where Cyber Threat Intelligence comes in, because the alerts and the queue does not stop.
  - Can you quickly spot the **real** threat amongst all the noise
  - Who, what, is on the other end of this alert indicator?
  - What is their normal pattern of actions?
  - What is my team's playbook response?
If an analyst can address these issues within minutes, this helps out the entire team down the line.
  - Suppose an alert is true-positive, quick response will buy the Incident Response team more time
### From Raw Data to Usable Intelligence
 
An IP address is just data until you provide some more information on it.
  - Who is this IP registered to?
  - Who owns the IP, malicious? Block it
In order to provide maximum information to help out the next level analyst handling the escalated alert we also need to include:
  - **IOC** Indicator of Compromise -> Evidence of a breach like a C2 address in logs
  - **IOA** Indicator of Attack -> A malicious action performed, like PowerShell launching an unknown service
  - **TTP** Tactics, Techniques and Procedures -> The attacker's actions explained by MITRE ATT&CK IDs and descriptions
| Layer | Definition | Alert-queue example | SOC L1 action |
|---|---|---|---|
| Data | An unprocessed observable | 45.155.205.3 :443 | Capture the artefact. |
| Information | Data plus factual annotation | IP registered to Hetzner, first seen 2023-07-14 | Record attributes. |
| Intelligence | Analysed information that answers so-what | IP belongs to the current BumbleBee C2; block immediately | Escalate or suppress. |
 
### Indicator Types Essential to First-Line Triage
 
Every clue left by the attacker needs a detailed enrichment about itself. Once the alert pops up, do you know where to look for evidence?
 
**Examples on How to Handle Different Types of Suspicious Evidence**
 
| Indicator | Example | First Resources | Associated IOA or TTP Examples |
|---|---|---|---|
| IPv4 / IPv6 | 45.155.205.3 | WHOIS (ASN, allocation date), VirusTotal Relations, Shodan banner scan | IOA: Repeated SSH failures. TTP: T1110.003 Password Guessing |
| Domain / FQDN | malicious-updates[.]net | WHOIS age, RiskIQ or SecurityTrails passive-DNS, urlscan.io | IOA: surge of DNS queries to a 24-hour-old domain |
| URL | hxxp://malicious-updates[.]net/login | URLhaus reputation, urlscan.io behaviour graph, Any.Run dynamic run (network off) | IOA: Browser POST to /gateway.php with payload |
| File hash | e99a18c428cb38d5… | VirusTotal static & dynamic, Hybrid-Analysis, MalShare corpus | TTP: T1055 Process Injection into regsvr32.exe |
| E-mail address | billing@evil-corp.com | MXToolbox header analysis, Have I Been Pwned | IOA: SPF failure plus recent domain registration |
| Local artefact | HKCU\Software\Run\updater.exe | Sigma rules, EDR prevalence query, Vendor knowledge base | TTP: T1060.001 Registry Run Keys |
 
---
 
**IMPORTANT TIP**
Have browser bookmarks and a folder ready, or a SIEM launcher panel that opens your preferred look-ups with the highlighted indicator pre-populated.
  - This could save some significant time between each alert
---
 
### Feeds, Platforms, and Why the Distinction Matters
 
SOCs need to work to build their intel resources; these are widely available from reliable security sources.
 
**Feeds** need to be curated properly, otherwise analysts might drown in false positives.
**Platform** is a structured repo that stores indicators, tracks enrichment, maps relationships and enforces sharing permissions.
Feeds should be introduced gradually because it takes time to adjust and make sure they fit your organization's goals, since those vary from business to business.
 
### Sources of Cyber Threat Intelligence
 
Your source of intelligence is only as good as the source that provided it. Trusted sources include:
  - Internal telemetry: SIEM logs, EDR detections, phishing-mailbox submissions
  - Commercial services: Paid sandboxes, paid vendor feeds, closed-source analytics
  - OSINT (Open Source Intelligence): AbuseIPDB, URLhaus, public blogs with IOCs, and academic research
  - Communities & ISACs: Sector-specific lists marked with labels and rich context
### Threat Intelligence Classifications
 
<p align="center">
<img src=screenshots/threat_cyber.png width="700">
</p>
What is the relationship between the victim's operational environment and the attacker? Threat intel can be broken down into the following classifications:
 
- **Strategic Intel**: High-level intelligence responsible for overlooking a company's threat landscape and the risks it runs based on current trends and emerging threats.
  - What are the current threats to our company that could disrupt business?
  - Example: A report detailing the rise of ransomware attacks in your business sector that includes data-wiping and extortion.
- **Tactical Intel**
  - Assessments of attackers' actions and behaviours through analysis of TTPs
  - Example: Sending out an advisory note detailing a new technique, T1059.005 (Visual Basic) abuse in malspam
- **Operational Intel**
  - Details on how an attacker intends to perform an attack
  - Important to understand your company's critical assets, important people and technologies that may be singled out for targeting
- **Technical Intel**
  - Atomic indicators and artefacts
  - Those are IPs and hashes, but also domains, URLs, file names, and registry keys, anything atomic enough to plug straight into a block list
**Note: L1 analysts will escalate many Technical IOCs, observe and document Tactical IOAs, and identify patterns that feed Operational reporting.**
 
---
 
**Answer the questions below**
 
---
 
**1. What does CTI stand for?**
 
**Answer:** Cyber Threat Intelligence
 
---
 
**2. IP addresses, Hashes and other threat artefacts would be found under which Threat Intelligence classification?**
 
**Answer:** Technical Intel
 
---
 
## Task 3: CTI Lifecycle
 
**6 Phases of the Intelligence Lifecycle**
 
<p align="center">
<img src=screenshots/threat_cti.png width="700">
</p>
### Traffic Light Protocol (TLP)—A Primer for Proper Sharing
 
**The Traffic Light Protocol is a four-colour labelling scheme defined by FIRST.org**
 
This is crucial for all analysts to know, and failing to appropriately label your triage can lead to a breach of contract and loss of trust with intelligence partners.
  - Analysts will gather intel after an attack; they must label it appropriately.
      - This information will be shared and updated between analysts
      - STIX, or Structured Threat Information Expression, is a template for how to appropriately share intel
| TLP label | Sharing boundary | Typical SOC L1 behaviour |
|---|---|---|
| TLP: CLEAR | No restriction | Post to the internal wiki or platform. |
| TLP: GREEN | Share with peer community but not publicly | Upload to MISP/Slack workspace restricted to partner SOCs. |
| TLP: AMBER | Organisation-wide, external sharing only with need-to-know clients | Keep within the company CTI platform; reference, do not copy, in tickets. |
| TLP: RED | Named recipients only | Store in an encrypted note; do not post to the ticketing system without clearance. |
 
### Scenario Premise
 
TryHatMe Corp holds sensitive customer data in a PostgreSQL server in a segmented subnet. The server is fronted by a next-generation firewall (NGFW) and monitored on-host by an Endpoint Detection and Response (EDR) agent. Senior management has asked the SOC to "bring in cyber-threat intelligence" so that the controls react rapidly to emerging threats. The morning-shift L1 analyst, Alex, is assigned to build and exercise that workflow.
 
### Step 1: Direction – Defining the Mission
 
Alex begins in a brief planning meeting with the CTI lead and the database administrator. Together, they translate the broad mandate into an intelligence requirement:
 
1. Primary asset. The PostgreSQL production database.
2. Business risk. Data breach fines under GDPR and loss of customer trust.
3. Available controls. NGFW is capable of IP- and domain-based blocking; EDR can quarantine files by hash.
4. Initial CTI goal. Use threat-feed indicators to:
   * Block or alert on suspicious IP addresses at the firewall, and
   * Detect known malicious file hashes at the EDR layer.
Alex constructs two measurable questions that will steer subsequent collection and analysis:
 
Q1 Which external IP addresses and domains are currently used to exploit PostgreSQL services or exfiltrate database records?
Q2 Which malware families targeting PostgreSQL drivers or credentials are active this week, and what are their file hashes?
 
The answers to Q1 and Q2 will become success criteria in later feedback.
 
### Step 2: Collection – Assembling the Raw Material
 
Guided by the questions, Alex gathers intelligence from four sources:
 
| Source | Rationale | Example artefacts collected |
|---|---|---|
| Commercial feed from the NGFW vendor | Tailored to the same firewall models, high-fidelity | 37 IPv4 addresses flagged as database-exfil C2 in the last 24 h |
| Open-source project AbuseIPDB filtered by tag PostgreSQL-brute-force | Rapid, community-driven updates | 15 IP addresses, four domains |
| The Malware Information Sharing Platform (MISP) internal to the company | Historical view of incidents | 2 SHA-256 hashes of PgSQL credential stealers |
| Vendor threat report released this week | Strategic insight distilled into technical IOCs | 1 new malware hash, 3 C2 domains |
 
### Step 3: Processing – Normalising and Correlating the Data
 
For Alex, processing threat intel will involve executing scheduled Python scripts that:
 
Normalise indicator syntax, such as having IPv6 addresses compressed, domains in lowercase, or stripping off IP subnet masks.
 
Correlate and deduplicate against the platform's existing indicator table.
 
Tag each entry with source, date, and TLP label.
 
Convert the final set of intel into two action files, such as:
 
firewall_blocklist.csv, a firewall-ready CSV that covers the indicator, action, and comments
edr_hash_rules.yar, an EDR YARA-rule file for hash blocking.
 
During processing, Alex spots a collision: one IP address appears in the NGFW feed labelled as TLP: AMBER and AbuseIPDB labels it as TLP: CLEAR. Because the stricter TLP prevails, the combined record inherits TLP: AMBER. This prevents inadvertent disclosure if the list is later shared.
 
### Step 4: Analysis – Turning Information Into Judgement
 
Blocking every indicator or artefact suspected of being malicious may create room for false positives. Alex needs to evaluate the relevance and matching between the intel gathered and the current status of the investigation. This results in providing answers to the questions from Phase 1:
 
Firewall indicators (Q1).
Cross-check: A Splunk query over the previous 30 days shows one of the NGFW IPs attempting but failing to open TCP 5432 against the production subnet. That event validates the indicator's applicability.
 
Hash indicators (Q2).
Reverse-search: OpenCTI links the new hash to the PgSteal malware family; Any.Run sandbox analysis demonstrates credential-dump behaviour. Because the organisation uses exactly the named ODBC driver, the hash is marked as high priority, and relevance is confirmed.
 
Seven IP addresses and one hash meet the high bar; the rest are queued as medium and set for monitoring.
 
Alex grades each indicator:
 
| Confidence | Source agreement | Local sightings | Action |
|---|---|---|---|
| High | Same IOC in ≥2 sources | ≥1 local attempt | Immediate block |
| Medium | Single trusted source | No local hits | Alert-only |
| Low | OSINT only | No context | Monitor for 14 days |
 
### Step 5: Dissemination – Getting Intelligence to the Right Consumers
 
Unused threat intelligence becomes shelfware. Alex delivers tailored output:
 
By tailoring depth to the audience, Alex ensures that each consumer receives only the detail required to act, nothing more and nothing less.
 
| Stakeholder | Format | Why they need it |
|---|---|---|
| Firewall team | CSV upload + change-ticket | They own the block rules; the ticket documents risk and indicator TLP |
| Endpoint team | YARA rule set in the EDR console | To load rules into the policy |
| CTI platform | Indicator objects with full tags | Keeps history of artefacts, supports future correlations, honours TLP |
| Management | 200-word summary in weekly cyber-risk memo | The report shows the return on investment for the process |
 
### Step 6: Feedback – Measuring and Refining the Cycle
 
Two weeks later, metrics and reports from the firewall and EDR show the progress of establishing an effective threat intelligence workflow:
 
Because the outcome meets the initial direction objectives, management approves expansion: the next sprint will add other threat parameters, such as DNS tunnelling IOCs for the same asset group. Alex updates the direction document, closes the feedback loop, and schedules the second iteration.
 
| KPI | Baseline | After the first cycle |
|---|---|---|
| Median dwell time of PgSQL brute-force IPs | 48 h | 0 h (pre-emptive block) |
| False-positive rate on new blocks | — | 0% (none revoked) |
 
---
 
**Answer the questions below**
 
---
 
**1. At which phase of the CTI lifecycle is data converted into usable formats through sorting, organising, correlation and presentation?**
 
**Answer:** Processing
 
---
 
**2. During which phase do security analysts get the chance to define the questions to investigate incidents?**
 
**Answer:** Direction
 
---
 
## Task 4: CTI Standards & Frameworks
 
Standards and frameworks are really important in the security field because they maintain a baseline of common terminology shared across the industry.
  - Using the same terms facilitates communication, collaboration and understanding throughout.
  - When writing up an alert, it makes it so anybody around you can understand exactly what you're saying.
### MITRE ATT&CK
 
Every ticket you write as an analyst requires a label; having one set per technique ensures that if you are talking about PowerShell, it is always **T1059.001**
  - No matter the language spoken, a DNS tunnel will always be T1048.003
  - When writing your ticket you can use MITRE ATT&CK to tell the story of the alert
      - Match the suspicious behaviour to a **Tactic/Technique** pair
      - Write the matching ID in the triage note: "Observed T1071.001 (web-based C2) against FINANCE-TRYHATME-00"
      - Escalate it to L2 and they will instantly know the mitigation process to start
### MITRE D3FEND
 
Attackers also have to be planned against defensively; this is where MITRE D3FEND tactics and techniques come into play.
Example: As a SOC, your proxy alerts on **T1048.003 DNS tunnel**
  - SOC searches D3FEND for the matching defensive technique: D3-NTDN DNS-request analysis. The page lists practical controls: block extensive TXT records and alert on uncommon query entropy.
  - Next on your ticket you write the diagnosis you just found, plus mitigation steps
### Cyber Kill Chain
 
<p align="center">
<img src=screenshots/threat_killchain.png width="700">
</p>
The Cyber Kill Chain can be defined as adversary actions broken down into steps.
  - This way analysts can pinpoint where in the chain they caught the alert.
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
 
As a SOC analyst you need to understand how to identify and organise vulnerability notifications.
 
  - CVE, Common Vulnerabilities and Exposures: a numbered catalogue of known vulnerabilities, e.g. CVE-2023-4863
  - CVSS, Common Vulnerability Scoring System: a 0 to 10 severity scale
  - NVD, National Vulnerability Database: the main repo that links CVEs to CVSS scores, exploits and affected products
### Sharing and Processing Intel
 
All shared intel should follow the STIX format or the TAXII protocol; sharing information is beneficial. Shared knowledge helps everyone around you stay safe from information they may not have had prior to your discovery. However, not every indicator should be shared. It's important to understand your company's privacy laws, customer NDAs, and internal competitive information, which may forbid disclosure. Sharing specific IOCs too early can also tip off attackers that you've caught on to their scent, letting them alter their plans.
 
**Answer the questions below**
 
---
 
**1. What sharing models are supported by TAXII?**
 
**Answer:** Collection and Channel
 
---
 
**2. When an adversary has obtained access to a network and is extracting data, what phase of the kill chain are they on?**
 
**Answer:** Actions on Objectives
 
---
 
## Task 5: Practical Analysis
 
<p align="center">
<img src=screenshots/threat_practical.png width="700">
</p>
As part of the dissemination phase of the lifecycle, Cyber Threat Intelligence is shared between organisations through what are called **threat reports.**
  - These reports inform on emerging and actively used threat vectors
  - This is valuable to share with your organisation's stakeholders
---
 
All the things we have discussed come together when mapping out an adversary based on threat intel. To better understand this, we will analyse a simplified engagement example. Click on the green "View Site" button in this task to open the Static Site Lab and navigate through the security monitoring tool on the right panel, and fill in the threat details.
 
---

**1. What was the source email address?**

<p align="center">
<img src=screenshots/threat_scenario.png width="700">
</p>

**Answer: vipivillain@badbank.com**

---

**2. What was the name of the file downloaded?**

**Answer: flbpfuh.exe**

---
 
**3. After building the threat profile, what message do you receive?**
 
<p align="center">
<img src=screenshots/threat_flag.png width="700">
</p>
- **Threat Actor Extraction IP Address?** Outbound network flow initiated to 91.185.23.222
- **Threat Actor Email Address?** Email received by John Doe from vipivillain@badbank.com
- **Malware Tool?** File download initiated by John Doe. File name: flbpfuh.exe
- **User Victim Logged Account?** Account logged on successfully. Account name: Administrator
- **Victim Email Recipient?** Account logged off successfully. Account name: John Doe
 
**Answer:** THM{NOW_I_CAN_CTI}
 
---
