---
title: Splunk - Dashboards and Reports
module: Advanced Splunk
path: SOC Level 2
platform: TryHackMe
tags: [splunk, siem, spl, reports, alerts, dashboards, soar]
status: In Progress
date: 2026-10-24
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/splunk3-intro.png width="1500">
</p>

---

# TASK 1 - Introduction

- Splunk is one of the most common SIEMs in enterprise environments, pulling data from tons of sources into one place
- Problem: that much data buries an analyst fast
- This room is about organizing it so analysis is quicker and clearer
- Covers reports for recurring searches, alerts and rule building, dashboards, and how Splunk extends into a full SOC platform (ES, UEBA, SOAR)
- Prereqs: Splunk: The Basics and Splunk: Exploring SPL

---

# TASK 2 - Creating Reports for Recurring Searches

## Building Reports

<p align="center">
<img src=screenshots/splunk3-task2.png width="700">
</p>

- Started with `index = *` on All time to get a feel for the data. Over 10,000 events, way too many to read raw
- SOCs run the same searches over and over, like at every shift change. A saved report runs it for you and stores the results
- Scheduling reports a few minutes apart also takes load off the search head
- Reports tab has defaults. Open in Search shows the query and time settings behind any report
- Built a report from this query, then Save As > Report:

<p align="center">
<img src=screenshots/splunk3-report.png width="700">
</p>

```
index = vpn_server | stats count by Username | sort - count
```

- 86 events total, Sarah had the most
- Because the query uses `stats count`, Content type auto-sets to Statistics Table

## Scheduling a Report

- Splunk Free can't schedule. Splunk Enterprise can
- Enterprise lets you set the schedule (e.g. daily at midnight over the last 24 hours), priority, and a schedule window so multiple reports don't fight over resources

---

## 1. Head to the Splunk Reports tab and open the Web Connections by Source IP report. Which Source_IP field value recorded the highest number of events?



<p align="center">
<img src=screenshots/splunk3-q1.png width="700">
</p>

**Answer: 10.0.0.1** 

---

## 2. While viewing the report above, click Edit → Open in Search to investigate the query behind the report. What is the flag value hidden within the search query?



<p align="center">
<img src=screenshots/splunk3-q2.png width="700">
</p>

**Answer: THM{splunk_report_wizard!}** 

---

# TASK 3 - Detecting With Alerts and Rules

## Building Alerts

- Reports show info. Alerts tell you when something happens, like repeated failed logins or an external IP hitting an internal portal
- Free license can't create alerts, so this part is follow-along with the queries
- In `web_logs`, the URI `/restricted.html` stood out. It should only be reached from internal IPs
- Query to catch anything outside the private ranges, then Save As > Alert:

```
index = web_logs URI = /restricted.html NOT Source_IP IN (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
```

- Alert settings used in the room:
  - Alert type: Real-time, runs continuously and fires as soon as a match shows up
  - Trigger alert when: Per-Result, fires on every single matching event
  - When triggered: Send email to the SOC

## Building a Baseline and Threshold Rule

- Focus here is `/payments.html` and its `status_code` values
- Goal: alert on a spike in 404s. To know what a spike is, you need a baseline first
- Baseline query, buckets events into 1 hour windows and averages them:

```
index = web_logs URI = /payments.html status_code = 404
| bin _time span=1h
| stats count AS hits BY _time
| eventstats avg(hits) AS avg_hits
| eval avg_hits = round(avg_hits, 1)
```

- Baseline came out to about 7.6 404s per hour
- Traffic shifts with weekends, holidays, campaigns. Too low a threshold means false positives, too high means missing real issues
- Threshold set at more than 11 per hour, roughly 1.45x the average:

```
index = web_logs URI = /payments.html status_code = 404
| bin _time span=1h
| stats count AS hits BY _time
| where hits > 11
| eval alert = "HIGH 404s: ".hits." in 1h (normal: ~7.6/hr)"
```

- Could run this as an hourly alert. Trigger history shows when it fired and what it caught

---

## 1. How many Source_IP addresses outside of the expected range accessed /restricted.html?

index = web_logs URI = /restricted.html NOT Source_IP IN (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)

<p align="center">
<img src=screenshots/splunk3-q3.png width="700">
</p>

**Answer: 2** 

---

## 2. How many total 404 status_code were recorded at /payments.html?

index = web_logs URI = /payments.html status_code = 404
| bin _time span=1h
| stats count AS hits BY _time
| eventstats avg(hits) AS avg_hits
| eval avg_hits = round(avg_hits, 1)

