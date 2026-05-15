# Room 01 — Junior Security Analyst Intro

---
title: "Junior_Security_Analyst_Intro"
tags:
  - soc
  - blue-team
  - tryhackme
  - fundamentals
  - investigation
room: "Junior Security Analyst Intro (THM)"
type: "thm-room"
priority: "medium"
status: "completed"
path: "SOC Level 1"
category: "introduction"
difficulty: "easy"
date_completed: "2026-03-24"
---

## Objective
<!-- What was the goal of this room? What were you supposed to understand or learn? -->
Understand the SOC investigation workflow and the role of Junior Security Analyst in handling alerts.
---

## Key Concepts
<!-- The most important ideas introduced — not everything, just what matters -->

- SOC team structures and roles
- SIEM alerts as starting point for investigation
- Escalation process within a SOC
- Using firewall rules to block malicious acitivity

---

## What a SOC Analyst Actually Does
<!-- Describe in your own words what a Junior Security Analyst does day-to-day -->

- Monitor and triage alerts from security tools (SIEM, IDS)
- Investigate suspicious activity using logs and context
- Determine if alerts are true positives or false positives
- Escalate confirmed threats to higher-level analysts
- Take basic respose actions such as blocking malicious IP
---

## Investigation Mindset
<!-- What kind of thinking does a SOC analyst need? How should they approach alerts and suspicious activity? -->

- Do not act immediately without validation, invetigate first
- Distinguish between false positives and true positives
- Look for supporting evidence in logs before taking action
- Stay methodical and avoid assumptions
-

---

## SOC Workflow (Investigation Flow)
<!-- The step-by-step process of how a SOC handles an alert from start to finish -->

1. Receive and review alert
2. Investigate the alert (logs, IP, context)
3.  Determine if it is a false positive or a true positive
4. If false positive> tune rules/document
5. If true positive> escalate and take response action
6.

---

## Commands & Tools

| Command / Tool | What It Did |
|----------------|-------------|
| —              | —           |
| SIEM Dashboard   | Displays alerts and system activity |
| IP Info          | Used to investigate IP reputation |
---

## Screenshots
(Only include meaningful screenshots — alerts, logs, or key findings)

<!-- ![description](screenshots/filename.png) -->

---

## Flags

| Task | Flag |
|------|------|
| Block Malicious IP on Firewall Rules | THM{until-we-meet-again} |
| —    | —    |

---

## What Stood Out
<!-- What personally stood out? What felt important, surprising, or different from what you expected? -->

- IP Info is a great tool.
- Multiple unsuccessful logins could indicate Brute Force attack.
-
-

---

## Detection Takeaways
<!-- If you were working in a SOC, what would you actually look for? What patterns or behaviors matter? -->

- Multiple failed login attempts indicate brute-force activity
- Suspicious IPs should be checked for reputation and history
- Alerts should always be validated with additional context
-

---

## What Confused Me & How I Resolved It
<!-- Be honest. This is the most valuable section. -->
Nothing in this room genuinely confused me; it was a high-level intro. 
However, I noted that in the real world, a Tier 1 analyst recommends 
firewall blocks but doesn't always execute them directly. The room 
simplified this step.
---

## What I Learned
<!-- How did this room change or improve your understanding? In your own words. -->
This room showed me that a Junior SOC analyst is the first line of defense, responsible for continuously monitoring and investigating alerts. The role is not just reacting, but carefully validating activity before taking action.
---

## Skills Practiced

- Alert analysis
- Escalation procedures
- Basic incident response

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: Miyu7*
```

---

Commit message:
```
add: final write-up template for 01_Junior-Security-Analyst-Intro
