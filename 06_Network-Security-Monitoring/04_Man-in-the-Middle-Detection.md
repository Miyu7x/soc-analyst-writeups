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
status: in-progress
date: 2026-05-14
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

<!-- Q: What is a Man-in-the-Middle attack and what position does the attacker occupy in the communication flow? -->

<!-- Q: What two-layer approach does blue team detection of MITM require? -->

<!-- Q: What are the three MITM techniques chained together in this room's scenario? -->

**Learning Objectives**
- Understand common MITM attack vectors and techniques
- Identify indicators of compromise related to MITM attacks
- Use network monitoring tools to detect suspicious traffic patterns
- Practice incident response procedures for MITM scenarios

### Task Questions

**1. Continue to the next task.**

Answer: N/A

---

## Task 2 - Lab Connection

### Key Concepts

<!-- Q: What three chained MITM techniques are investigated in this scenario? -->

<!-- Q: What are the three analysis paths available to investigate this room (pcap, log file, Splunk)? -->

### Task Questions

**1. Connect with the Lab.**

Answer: N/A

---

## Task 3 - MITM Attacks: An Overview

### Key Concepts

<!-- Q: What are the two main steps that MITM attacks generally involve? -->

<!-- Q: What is SSL stripping and how does it differ from DNS spoofing as an attack technique? -->

<!-- Q: Where does MITM fit within the Cyber Kill Chain, and why can it appear in more than one phase? -->

<!-- Q: What makes detecting an active MITM a critical finding from a SOC perspective? -->

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

Answer: N/A

---

## Task 4 - Detecting ARP Spoofing

### Key Concepts

<!-- Q: What does ARP do and why does it have no authentication mechanism? -->

<!-- Q: What is a gratuitous ARP reply and why is it a strong indicator of ARP spoofing? -->

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
| `arp.opcode == 2 && _ws.col.info contains "192.168.10.1 is at"` | Responses mapping gateway IP to a MAC |
| `arp.duplicate-address-detected \|\| arp.duplicate-address-frame` | Duplicate IP-to-MAC mappings |

**Network Info:**

| Role | IP | MAC | Notes |
|---|---|---|---|
| Gateway | 192.168.10.1 | | Legit router |
| Attacker | | | |
| Victim | | | |
| Domain | corp-login.acme-corp.local | | |

### Task Questions

**1. How many ARP packets from the gateway MAC Address were observed?**

Answer:

**2. What MAC address was used by the attacker to impersonate the gateway?**

Answer:

**3. How many Gratuitous ARP replies were observed for 192.168.10.1?**

Answer:

**4. How many unique MAC addresses claimed the same IP (192.168.10.1)?**

Answer:

**5. How many ARP spoofing packets were observed in total from the attacker?**

Answer:

---

## Task 5 - Unmasking DNS Spoofing

### Key Concepts

<!-- Q: What is DNS spoofing (cache poisoning) and how does it extend an ARP spoofing attack into a full MITM? -->

<!-- Q: What is the single most reliable indicator of DNS spoofing in packet capture? -->

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

Answer:

**2. How many DNS requests were observed from IPs other than 8.8.8.8?**

Answer:

**3. What IP did the attacker's forged DNS response return for the domain?**

Answer:

---

## Task 6 - Spotting SSL Stripping in Action

### Key Concepts

<!-- Q: What is SSL stripping and what does the victim's traffic look like after the attacker performs it? -->

<!-- Q: What is the relationship between DNS spoofing and SSL stripping in a chained MITM attack? -->

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

Answer:

**2. What is the password of the victim found in the plaintext after the successful SSL stripping attack?**

Answer:

---

## Task 7 - Conclusion and Room Wrap-up

### Key Concepts

<!-- Q: What are the three detection methods for each MITM technique covered in this room (ARP, DNS, SSL)? -->

<!-- Q: What Splunk index were the logs pre-ingested into, and what is the log file name for offline review? -->

| MITM Technique | Primary Detection Method |
|---|---|
| ARP Spoofing | Duplicate MAC addresses for same IP in ARP cache; gratuitous ARP replies in Wireshark |
| DNS Spoofing | Multiple conflicting DNS responses for same query |
| SSL Stripping | Sensitive data sent in plaintext HTTP to a domain that should use HTTPS |

### Task Questions

**1. Continue to complete the room.**

Answer: N/A
