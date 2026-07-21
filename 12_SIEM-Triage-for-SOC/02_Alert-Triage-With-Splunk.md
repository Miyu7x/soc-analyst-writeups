---
title: Alert Triage With Splunk
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [siem, log-analysis, splunk, alert-triage, brute-force, persistence, web-shell, linux-logs, windows-logs, web-logs]
status: in-progress
date:
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/analysis_intro2.png width="1500">
</p>
---

## Task 1: Introduction

As a SOC analyst, it's important to be able to investigate different types of suspicious activity across a variety of assets in the environment. Knowing what to look for and which details matter most during an investigation is a key part of the role.

#### Learning Objectives

- Learn how to properly investigate alerts in a SOC environment.
- Understand how to investigate brute-force attacks on Linux systems.
- Discover the persistence mechanism on Windows systems.
- Analyse a web shell on a vulnerable web server.
- Learn how to investigate alerts for three given scenarios using Splunk.

---

## Task 2: Initial Access Alert

We will explore three different scenarios that you, as an analyst, may encounter during your shift. The first one takes place on a Linux host, and you will uncover exactly what happens there during our investigation.

#### Alert Scenario

You've just started your first shift as a SOC analyst at an MSSP. Only a few minutes have passed since an alert about a possible brute-force attack appeared on the platform.

| Field | Value |
|---|---|
| Alert Name | Brute Force Activity Detection |
| Time | 17/09/2025 9:00:21 AM |
| Target Host | tryhackme-2404 |
| Source IP | 10.10.242.248 |

Your job is to investigate this activity and decide whether it should be considered suspicious.

---
**Example:** Query for successful/failed logins, invalid user events could indicate enumeration of accounts
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248 
| search "Accepted password for" OR "Failed password for" OR "Invalid user"
| sort + _time
```

1. Large Event Count Associated with 10.10.242.248 Activity
2. Invalid User Login Attempts
3. Failed Logins Due to Invalid User 
<p align="center">
<img src=screenshots/anlysis_example2.png width="700">
</p>

---

**Example:** Number of Login attempts made for **each** user
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\S+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

1. List of Users Targeted by Login Attempts
2. Number of Login Attempts per User
3. Threat Actor IP Address
4. Strong Evidence of Brute Force Attempts on user John Smith
<p align="center">
<img src=screenshots/analysis_example3.png width="700">
</p>
<p align="center">
<img src=screenshots/analysis_example4.png width="700">
</p>

---
**Example:** How to search if an attacker **gained access** to the system
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\S+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(action) values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process  by username
```
1. Status of Login Attempts
2. Evidence of Successful System Login
<p align="center">
<img src=screenshots/analysis_example5.png width="700">
</p>
<p align="center">
<img src=screenshots/analysis_example6.png width="700">
</p>

---
**1. How many failed login attempts were made on the user john.smith?**

<p align="center">
<img src=screenshots/analysis_failed1.png width="700">
</p>
In case of a suspected brute force attack we can always filter for failed or failed password... In this case we have the suspected username john.smith.
**Filter:** index="linux-alert" *john.smith* AND *failed*
 
**Answer: 500**

---

**2. What was the duration of the brute-force attack in minutes?**

<p align="center">
<img src=screenshots/analysis_time.png width="700">
</p>
As an SOC you could just use the above filter and observe the last event and last event on the last page when the attack started...However, to make it easier, there is a better way to filter than what THM shows us. We can filter by minute intervals from start to end. 
```
index="linux-alert" john.smith failed
| stats earliest(_time) as start latest(_time) as end
| eval duration_min=(end-start)/60
```

**Answer: 5**

---

**3. What username was the attacker able to privilege escalate to?**

<p align="center">
<img src=screenshots/analysis_root.png width="700">
</p>
Privilege escalation is done through sudo or su if the attacker has breached a low privilege account. Lets filter out for those commands exactly.
Filter: index="linux-alert" sudo su

**Answer: root** 

---

**4. What is the name of the user account created by the attacker for persistence?**

<p align="center">
<img src=screenshots/analysis_time.png width="700">
</p>
If we suspect a threat actor has successfully escalated or if we want to know if they have, we can filter out by new user.
Filter: index="linux-alert" new user

**Answer: system-utm**

---

## Task 3: Persistence Alert

The second scenario focuses on activity in a Windows environment, which SOC analysts come across regularly. That's why knowing how to carry out this kind of analysis is really important.

#### Alert Scenario

You are working as a Level 1 SOC Analyst on shift at an MSSP. An alert has come through indicating that a suspicious scheduled task was created on a host.

**Alert Details:**

