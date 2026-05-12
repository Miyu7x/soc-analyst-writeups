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

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 — Introduction

**Key Concepts:**

**Network Discovery in Detail**
  - why attackers perform network discovery
  - what are the different types of network dicovery
  - how network discovery techniques work, and how to detect them
    
---

**1. Let's begin!**

- **Answer:** N/A

---

## Task 2 — Network Discovery

**Key Concepts:**

**Attackers and Network Discovery**
  - attackers use information avaliable publicly that is called attack surface to gain information on the organizations  assets
  - attackers look for which assets can be accessed
  - what services the organization has working on these assets
      - IP addresses, ports OS... anymore?
      - version of these services and does any have a vulnerability?

*Attackers look for an opening anywhere they can*
<img src="screenshots/attackerscanningports.png" width="400">

**Defenders and Network Discovery**
  - defenders might run network discovery software when doing inventory on assets
  - check for IP, ports or services that are open and have no reason to be open
  - confirming that vulnerabilities are patched
  - low severity

**Challanges in Detecting Network Discovery**
  - both attackers and defenders must perform network discovery
  - SOC teams must know how to to differentiate between good and bad discovery, SOCs will implement techniques to achieve this
      - allowlist for known internal and benign external scanner, ensuring no flase positives happen
      - use threat intelligence to alert on malicious activities from known surpicious or suspicious sources
  - this type of scan is high severity since the attacker is now within the system
      - SOCs now escalate the alert to initiate the **Incident Response** proccess
   
  <img src="screenshots/mitrediscoverphase.png" width="400">

---

**1. What do attackers scan, other than IP addresses, ports, and OS version, in order to identify vulnerabilities in a network?**

*From our notes we can see besides IP addresses, ports and OS version we saw that services are also used for recon by the attackers*
- **Answer: Services**

---

## Task 3 — External vs Internal Scanning

**Key Concepts:**

Attackers when they are planning an attack they do whats called "mapping hosts" there are different types of investigative methods

**External Scanning Activity**
  - When an attcker uses their external IP to scan the organizations destination IP addresses
  - attackers scan the public facing assets on the perimeter
  - this type of attacks are what SOC consider the **Reconnaissance** phase of the **MITRE ATT&CK** lifecycle

SOCs in this phase can block the malicious IP
  - attackers can get new IPs

**Internal Scanning Activity**
  - SOCs will note that the source IP and destination IP addresses are now private inside the organisations network
  - this scan is starting and scanning assets from the same network
  - this attack has now progressed into the **Discovery** phase of the **MITRE ATT&CK**

---

Logs can be hard to read in their raw format and spacing on the CLI can be also confusing, in order to better organize our view we we can cut by counting the rows, but how do we know what row is what number? we firstly run (-n2 shows us the first 2 lines of the log) (cut -d',' -f3)
     **head -n2 log-session-1.csv**

Then it reuturns something like:
"@timestamp","source.ip","source.port","destination.ip"...

From here we count by commans since timestamp has 2 commnas that mean row for souyrce_ip would be #3
     
****

| | External Scanning | Internal Scanning |
|---|---|---|
| Source IP | Public/external | Private (RFC 1918) |
| MITRE Phase | Reconnaissance (TA0043) | Discovery (TA0007) |
| Severity | Low | High |
| Response | Block at perimeter firewall | Full IR process, root cause analysis |

---

**1. Which file contains logs that showcase internal scanning activity?**

<img src=screenshots/logsession2example.png width="600">
*This reminded me of what we noted that source and destination IP are private, I cut the log using **cut -d',' -f3,5** to filter only the source IP and destination IP*
- **Answer: log-session-2.csv**

---

**2. How many log entries are present for the internal IP performing internal scanning activity?**

<img src=screenshots/logentriesinternal.png width="600">
- **Answer: 2275**

---

**3. What is the external IP address that is performing external scanning activity?**

<img src=screenshots/externalscanexample.png width="600">
*same thought process as the internal scanning, cut source port and destination ports and show first 10 lines **cut -d',' -f3,5 log-session-0.csv | head -n10***
- **Answer: 203.0.113.25**

---

## Task 4 — Horizontal vs Vertical Scanning

**Key Concepts:**

| Scan Type | Destination IP | Destination Port | Purpose |
|---|---|---|---|
| Horizontal | Multiple | Single | Find all hosts with a specific port open |
| Vertical | Single | Multiple | Footprint one host's services |
| Mixed | Multiple | Multiple | Both advantages combined |

---

**1. One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X**

- **Answer:**

---

**2. In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?**

- **Answer:**

---

**3. On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.**

- **Answer:**

---

## Task 5 — The Mechanics of Scanning

**Key Concepts:**

| Technique | Protocol | How It Works | Detection Signal |
|---|---|---|---|
| Ping Sweep | ICMP | ICMP echo to identify live hosts | High-volume ICMP requests to sequential IPs |
| TCP SYN Scan | TCP | SYN sent; SYN-ACK = port open; no full handshake | Mass SYN packets, conn_state S0, no completed handshakes |
| UDP Scan | UDP | Empty UDP; ICMP port unreachable = closed but host online | Slow; many timeouts; unreliable signal |

| conn_state | Meaning |
|---|---|
| S0 | SYN sent, no reply |
| S1 | SYN + SYN-ACK, no data |
| REJ | SYN sent, RST received (port closed) |
| SF | Normal full connection established and closed |

---

**1. Which source IP performs a ping sweep attack across a whole subnet?**

- **Answer:**

---

**2. The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145**

- **Answer:**

---

**3. Is there any UDP scanning attempt in the logs? Y/N**

- **Answer:**

---

## Task 6 — Conclusion

**Key Concepts:**

---

**1. Heading on to the next room!**

- **Answer:** N/A
