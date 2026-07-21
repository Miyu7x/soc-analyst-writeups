---
title: Alert Triage With Elastic
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [siem, elastic, kibana, alert-triage, proxylogon, windows-logging, sysmon]
status: 
date: 
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/analysis_intro2.png width="1500">
</p>

---

## TASK 1 - Introduction

SomeCorp's infrastructure triggered a string of SOC alerts. Using Kibana, this investigation correlates web logs, Windows Security logs, Sysmon, and PowerShell script block logging to confirm a ProxyLogon-based breach, an Administrator logon outside business hours, a backdoor account creation, and privilege escalation via group membership changes.

---

## TASK 2 - Scenario Briefing

Your team is responsible for managing several small businesses' servers, applications, and network infrastructure. Suspicious activity in your client, SomeCorp's infrastructure, has triggered multiple alerts. As the on-call SOC analyst, use the provided logs, dashboards, and tools to investigate the activity, determine if it is malicious, and reconstruct the attack sequence.

You log in to the SOC dashboard provided above and see many alerts to triage. But first, you must build out your environment on the Kibana interface. Let's start by accessing the Kibana dashboard and choosing the correct filters.

1. Access Kibana: Start the VM, wait up to five minutes, and open `https://LAB_WEB_URL.p.thmlabs.com/`
2. Select the Data view: You need to choose `Alert Triage With Elastic` for this room
3. Select the time range: For now, select `Entire data range` to view all ingested logs

The Data view selected includes two indices. For the second question, please enter your first query `_index:weblogs` to filter for only IIS web server logs, which we will begin with in the next task. Before moving on, take a moment to check out the available fields on the left side of the dashboard.

**How many logs are available for analysis within the entire time range?**

**Answer:**

---

**What is the field value for the `client.ip` in the `weblogs` index?**

**Answer:**

---

## TASK 3 - Investigating Web Attacks

Now that we have set up the Kibana interface, we can begin to investigate the alerts. Let's take a moment to consider the information we have. The alert above shows the IP address `203.0.113.55` made `POST` requests to the page `proxyLogon.ecp`. Even if you don't know how the page can be exploited, your first goal is to visualize all web requests from the IP address.

Let's build out a table using the information below and answer the first two questions about the requests.

1. Append your current query to include the IP address and highlight `POST` requests
`_index:weblogs and client.ip:203.0.113.55 and http.request.method:POST`
2. Start building your table by hitting the `+` next to any field on the left side to add it as a column
Add `client.ip`, `user.agent`, `http.request.method`, `url.path`, and `http.response.status_code`

Highlighting the above fields will help you view the log data in a more structured and legible manner, making it easier to identify indicators. You can use the query and table to answer the first two questions.

You have now utilized Elastic to visualize the first alert. Great job! Upon learning more about the user-agent and the page name, you will see that the requests are automated and related to the ProxyLogon vulnerability. This is already a red flag, but let's quickly glance at another high-severity alert:

The alert concerns the same client IP address and comes just seven minutes after the first one. We will use the alert details to investigate whether this is indeed a True Positive. Look at the Trigger field; the `cmd=` parameter is a hallmark of web shell activity, as attackers can append commands directly to it. For example, `cmd=id` where `id` is the command being executed. Let's see whether we can confirm a web shell:

1. Alter your query to search for `GET` requests and the file name `errorEE.aspx`
`_index:weblogs and client.ip:203.0.113.55 and http.request.method:GET and errorEE.aspx`
2. Sort `Old-New` to see the requests that came in first at the top

Great, now you are getting somewhere. In the screenshot below, and in your Elastic instance, you can see some of the commands that were run using the web shell in the `url.path` field of the table. The results indicate a breach! You'd need to escalate both alerts to your senior as True Positives. Using the query, begin answering the last questions in this task.

**How many `POST` requests did the IP address `203.0.113.55` make to `proxyLogon.ecp`?**

**Answer:**

---

**Which `user.agent` paired with the IP address `203.0.113.55` made the `POST` requests?**

**Answer:**

---

**How many logs contain the `cmd=` query parameter in the `url.path` field?**

**Answer:**

---

**Which command was run utilizing `errorEE.aspx` on `Jul 20, 2025 @ 04:45:50.000`?**

**Answer:**

---

## TASK 4 - Uncovering Account Activity

