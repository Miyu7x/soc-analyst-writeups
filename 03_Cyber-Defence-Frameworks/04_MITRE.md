---
title: MITRE
module: Cyber Defence Frameworks
path: SOC Level 1
platform: TryHackMe
tags:
  - MITRE
  - ATT&CK
  - CAR
  - D3FEND
  - CTI
  - threat-intelligence
  - SOC
status: completed
date: 2026-04-17
date completed: 2026-04-18
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

**MITRE: To solve problems for a safer world** is a security organization that conducts research across a range of domains:
- Cybersecurity
- Artificial Intelligence
- Healthcare
- Space Systems

MITRE's frameworks include:
- MITRE ATT&CK framework
- The CAR knowledge base
- D3FEND
- MITRE Engage and others

### Task Questions

1. I understand the learning objectives and am ready to learn about MITRE!
   - **Answer:**

---

## Task 2 - ATT&CK® Framework

### Key Concepts

**MITRE ATT&CK** is a public database of known adversary **tactics and techniques.**
- This information is compiled to create **threat models and methodologies** for companies, government, the cybersecurity industry, and the public.

| Term | Definition | Example |
|------|-----------|---------|
| Tactic | The attacker's goal and objective -- the "why" behind the attack | Reconnaissance |
| Technique | How the attacker achieves their goal | Active Scanning |
| Procedure | The specific method used to execute the technique | Scanning IP blocks to map the target network |

![MITRE](screenshots/T2_mitre_matrix.png)

### Task Questions

1. What Tactic does the Hide Artifacts technique belong to in the ATT&CK Matrix?
   - **Answer: Defense Evasion**

2. Which ID is associated with the Create Account technique?
   ![T1136 Create Account](screenshots/T2_t1136_createaccount.png)
   - **Answer: T1136**

---

## Task 3 - ATT&CK in Operation

### Key Concepts

**ATT&CK** catalogs information using standard terminology and unique IDs, making it easy to compare data across any platform, job role, or incident. Defenders use it to translate threat intelligence into real detection logic, queries, and SOC playbooks.

| Role | Goal | How They Use ATT&CK |
|------|------|----------------------|
| CTI Teams | Collect and analyze threat information | Map observed threat actor behavior to ATT&CK TTPs to build actionable profiles |
| SOC Analysts | Investigate and triage security alerts | Link activity to tactics and techniques to add context and prioritize incidents |
| Detection Engineers | Design and improve detection systems | Map SIEM and EDR rules to ATT&CK to ensure better detection coverage |
| Incident Responders | Respond to and investigate security incidents | Map incident timelines to MITRE tactics and techniques to visualize the attack |
| Red & Purple Teams | Emulate adversary behavior to test defenses | Build emulation plans aligned with techniques and known group operations |

**Mustang Panda** is a threat actor group that uses phishing as their preferred attack technique. Their behavior is mapped out in MITRE ATT&CK below.

