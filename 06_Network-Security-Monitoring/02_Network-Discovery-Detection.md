---
title: Network Discovery Detection
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [network-discovery, port-scanning, firewall-logs, zeek, kibana, mitre-attck, soc-level-1]
status: in-progress
date: 2026-05-12
date_completed:
---

*by Miyu7 | TryHackMe SOC Level 1 Path*

---

## Task 1 — Introduction

**Key Concepts:**

---

**1. Let's begin!**

- **Answer:** N/A

---

## Task 2 — Network Discovery

**Key Concepts:**

---

**1. What do attackers scan, other than IP addresses, ports, and OS version, in order to identify vulnerabilities in a network?**

- **Answer:**

---

## Task 3 — External vs Internal Scanning

**Key Concepts:**

---

**1. Which file contains logs that showcase internal scanning activity?**

- **Answer:**

**2. How many log entries are present for the internal IP performing internal scanning activity?**

- **Answer:**

**3. What is the external IP address that is performing external scanning activity?**

- **Answer:**

---

## Task 4 — Horizontal vs Vertical Scanning

**Key Concepts:**

---

**1. One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X**

- **Answer:**

**2. In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?**

- **Answer:**

**3. On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.**

- **Answer:**

---

## Task 5 — The Mechanics of Scanning

**Key Concepts:**

---

**1. Which source IP performs a ping sweep attack across a whole subnet?**

- **Answer:**

**2. The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145**

- **Answer:**

**3. Is there any UDP scanning attempt in the logs? Y/N**

- **Answer:**

---

## Task 6 — Conclusion

**Key Concepts:**

---

**1. Heading on to the next room!**

- **Answer:** N/A

---

## Room Summary

**Key Concepts:**

---

| Concept | External Scanning | Internal Scanning |
|---|---|---|
| Source IP | Public/external | Private (RFC 1918) |
| MITRE Phase | Reconnaissance (TA0043) | Discovery (TA0007) |
| Severity | Low | High |
| Response | Block at perimeter firewall | Full IR process, root cause analysis |

| Scan Type | Destination IP | Destination Port | Purpose |
|---|---|---|---|
| Horizontal | Multiple | Single | Find all hosts with a specific port open |
| Vertical | Single | Multiple | Footprint one host's services |
| Mixed | Multiple | Multiple | Both advantages combined |

| Scan Technique | Protocol | Detection Signal |
|---|---|---|
| Ping Sweep | ICMP | High-volume ICMP echo requests to sequential IPs |
| TCP SYN Scan | TCP | Mass SYN packets, no completed handshakes (conn_state: S0) |
| UDP Scan | UDP | Slow; many timeouts; ICMP port unreachable responses |
