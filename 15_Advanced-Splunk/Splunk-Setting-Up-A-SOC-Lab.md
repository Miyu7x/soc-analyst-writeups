---
title: Splunk Setting up a SOC Lab
module: Advanced Splunk
path: SOC Level 2
platform: TryHackMe
tags:
  - splunk
  - siem
  - universal-forwarder
  - log-ingestion
  - linux-logs
  - apache
  - deployment-server
status: In Progress
date: 2026-10-23
date_completed: 2026-10-23
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/splunk2-intro.png width="1500">
</p>

---

# TASK 1 - Introduction

<p align="center">
<img src=screenshots/coffely.png width="400">
</p>

- Coffely got hit by an insider breach. Someone in IT stole Jasmine's secret recipe.
- Forensics caught the culprit and got the recipe back, but now Jasmine wants her own SOC so it doesn't happen again.
- My job is to build it: install Splunk locally, then feed it logs from a Linux host and the Coffely web server.
- Skills covered: installing Splunk and the Universal Forwarder on Linux, managing Splunk from the CLI, and ingesting OS logs and web logs.

---

# TASK 2 - Splunk Deployment Architecture

<p align="center"> 
<img src=screenshots/splunk2-arch.png width="700">
</p>
## Architecture Components

- Splunk is split into roles. Each piece does one job in the pipeline from endpoint to analyst.

---

## Forwarders

- Lightweight agents installed on endpoints and servers. They collect data and ship it to Splunk.
- Universal forwarder sends raw data.
- Heavy forwarder can parse or filter data before sending it.

---

## Indexers

- Receive the data from forwarders, process it, and store it in an index.
- This is what makes logs searchable and lets me correlate events across sources.

---

## Search Head

- The part analysts actually use to run searches, dashboards, reports, and alerts.
- Sends search requests to the indexers and displays the results.
- Splunk does search-time parsing: raw events are stored as-is and fields get extracted when I search. Flexible, but heavier on the search head CPU.
- Elastic leans toward index-time parsing instead, so searches are faster but indexing costs more.
- In this room, one Splunk instance plays both indexer and search head.

---

## Deployment Server

<p align="center">
<img src=screenshots/splunk2-server.png width="700">
</p>

- Optional central manager for lots of Splunk instances and forwarders.
- Lets me push configs, inputs, and outputs to hundreds of endpoints remotely instead of logging into each one.

---

## 1. Which part of the Splunk deployment architecture is responsible for processing incoming data from forwarders?



<p align="center">
<img src=screenshots/splunk2-q1.png width="700">
</p>

**Answer: Indexers**

---

## 2. Part of your client's infrastructure includes hundreds of employee workstations. Which Splunk feature would enable you to remotely manage forwarders on each system?



<p align="center">
<img src=screenshots/splunk2-q2.png width="700">
</p>

**Answer: Deployment Server**

---

# TASK 3 - Linux Deployment

## Connect to the Lab

- The Splunk installer is already waiting in `~/Downloads/splunk`, no account or download needed.
- If Splunk freezes, run `/opt/splunk/bin/splunk restart` as root and give it a few minutes.

---

## Splunk Install

- Move into the folder, switch to root with `sudo su`, then extract the installer into `/opt` with `tar xvzf splunk_installer.tgz -C /opt`.
- Root is fine for the lab, but in a real environment Splunk should run under a dedicated low-privileged account with just enough access to read logs and manage Splunk.

---

## Starting Splunk

- From `/opt/splunk/bin`, run `./splunk start --accept-license`.
- First run makes me create the admin username and password (minimum 8 characters).

---

## Accessing Splunk

- Splunk Web lives at `http://coffely:8000` inside the VM, or `http://MACHINE_IP:8000` over the VPN.
- Log in with the admin credentials I just created.

---

## 1. In which Linux directory, typically used for add-on or third-party software, should Splunk be installed?



<p align="center">
<img src=screenshots/splunk2-q3.png width="700">
</p>

**Answer: opt**

---

## 2. Which port does the Splunk web interface run on by default?



<p align="center">
<img src=screenshots/splunk2-q4.png width="700">
</p>

**Answer: 8000**

---

# TASK 4 - Managing Splunk From CLI

- All of these run from `/opt/splunk/bin`.
- `splunk start` starts every Splunk process. No effect if it's already running.
- `splunk stop` stops it.
- `splunk restart` is what I use after config changes.

---

## Further Useful Commands

- `splunk enable boot-start` makes Splunk start as a service on boot.
- `splunk status` shows whether splunkd and its helpers are running, with PIDs.
- `splunk add oneshot` adds a single file to an index one time. Good for testing.
- `splunk search` runs a search straight from the terminal.
- `splunk help` lists every CLI command and its syntax.

---

## 1. Using the CLI, what command would you run to search Splunk for the term `coffely`?



<p align="center">
<img src=screenshots/splunk2-q5.png width="700">
</p>

**Answer: ./splunk search coffely**

---

## 2. Which Splunk command would you run to get help with Splunk searches?



<p align="center">
<img src=screenshots/splunk2-q6.png width="700">
</p>

**Answer: ./splunk help search**

---

# TASK 5 - Splunk Universal Forwarder

- In the real world, forwarders sit on remote endpoints and send data to the central Splunk server.
- Here, the forwarder goes on the same machine as Splunk.

---

## Installing the Universal Forwarder

- Back in `/home/ubuntu/Downloads/splunk` as root.
- Extract it into `/opt` with `tar xvzf splunkforwarder.tgz -C /opt`.

---

## Starting the Forwarder

