---
title: Unified Kill Chain
module: Cyber Defence Frameworks
path: SOC Level 1
platform: TryHackMe
tags:
  - ukc
  - kill-chain
  - threat-modelling
  - mitre
  - frameworks
status: completed
date: 2026-04-17
date_completed: 2026-04-17
---

# Unified Kill Chain
![Unified Kill Chain](screenshots/T1_Miyus_UnifiedKillchain.jpg)

---

## Task 1 - Introduction

### Key Concepts

When you say the offense is a good defense this is what it means! If we understand what the attacker is doing or going to do we can anticipate and block those avenues


### Task Questions
1. Let's proceed!
   - **Answer: Check**

---

## Task 2 - What is a "Kill Chain"

### Key Concepts

The full attack a hacker creates from beginning to the end, a chain of events;
- recon
- weaponization
- delivery
- exploitation
- installation
-  C2
- Actions on objectives


### Task Questions
1. Where does the term "Kill Chain" originate from?
   - **Answer: Military**

---

## Task 3 - What is "Threat Modelling"

### Key Concepts
**Threat Modelling** in cyberseucirty are the steps taken to improve a security system
- Identify Risk
- Asses Vulnerabilities
- Create a Plan of Action
- Putting in Place Policies to avoid future vulnerabilities

**Threat Modelling** is also responsible for creating a high level overview of an organisation IT assets which includes the companys hardware and all software

**Threat Modelling Frameworks** 
- **STRIDE** Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation Privilege

- **DREAD** A system used by Microsoft to asses risk to computer security threats

- **CVSS** Common Vulnerability Scoring System

---
### Task Questions
1. What is the technical term for a piece of software or hardware in IT?
   - **Answer: Asset**

---

## Task 4 - Introducing the Unified Kill Chain

### Key Concepts

The **Unified Kill Chain** believes there is 18 phases to an attack, this model aims to complement MITRE ATT&CK and Lockheed Martins
- Extremely detailed
- Modern released in 2017

![18 Phases of the Unified kill chain](screenshots/T4_18phases.png)


### Task Questions
1. In what year was the Unified Kill Chain framework released?
   - **Answer: 2017**
1. According to the Unified Kill Chain, how many phases are there to an attack?
   - **Answer: 18**
1. What is the name of the attack phase where an attacker employs techniques to evade detection?
   - **Answer: Defense Evasion**
1. What is the name of the attack phase where an attacker employs techniques to remove data from a network?
   - **Answer: Exfiltration**
1. What is the name of the attack phase where an attacker achieves their objectives?
   - **Answer: Objectives**

---

## Task 5 - Goal: In (Initial Foothold)

### Key Concepts
<!-- Purpose of the "In" phase -->
<!-- Reconnaissance: passive vs active -->
<!-- Weaponization: C2 setup, payload delivery -->
<!-- Social Engineering tactics -->
<!-- Exploitation: code execution abuse -->
<!-- Persistence mechanisms -->
<!-- Defence Evasion: bypassing firewalls, AV, IDS -->
<!-- Command & Control: communications establishment -->
<!-- Pivoting: reaching internal systems -->

| Phase | MITRE Tactic | Description |
|---|---|---|
| Reconnaissance | TA0043 | |
| Weaponization | TA0001 | |
| Social Engineering | TA0001 | |
| Exploitation | TA0002 | |
| Persistence | TA0003 | |
| Defence Evasion | TA0005 | |
| Command & Control | TA0011 | |
| Pivoting | TA0008 | |

### Task Questions
1. What is an example of a tactic to gain a foothold using emails?
   - **Answer: Phishing**
1. Impersonating an employee to request a password reset is a form of what?
   - **Answer: Social Engineering**
1. An adversary setting up the Command & Control server is what phase of the Unified Kill Chain?
   - **Answer: Weaponization**
1. Exploiting a vulnerability present on a system is what phase of the Unified Kill Chain?
   - **Answer: Exploitation**
1. Moving from one system to another is an example of?
   - **Answer: Pivoting**
1. Leaving behind a malicious service that allows the adversary to log back into the target is what?
   - **Answer: Persistence**

---

## Task 6 - Goal: Through (Network Propagation)

### Key Concepts

The **Through phase** means the attacker has successfully established a foothold on the victims system

**Pivoting** is an essential step for any attacker, they set up C2, this is their gateway to all other systems on the network

**Discovery** the attcker pulls information about the entire system and the network its connected to

**Privilage Escalation** this is crucial to gain acces to more important accounts and information
- SYSTEM/ROOT
- Local Amdinistrator
- A user with amdin like access
- A user account with specific allowed access

**Execution** the attacker deploys the malicious code

**Credential Access** works hand in hand with the Privilege Escalation, the attacker is now tryin to steal account info, passwords..

**Lateral Movement** the attacker can move through the computers in the compromised network its no longer just in the infected machine

| Phase | MITRE Tactic | Description |
|---|---|---|
| Pivoting | TA0008 | |
| Discovery | TA0007 | |
| Privilege Escalation | TA0004 | |
| Execution | TA0002 | |
| Credential Access | TA0006 | |
| Lateral Movement | TA0008 | |

### Task Questions
1. Failed logins from an administrator account -- what phase is the attacker seeking to achieve?
   - **Answer: Privilege Escalation**
1. Mimikatz detected attempting to dump OS and user secrets -- which phase does this correspond to?
   - **Answer: Credential Access**

---

## Task 7 - Goal: Out (Action on Objectives)


![The hacker gets their prize](screenshots/Miyus_Exfiltration.png)
### Key Concepts

The **Out** phase means the hacker has concluded their goal successfully
- Compromised the **Confidentiality Integrity and Availability** of the system

| Phase        | MITRE Tactic | Description                                                                                                        |
| ------------ | ------------ | ------------------------------------------------------------------------------------------------------------------ |
| Collection   | TA0009       | The hacker collects all the data they pretend to exfil                                                             |
| Exfiltration | TA0010       | The hacker now attempts to exfiltrate the data out of the victims system                                           |
| Impact       | TA0040       | The hacker compromises the integrity and availability of the assets                                                |
| Objectives   | -            | Depending on the hackers motivation, ransomware, reputation damage they will release the information to the public |

### Task Questions
1. Big traffic spike sent to unknown suspicious IP -- what UKC phase could describe this?
   - **Answer: Exfiltration**
1. PII released publicly -- what part of the CIA triad is affected?
   - **Answer: Confidentiality**

---

## Task 8 - Practical

### Key Concepts

The Attacker uses tools to gather information about a system: **Reconnaissance**
The Attacker installs a malicious script to allow them remote access at a later date: **Persistence**
The hacked machine is being controlled from an Attacker's own server: **Command and Control**
The Attacker uses the hacked machine to access other servers on the same network: **Pivoting**
The Attacker steals a database and sells this to a 3rd party: **Action and Objectives**

### Task Questions
1. What is the flag?
   - **Answer: THM{UKC_SCENARIO}**

---

## Task 9 - Conclusion

### Key Concepts

This was a nice refresher com the cyber kill chain, you can really see how vital having 18 phases really is. This modern take on the UKC really put the whole picture together

### Task Questions
1. Complete this task to finish the room!
   - **Answer:**

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*