---
title: "Humans_as_Attack_Vectors"
tags:
  - soc
  - blue-team
  - tryhackme
  - social-engineering
  - phishing
  - human-risk
room: "Humans as Attack Vectors (THM)"
type: "thm-room"
priority: "high"
status: "completed"
path: "SOC Level 1"
category: "introduction"
difficulty: "easy"
date_completed: "2026-03-24"
---

# Room 03 — Humans as Attack Vectors

## Objective
Understand why humans are considered the weakest link in cybersecurity, how attackers exploit human behavior, and how a SOC defends against human-targeted attacks.

---

## Key Concepts
- Getting a human to execute a phishing email is often easier than bypassing a firewall
- Attackers target both individuals and executives depending on the goal
- Detection and mitigation both matter. Prevention alone is not enough

---

## The Human Element
- Attackers disguise themselves convincingly, for example MlCROSOFT vs MICROSOFT (lowercase L replacing capital I)
- Targeting executives (whaling) leads to higher-value compromises
- 3.4 billion malicious emails are sent every day. Volume is a weapon

---

## Common Attack Types
- **Phishing**: mass malicious emails
- **Spear Phishing**: targeted phishing at a specific person
- **Whaling**: spear phishing aimed at executives
- **Vishing**: voice or phone-based social engineering
- **Smishing**: SMS-based phishing

---

## Attacker Mindset
- Deepfakes used to impersonate executives or trusted figures
- Social engineering exploits trust, urgency, and authority
- Attackers research targets before striking (LinkedIn, company sites)

---

## Defending Humans
- **Mitigation**: security awareness training, anti-phishing tools, clear policies
- **Detection**: SIEM alerts, anomalous login monitoring, email filtering
- **Response**: automation to contain threats, clear escalation paths for employees

---

## Investigation Mindset (Human-Focused)
- Social engineering changes triage. Intent matters as much as the technical indicator
- Be available and approachable so employees report suspicious activity early
- Propose security training and policy improvements based on observed attack patterns

---

## Scenario Decisions

| Scenario | Decision |
|----------|----------|
| Lucas downloaded Setup.exe from best-freeapps-2025.top, blocked by AV 6 times | Quarantine Setup.exe. Instruct Lucas to use official 7-Zip installer |
| Stripe invoice email from stripe-payments.xyz with password-protected RAR | Block email and begin analysis. This is a phishing attempt |
| CEO Ben called at 9PM from hidden number requesting Gmail password reset | Disable Ben's Gmail until he personally confirms the login |
| Rose Lewis logged into M365 from London after visiting suspicious URLs | Disable Rose's account pending investigation |

---

## Commands & Tools

| Command / Tool | What It Did |
|----------------|-------------|

---

## Screenshots
![Mitigation and Detection](screenshots/mitigation-policy.png)
![Security Policy](screenshots/security-policy.png)

---

## Flags

| Task | Flag |
|------|------|
| Employees at Risk scenarios | THM{anyone_else_at_risk?} |
| Available Policies | THM{human_protection_expert!} |

---

## What Stood Out
- 3.4 billion malicious emails are sent every day. The scale of human-targeted attacks is staggering
- A single human mistake can bypass every technical control an organization has
- MSSPs and internal SOCs both need human-risk strategies, not just technical ones

---

## Detection Takeaways
- Maintain close relationships with IT and HR. Human attacks often surface through them first
- Monitor for anomalous login locations, times, and patterns
- Propose security improvements based on real attack patterns observed in alerts

---

## What Confused Me & How I Resolved It
The policies section felt vague. In reality, policies depend heavily on the organization and its needs.
For example, limiting internet access can reduce risk, but it has to be balanced with usability. Too many restrictions can slow down employees or lead them to bypass controls.
So the solution is not simply to restrict more. It is to apply the right level of control based on the company’s risk profile.

---

## What I Learned
If you throw millions of baited hooks out in the ocean you will definitely get something on the line. Phishing works because of scale.
If attackers send out millions of emails, even a tiny success rate results in real compromises. This means phishing is not just a basic attack. It is one of the most effective and reliable methods attackers use, which is why SOC teams must take it seriously.

---

## Skills Practiced
- Scenario-based decision making under ambiguity
- Identifying social engineering attack types
- Applying security policies to real-world incidents

---
