---
title: Benign
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [splunk, windows-event-logs, lolbin, c2, incident-response]
status: 
date: 2026-07-23
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/benign-intro.png width="1500">
</p>

---

## TASK 1 - Introduction

This room investigates host-centric Windows process execution logs (Event ID 4688) ingested into Splunk under the `win_eventlogs` index, to uncover a compromised host on the network. Familiarity with Splunk basics helps here — see the splunk101 and splunk201 rooms for background.

---

## TASK 2 - Scenario: Identify and Investigate an Infected Host

### Scenario

One of the client's IDS solutions flagged suspicious process execution pointing to a compromised host in the HR department. Tools tied to network information gathering and scheduled tasks were run on the machine, confirming the suspicion. Due to limited resources, only the process execution logs (Event ID 4688) were pulled and ingested into Splunk under the `win_eventlogs` index for investigation.

### About the Network Information

The network is divided into three logical segments — it will help in the investigation.

**IT Department**
- James
- Moin
- Katrina

**HR department**
- Haroon
- Chris
- Diana

**Marketing department**
- Bell
- Amelia
- Deepak

---

How many logs are ingested from the month of March, 2022?

**Answer:** 

---

Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?

**Answer:** 

---

Which user from the HR department was observed to be running scheduled tasks?

**Answer:** 

---

Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host.

**Answer:** 

---

To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?

**Answer:** 

---

What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)

**Answer:** 

---

Which third-party site was accessed to download the malicious payload?

**Answer:** 

---

What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?

**Answer:** 

---

The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?

**Answer:** 

---

What is the URL that the infected host connected to?

**Answer:** 

---