<p align="center">
<img src=screenshots/splunk3-q4.png width="700">
</p>

**Answer: 189** 

---

## 3. What is the highest number of 404's received in a given hour with the data available?

index = web_logs URI = /payments.html status_code = 404
| bin _time span=1h
| stats count AS hits BY _time
| where hits > 11
| eval alert = "HIGH 404s: ".hits." in 1h (normal: ~7.6/hr)"

<p align="center">
<img src=screenshots/splunk3-q5.png width="700">
</p>

**Answer: 16** 

---

# TASK 4 - Creating Dashboards for Summarizing Results

<p align="center">
<img src=screenshots/splunk3-web.png width="700">
</p>

- Dashboards give a quick summarized view of key fields and stats, good for spotting spikes or dips
- New dashboards need a title, optional description, permissions, and a choice of Classic Dashboards or Dashboard Studio
- Dashboard Studio has more customization but a steeper learning curve. Classic is what you see most and supports all standard visualizations, so the room uses Classic
- Worked in the Web Logs Overview dashboard, which starts with one panel of event count over time from `web_logs`
- Edit > + Add Panel > New > Pie Chart, time set to All time, with this search:

```
index = web_logs | stats count by URI | sort - count
```

- Second panel, New > Statistics Table, All time, showing `status_code`, count, percent, and total for `/restricted.html`:

```
index = web_logs URI = /restricted.html
| stats count by status_code
| eventstats sum(count) as total
| eval percent = round(count * 100.0 / total, 2)
| sort - count
```

---

## 1. Inspect the URI pie chart you built in the dashboard above. Which URI field value has the least amount of events present?

index = web_logs | stats count by URI | sort count

<p align="center">
<img src=screenshots/splunk3-q6.png width="700">
</p>

**Answer: pictures.html** 

---

## 2. Add another statistics table to your dashboard to view the Source_IP, URI, and status_code fields. How many times did 172.16.0.1 receive the status_code 200 from /payments.html?

index = web_logs  Source_IP="172.16.0.1"  status_code=200 | stats  count by Source_IP, URI, status_code

<p align="center">
<img src=screenshots/splunk3-q7.png width="700">
</p>

**Answer: 50** 

---

# TASK 5 - Extending Splunk's Functionality

- At its core Splunk ingests, indexes, and gives visibility into logs. A modern SOC needs more than visibility, it needs to manage and stop threats
- None of these add-ons are installed in the lab, the screenshots are just examples

## Splunk Enterprise Security

<p align="center">
<img src=screenshots/splunk3-security.png width="700">
</p>

- ES is a paid add-on that sits on top of Splunk Enterprise or Cloud
- SOC Operations view shows efficiency, workload split, and where bottlenecks are forming
- Adds context, correlation, and threat models instead of just searching data
- Maps detections to MITRE ATT&CK, builds risk scores, and flags events that are real threats
- Includes notable events, threat intel integration, correlation searches, and investigation workflows

<p align="center">
<img src=screenshots/splunk3-intrusion.png width="700">
</p>

## Splunk UEBA

<p align="center">
<img src=screenshots/splunk3-ueba.png width="700">
</p>

- User and Entity Behavior Analytics, a tool within ES
- Built for insider threats, compromised accounts, and suspicious activity other detections miss
- Watches behavior of users and entities like hosts, servers, and apps to find anomalies
- Stacks events over time (unusual logins, weird access patterns, suspicious processes) into a risk score
- Maps to MITRE ATT&CK so you can see which attack phase it lines up with

## Splunk SOAR

<p align="center">
<img src=screenshots/splunk3-soar.png width="700">
</p>

- Formerly Splunk Phantom. Paid, but there's a free Community Edition for on-prem
- Turns detections into automated responses: IP reputation checks, isolating hosts, disabling accounts
- Uses playbooks that integrate with other security tools
- Cuts response time, removes repetitive work, keeps handling consistent
- ES finds the threat, SOAR acts on it

---

## 1. Which feature in Splunk Enterprise Security provides visibility into SOC efficiency and performance?



<p align="center">
<img src=screenshots/splunk3-q8.png width="700">
</p>

**Answer: SOC Operations** 

---

## 2. What does Splunk SOAR use to automate response actions based on conditions, filters, and decision logic?



<p align="center">
<img src=screenshots/splunk3-q9.png width="700">
</p>

**Answer: Playbooks** 

---

# TASK 6 - Conclusion


