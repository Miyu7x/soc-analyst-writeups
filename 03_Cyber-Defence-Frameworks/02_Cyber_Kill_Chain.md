---
title: Cyber Kill Chain
module: Cyber Defence Frameworks
path: SOC Level 1
platform: TryHackMe
tags: [cyber-kill-chain, reconnaissance, osint, phishing, malware, persistence, c2, exfiltration, threat-intelligence, blue-team, defence-frameworks, mitre-attack]
status:
date:
date_completed:
---

# Cyber Kill Chain

## Task 1 - Introduction

### Key Concepts

**Cyber Kill Chain** is the military concept of the structure of an attack
- Identify target
- Decide how to proceed
- Attack target
- Eliminate target

In the **Cyber Kill Chain** framework the attacker must go through all the steps of the Kill Chain in order to perform a successful attack 

As an **SOC** understanding the Cyber Kill Chain will allow us to recognize the intrusion attempts and the attackers goals

### Task Questions

1. Read the above.

---

## Task 2 - Reconnaissance

### Key Concepts

**Reconnaissance**

![Recon Miyu](screenshots/recon_miyu.png)

**Reconnaissance** is the research the attacker does on the target
- Gather Information
	- structure details
	- employee data
	- business processes 
	- exposed technologies
	  
**OSINT or Open Source Intellignece** gives attackers all public details about a target, the attackers will use:
- Search engines
- Media
- Social Media accounts
- Forums and Blogs
- Online Public Records Database
- WHOIS and Techinical data

**Passive Recon** NO direct contact with the target
**Active Recon** DIRECT contact with the target
	- social engineering
	- port scanning
	- banner grabbing

![Email harvesting examples](screenshots/T2_emailharvesting_examples.png)
### Task Questions

1. What is the name of the Intel Gathering Tool that is a web-based interface to the common tools and resources for open-source intelligence?
   - **Answer: OSINT Framework**

2. What is the definition for the email gathering process during the stage of reconnaissance?
   - **Answer: Email Harvesting**

---

## Task 3 - Weaponization

![All weapons loaded](screenshots/T2_miyusfullassault_room.png)

### Key Concepts

In this stage the attacker is choosing its **WEAPON**
- **Malware** a program or software; damages, disrupts, controls the target
- **Exploit** a program or code that takes advantage of *vulnerability or flaw* in the application or system
- **Payload** a malicious code the **attacker** runs on the victims system


### Task Questions

1. What is the term for automated scripts embedded in Microsoft Office documents that can be used to perform tasks or exploited by attackers for malicious purposes?
   - **Answer: Macro**

---

## Task 4 - Delivery

![Fresh Phish Delivery](screenshots/T4_fresh_phish_delivery.png)
### Key Concepts

How does the **attacker deliver** his goods? 
- **Phishing email**
- **USB drops**
- **Watering Hole Attacks** targets a specific group of people who visit a specific website
	- Attackers will create redirection links
	- Provide downloadable malware

### Task Questions

1. What do you call an attack targeting a specific group by infecting their frequently visited website?
   - **Answer: Watering Hole Attack**

---

## Task 5 - Exploitation

### Key Concepts

![](T5_accessgranted.png)

This is the hackers **AH-HA** moment, their code has executed in the victims machine

The victim could have been compromised by;
- Mailicious macro, phishing email
- Zero-day exploits: an exploit that hasnt been seen or documented before
- Known CVE's the attacker found a open known vulnerability and exploited it

### Task Questions

1. What is the term for a cyber attack that exploits a software vulnerability that is unknown by software vendors?
   - **Answer: Zero-Day**

---

## Task 6 - Installation

![Installation Process](screenshots/Installation_Miyu.png)

### Key Concepts

Installation is key if the attacker wants to maintain access to the victims system
- Install a web shell
- Use Meterpreter to install a backdoor
- Modify a Windows Service to execute malicious scripts
- Adding the entry to the "run keys" for the malicious payload in the Registry or the Startup Folder
		- Will execute every time the victim logs on 
**Timestomping** will disguise malware as a legitimate program 
		- Modifies timestamps
		- Modify, Access, Create, Change Times
### Task Questions

1. What technique is used to modify file time attributes to hide new or changes to existing files?
   - **Answer: Timestomping**

2. What malicious script can be planted by an attacker on the web server to maintain access to the compromised system and enables the web server to be accessed remotely?
   - **Answer: Web Shell**

---

## Task 7 - Command & Control

![Command and Control Miyu](screenshots/C2_Miyu.png)
### Key Concepts

Following persistence the attacker executed the malware now they **FULL CONTROL**

**C2 or Command and Control** lets the attacker take full ownership of the victims system


| C2 Channel    | Port(s) | Why It Blends In                    |
| ------------- | ------- | ----------------------------------- |
| HTTP          | 80      | Blends Legit and Malicious Traffic  |
| HTTPS         | 443     |                                     |
| DNS Tunneling |         |                                     |

### Task Questions

1. What is the C2 communication where the victim makes regular DNS requests to a DNS server and domain which belong to an attacker?
   - **Answer: DNS Tunneling**

---

## Task 8 - Actions on Objectives (Exfiltration)

![Jackpot for the attacker](Miyus_Exfiltration.png)

### Key Concepts

After the 6 phases of the attack, the attacker gets their prize:
- User credentials
- Privilege Escalation
- Lateral Movement 
- Collect and exfiltrate data
- Delete backups and Shadow Copy(Microsoft Tech that creates backup copies, files, volumes)
### Task Questions

1. What technology is included in Microsoft Windows that can create backup copies or snapshots of files or volumes on the computer, even when they are in use?
   - **Answer: Shadow Copy**

---

## Task 9 - Practice Analysis

### Key Concepts
<!-- The Target breach (November 2013) -- map each Kill Chain phase to the attack -->
<!-- Six items to place: exploit public-facing application, data from local system, powershell, dynamic linker hijacking, spearphishing attachment, fallback channels -->

### Task Questions

1. What is the flag after you complete the static site?
   - **Answer: THM{7HR347_1N73L_12_4w35om3}**

---

## Task 10 - Conclusion

### Key Concepts

The last part to get the flag im still unclear why powershell sits at the "bug" level their "second" stage.... I understand why it could be under weaponization but still unclear with some of these THM challenges.
### Task Questions

1. Read the above.

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*