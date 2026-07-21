---
title: Alert Triage With Splunk
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [siem, log-analysis, splunk, alert-triage, brute-force, persistence, web-shell, linux-logs, windows-logs, web-logs]
status: completed
date: 2026-07-20
date_completed: 2026-07-20
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/analysis_intro2.png width="1500">
</p>

---

## Task 1: Introduction

As a SOC analyst, it's important to be able to investigate different types of suspicious activity across a variety of assets in the environment. Knowing what to look for, and which details actually matter during an investigation, is a key part of the role.

#### Learning Objectives

- Learn how to properly investigate alerts in a SOC environment.
- Understand how to investigate brute-force attacks on Linux systems.
- Discover the persistence mechanism on Windows systems.
- Analyse a web shell on a vulnerable web server.
- Learn how to investigate alerts for three given scenarios using Splunk.

---

## Task 2: Initial Access Alert

We'll explore three different scenarios that you, as an analyst, may encounter during your shift. The first takes place on a Linux host, and you'll uncover exactly what happens there during the investigation.

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

**Example:** Query for successful/failed logins and invalid user events, which could indicate account enumeration.
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248 
| search "Accepted password for" OR "Failed password for" OR "Invalid user"
| sort + _time
```

1. Large event count associated with 10.10.242.248 activity
2. Invalid user login attempts
3. Failed logins due to invalid user

<p align="center">
<img src=screenshots/anlysis_example2.png width="700">
</p>

---

**Example:** Number of login attempts made for each user.
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\S+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

1. List of users targeted by login attempts
2. Number of login attempts per user
3. Threat actor IP address
4. Strong evidence of brute force attempts on user john.smith

<p align="center">
<img src=screenshots/analysis_example3.png width="700">
</p>
<p align="center">
<img src=screenshots/analysis_example4.png width="700">
</p>

---

**Example:** How to search whether an attacker gained access to the system.
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\S+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(action) values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

1. Status of login attempts
2. Evidence of successful system login

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

In a suspected brute force attack, filtering for "failed" or "failed password" is usually enough. Here we already have the suspected username, john.smith, so we can go straight to it.

**Filter:** `index="linux-alert" *john.smith* AND *failed*`

**Answer: 500**

---

**2. What was the duration of the brute-force attack in minutes?**

<p align="center">
<img src=screenshots/analysis_time.png width="700">
</p>

You could use the filter above and manually check the timestamps of the first and last event to work out the duration. There's a cleaner way to get there though, using `earliest`/`latest` with `eval` to calculate it directly instead of eyeballing it.

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

Privilege escalation here is done through sudo or su, since the attacker breached a low-privilege account first. Filtering for those commands directly gets us there.

**Filter:** `index="linux-alert" sudo su`

**Answer: root**

---

**4. What is the name of the user account created by the attacker for persistence?**

<p align="center">
<img src=screenshots/analysis_time.png width="700">
</p>

If we suspect the threat actor successfully escalated (or want to confirm whether they did), we can filter for new user creation.

**Filter:** `index="linux-alert" new user`

**Answer: system-utm**

---

## Task 3: Persistence Alert

The second scenario focuses on activity in a Windows environment, which SOC analysts come across regularly. That's why knowing how to carry out this kind of analysis matters.

#### Alert Scenario

You are working as a Level 1 SOC analyst on shift at an MSSP. An alert has come through indicating that a suspicious scheduled task was created on a host.

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

Normally, these questions would be addressed by an SOC L2 analyst. But since this is a training exercise, we get to answer them ourselves in the practical part of this task.

---

**1. What is the ProcessId of the process that created this malicious task?**

<p align="center">
<img src=screenshots/analysis_certutil.png width="700">
</p>

Staying alert for scheduled tasks matters, since they're one of the most common persistence methods attackers use. We can look for any scheduled task or certutil process creation.

**Filter:** `index=win-alert certutil`

**Answer: 5816**

---

**2. What is the name of the parent process for the process that created this malicious task?**

<p align="center">
<img src=screenshots/analysis_parent.png width="700">
</p>

To build out the attack chain, we need to see which parent process spawned the malicious task. The search from the previous question already gives us the full event details, including the parent process, command lines, and hashes.

**Answer: cmd.exe**

---

**3. Which local group did the attacker enumerate during discovery?**

<p align="center">
<img src=screenshots/analysis_enumerate.png width="700">
</p>

Watching for group enumeration is another useful way for SOCs to catch an attacker attempting to add a user for persistence.

**Answer: Administrators**

---

**4. What is the name of the workstation from which the threat actor logged into this host?**

<p align="center">
<img src=screenshots/analysis_workstation.png width="700">
</p>

**Filter:** `index=win-alert oliver.thompson logon workstation`

**Answer: DEV-QA-SERVER**

---

## Task 4: Web Shell Alert

The last scenario focuses on activity on a vulnerable web server. We'll be investigating possible web shell exploitation.

#### Alert Scenario

Your shift as an L1 SOC analyst continues, and the next alert needs to be investigated. This time, the activity is web-related.

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

---

**1. What time did the brute-force activity using Hydra begin?**

*Answer format: 2025-01-15 12:30:45*

<p align="center">
<img src=screenshots/analysis_hydratime.png width="700">
</p>

**Answer: 2025-09-14 21:20:27**

---

**2. Which user agent did the attacker use when interacting with the web shell?**

<p align="center">
<img src=screenshots/analysis_notmozilla.png width="700">
</p>

This one took a bit more thought. THM hands us a brute-force question, but this one is meant to move past Hydra, so we exclude "Mozilla/5.0 (Hydra)" and see what's left.

**Filter:**
```
index=web-alert 171.251.232.40 useragent!="Mozilla/5.0 (Hydra)" 
| table _time clientip useragent uri_path referer referer_domain method status
```

**Answer: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36**

---

**3. What was the number of requests made by the attacker to the server via the web shell?**

<p align="center">
<img src=screenshots/analysis_4post.png width="700">
</p>

In the previous question we saw POST requests to `http://web.trywinme.thm/wp-admin/theme-editor.php?file=b374k.php&theme=blocksy`. The `file=b374k.php` parameter stood out, so I filtered for POST requests containing "b374k."

**Filter:**
```
index=web-alert 171.251.232.40 b374k method=POST 
| table _time clientip useragent uri_path referer referer_domain method status
```

**Answer: 4**

---

## Task 5: Conclusion

Completed this room, gaining hands-on experience investigating the kinds of alerts SOC analysts run into in the real world:

- Detecting anomalies on Windows and Linux systems
- Analysing web shell activity and identifying its traces
