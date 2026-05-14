---
title: Man-in-the-Middle Detection
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags:
  - mitm
  - arp-spoofing
  - dns-spoofing
  - ssl-stripping
  - wireshark
  - network-analysis
  - blue-team
  - soc-level-1
status: completed
date: 2026-05-14
date_completed: 2026-05-14
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

**Man-in-the-Middle**

<img src=screenshots/mitmintro.png width="600">

<!-- Q: What is a Man-in-the-Middle attack and what position does the attacker occupy in the communication flow? -->

**MITM Attacks:** Attackers put themselves in the middle of your endpoints, intercepting, modifying, and redirecting traffic.

<!-- Q: What two-layer approach does blue team detection of MITM require? -->

**SOC Detection** requires a multi-layered approach:
- Network monitoring
- Certificate validation
- Behavioral analysis

<!-- Q: What are the three MITM techniques chained together in this room's scenario? -->

**Learning Objectives**
- Understand common MITM attack vectors and techniques
- Identify indicators of compromise related to MITM attacks
- Use network monitoring tools to detect suspicious traffic patterns
- Practice incident response procedures for MITM scenarios

### Task Questions

**1. Continue to the next task.**

**Answer:** N/A

---

## Task 2 - Lab Connection

### Key Concepts

<!-- Q: What three chained MITM techniques are investigated in this scenario? -->

**Attack Scenario:** As a SOC analyst, we investigate a MITM attack inside the corporate LAN where an attacker quietly intercepted communications, redirected connections, and captured user credentials.

<!-- Q: What are the three analysis paths available to investigate this room (pcap, log file, Splunk)? -->

**Three chained MITM techniques investigated:**
- ARP spoofing: network interception
- DNS spoofing: redirection
- SSL stripping: credential capture

### Task Questions

**1. Connect with the Lab.**

**Answer:** N/A

---

## Task 3 - MITM Attacks: An Overview

### Key Concepts

**MITM Overview**

<img src=screenshots/mitmuserattacker.png width="400">

<!-- Q: What are the two main steps that MITM attacks generally involve? -->

**How MITM Works** involves two main steps:
- **Interception:** Attackers insert themselves into the communication stream by exploiting weaknesses in network protocols such as ARP, DNS, or IP spoofing.
- **Manipulation/Decryption:** The attacker accesses or modifies the communication by decrypting encoded data or injecting harmful content such as altered website responses or fake login forms.

<!-- Q: What is SSL stripping and how does it differ from DNS spoofing as an attack technique? -->

**SSL Stripping vs. DNS Spoofing:**
- SSL stripping: removes the secure connection to steal or alter transferred data
- DNS spoofing: redirects legitimate traffic to a malicious domain

<!-- Q: Where does MITM fit within the Cyber Kill Chain, and why can it appear in more than one phase? -->

**Cyber Kill Chain** is a framework based on military knowledge of how an intruder performs their attack from beginning to end, mapping every step through TTPs (Tactics, Techniques, and Procedures):

| Phase | Description |
|---|---|
| Reconnaissance | Attacker gathers intel on the target to find vulnerabilities |
| Weaponization | Attacker prepares an exploit with a malicious payload into a deliverable package |
| Delivery | Attacker transmits the malicious package to the target system |
| Exploitation | Attacker's code activates, gaining control via software, hardware, or human vulnerability |
| Installation | Attacker installs malware or establishes a persistent backdoor on the compromised asset |
| Command and Control | Attacker establishes a channel to communicate with and control the compromised target |
| Actions on Objectives | Attacker accomplishes their goals: data exfiltration, destruction, lateral movement |

<!-- Q: What makes detecting an active MITM a critical finding from a SOC perspective? -->

**MITM signals that an adversary is in the middle stage of an intrusion**, presenting a crucial opportunity for the SOC to intervene and stop the attack before the attacker achieves their final objective.

| MITM Type | Mechanism | Primary Target |
|---|---|---|
| Packet Sniffing | Captures unencrypted packets | Open Wi-Fi traffic |
| Session Hijacking | Steals session tokens | Authenticated sessions |
| SSL Stripping | Downgrades HTTPS to HTTP | Encrypted web traffic |
| DNS Spoofing | Manipulates DNS responses | Website routing |
| IP Spoofing | Forges source IP packets | Trusted system impersonation |
| Rogue Wi-Fi AP | Creates fake network | All user traffic |

| Kill Chain Phase | MITM Role |
|---|---|
| Exploitation | Abuses trust in ARP/DNS protocols to intercept the communication channel |
| Installation | Uses the established MITM position to inject malicious payloads into traffic |

### Task Questions

**1. Continue to the next task.**

**Answer:** N/A

---

## Task 4 - Detecting ARP Spoofing

### Key Concepts

<!-- Q: What does ARP do and why does it have no authentication mechanism? -->

**ARP maps IP addresses to MAC addresses in the LAN.** When a device wants to send data to another IP, it asks "Who has this IP?" and the correct device replies with its MAC address.

<!-- Q: What is a gratuitous ARP reply and why is it a strong indicator of ARP spoofing? -->