| Severity | Alert ID | Account | Hostname | Alert Time | Trigger |
|---|---|---|---|---|---|
| High | SOC-20250720-0014 | Administrator | winserv2019.some.corp | Jul 20, 2025 @ 05:11:22.000 | EDR - Administrator authentication outside of expected hours |
| Critical | SOC-20250720-0015 | Administrator | winserv2019.some.corp | Jul 20, 2025 @ 05:13:09.000 | EDR - User Account Management: new user created |

Your web log investigation raised red flags, but suspicious network traffic is only part of the story. You need to pivot to host-based evidence to determine if the attacker has moved further. The alert shows that the Administrator account accessed the server outside of business hours. Your first step is to confirm the logon. When, how, and from where did it occur? Investigate Windows Security Event ID 4624 (Logon) to gather the context.

Let's begin by focusing on events that occurred on or after the specified time in the alert. We will focus on Security Event ID `4624` (Logon), our client's hostname, and the `Administrator` user.

`@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator`

Again, we will add fields as columns to construct a table, allowing relevant information to be easily highlighted.

1. `winlog.event_id`: Windows Event ID
2. `host.name`: Target hostname on which the logon occurred
3. `winlog.event_data.TargetUserName`: User account that logged in
4. `winlog.logon.type`: How the user accessed the system (e.g., remotely via RDP)
5. `winlog.event_data.IpAddress`: The source IP address of the client, a very important field

Based on the event in the screenshot and your Elastic instance, you can confirm that the Administrator logged on at the specified time and from the same IP `203.0.113.55` as in the previous alerts. Expand the log and answer the first question.

Alongside Windows Security logs, you also have Sysmon logs available for investigation. These can be correlated to validate the Administrator authentication and to investigate what happened afterward. To begin, take the timestamp from the previous `4624` logon event and adjust your query to look for Sysmon Event ID `1` (Process Creation) and the Administrator username.

`@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:1 and user.name:Administrator`

These fields have been added to a table in the screenshot below. You can do the same in your VM.

1. `user.name`: User account that launched the process
2. `process.parent.name`: Executable name of the parent process
3. `process.command_line`: The actual process with its full command line

In the screenshot, we can verify the Administrator logon and gain further insight into the process chain. In this case, the Administrator logon initiated a process chain that aligns with Windows session initialization. Try using the above query to answer the second question.

Great, so those two queries helped you confirm the logon occurred, but the evidence you have so far is not enough to validate whether this behavior is malicious. Perhaps the Administrator of SomeCorp started work early today. Let's investigate the next alert to determine what the Administrator account was up to.

To begin investigating this alert, we can use similar queries that we have already employed. Here is a query to begin your investigation.

`@timestamp >= "2025-07-20T05:13:10.000" and winlog.channel:Security and winlog.task:User Account Management`

In the above query, we are focusing on Windows Security logs and the `winlog.task` field. This field gives a human-readable context about what type of activity the event represents. Try creating a table with these three fields after submitting the query and answering the remaining task questions. Don't forget to sort Old-New to see the events that occurred first, at the top.

1. `winlog.event_id`
2. `winlog.task`
3. `message`

**What is the `winlog.record_id` of the Administrator 4624 logon event?**

**Answer:**

---

**What is the `process.pid` of the Sysmon 1 event that occurred on Jul 20, 2025 @ 05:11:27.996?**

**Answer:**

---

**What is the `winlog.event_id` for the new user account being created?**

**Answer:**

---

**What is the name of the new user account?**

**Answer:**

---

## TASK 5 - Exposing Command Execution

| Severity | Alert ID | Account | Hostname | Alert Time | Trigger |
|---|---|---|---|---|---|
| High | SOC-20250720-0016 | Administrator | winserv2019.some.corp | Jul 20, 2025 @ 05:13:15.000 | EDR - command-line usage: C:\Windows\system32\cmd.exe |

This next alert flags suspicious command-line usage from the same Administrator account. Could the attacker be escalating their actions? Let's investigate.

As a Level 1 SOC analyst, your first steps should be:

1. Scope the alert: Identify the child processes launched by cmd.exe
2. Confirm the origin: Find out who launched cmd.exe and why
3. Check for privilege changes: Look for commands like net used to add users to groups
4. Correlate across log sources: Use Sysmon and Windows Security logs to confirm the malicious behavior

Crafting a query to highlight Sysmon events that occurred on or after the stated time and include the parent process cmd.exe will help you clearly see what commands were executed.

`@timestamp >= "2025-07-20T05:13:15" and process.parent.name:cmd.exe and user.name:Administrator`

Let's make a simple table to investigate the commands executed by adding `process.command_line`, `process.name`, and `process.parent.name` as columns.

The first command executed is shown in