| Field | Value |
|---|---|
| Alert Name | Potential Task Scheduler Persistence Identified |
| Time | 30/08/2025 10:06:07 AM |
| Host | WIN-H015 |
| User | oliver.thompson |
| Task Name | AssessmentTaskOne |

Your job is to investigate this activity and decide whether it should be considered suspicious.

The logs for this task are located in the Splunk index `win-alert`. Use the following query: `index=win-alert`



#### Next Investigation Steps


Open questions remain, such as:

- How was this scheduled task created?
- How did the attacker gain access to the WIN-H015 host?
- How was the oliver.thompson account compromised?

Normally, these questions would be addressed by an SOC L2 analyst. However, since this is a training exercise, you'll have the chance to answer them yourself during the practical part of this task.

---

**1. What is the ProcessId of the process that created this malicious task?**

<p align="center">
<img src=screenshots/analysis_certutil.png width="700">
</p>
The SOCs job is to stay vigilant, scheduled tasks is one of the main preferred methods of persistence for attackers. We can look for any scheduled tasks or certutil process created.
Fiter: index=win-alert certutil

**Answer: 5816**

---

**2. What is the name of the parent process for the process that created this malicious task?**

<p align="center">
<img src=screenshots/analysis_parent.png width="700">
</p>
In order to build attack chains, we need to watch for which parent spawned the malicious task. In the above question, when we searched for the creation of a scheduled task, we can see the full details of the task creation, as well as a lot more info about the event, including the parent process, command lines, hashes, etc.

**Answer: cmd.exe** 
---

**3. Which local group did the attacker enumerate during discovery?**

<p align="center">
<img src=screenshots/analysis_enumerate.png width="700">
</p>
Investigating groups is also another valuable method for SOCs to keep track if the attacker is adding another user to achieve persistence.

**Answer: Administrators ** 

---

**4. What is the name of the workstation from which the threat actor logged into this host?**

<p align="center">
<img src=screenshots/analysis_workstation.png width="700">
</p>
Filter: index=win-alert oliver.thompson logon workstation

**Answer: DEV-QA-SERVER** 

---

## Task 4: Web Shell Alert

The last scenario focuses on activity on a vulnerable web server. We'll be investigating possible web shell exploitation.

#### Alert Scenario

Your shift as an L1 SOC analyst continues, and you've now received the next alert that needs to be investigated. This time, the activity is related to the web.

**Alert Details:**

| Field | Value |
|---|---|
| Alert Name | Potential Web Shell Upload Detected |
| Time | 14/09/2025 09:31:51 AM |
| Resource | http://web.trywinme.thm |
| Suspicious IP | 171.251.232.40 |

Your job is to investigate this activity and decide whether it should be considered suspicious.

The logs for this task are located in the Splunk index `web-alert`. Use the following query: `index=web-alert`

These questions should be reviewed by the SOC Level 2 analyst:

- Was the brute-force attack using Hydra successful?
- How did the attacker upload the web shell to the server, given that SOC L1 did not identify any traces of the upload?
- What specific actions did the attacker perform on the server using commands through the web shell?

After this, containment and remediation actions should take place.

**Answer the questions below**

---

**1. What time did the brute-force activity using Hydra begin? Answer Format Example: 2025-01-15 12:30:45**

<p align="center">
<img src=screenshots/analysis_hydratime.png width="700">
</p>

**Answer: 2025-09-14 21:20:27**

---

**2. Which user agent did the attacker use when interacting with the web shell?**

<p align="center">
<img src=screenshots/analysis_notmozilla.png width="700">
</p>
This was a difficult process to think around as THM handed us a brute-force question and this question was meant to move past hydra, so "Mozilla/5.0 (Hydra)"
Filter: index=web-alert 171.251.232.40 useragent!="Mozilla/5.0 (Hydra)" 
| table  _time clientip useragent uri_path referer referer_domain method status

**Answer: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36** 

---

**3. What was the number of requests made by the attacker to the server via the web shell?**

<p align="center">
<img src=screenshots/analysis_notmozilla.png width="700">
</p>
In the previous question, we observed POST requests http://web.trywinme.thm/wp-admin/theme-editor.php?file=b374k.php&theme=blocksy
The file=b374k.php seems suspicious. I filtered out for POST requests and the file name b374k.
Filter: index=web-alert 171.251.232.40 b374k method=POST 
| table  _time clientip useragent uri_path referer referer_domain method status

**Answer: 4**

---

## Task 5: Conclusion

Great job completing this room! You've now gained practical experience investigating different types of alerts you can encounter in the real world.

- Detecting anomalies on Windows and Linux systems.
- Analysing web shell activity and identifying its traces.

More exciting challenges await you next!