**ARP has no authentication**, so any device can send an unsolicited `is-at` message. Attackers exploit this by sending fake ARP replies to victims and gateways, essentially saying "Hey, I have that MAC address you asked for" when nobody asked. That is what makes it so dangerous.

**Indicators of ARP Spoofing:**
- **Duplicate MAC-to-IP mapping:** Multiple MAC addresses claiming the same IP address
- **Unsolicited ARP replies:** High number of ARP replies without matching requests (gratuitous ARP)
- **Abnormal ARP traffic volume:** Large number of ARP packets in short intervals
- **Unusual traffic routing:** Traffic rerouted through the attacker's MAC
- **Gateway redirection patterns:** Multiple destination MACs for the same gateway IP
- **ARP probe/reply loops:** Many ARP requests with "Who has 192.168.1.x? Tell 192.168.1.y" patterns

<!-- Q: What Wireshark filter isolates gratuitous ARP packets? -->

<!-- Q: What does the arp.duplicate-address-detected filter confirm about the attack? -->

| Wireshark Filter | Purpose |
|---|---|
| `arp` | Isolate all ARP traffic |
| `arp.opcode == 1` | Show ARP requests only |
| `arp.opcode == 2` | Show ARP responses only |
| `arp.isgratuitous` | Show gratuitous (unsolicited) ARP replies |
| `arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == <gateway_mac>` | ARP traffic from legitimate gateway |
| `arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1` | All replies claiming gateway IP |
| `arp.opcode == 2 && _ws.col.info contains "192.168.10.1 is at"` | Responses mapping gateway IP to a MAC; also shows INFO column |
| `arp.duplicate-address-detected \|\| arp.duplicate-address-frame` | Duplicate IP-to-MAC mappings |

**Network Info:**

| Role | IP | MAC | Notes |
|---|---|---|---|
| Gateway | 192.168.10.1 | 02:aa:bb:cc:00:01 | Legit router |
| Attacker | 192.168.10.55 | 02:fe:fe:fe:55:55 | |
| Victim | 192.168.10.10 | | |
| Domain | corp-login.acme-corp.local | | |

### Task Questions

**1. How many ARP packets from the gateway MAC Address were observed?**

<img src=screenshots/gatewaymac.png width="400">
<img src=screenshots/gatewayfirst.png width="400">

*Filter by the gateway IP `arp.src.proto_ipv4 == 192.168.10.1` to observe two MAC addresses. Then filter by the legitimate gateway MAC `arp && eth.src == 02:aa:bb:cc:00:01` to isolate packets from the real gateway.*

**Answer:** 10

---

**2. What MAC address was used by the attacker to impersonate the gateway?**

<img src=screenshots/attackersmac.png width="400">

**Answer:** 02:fe:fe:fe:55:55

---

**3. How many Gratuitous ARP replies were observed for 192.168.10.1?**

<img src=screenshots/gratuitousipv4.png width="400">

**Answer:** 2

---

**4. How many unique MAC addresses claimed the same IP (192.168.10.1)?**

<img src=screenshots/sameipmac.png width="400">

*Filtered by the gateway IP `arp.src.proto_ipv4 == 192.168.10.1`.*

**Answer:** 2

---

**5. How many ARP spoofing packets were observed in total from the attacker?**

<img src=screenshots/totalspoofingpackets.png width="400">

*Filter by ARP responses containing the gateway IP mapped to the spoofed MAC: `arp.opcode == 2 && _ws.col.info contains "192.168.10.1 is at 02:fe:fe:fe:55:55"`.*

**Answer:** 14

---

## Task 5 - Unmasking DNS Spoofing

### Key Concepts

**DNS Spoofing**

<img src=screenshots/dnsthmspoofing.png width="400">

<!-- Q: What is DNS spoofing (cache poisoning) and how does it extend an ARP spoofing attack into a full MITM? -->

**DNS translates domain names like google.com into their numbered IP addresses.** DNS poisoning is when an attacker gives your computer the wrong IP for a domain name, pointing it to their malicious server instead.

<!-- Q: What is the single most reliable indicator of DNS spoofing in packet capture? -->

**Indicators of DNS Spoofing:**
- Multiple DNS responses for the same query
- DNS responses from an unexpected source
- Suspiciously short TTL values (attackers use 1-30 seconds)
- Unsolicited DNS responses

<!-- Q: Why do attackers use suspiciously short TTL values in forged DNS responses? -->

<!-- Q: What Wireshark filter reveals DNS responses from sources other than the legitimate DNS server? -->

| Wireshark Filter | Purpose |
|---|---|
| `dns` | Isolate all DNS traffic |
| `dns.flags.response == 1 && ip.src == 8.8.8.8` | Legitimate DNS responses from known resolver |
| `dns.flags.response == 1` | All DNS responses |
| `dns && dns.qry.name == "corp-login.acme-corp.local"` | DNS traffic for domain of interest |
| `dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"` | Spoofed DNS responses from rogue source |

| DNS Spoofing Indicator | What to Look For |
|---|---|
| Multiple responses for same query | Legitimate and forged responder both reply |
| Response from unexpected source | Reply IP does not match configured resolver |
| Short TTL values | TTL of 1-30 seconds |
| Unsolicited DNS responses | Reply with no matching request |

