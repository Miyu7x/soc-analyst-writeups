---
title: IDS Fundamentals
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags:
  - IDS
  - Snort
  - NIDS
  - HIDS
  - signature-based
  - anomaly-based
  - network-security
  - pcap
status: in-progress
date: 2026-05-15
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - What Is an IDS

### Key Concepts

<!-- Q: What is the core difference between a firewall and an IDS in terms of when they act? -->
A firewall stands at the boundary of network it allows traffic in or out based on the rules set by the organization, what happens if a connection that wasnt supposed to be allowed past the firewall gets in?

An **Intrusion Detection System** does just that detects a malicious connection that passed by the firewall.
<!-- Q: What analogy does the room use to describe how an IDS relates to a firewall, and what does each component represent? -->
The **firewall** can be thought as a gatekeeper, checking the people that come in and out
	- theres a chance they might unknowingly let some bad people in

The **IDS** is the camera security system, they can now see this bad person has gotten past security and is now inside.

<!-- Q: What does an IDS do when it detects malicious activity -- does it block the threat or only notify? -->
**IDS**
	- monitors the network traffic through the networks signatures and also anomaly based detection
	-  if malicious traffic is detected it generates and alert to the security administrators so the SOCs on shift can act and investigate the traffic right away
<!-- Q: What are the two detection methods an IDS uses to identify abnormal traffic? -->

**Host Intrusion Detection System** HIDS: Host based detection . Machine based, meaining what is happening inside this one host.

**Network INtrusion Detection System**: NIDS Network based. What is happening inside the entire network if this organization.

### Task Questions

**Q1. Can an intrusion detection system (IDS) prevent the threat after it detects it? Yea/Nay**

**Answer:**

---

## Task 2 - Types of IDS

### Key Concepts

<img src=screenshots/nidsvshids.png width="400">

<!-- Q: What is the key difference between HIDS and NIDS in terms of scope and deployment? -->
**HIDS**: Host-based
	- each individual machine
	- syslogs, Windows events logs, auditd(Linux)

<!-- Q: What is the main weakness of signature-based IDS, and what type of attack does it fail to detect? -->

**Signature-Based IDS** each attack has a unique pattern known as a signature
	- this means if a type of attack has never happened before this IDS has no way of kowing about it
	- previous attacks have their signature asaved in the db

**Anomaly-Based IDS** works off the systems nromal behavior, what is expected to run on day to day basis
	- deviation form normal behavior triggers alerts
	- natural programs can match malicious behavior causing IDS to generate many false positives
	- fine tuning the rules is critical here

**Hybrid IDS** uses signatures and anomalies to detect threats
<!-- Q: How does anomaly-based IDS detect threats, and why does it produce more false positives than signature-based IDS? -->

<!-- Q: What makes hybrid IDS stronger than either signature-based or anomaly-based IDS alone? -->

<!-- Q: What is a zero-day attack, and which IDS detection mode(s) can handle it? -->

### Comparison Table: IDS Deployment Modes

| Type | Scope | Strength | Limitation |
|------|-------|----------|------------|
| HIDS | Single host | Detailed host visibility | Resource-intensive; hard to scale |
| NIDS | Whole network | Centralized network view | Less host-level granularity |

### Comparison Table: IDS Detection Modes

| Mode | How It Detects | Detects Zero-Day? | Risk |
|------|---------------|-------------------|------|
| Signature-based | Matches known attack patterns (signatures) | No | Misses new/unknown threats |
| Anomaly-based | Detects deviation from baseline behavior | Yes | High false positive rate |
| Hybrid | Combines both methods | Yes | Depends on tuning |

### Task Questions

**Q1. Which type of IDS is deployed to detect threats throughout the network?**

**Answer: Network Intrusion Detection System**

---

**Q2. Which IDS leverages both signature-based and anomaly-based detection techniques?**

**Answer: Hybrid IDS**

---

## Task 3 - IDS Example: Snort

### Key Concepts

<img src=screenshots/modesofsnort.png width="400">

