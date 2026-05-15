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
status: completed
date: 2026-05-15
date_completed: 2026-05-15
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - What Is an IDS

### Key Concepts

<!-- Q: What is the core difference between a firewall and an IDS in terms of when they act? -->
A firewall stands at the boundary of a network -- it allows traffic in or out based on the rules set by the organization. But what happens if a connection that was not supposed to be allowed past the firewall gets in?

An **Intrusion Detection System (IDS)** does just that -- it detects malicious connections that slipped past the firewall.

<!-- Q: What analogy does the room use to describe how an IDS relates to a firewall, and what does each component represent? -->
The **firewall** can be thought of as a gatekeeper, checking the people that come in and out -- but there is always a chance it unknowingly lets someone malicious through.

The **IDS** is the security camera system. Even after a bad actor gets past the gatekeeper, the cameras catch them operating inside.

<!-- Q: What does an IDS do when it detects malicious activity -- does it block the threat or only notify? -->
The IDS monitors network traffic through signature-based and anomaly-based detection. If malicious traffic is detected, it generates an alert for security administrators -- it does not block the threat, only notifies.

<!-- Q: What are the two detection methods an IDS uses to identify abnormal traffic? -->
Signature-based and anomaly-based detection. Signature-based matches known attack patterns; anomaly-based flags deviations from normal baseline behavior.

### Task Questions

**Q1. Can an intrusion detection system (IDS) prevent the threat after it detects it? Yea/Nay**

**Answer: Nay**

---

## Task 2 - Types of IDS

### Key Concepts

<img src=screenshots/nidsvshids.png width="400">

<!-- Q: What is the key difference between HIDS and NIDS in terms of scope and deployment? -->
**HIDS (Host-based):** Installed on individual machines. Monitors activity on that specific host only -- examples include syslogs, Windows Event Logs, Sysmon, and auditd on Linux. Resource-intensive and difficult to scale across large networks.

**NIDS (Network-based):** Deployed to monitor the entire network. Provides a centralized view of all traffic across all hosts.

<!-- Q: What is the main weakness of signature-based IDS, and what type of attack does it fail to detect? -->
**Signature-based IDS:** Each attack has a unique pattern known as a signature. Previous attacks have their signatures saved in the database. The major weakness is that if an attack has never happened before, the IDS has no signature for it and will miss it entirely -- these are called zero-day attacks.

<!-- Q: How does anomaly-based IDS detect threats, and why does it produce more false positives than signature-based IDS? -->
**Anomaly-based IDS:** Works off the system's normal behavior baseline. Any deviation from expected behavior triggers an alert. Because many legitimate programs can behave similarly to malicious ones, this type generates a high volume of false positives. Fine-tuning the rules is critical.

<!-- Q: What makes hybrid IDS stronger than either signature-based or anomaly-based IDS alone? -->
**Hybrid IDS:** Combines both signature-based and anomaly-based detection. Known threats are caught by signatures; unknown or zero-day threats can still be flagged through anomaly detection.

<!-- Q: What is a zero-day attack, and which IDS detection mode(s) can handle it? -->
A zero-day attack has no prior signature -- it has never been seen before so it is not in any database. Only anomaly-based and hybrid IDS can detect zero-day attacks since they do not rely on known signatures.

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
Snort was developed in 1998 as an open-source IDS solution. It uses both signature-based and anomaly-based detection defined in its rule files.

<!-- Q: What is the purpose of Snort's built-in rule files, and can you create custom ones? -->
Snort ships with built-in rule files containing known attack patterns. You can also write custom rules to extend detection beyond what the defaults cover, or disable built-in rules that are not relevant to your environment.

<!-- Q: What are the three modes of Snort, and what does each one do? -->
**Packet Sniffer Mode:** Reads and displays packets without performing any analysis. Useful for network troubleshooting.

**Packet Logging Mode:** Logs network traffic to a PCAP file for later analysis. Used by forensic investigators for root cause analysis after an incident.

**NIDS Mode:** Snort's primary mode. Monitors traffic in real-time, applies rule files, and generates alerts on matches.