### Task Questions

**1. How many DNS responses were observed for the domain corp-login.acme-corp.local?**

<img src=screenshots/dnscorpacme.png width="400">

*Filtered by the domain: `dns.qry.name == "corp-login.acme-corp.local"`.*

**Answer:** 211

---

**2. How many DNS requests were observed from IPs other than 8.8.8.8?**

<img src=screenshots/otheripdns.png width="400">

*Filter by domain query name excluding the legitimate DNS server: `dns.flags.response == 1 && dns.qry.name == "corp-login.acme-corp.local" && ip.src != 8.8.8.8`.*

**Answer:** 2

---

**3. What IP did the attacker's forged DNS response return for the domain?**

<img src=screenshots/forgedattackerip.png width="400">

*The packet details show `[Unsolicited: True]` confirming a forged reply. The answer IP is visible in the Answers section of the DNS packet.*

**Answer:** 192.168.10.55

---

## Task 6 - Spotting SSL Stripping in Action

### Key Concepts

<img src=screenshots/httpstohttp.png width="400">

<!-- Q: What is SSL stripping and what does the victim's traffic look like after the attacker performs it? -->

**SSL Stripping** removes the security layer of HTTPS, downgrading it to plain HTTP. The attacker maintains a secure HTTPS session with the real server while relaying unencrypted HTTP to the victim, exposing everything in plaintext.

<!-- Q: What is the relationship between DNS spoofing and SSL stripping in a chained MITM attack? -->

**Indicators of SSL Stripping:**
- User requests HTTPS (port 443) but subsequent packets shift to HTTP (port 80) for the same domain
- Redirects and link rewriting from HTTPS to HTTP
- TLS/SSL handshake fails, shows errors, or presents a self-signed certificate

<!-- Q: What HTTP filter confirms the victim communicated with the attacker in plaintext after SSL stripping? -->

<!-- Q: What indicators in packet capture expose SSL stripping? -->

| Wireshark Filter | Purpose |
|---|---|
| `tls \|\| ssl` | All SSL/TLS traffic |
| `tls.handshake.type == 1 && tls.handshake.extensions_server_name == "corp-login.acme-corp.local"` | TLS handshakes to legitimate server |
| `dns.flags.response == 1 && ip.src == 192.168.10.55 && dns.qry.name == "corp-login.acme-corp.local"` | Forged DNS responses from attacker IP |
| `http && ip.src == 192.168.10.10 && ip.dst == 192.168.10.55` | Victim HTTP traffic to attacker (plaintext) |

| SSL Stripping Indicator | What to Look For |
|---|---|
| HTTPS to HTTP shift | Initial request port 443, subsequent traffic port 80 for same domain |
| Redirect rewriting | HTTP 301/302 redirects from HTTPS to HTTP |
| TLS handshake disappears | No TLS to real server after DNS spoof |
| Plaintext POST | Credentials visible in HTTP body |

**Attack Timeline Summary:**

| Phase | Event |
|---|---|
| ARP Spoofing | Attacker sends unsolicited ARP is-at claiming gateway IP, poisons victim cache |
| DNS Spoofing | Victim queries corp-login.acme-corp.local; forged DNS response redirects to 192.168.10.55 |
| SSL Stripping | Victim connects via HTTP to attacker; credential POST captured in cleartext |

### Task Questions

**1. How many POST requests were observed for our domain corp-login.acme-corp.local?**

<img src=screenshots/requestmethodpost.png width="400">

*Filtered by HTTP request method: `http.request.method == "POST"`.*

**Answer:** 1

---

**2. What is the password of the victim found in the plaintext after the successful SSL stripping attack?**

<img src=screenshots/strippedpass.png width="400">

**Answer:** Secret123!

---

## Task 7 - Conclusion and Room Wrap-up

### Key Concepts

<!-- Q: What are the three detection methods for each MITM technique covered in this room (ARP, DNS, SSL)? -->

**SOC Investigation Summary:**

**ARP Spoofing** (cache poisoning): Attacker sends unsolicited ARP `is-at` messages to claim the gateway IP, poisoning the victim's ARP cache.

**DNS Spoofing** (forged DNS responses): Victim's DNS query for corp-login.acme-corp.local is intercepted and a forged response redirects traffic to the attacker's IP 192.168.10.55.

**SSL Stripping** (TLS downgrade and credential capture): Victim initiates a connection to the resolved IP over HTTP; credential POST captured in cleartext.

<!-- Q: What Splunk index were the logs pre-ingested into, and what is the log file name for offline review? -->

| MITM Technique | Primary Detection Method |
|---|---|
| ARP Spoofing | Duplicate MAC addresses for same IP in ARP cache; gratuitous ARP replies in Wireshark |
| DNS Spoofing | Multiple conflicting DNS responses for same query |
| SSL Stripping | Sensitive data sent in plaintext HTTP to a domain that should use HTTPS |

### Task Questions

**1. Continue to complete the room.**

**Answer:** N/A