- From `/opt/splunkforwarder/bin`, start it with the same license flag and create its admin credentials.
- Splunk Enterprise and the forwarder both default to management port `8089`, so they collide on one machine.
- When prompted, change the forwarder's management port to `8090` so both can run.

---

## 1. What is the full command you used to start the forwarder?



<p align="center">
<img src=screenshots/splunk2-q7.png width="700">
</p>

**Answer: ./splunk start --accept-license**

---

## 2. Which port does the Splunk forwarder run on by default?



<p align="center">
<img src=screenshots/splunk2-q8.png width="700">
</p>

**Answer: 8089**

---

# TASK 6 - Configuring the Forwarder

- Two sides to set up: Splunk needs to know it's receiving, and the forwarder needs to know where to send.

---

## Splunk Configuration

<p align="center">
<img src=screenshots/splunk2-config.png width="700">
</p>

- `Settings > Forwarding and receiving > Receive data > + Add new`.
- Splunk listens for forwarder data on port `9997` by default. Set it to `9997` and save.

---

## Creating an Index

<p align="center">
<img src=screenshots/splunk2-index.png width="700">
</p>

- `Settings > Indexes > New Index`, name it `linux_host`, keep the defaults, save.
- No index specified means data lands in the default `main` index.
- The built-in internal indexes are Splunk monitoring its own health.

---

## Configuring the Forwarder

- On the Linux host, point the forwarder at Splunk with `./splunk add forward-server MACHINE_IP:9997`.

---

## Linux Log Sources

- Linux keeps its important logs in `/var/log`.
- `syslog` gets ingested first. Every other log there works the same way.

---

## Inputs

- `./splunk add monitor /var/log/syslog -index linux_host` tells the forwarder to watch syslog and send it to `linux_host`.
- `inputs.conf` defines what Splunk collects. Mine lives at `/opt/splunkforwarder/etc/apps/search/local/inputs.conf`.
- It can be edited by hand to add or change sources.

---

## Using the Logger Utility

- `logger "text"` writes a test entry into syslog.
- `tail -1 /var/log/syslog` confirms it landed locally.
- Then search `index = linux_host "coffely-has-the-best-coffee-in-town"` in Search & Reporting to confirm Splunk received it.

---

## 1. Follow the steps from above, but this time to ingest the /var/log/auth.log file into the linux_host index. What is the value in the sourcetype field?



<p align="center">
<img src=screenshots/splunk2-q9.png width="700">
</p>

**Answer: auth**

---

## 2. Create a new user named analyst using the command adduser analyst. Once created, look at the syslog events generated in Splunk related to the user creation activity. What is the value of the process field when the new user is added?



<p align="center">
<img src=screenshots/splunk2-q10.png width="700">
</p>

**Answer: adduser**

---

# TASK 7 - Windows Forwarding

- Real SOCs pull logs from many sources: Windows servers, email servers, firewalls.
- Log aggregation means collecting, normalizing, and indexing all of it centrally so events can be correlated across systems.
- This task is read-along only, since the lab can only run one VM.
- On Windows, install the forwarder and give it the Linux Splunk IP as the indexer.
- From PowerShell in `C:\Program Files\SplunkUniversalForwarder\bin`, run `.\splunk.exe add monitor C:\Windows\System32\winevt\Logs\Security.evtx` to forward Security logs.

---

## Deployment Server

- `./splunk enable deploy-server` then `./splunk restart` turns my Splunk instance into a deployment server.
- That lets me manage the Windows forwarder from the UI.
- `Settings > Data Inputs > Forwarded inputs > + Add new` next to Windows Event Logs.
- Pick the Windows host from Available hosts and give it a server class name. A server class groups deployment clients together.
- Choose the logs to forward (Application, Security, Setup, System), create a `windows` index, review, and submit.

---

## 1. What command would you use on Linux to enable your Splunk instance as a deployment server?



<p align="center">
<img src=screenshots/splunk2-10.png width="700">
</p>

**Answer: ./splunk enable deploy-server**

---

# TASK 8 - Ingesting Web Logs

<p align="center">
<img src=screenshots/splunk2-ingest.png width="700">
</p>

- Coffely's dev website runs on the same Linux box at `http://coffely.thm:8080`.
- It's Apache, so access logs are at `/var/log/apache2/access.log`.
- Goal is to ingest them to track requests, responses, and orders. This time through the UI instead of the CLI.

---

## Adding Data

<p align="center">
<img src=screenshots/splunk2-add.png width="700">
</p>

- `Settings > Add Data > Monitor > Files & Directories`, path `/var/log/apache2/access.log`.
- Choose continuous monitoring, not index once, so new logs keep flowing in.
- Source type: `Select Source Type > Web > access_combined`.
- Set the host field value to `coffelyweb` and create a new index named `web`.
- Review and submit. Browse the site to generate traffic, since the log starts empty.

---

## 1. Which `sourcetype` should be used to parse and categorize Apache access logs correctly?



<p align="center">
<img src=screenshots/splunk2-q11.png width="700">
</p>

**Answer: access_combined**

---

## 2. After browsing the Coffely site, which `root` field value has the highest count?



<p align="center">
<img src=screenshots/splunk2-q12.png width="700">
</p>

**Answer: images**

---

## 3. What is the full `uri_path` field value for `cappuchino.png`?



<p align="center">
<img src=screenshots/splunk2-q13.png width="700">
</p>

**Answer: /images/coffeerecipes/cappuchino.png**

---

## 4. Navigate to `http://coffely.thm:8080/secret-flag.html` Check out the list of recent orders and find the flag!



<p align="center">
<img src=screenshots/splunk2-q14.png width="700">
</p>

**Answer: THM{best_coffee_in_town!}**

---

# TASK 9 - Conclusion