<!-- Q: Which Snort mode is most relevant for IDS use, and which mode would a forensic investigator use after an incident? -->
NIDS mode is the primary IDS use case. A forensic investigator would use Packet Logging mode -- or run Snort against a previously captured PCAP file using the `-r` flag.

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
By default, Snort only captures traffic intended for its own host. To monitor the entire network, promiscuous mode must be enabled on the network interface -- this allows the interface to capture all traffic on the network segment, not just traffic addressed to it.

<!-- Q: What is the main configuration file for Snort and where is it located? -->
Snort's files live in `/etc/snort`. The main configuration file is `snort.conf`, where you define which rule files to enable, the network range to monitor (`$HOME_NET`), and other settings. Rule files are stored in the `rules/` subdirectory.

<!-- Q: What are the components of a Snort rule, in order? -->
Snort rules follow a strict format: action, protocol, source IP, source port, direction operator, destination IP, destination port, and rule metadata in parentheses.

<!-- Q: What does $HOME_NET represent in a Snort rule, and where is it defined? -->
`$HOME_NET` is a variable representing the monitored network range. Its value is defined in `snort.conf`. Using a variable instead of a hardcoded IP makes rules portable across different network configurations.

<!-- Q: What is the purpose of the sid field in Snort rule metadata? -->
The `sid` (signature ID) is a unique identifier for each rule. It differentiates rules from one another and is used when referencing or suppressing specific alerts.

<!-- Q: What does the rev field track, and when is it incremented? -->
The `rev` field is the rule's revision number. Every time the rule is modified, `rev` is incremented -- this helps track changes to rules over time.

<!-- Q: What command runs Snort in NIDS mode on a live interface, and what does each flag do? -->
`sudo snort -q -l /var/log/snort -i lo -A console -c /etc/snort/snort.conf`
- `-q` quiet mode (suppresses banner)
- `-l` log directory
- `-i` network interface to listen on
- `-A console` outputs alerts to the terminal
- `-c` path to the configuration file

<!-- Q: What command runs Snort against a PCAP file for forensic analysis? -->
`sudo snort -q -l /var/log/snort -r Task.pcap -A console -c /etc/snort/snort.conf`

The `-r` flag replaces `-i` and tells Snort to read from a PCAP file instead of a live interface.

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

**Answer: /etc/snort**

---

**Q2. Which field in the Snort rule indicates the revision number of the rule?**

**Answer: rev**

---

**Q3. Which protocol is defined in the sample rule created in the task?**

**Answer: ICMP**

---

**Q4. What is the file name that contains custom rules for Snort?**

**Answer: local.rules**

---

## Task 5 - Practical Lab

### Key Concepts

<!-- Q: What command would you run to analyze the Intro_to_IDS.pcap file with Snort, starting from the correct directory? -->
`cd /etc/snort` then `sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A console -c /etc/snort/snort.conf`

<!-- Q: How do you identify the source IP of an SSH connection attempt from Snort alert output? -->
In the alert output, the source IP appears before the `->` operator. For example: `{TCP} 10.11.90.211:54334 -> 10.10.161.151:22` -- the source is `10.11.90.211`.

<!-- Q: What fields in a Snort alert line tell you the rule message and the sid? -->
The rule message appears between `[**]` markers. The sid is in the bracket notation `[1:SID:rev]` -- for example `[1:1000002:1]` means sid `1000002`, revision `1`.

### Lab Notes

```bash
# Change to Snort directory
cd /etc/snort

# Run Snort on the lab PCAP
sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A console -c /etc/snort/snort.conf
```

<img src=screenshots/pingdetected.png width="600">

### Task Questions

**Q1. What is the IP address of the machine that tried to connect to the subject machine using SSH?**

**Answer: 10.11.90.211**

---

**Q2. What other rule message besides the SSH message is detected in the PCAP file?**

**Answer: Ping Detected**

---

**Q3. What is the sid of the rule that detects SSH?**

**Answer: 1000002**

---
