---
title: ItsyBitsy
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [elk, kibana, siem, c2, incident-response]
status: 
date: 2026-07-23
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/itsybitsy-intro.png width="1500">
</p>

---

## TASK 1 - Introduction

This room sets up an ELK-based lab environment (Kibana, credentials `Admin`/`elastic123`) used to investigate an IDS alert flagging potential C2 communication. A week's worth of HTTP connection logs have been ingested into the `connection_logs` index for analysis.

---

## TASK 2 - Scenario - Investigate a potential C2 communication alert

### Scenario

A SOC analyst noticed an IDS alert pointing to possible C2 activity coming from an HR employee named Browne. It looks like a suspicious file was opened on his machine, and that file holds a hidden flag in the format `THM{ ________ }`. Since resources are limited, only a week of HTTP connection logs were available for the investigation — the job is to trace the connection, find the link to the file, and pull out its contents.

---

**Q:** How many events were returned for the month of March 2022?

**Answer:** 

---

**Q:** What is the IP associated with the suspected user in the logs?

**Answer:** 

---

**Q:** The user's machine used a legit windows binary to download a file from the C2 server. What is the name of the binary?

**Answer:** 

---

**Q:** The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?

**Answer:** 

---

**Q:** What is the full URL of the C2 to which the infected host is connected?

**Answer:** 

---

**Q:** A file was accessed on the filesharing site. What is the name of the file accessed?

**Answer:** 

---

**Q:** The file contains a secret code with the format THM{_____}.

**Answer:** 

---