<!-- Q: When was Snort developed and what detection methods does it use? -->
**SNORT** was developed in 1998 as an open source IDS solution
<!-- Q: What is the purpose of Snort's built-in rule files, and can you create custom ones? -->
SNORT uses rule files that contain various known attack patterns and you can also configure new rules to broaden the protection
<!-- Q: What are the three modes of Snort, and what does each one do? -->
**Packet Sniffer Mode** observes paclets without doing any analysis on them

**Packet Logging Mode** performs detection on live traffic in real time, great for log analysis 

**Network Intrusion Detection System Mode** SNORTS prmary mode that monitors traffic in real-time 
<!-- Q: Which Snort mode is most relevant for IDS use, and which mode would a forensic investigator use after an incident? -->

### Comparison Table: Snort Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| Packet Sniffer | Reads and displays packets; no analysis | Network troubleshooting/monitoring |
| Packet Logging | Logs network traffic to PCAP for later analysis | Forensic investigation / root cause analysis |
| NIDS Mode | Real-time traffic monitoring with rule matching; generates alerts | Proactive threat detection |

### Task Questions

**Q1. Which mode of Snort helps us to log the network traffic in a PCAP file?**

**Answer: Packet Logging Mode**

---

**Q2. What is the primary mode of Snort called?**

**Answer: Network Intrusion Detection System Mode**

---

## Task 4 - Snort Usage

### Key Concepts

<!-- Q: What is promiscuous mode, and why would you enable it on a network interface running Snort? -->

<!-- Q: What is the main configuration file for Snort and where is it located? -->

<!-- Q: What are the components of a Snort rule, in order? -->

<!-- Q: What does $HOME_NET represent in a Snort rule, and where is it defined? -->

<!-- Q: What is the purpose of the sid field in Snort rule metadata? -->

<!-- Q: What does the rev field track, and when is it incremented? -->

<!-- Q: What command runs Snort in NIDS mode on a live interface, and what does each flag do? -->

<!-- Q: What command runs Snort against a PCAP file for forensic analysis? -->

### Snort Rule Anatomy

```
alert icmp any any -> $HOME_NET any (msg:"Ping Detected"; sid:10001; rev:1;)
```

| Field | Value | Meaning |
|-------|-------|---------|
| Action | alert | What to do when rule triggers |
| Protocol | icmp | Traffic protocol to match |
| Source IP | any | Any originating IP |
| Source Port | any | Any originating port |
| Direction | -> | Traffic flow direction |
| Destination IP | $HOME_NET | Variable defined in snort.conf |
| Destination Port | any | Any destination port |
| msg | "Ping Detected" | Alert message displayed on trigger |
| sid | 10001 | Unique rule signature ID |
| rev | 1 | Rule revision number |

### Key Commands (Reference)

```bash
# List Snort directory contents
ls /etc/snort

# Edit custom rules file
sudo nano /etc/snort/rules/local.rules

# Run Snort in NIDS mode on loopback interface
sudo snort -q -l /var/log/snort -i lo -A console -c /etc/snort/snort.conf

# Run Snort against a PCAP file
sudo snort -q -l /var/log/snort -r Task.pcap -A console -c /etc/snort/snort.conf

# Ping loopback to test rule
ping 127.0.0.1
```

### Task Questions

**Q1. Where is the main directory of Snort that stores its files?**

**Answer:**

---

**Q2. Which field in the Snort rule indicates the revision number of the rule?**

**Answer:**

---

**Q3. Which protocol is defined in the sample rule created in the task?**

**Answer:**

---

**Q4. What is the file name that contains custom rules for Snort?**

**Answer:**

---

## Task 5 - Practical Lab

### Key Concepts

<!-- Q: What command would you run to analyze the Intro_to_IDS.pcap file with Snort, starting from the correct directory? -->

<!-- Q: How do you identify the source IP of an SSH connection attempt from Snort alert output? -->

<!-- Q: What fields in a Snort alert line tell you the rule message and the sid? -->

### Lab Notes

```bash
# Change to Snort directory
cd /etc/snort

# Run Snort on the lab PCAP
sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A console -c /etc/snort/snort.conf
```

### Task Questions

**Q1. What is the IP address of the machine that tried to connect to the subject machine using SSH?**

**Answer:**

---

**Q2. What other rule message besides the SSH message is detected in the PCAP file?**

**Answer:**

---

**Q3. What is the sid of the rule that detects SSH?**

**Answer:**

---