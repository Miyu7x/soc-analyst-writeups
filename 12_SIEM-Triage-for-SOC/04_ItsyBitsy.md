---
title: ItsyBitsy
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [elk, kibana, siem, c2, incident-response]
status: Completed
date: 2026-07-23
date_completed: 2026-07-23
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

How many events were returned for the month of March 2022?

<p align="center">
<img src=screenshots/itsbitsy_total.png width="700">
</p>

It's important to know when an attack started and when it ended. Since the incident took place sometime within a week in March, I set the date range to cover the entire month for broader visibility.

**Answer: 1,482**

---

What is the IP associated with the suspected user in the logs?

<p align="center">
<img src=screenshots/itsybitsy_maliciousip.png width="700">
</p>

There are only two source IPs in the logs. I first checked the IP responsible for 99.6% of hits, which looked like normal traffic to storage.googleapis.com and redirector.gvy1.com. Checking the second IP, 192.166.65.54, revealed suspicious activity toward pastebin.com using the user agent bitsadmin.

**Answer: 192.166.65.54**

---

The user's machine used a legit Windows binary to download a file from the C2 server. What is the name of the binary?

<p align="center">
<img src=screenshots/itsybitsy_bitsadmin.png width="700">
</p>

This is where OSINT comes in — I looked up pastebin.com and the user agent bitsadmin. It turns out bitsadmin.exe is a legitimate Windows command-line binary, and the user agent string is the identifier this binary sends when it makes a network request.

**Answer: bitsadmin**

---

The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?

<p align="center">
<img src=screenshots/itsybitsy_pastebin.png width="700">
</p>

I searched for any research on pastebin.com and found a Fortinet report explaining how threat actors use pastebin.com to evade detection and obscure their activity — encoding scripts, hashes, and other binary data in Base64.

**Answer: pastebin.com**

---

What is the full URL of the C2 to which the infected host is connected?

<p align="center">
<img src=screenshots/itsybitsy_uri.png width="700">
</p>

I set up filters for the HTTP investigation, including: time, source_ip, source_port, destination_port, method, destination_ip, host, uri, user_agent, and type. This gives good visibility when hunting for threats over the network. The host field shows it's pastebin.com, with the uri /yTg0Ah6a.

**Answer: pastebin.com/yTg0Ah6a**

---

A file was accessed on the filesharing site. What is the name of the file accessed?

<p align="center">
<img src=screenshots/itsybitsy_virustotal.png width="700">
</p>

With the full URL path pastebin.com/yTg0Ah6a, I ran it through VirusTotal, which returned a lot of useful info, including a screenshot of the page — from there I could see the file type, and even the THM flag itself.

**Answer: secret.txt**

---

The file contains a secret code with the format THM{_____}.

<p align="center">
<img src=screenshots/itsybitsy_flag.png width="700">
</p>

This is a good example of how threat actors use pastebin.com to hide secrets inside plain text files.

**Answer: THM{SECRET__CODE}**

---