![Mustang Panda's preferred attack method](screenshots/T3_mustangpanda.png)

### Task Questions

1. In which country is Mustang Panda based?
   - **Answer: China**

2. Which ATT&CK technique ID maps to Mustang Panda's Reconnaissance tactics?
   ![Mustang Panda Recon](screenshots/T3_t1598.png)
   - **Answer: T1598**

3. Which software is Mustang Panda known to use for Access Token Manipulation?
   - **Answer: Cobalt Strike**

---

## Task 4 - ATT&CK for Threat Intelligence

![CTI Mapping](screenshots/T4_cti_mapping.png)

### Key Concepts

![THM Scenario](screenshots/T4_scenario_mitre.png)

### Task Questions

1. Which APT group has targeted the aviation sector and has been active since at least 2013?
   ![APT33 aviation threat group](screenshots/T5_apt33.png)
   - **Answer: APT33**

2. Which ATT&CK sub-technique used by this group is a key area of concern for companies using Office 365?
   ![APT33 and Office 365](screenshots/T4_apt33_office365.png)
   - **Answer: Cloud Accounts**

3. According to ATT&CK, what tool is linked to the APT group and the sub-technique you identified?
   ![Sub-technique used by APT33](screenshots/T4_apt33_ruler.png)
   - **Answer: Ruler**

4. Which mitigation strategy advises removing inactive or unused accounts to reduce exposure to this sub-technique?
   ![Mitigation for Ruler](screenshots/T4_apt33_mitigation.png)
   - **Answer: User Access Management**

5. What Detection Strategy ID would you implement to detect abused or compromised cloud accounts?
   ![DET0546 Detection Strategy for compromised cloud accounts](screenshots/T4_detection_strategy.png)
   - **Answer: DET0546**

---

## Task 5 - Cyber Analytics Repository (CAR)

### Key Concepts

**ATT&CK** tells you what attackers do. **CAR** tells you how to catch them doing it.

Think of CAR as a detection recipe. For each attacker behavior it documents, CAR provides the logic for spotting it and ready-made queries you can drop into Splunk or run in EQL. Like ATT&CK, CAR uses a consistent data model with uniform terminology and standardized IDs, so analysts can find and understand detections quickly without having to reverse-engineer the reasoning behind them.

**Short version: CAR is ATT&CK with the "okay but how do I actually detect this in my SIEM" problem already solved for you.**

### Task Questions

1. Which ATT&CK Tactic is associated with CAR-2019-07-001?
   ![Defense Evasion associated with CAR-2019-07-001](screenshots/T5_car_repo.png)
   - **Answer: Defense Evasion**

2. What is the Analytic Type for Access Permission Modification?
   ![Situational Awareness is the analytic type for Access Permission Modification](screenshots/T5_analytic_type.png)
   - **Answer: Situational Awareness**

---

## Task 6 - MITRE D3FEND Framework

### Key Concepts

![D3FEND matrix](screenshots/T6_d3fend.png)

Where ATT&CK maps how attackers operate, **D3FEND** maps how defenders can stop them. The two frameworks are designed to complement each other -- ATT&CK gives you the attacker's move, D3FEND gives you the countermeasure.

**D3FEND stands for:** Detection, Denial, Disruption Framework Empowering Network Defense.

| Framework | Focus | Purpose |
|-----------|-------|---------|
| ATT&CK | Adversary tactics and techniques | Understand how attackers operate and map threat behavior |
| D3FEND | Defensive techniques and controls | Understand how to detect, deny, and disrupt those attacks |

### Task Questions

1. Which sub-technique of User Behavior Analysis would you use to analyze the geolocation data of user logon attempts?
   ![User Behavior Analysis](screenshots/T7_user_geolocation.png)
   - **Answer: User Geolocation Logon Pattern Analysis**

2. Which digital artifact does this sub-technique rely on analyzing?
   ![User Geo artifacts](screenshots/T7_usergeo_logon.png)
   - **Answer: Network Traffic**

---

## Task 7 - Other MITRE Projects

### Key Concepts

| Project | Focus Area | Key Use Case |
|---------|-----------|--------------|
| Adversary Emulation Library | Real-world threat group emulation plans | Step-by-step guides to replicate specific threat group behavior in controlled exercises |
| CALDERA | Automated adversary emulation | Simulate attacker behavior using ATT&CK to test and improve detection and response |
| AADAPT | Digital asset and blockchain threats | Helps defenders understand adversarial tactics targeting blockchain, smart contracts, and digital wallets |
| ATLAS | AI and machine learning system threats | Documents real-world attack techniques and mitigations specific to AI technology |

### Task Questions

1. What technique ID is associated with Scrape Blockchain Data in the AADAPT framework?
   ![Technique ID for Scrape Blockchain Data](screenshots/T8_scrapeblockchain.png)
   - **Answer: ADT3025**

2. Which tactic does LLM Prompt Obfuscation belong to in the ATLAS framework?
   ![LLM Prompt Obfuscation tactic](screenshots/T8_llmobfuscation.png)
   - **Answer: Defense Evasion**

---

## Task 8 - Conclusion

### Key Concepts

MITRE feels like the backbone of any SOC playbook. Between ATT&CK, CAR, D3FEND, and the emulation tools, there is a full ecosystem built around understanding and stopping adversaries. The Adversary Emulation Library is especially interesting -- I cannot wait to get to the point in my career where I am running those exercises.

### Task Questions

1. Complete the room and continue on your cyber learning journey!
   - **Answer:**

---

## Personal Notes

<!-- Anything that surprised you, clicked hard, or you want to revisit -->
<!-- Connections to other rooms or real-world scenarios -->
<!-- Questions to research further -->
