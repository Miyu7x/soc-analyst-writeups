---
title: Introduction to SOAR
module: Core SOC Solutions
path: SOC Level 1
platform: TryHackMe
tags:
  - soar
  - soc
  - orchestration
  - automation
  - response
  - playbooks
  - alert-triage
status: in-progress
date: 2026-04-13
date completed: 2026-04-13
---
# Introduction to SOAR

---

## Task 1 - Introduction

### Key Concepts

A SOC analyst's job can be exhausting - too many manual processes, too many disconnected tools, and constant communication overhead across teams. SOAR (Security Orchestration, Automation, and Response) was built to address exactly those pain points.

### Task Questions

1. Let's get started!
   - **Answer:**

---

## Task 2 - Traditional SOC and Challenges

### Key Concepts

A SOC's effectiveness is measured by its ability to continuously monitor, analyze, and respond to security incidents. That work happens across four core capabilities.

**Monitoring and Detection** - 24/7 scanning and flagging of suspicious activity within the network. Primary tool: SIEM.

**Recovery and Remediation** - When a threat is confirmed, the SOC team acts as first responders. Tools used: EDR, firewalls, IAM. Actions include isolating endpoints, removing malware, shutting down infected systems, and stopping malicious processes.

**Threat Intelligence** - Attackers constantly develop new exploits. Analysts maintain awareness through continuous threat feeds covering IP addresses, domains, file hashes, and other indicators.

**Communication** - SOC analysts do not work in isolation. Coordinating with IT teams and management is a core skill, including knowing how to escalate an incident clearly and effectively to a Tier 2 analyst or above.

**Challenges faced by SOCs:**
- Alert fatigue - too many events, many of them false positives
- Too many disconnected tools - no unified view of the environment
- Manual processes - undocumented tribal knowledge slows response
- Talent shortage - growing threat landscape with limited experienced analysts

### Task Questions

1. How would you describe the experience of an overload of security events being triggered within a SOC?
   - **Answer: Alert Fatigue**

---

## Task 3 - Overcoming SOC Challenges with SOAR

### Key Concepts

SOAR (Security Orchestration, Automation, and Response) unifies the security tools a SOC uses - SIEM, EDR, firewall, IAM, threat intelligence platforms - into a single interface. Analysts no longer need to switch between tools during an investigation.

The three capabilities that make SOAR work:

**Orchestration** - Connects all security tools and defines workflows called playbooks. A playbook is a predefined set of steps that tells SOAR how to investigate a specific type of alert. The quality of automation is only as good as the analyst who wrote the playbook.

**Automation** - SOAR executes the playbook automatically, without manual intervention. Example for a VPN brute force alert:
- SOAR receives the alert from SIEM
- Automatically queries login history for the user
- Automatically checks the IP against threat intelligence platforms
- If the IP is malicious, automatically disables the user in IAM
- Automatically opens a ticket with all investigation details

**Response** - SOAR takes containment and remediation actions across tools from one interface, automatically and at speed.

![What SOAR does](screenshots/T3_soar_table.png)

![Before and after SOAR](screenshots/T3_before_after_soar.png)

**Do we still need SOC analysts?** Yes - and this is important. SOAR handles the repetitive, documented work. Analysts are still essential for judgment calls at critical decision points, understanding threats in business context, and building and maintaining the playbooks themselves. The automation is only as smart as the person who designed it.

### Task Questions

1. The act of connecting and integrating security tools and systems into seamless workflows is known as?
   - **Answer: Orchestration**

2. What do we call a predefined list of actions to handle an incident?
   - **Answer: Playbook**

---

## Task 4 - Building SOAR Playbooks

### Key Concepts

Playbooks are built for recurring, predictable alert categories. The two examples covered here - phishing and CVE patching - both follow branching logic rather than a linear checklist. Each decision point changes the path, which means the playbook can handle variation without requiring a human at every step.

**Phishing Playbook** - Phishing is one of the most common attack vectors. The playbook starts at "suspicious email received," creates a ticket, then checks whether the email contains a URL or attachment. From there the path branches based on what is found. Most steps are automated; an analyst steps in only when a judgment call is needed.

![Phishing playbook](screenshots/T4_phishing_playbook_example.png)

**CVE Patching Playbook** - New CVEs are published constantly. Without automation, the patching backlog grows and environments stay vulnerable. The playbook fetches new CVEs from advisory lists, assesses risk, creates a patching ticket, tests the patch, and pushes to production. If assets remain vulnerable after patching, the SOC develops a mitigation plan. An analyst appears at the points where human verification or decision-making is required.

![CVE patching playbook](screenshots/T4_cve_playbook_example.png)

### Phishing Playbook - Key Decision Points

| Step | Condition | Path A | Path B |
|---|---|---|---|
| Initial triage | Contains URL or attachment? | Yes - continue analysis | No - notify user and close |
| Content check | URL or attachment? | URL - check reputation via TI | Attachment - detonate in sandbox |
| Verdict | Malicious confirmed? | Remediate and escalate | Mark false positive and close |

### Task Questions

1. Is manual analysis vital within a SOAR workflow? Yay or Nay?
   - **Answer: Yay**

2. From where is the CVE Patching playbook fetching the new CVEs?
   - **Answer: Advisory Lists**

3. In the CVE Patching playbook, if the assets are found vulnerable even after the patch is deployed, what does the SOC develop next?
   - **Answer: Mitigation Plan**

---

## Task 5 - Threat Intel Workflow Practical

### Key Concepts

The practical puts you in the role of a SOC analyst adopting SOAR for the first time after a slow, manual breach investigation. The task is to build a threat intelligence integration workflow by configuring each component of the automation chain and testing it until the full flow runs cleanly.

Each stage had to be activated in the correct order - case ticket management, threat intel automation, incident data extraction, reputation checks, and course of action. Getting one step wrong breaks the chain downstream, which is a good lesson in how dependent these automated workflows are on correct configuration.

![Building the first SOAR workflow](screenshots/T5_building_first_soar.png)

![Setting up automation](screenshots/T5_settingup_automation_soar.png)

![Case ticket management](screenshots/T5_case_management_settings.png)

![Threat intel automation](screenshots/T5_threatintell_automation.png)

![Incident data extraction](screenshots/T5_incidentdata_extraction_automation.png)

![Reputation checks](screenshots/T5_reputation_checks.png)

![Course of action](screenshots/T5_correct_course_action.png)

### Task Questions

1. What is the flag received?
   - **Answer: THM{AUT0M@T1N6_S3CUR1T¥}**

---

## Task 6 - Conclusion

### Key Concepts

SOAR was built to ease the load on SOC analysts - but it cannot replace them. The relationship is symbiotic. SOAR handles the speed and volume; analysts handle the judgment and context. The playbook is only as effective as the analyst who built it. One does not work without the other.

### Task Questions

1. Power to Security Orchestration and Automation.
   - **Answer:**

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*
