---
title: Summit
module: Cyber Defence Frameworks
path: SOC Level 1
platform: TryHackMe
tags:
  - pyramid-of-pain
  - detection-engineering
  - purple-team
  - malware-analysis
  - threat-simulation
status: in-progress
date: 2026-04-20
date_completed:
---

# Summit

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Objective

PicoSecure is running a threat simulation and detection engineering engagement to strengthen malware detection. Working alongside an external penetration tester in an iterative purple-team scenario, the goal is to configure security tools to detect and block malware samples as the tester attempts to execute them on a simulated workstation.

The framework guiding the exercise is the **Pyramid of Pain** - each sample corresponds to an ascending tier of the pyramid, increasing the adversary's cost of operations with every detection.

---

## Environment Setup

- **Lab URL:** `https://LAB_WEB_URL.p.thmlabs.com`
- **Machine start time:**
- **Notes:**
  
  ![Summit Intro](T1_summit_intro.png)

---

## Pyramid of Pain - Detection Log

| Sample | Pyramid Tier | Indicator Type | Detection Method | Flag |
|--------|-------------|----------------|-----------------|------|
| sample1.exe | | | | |
| sample2.exe | | | | |
| sample3.exe | | | | |
| sample4.exe | | | | |
| sample5.exe | | | | |
| Sphinx (final) | | | | |

---

**Sandbox the malware**

![Sandbox malware](screenshots/T1_sandbox_malware.png)

## Task 1 - Challenge

![Added malware MD5 hash to blocklist](screenshots/Q1_add_hash_blocklist.png)

![First flag was in mailbox](screenshots/T1_firstflag.png)

### Sample 1

**Pyramid Tier: 1 **

**Indicator Used:**

**Detection Approach: Used sandbox on suspicious file**

**Notes: Sandboxed sample1.exe found trojan malware and hases MD5, SHA1, and SHa256
- Added MD5 hash to the blocklist**

- **Flag: THM{f3cbf08151a11a6a331db9c6cf5f4fe4}**

---

### Sample 2

![Malware sample2.exe sandbox](screenshots/T2_sandbox_example.png)

![Created a firewall rule to block connection with malicious IP](screenshots/T2_q2firewall_rule.png)


**Pyramid Tier:**

**Indicator Used: Malicious IP**

**Detection Approach: Used sandbox to scan sample2.exe**

**Notes: Noticed malicious IP connection so we created a firewall rule to block connection with the malicious IP**

- **Flag: THM{2ff48a3421a938b388418be273f4806d}**

---

### Sample 3

![Results for sandboxed sample3.exe](screenshots/T3_q3_sandbox_example.png)

![Created a rule for blocking DNS backdoor Trohjan](screenshots/T3_dns_backdoor.png)

**Pyramid Tier:**

**Indicator Used: Supicious DNS**

**Detection Approach: Sandboxed sample3.exe found suspicious DNS and backdoor Trojan**

**Notes: Upon sandboxing the sample3.exe we found it was a persistence backdoor Trojan so we blocked the DNS connection **

- **Flag: THM{4eca9e2f61a19ecd5df34c788e7dce16}**

---

### Sample 4

![](screenshots/T4_q4sandbox_example.png)

![Sigma rule builder](screenshots/T4_defense_evasion_sysmon.png)

**Pyramid Tier:**

**Indicator Used: **

**Detection Approach: **

**Notes: Observed a malicious process tree in HKEYs in attempt to disable security monitoring*

- **Flag: THM{c956f455fc076aea829799c0876ee399}**

---

### Sample 5

![Sandbox sample5.exe](screenshots/T5_q5sandbox_example.png)

![](T5_sigma_rule.png)
**Pyramid Tier:**

**Indicator Used:**

**Detection Approach:**

**Notes: Review logs and determine the frequency of POST reuqests**

- **Flag: THM{46b21c4410e47dc5729ceadef0fc722e}**

---

### Final - Sphinx

![Exfil Log](screenshots/T5_exfil_log.png)

![Sample6.exe sandbox](screenshots/T6_sandbox_example.png)

![](T6_sigma_exfill_rule.png)

**Pyramid Tier:**


**Indicator Used:**

**Detection Approach Review logs**

**Notes: Created a sigma rule to stop the process of exfiltration*

- **Flag: THM{c8951b2ad24bbcbac60c16cf2c83d92c}**

---

## Key Concepts



---

## Reflections

This was a very good exercise to get me thinking like an SOC, reviewing the attack and on the sandbox and having to setp firewall rules, dns rules, and sigma rules was really good for building the mental blocks of a daily SOC work flow.