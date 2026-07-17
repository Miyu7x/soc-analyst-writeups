---
title: Log Analysis with SIEM
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [siem, log-analysis, splunk, windows-logs, linux-logs, web-logs, network-logs, correlation, normalization]
status: in-progress
date:
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/analysis_intro.png width="1500">
</p>

---

## Task 1: Introduction

Any modern SOC analyst must be able to effectively use a SIEM to analyze and correlate logs while quickly identifying malicious activity and compromised assets. Equally important is understanding the different data sources behind the logs and how each one helps you see the full picture.

#### Learning Objectives

*[reference]*

- Discover various data sources that are ingested into a SIEM.
- Understand the importance of data correlation.
- Learn the value of Windows, Linux, Web, and Network logs during an investigation.
- Practice analyzing malicious behavior.

#### Room Prerequisites

*[reference]*

It is suggested to complete the following rooms first before proceeding:

- [Introduction to SIEM](https://tryhackme.com/room/introtosiem)
- [Windows Logging Capabilities](https://tryhackme.com/room/windowsloggingforsoc)
- [Cyber Kill Chain](https://tryhackme.com/r/room/cyberkillchainzmt)
- [Sysmon](https://tryhackme.com/r/room/sysmon)

#### Lab Access


Before proceeding, start the lab by clicking the Start Lab Machine button below. You will then have access to the Splunk Web Interface. To access Splunk, please follow this link: `https://LAB_WEB_URL.p.thmlabs.com`. Please wait 4-5 minutes for the Splunk instance to launch. Use Splunk's All Time range to search. The indexes where logs are stored for each practical exercise are present in each task.

#### Set up your virtual environment


To successfully complete this room, you'll need to set up your virtual environment. This involves starting the Lab Machine, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.

**Answer the questions below**

---

**1.** Let's go!

**Answer:** N/A

---

## Task 2: Benefits of SIEM for Analysts

SIEM solutions play a vital role in every Security Operations Centre, and any SOC analyst's day-to-day life. Let's take a moment to understand why SIEM is so valuable and explore its key benefits for analysis.

#### Centralisation

<p align="center">
<img src=screenshots/analysis_central.png width="400">
</p>

One of the first things that makes SIEM so helpful for a SOC is centralization. Instead of checking logs in different places, like network devices, cloud services, identity providers, and more, a SIEM allows you to gather all that data in one place. This means an analyst doesn't have to switch between systems during an investigation. Everything is available in a single solution, making their work much smoother and more efficient.

Let's take an example. We have two SOC Level 1 analysts: Ted and Emily. Ted works in a SOC with a SIEM solution, but Emily doesn't. Both analysts receive similar alerts at the same time:

- A suspicious spike in network activity.
- A malicious command was detected on a host.

With a SIEM, Ted can investigate both alerts from a single platform. He has access to logs from the IPS, endpoints, and other systems, all in one place, ready to be searched and analysed. On the other hand, Emily has to log into each system separately, for example, the IPS and EDR. She must collect and review data manually from each one.

While Ted quickly sees the bigger picture, Emily spends valuable time just gathering information. This highlights how centralized visibility through SIEM enables faster and more effective analysis.

#### Correlation

<p align="center">
<img src=screenshots/analysis_correlation.png width="700">
</p>

Another core strength of SIEM is correlation, the ability to link separate events and piece them together like pieces of a puzzle to form a complete picture. Let's walk through a scenario.

You receive an alert in your SIEM about internal network discovery activity. The only information you have is the IP address of the host performing the scan. Nothing else. The alert comes from your IDS logs.

That's not much to go on, is it? To make sense of it, you need to enrich the data, find out which device the IP belongs to, and who triggered the activity.

You can check Windows Event Logs or Sysmon by correlating this with the IDS alert and build context: Who performed this activity, where, and possibly which tool was used.

Piece by piece, the puzzle forms, helping you decide if the activity was malicious or just noise.

#### Historical Events

<p align="center">
<img src=screenshots/analysis_historical.png width="400">
</p>

SIEM also allows you to look at past events, not just current activity. This helps you spot patterns or threats that may have started earlier but weren't noticed at the time.

For instance, if you get an alert about an unusual login location for a user, you can look back at historical logs to see if he has logged in from that location or IP before. This helps you identify patterns in their behaviour and assess whether the activity is part of a malicious attempt or a legitimate action.

Of course, these aren't all the benefits SIEM offers for the SOC. There are many others, such as visualisation, detection rules, and automation paths, which we'll cover partially later in this room.

**Answer the questions below**

---

**1. What is the process of linking data from multiple sources to identify relationships between individual events called?**

**Answer: Correlation**

---

**2. What is the process of collecting and storing log data from multiple systems and sources into a single, unified location for easier analysis called?**

**Answer: Centralisation**

---

## Task 3: Log Sources Overview

Let's explore the Log Sources an analyst can encounter in SIEM and the value they can provide. Every organization has different types of resources from which logs can be collected into the SIEM, such as workstations, servers, network devices, identity providers, cloud services, various applications, and more. It can be helpful to think of this as a tree, where the organisation is the tree itself, and the roots represent the resources that make up this tree. While these components are often unique to each organisation, they all come together to form a logical structure.

Logs from all or just a few of these resources are sent to the SIEM platform, where the analyst can use them for analysis and correlation of events.

#### Host-Based Log Sources

<p align="center">
<img src=screenshots/analysis_host.png width="400">
</p>

Host-based logs come from an individual's computer at a company, a workstation, or a server. This server can be:
  - An SQL server used to store data in a table format with various attributes as well as different relationships between the data values.
  - DNS servers are machines or software responsible for resolving human-readable URL addresses to their IP addresses
  - and many more...

#### Network-Based Log Sources

<p align="center">
<img src=screenshots/analysis_network.png width="400">
</p>

Network-Based logs keep record of anything that touches the web/network: firewalls, routers, IDS and IPS and others.
  - Monitors traffic and Connections accross your network

#### Web-Based Log Sources

<p align="center">
<img src=screenshots/analysis_web.png width="400">
</p>

Organizations have their own web app most of the time, and these are where the Web-Based logs come from.
  - Attackers often gain access to an organization through web app vulnerabilities so its important for SOCs to pay close attention to their web events


In addition to the above-mentioned data sources, SIEM also collects logs from cloud platforms such as AWS and Azure, various identity providers like Entra ID, and third-party applications. However, this is beyond the scope of this room, so we'll leave it for future discussions.

However, this task will touch upon two other important topics, such as time pitfalls and normalisation in SIEM.

#### Time Pitfalls


One common challenge analysts face when working in an SIEM is time, specifically how time is recorded across different log sources.

Logs can come from systems in different time zones. Some can be in UTC, others in local time, and some may not include a timezone at all.

It's important to understand that your local time can be different from the time set in the SIEM.

Let's say you're working in UTC-2, but the logs in Splunk SIEM are normalized to UTC+2. If it's 5 PM for you, those same logs might appear as 9 PM in Splunk. That doesn't mean the logs are ingested four hours late; it's just a difference in time settings.

So always be aware of which time zones you're dealing with when analyzing events. It can make all the difference in understanding what really happened, and when.

#### Logs Normalization

Logs come in various formats: JSON, XML, and some even in plain text.
  - Each system logs events in its **own** way: different field names and structures.
  - This is where **Normalization** comes in
      - Converts all these formats into a single cohesive structure
      - All can be viewed in human-readable format in the SIEM


**Answer the questions below**

---

**1. What is the process of converting logs from different formats into a single format for easier analysis in a SIEM?**

**Answer: Normalization**

---

**2. Which log source type can be used to detect the execution of a malicious script?**

**Answer: Host-based**

---

## Task 4: Windows Logs

When using a SIEM to analyse Windows logs, we usually talk about two main data sources: WinEventLogs and Sysmon. The second one must be installed and configured separately to start receiving logs.

However, combining these two data sources actually provides clear visibility for analysing activity.

Let's briefly look at each of them.

<p align="center">
<img src=screenshots/analysis_syswin.png width="400">
</p>

#### Sysmon

Sysmon is a powerful Windows logging tool; it's capable of keeping logs of various activities, giving SOCs a wide range of visibility during analysis.
  - Deep visibility into system behavior
  - Identify the execution of malicious processes and many others such as:
      - network connections
      - possible process injection
      - registry changes
      - file creation
  


#### Malicious Process Execution


Let's consider a situation where you've received an alert about the execution of a suspicious encoded PowerShell command. You now need to analyse the activity. How would you use a SIEM in this case? Let's try using a Splunk query to see what useful information we can find in the logs using Splunk SIEM. Let's write a search that detects launched processes (EventCode=1) related to PowerShell that include the EncodedCommand PowerShell argument.

```spl
index=winenv EventCode=1 *powershell* AND *EncodedCommand*
| table _time ComputerName ParentUser ParentImage ParentCommandLine Image CommandLine
```

In this example, we detected that on host WINHOST05, a malicious update_config.js file was executed from the C:\Users\Public directory, which resulted in cmd.exe launching PowerShell with an encoded command.

Note: The activities from the following examples will not be visible in your Splunk instance, it only contains logs for your practical tasks.
<p align="center">
<img src=screenshots/analysis_example.png width="700">
</p>


#### Suspicious Network Connection


This story didn't end there, as just a few minutes later, we received another alert from WINHOST05, this time about a suspicious network connection. Let's try to determine what happened using Sysmon logs. To do this, we will search for EventCode 3, which identifies network connections, and filter by the host WINHOST05, since that is where the activity originated.

```spl
index=winenv EventCode=3 ComputerName=WINHOST05
| table _time ComputerName Image SourceIp SourcePort DestinationIp DestinationPort Protocol
```

As shown in the image below, a suspicious connection was initiated by the suspicious process PPn423.exe from the Temp folder, targeting the unusual port 9999 on IP address 83.222.191.2. We also recommend checking this IP on TI platforms.
<p align="center">
<img src=screenshots/analysis_suspicious.png width="700">
</p>

These are just two examples of when Sysmon can be useful; in reality, there are many other situations where these logs can help detect anomalies.

#### WinEventLogs

WinEventLogs or Windows Event Logs includes hundreds of unique log files beyond the common three were used to: Security, System and Application Logs


#### Windows Security Logs

The most commonly viewed and referenced logs are the Security logs, but why is that the case? Let's break down why these logs are so important and what makes them essential for investigation. Analysts can detect in Security logs activities such as user authentication attempts, account creation or modification, access to files and registry keys, process execution, system restarts or log clearing, and changes to audit or security policies.

We hope you still remember our examples with the WINHOST05 host. A SOC analyst started a new shift and was handed information about suspicious activity on this host. Within just a couple of hours, another alert came in from the same machine, this time about a user account creation. This suggests the attacker is still present on the host. We need to review the Security logs to detect what's happening and share this information with our SOC L2 analyst.

```spl
index=winenv EventCode=4720 OR EventCode=4722
| table _time EventCode ComputerName Subject_Account_Name Target_Account_Name New_Account_Account_Name Keywords
```

The attacker likely decided to create a persistence mechanism in the form of a backup user account, which was created and enabled by ted-admin on the WINHOST05 host.
<p align="center">
<img src=screenshots/analysis_winev.png width="700">
</p>


#### Windows System Logs

These logs record events generated by the operating system and its core services. They help detect various events related to services, system-level activity, and potential errors.

These logs are an excellent place to look for potential persistence or privilege escalation attempts via services. Let's examine the activity on our compromised host WINHOST05 and try to identify anomalies. We will use two event codes, 7045 and 7036, which indicate service creation and service start/stop events.

```spl
index=winenv EventCode=7045 OR EventCode=7036 ComputerName=WINHOST05
|  table _time EventCode ComputerName Service_Name Service_Account Service_File_Name Message
```

From the search results, we can see that on the host, a service named "User Updates" was created and started, which launches the malicious RNSfnsjdf.exe file from the Temp directory under the SYSTEM account. This is most likely a privilege escalation attempt, as we recall that the attacker previously only had access to the ted-admin account.
<p align="center">
<img src=screenshots/analysis_winsys.png width="700">
</p>

In that task, we examined a number of important log sources for an SOC L1 analyst and the value they can provide. Of course, there are many other logs that we will explore together in future rooms.

#### Practice Scenario

*[reference]*

You are an SOC Level 1 Analyst on shift and have received an alert indicating a suspicious network connection using port 5678 on the WIN-105 host. Your task is to conduct an investigation and determine whether this activity is suspicious.

The logs for this task are located in the Splunk index task4. Use the following query: `index=task4`

**Answer the questions below**

---

**1. Which IP address was the connection established with?**

**Answer:**

---

**2. Which process initiated this suspicious connection?**

**Answer:**

---

**3. What is the MD5 hash of the malicious process from the previous question?**

**Answer:**

---

**4. What is the name of the scheduled task that was created on the system?**

**Answer:**

---

## Task 5: Linux Logs

When you're analysing Linux systems in a SIEM, you'll often start by looking at two key log sources.

First, you'll see auth.log, which tracks authentication-related activity, such as user logins and sudo usage, among other things. These logs are crucial for spotting failed logins, unauthorised access attempts, or privilege escalation.

Next, you'll come across syslog, which captures general system-level events. Here, you can monitor service restarts, cron jobs, and background processes - all helpful when building timelines or understanding how the system behaves over time. These tools significantly improve visibility.

#### Authentication Logs

*[quick definition]*



#### Unusual login activities

*[reference]*

Let's move on to real scenarios and examine a situation in the SOC. As an SOC L1 analyst, you have received an alert about an unusual SSH login to the ubuntu username on the system. In this case, let's write a query to search for successful and failed login attempts to the system for the ubuntu user.

```spl
index=linux source="auth.log" *ubuntu* process=sshd 
| search "Accepted password" OR "Failed password"
```

There were 97 events, and the last ones show successful login attempts to the ubuntu user. This can likely be classified as a successful brute-force attack - an activity that should be escalated to L2. From a learning perspective, let's take a look at what happened.

#### Privilege Escalation behaviours

*[reference]*

Attackers often need root access on a system to gain access to certain files and more. Let's see if, in our case, there are any signs of such activities.

```spl
index=linux source="auth.log" *su*
| sort + _time
```

From the image below, it is clear that the attacker managed to gain access to the root account. How exactly this was achieved cannot be determined from auth.log alone; additional logs would be needed for that.

#### System Logs

*[quick definition]*



#### Persistence Mechanisms

*[reference]*

We hope you still remember our case in this task about the threat actor who successfully switched from ubuntu to root. In this scenario, syslog can also be useful, specifically for searching for activity related to persistence through cron jobs or services.

```spl
index=linux sourcetype=syslog ("CRON" OR "cron") 
|  search ("python" OR "perl" OR "ruby" OR ".sh" OR "bash" OR "nc")
```

We detected three interesting events. First, we can see that a suspicious pnr5433sw.sh file from the /tmp folder is being executed via cron every 5 minutes. Next, there are clear signs of a Perl reverse shell attempting to establish a connection to 10.10.101.12 IP on port 9999.

As mentioned earlier, in addition to the core data sources, it's very common to see tools like auditd and osquery used in real-world environments. These tools fall outside the scope of this room, so if you're interested in learning more about them, feel free to explore other related rooms available on our platform.

#### Practice Scenario

*[reference]*

You are an SOC Level 1 Analyst on shift and have received an alert indicating possible persistence through the creation of a new remote-ssh user on an Ubuntu server. Your task is to dive into the logs and determine exactly what happened on the system.

The logs for this task are located in the Splunk index task5. Use the following query: `index=task5`

**Answer the questions below**

---

**1. What was the timestamp of the remote-ssh account creation? Answer Format Example: 2025-01-15 12:30:45**

**Answer:**

---

**2. Which user successfully escalated their privileges to root prior to the action from the first question?**

**Answer:**

---

**3. From which IP address did the user from the previous question successfully log in to the system?**

**Answer:**

---

**4. How many failed login attempts occurred prior to this successful login?**

**Answer:**

---

**5. Which port is the persistence mechanism configured to connect to?**

**Answer:**

---

## Task 6: Web Application Logs

Other important data sources for analysis include the Web. Let's discuss them in more detail.

#### Web Log Sources

*[quick definition]*



Let's explore how different types of malicious activity can be detected using web access logs.

#### Brute Force Activity

*[reference]*

Imagine you are a SOC L1 analyst on shift, and you've just received an alert about a possible brute-force attack targeting a WordPress login page. To investigate, we recommend starting by identifying the login page URL (e.g., /wp-login.php) and filtering for POST requests. Brute-force activity involves many repeated attempts, so set a threshold like count > 25 during 5 mins. Group the results by clientip to pinpoint the source of the activity. Other fields, such as user-agent, can also provide useful context. Below is the ready query.

```spl
index=* method=POST uri_path="/wp-login.php"
| bin _time span=5m
| stats values(referer_domain) as referer_domain values(status) as status values(useragent) as UserAgent values(uri_path) as uri_path count by clientip _time
| where count > 25
| table referer_domain clientip UserAgent uri_path count status
```

As a result, we can detect that in our example, the IP address 167.172.41.141 made as many as 160 requests to the wp-login.php page. Interestingly, the User-Agent string shows Hydra, a popular tool often used by attackers to perform brute-force attacks.

#### Possible Web Shell

*[reference]*

Shortly after, you receive a second alert about possible web shell activity during your shift. To investigate potential web shell activity in Splunk, search for requests to script or executable file types such as .php, .asp, .jsp, or .exe, combined with POST and GET methods and a status=200 response. Web shells often generate a few suspicious requests in a short time frame, so set a threshold such as count > 2. Group the results by domain to identify patterns and review fields like clientip and user-agent for attacker fingerprints. Below is the ready query.

```spl
index=*
| search status=200 AND uri_path IN(*.php, *.phtm, *.asp, *.aspx, *.jsp, *.exe) AND (method=POST AND method=GET)
| stats values(status) as status values(useragent) as UserAgent values(method) as method
  values(uri) as uri values(clientip) as clientip count by referer_domain
| where count > 2
| table referer_domain count method status clientip UserAgent uri
```

As you can see in the results, we detect a variety of different URIs, but what caught our attention the most was 505.php, which could be a web shell. This means we need to take a closer look at it in more detail.

#### DDoS Activity

*[reference]*

Your shift is almost over, but don't rush to close your laptop just yet, a new alert has just come in, likely the last one for today, about a possible DDoS attack. When checking for signs of a potential DDoS in Splunk, look for status code 503, which can mean the server is overloaded. Review for huge request counts in a short time, for example, more than 100 000 in 10 minutes filtered by IPs. Review the targeted domain, which user-agent was used, and the URI path to spot patterns or attacker fingerprints. Below is the ready query.

```spl
index=* status=503
| bin _time span=10m
| stats values(referer_domain) as referer_domain values(status) as status values(useragent) as UserAgent values(uri_path) as uri_path count by clientip _time
| where count > 100000
| table _time referer_domain clientip UserAgent uri_path count status
```

And from this example, we can see that the resource was unavailable for the last 10 minutes and received more than 1.5 million requests, which confirms a possible DDoS attack.

#### Practice Scenario

*[reference]*

You are an SOC Level 1 Analyst on shift and have received an alert indicating a spike in activity on the organisation's web server. Your task is to dive into the logs and determine exactly what happened.

The logs for this task are located in the Splunk index task6. Use the following query: `index=task6`

**Answer the questions below**

---

**1. Which URI path had the highest number of requests?**

**Answer:**

---

**2. Which IP address was the source of the activity?**

**Answer:**

---

**3. How can this activity be classified?**

**Answer:**

---

**4. Which tool did the threat actor use?**

**Answer:**

---

## Task 7: Conclusion

Great job completing this room! You've now gained a solid understanding of the key log sources commonly found in SIEM platforms and the value they provide during analysis.

- Explored the value of SIEM during log analysis.
- Learned how Splunk queries can be used to detect malicious behaviours.
- Gained an introduction to the processes of log correlation and normalisation.

More exciting challenges await you next!

**Answer the questions below**

---

**1.** Good work!

**Answer:** N/A

---
