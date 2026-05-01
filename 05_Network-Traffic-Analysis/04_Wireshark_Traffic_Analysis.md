---
title: Wireshark Traffic Analysis
module: Network Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [wireshark, traffic-analysis, nmap, arp, dhcp, dns, icmp, ftp, http, https, kerberos, netbios, soc-level-1]
status: in-progress
date: 
date completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 – Introduction

### Key Concepts

<!-- What distinguishes packet-level analysis from big-picture traffic correlation -->
<!-- Role of analyst knowledge vs tool functionality in anomaly detection -->
<!-- What kinds of anomalies and malicious activities this room targets -->

### Task Questions

**1. Read the task above.**

- **Answer:**

---

## Task 2 – Nmap Scans

### Key Concepts

<!-- Three Nmap scan types covered and what makes each distinct at the packet level -->
<!-- TCP Connect scan: handshake behavior, privilege level, window size indicator -->
<!-- SYN scan: half-open behavior, privilege level, window size indicator -->
<!-- UDP scan: no handshake, how closed vs open ports respond differently -->
<!-- Key Wireshark filters for each scan type and the logic behind them -->

| Scan Type | Nmap Flag | Handshake | Window Size | Closed Port Response |
|-----------|-----------|-----------|-------------|----------------------|
| TCP Connect | -sT | Full | > 1024 | RST, ACK |
| SYN | -sS | Half-open | <= 1024 | RST, ACK |
| UDP | -sU | None | N/A | ICMP Type 3 Code 3 |

### Task Questions

**1. What is the total number of the "TCP Connect" scans?**

- **Answer:**

**2. Which scan type is used to scan the TCP port 80?**

- **Answer:**

**3. How many "UDP close port" messages are there?**

- **Answer:**

**4. Which UDP port in the 55-70 port range is open?**

- **Answer:**

---

## Task 3 – ARP Poisoning & Man In The Middle

### Key Concepts

<!-- What ARP does and why its lack of authentication makes it exploitable -->
<!-- Difference between a legitimate ARP exchange and a spoofed one -->
<!-- How duplicate ARP responses signal a conflict and what Wireshark shows -->
<!-- Three-phase attacker pattern: announce own IP, claim gateway IP, flood requests -->
<!-- How to correlate ARP anomalies with HTTP traffic using MAC address columns -->

| Role | MAC Address | IP Address |
|------|-------------|------------|
| Attacker | | |
| Gateway | | |
| Victim | | |

### Task Questions

**1. What is the number of ARP requests crafted by the attacker?**

- **Answer:**

**2. What is the number of HTTP packets received by the attacker?**

- **Answer:**

**3. What is the number of sniffed username & password entries?**

- **Answer:**

**4. What is the password of the "Client986"?**

- **Answer:**

**5. What is the comment provided by the "Client354"?**

- **Answer:**

---

## Task 4 – Identifying Hosts: DHCP, NetBIOS and Kerberos

### Key Concepts

<!-- Why host and user identification matters at the start of an investigation -->
<!-- DHCP: which packet types carry hostname vs domain vs MAC vs IP info -->
<!-- DHCP option numbers worth memorizing and what each contains -->
<!-- NBNS: what it does and how to search by name -->
<!-- Kerberos: CNameString field, how to filter out hostnames vs usernames using $ -->

| Protocol | Key Filter | Low-Hanging Fruit |
|----------|------------|-------------------|
| DHCP | dhcp.option.dhcp == 3 | Option 12 hostname, Option 61 MAC |
| NBNS | nbns.name contains "keyword" | Query name, TTL, IP |
| Kerberos | kerberos.CNameString | CNameString, realm, sname |

### Task Questions

**1. What is the MAC address of the host "Galaxy A30"?**

- **Answer:**

**2. How many NetBIOS registration requests does the "LIVALJM" workstation have?**

- **Answer:**

**3. Which host requested the IP address "172.16.13.85"?**

- **Answer:**

**4. What is the IP address of the user "u5"? (Enter the address in defanged format.)**

- **Answer:**

**5. What is the hostname of the available host in the Kerberos packets?**

- **Answer:**

---

## Task 5 – Tunneling Traffic: DNS and ICMP

### Key Concepts

<!-- What traffic tunnelling is and why both defenders and attackers use it -->
<!-- ICMP tunnelling: how extra payload is used for exfil and C2, detection indicators -->
<!-- Normal ICMP packet size baseline and why that matters for anomaly detection -->
<!-- DNS tunnelling: encoded subdomain pattern, how C2 commands are routed -->
<!-- Key filters and what makes a DNS query length suspicious -->

| Protocol | Normal Behavior | Tunnelling Indicator |
|----------|----------------|----------------------|
| ICMP | 64-byte payload | data.len > 64, encoded payload |
| DNS | Short standard queries | Long subdomains, encoded strings, high query volume |

### Task Questions

**1. Use the "Desktop/exercise-pcaps/dns-icmp/icmp-tunnel.pcap" file. Investigate the anomalous packets. Which protocol is used in ICMP tunnelling?**

- **Answer:**

**2. Use the "Desktop/exercise-pcaps/dns-icmp/dns.pcap" file. Investigate the anomalous packets. What is the suspicious main domain address that receives anomalous DNS queries? (Enter the address in defanged format.)**

- **Answer:**

---

## Task 6 – Cleartext Protocol Analysis: FTP

### Key Concepts

<!-- Why FTP is a cleartext risk and what attack types it enables -->
<!-- FTP response code series: x1x, x2x, x3x and what each category covers -->
<!-- Key response codes for authentication events: 230, 331, 430, 530 -->
<!-- How to identify brute-force vs password spray from FTP filter combinations -->
<!-- Key FTP commands: USER, PASS, CWD, LIST and their filter syntax -->

| Code | Meaning | Detection Use |
|------|---------|---------------|
| 230 | Successful login | Confirmed access |
| 331 | Valid username | Username enumeration |
| 430 | Invalid user or password | Failed attempt |
| 530 | No login, invalid password | Brute-force signal |

### Task Questions

**1. How many incorrect login attempts are there?**

- **Answer:**

**2. What is the size of the file accessed by the "ftp" account?**

- **Answer:**

**3. The adversary uploaded a document to the FTP server. What is the filename?**

- **Answer:**

**4. The adversary tried to assign special flags to change the executing permissions of the uploaded file. What is the command used by the adversary?**

- **Answer:**

---

## Task 7 – Cleartext Protocol Analysis: HTTP

### Key Concepts

<!-- HTTP attacks detectable through traffic analysis: phishing, web attacks, exfil, C2 -->
<!-- Key request method and response code filters and what each status code signals -->
<!-- User-agent field: what makes one suspicious and why you cannot whitelist any value -->
<!-- Log4j attack pattern: POST request, jndi:ldap string, Exploit.class, base64 in user-agent -->
<!-- Why HTTP2 matters and how it differs from HTTP -->

| Anomaly Type | Wireshark Filter |
|--------------|-----------------|
| Nmap/scanner user agents | http.user_agent contains "Nmap" |
| Log4j initial POST | http.request.method == "POST" |
| Log4j payload | frame contains "jndi" |
| Encoded user agent | http.user_agent contains "$" |

### Task Questions

**1. Investigate the user agents. What is the number of anomalous "user-agent" types?**

- **Answer:**

**2. What is the packet number with a subtle spelling difference in the user agent field?**

- **Answer:**

**3. Locate the "Log4j" attack starting phase. What is the packet number?**

- **Answer:**

**4. Locate the "Log4j" attack starting phase and decode the base64 command. What is the IP address contacted by the adversary? (Enter the address in defanged format and exclude "{}")**

- **Answer:**

---

## Task 8 – Encrypted Protocol Analysis: Decrypting HTTPS

### Key Concepts

<!-- Why HTTPS obscures traffic and what TLS handshake steps look like in Wireshark -->
<!-- Client Hello vs Server Hello filters and what they reveal about TLS sessions -->
<!-- What a key log file is, how it is generated, and how to load it in Wireshark -->
<!-- What becomes visible post-decryption: HTTP2 packets, decompressed headers, flags -->
<!-- SSDP and why it is excluded from TLS hello filters -->

### Task Questions

**1. What is the frame number of the "Client Hello" message sent to "accounts.google.com"?**

- **Answer:**

**2. Decrypt the traffic with the "KeysLogFile.txt" file. What is the number of HTTP2 packets?**

- **Answer:**

**3. Go to Frame 322. What is the authority header of the HTTP2 packet? (Enter the address in defanged format.)**

- **Answer:**

**4. Investigate the decrypted packets and find the flag! What is the flag?**

- **Answer:**

---

## Task 9 – Bonus: Hunt Cleartext Credentials

### Key Concepts

<!-- Why spotting multiple credential entries at packet level is difficult manually -->
<!-- Which Wireshark dissectors support the Credentials feature: FTP, HTTP, IMAP, POP, SMTP -->
<!-- Minimum Wireshark version required and why manual checks still matter -->
<!-- What the Credentials window shows and how clicking fields navigates to packets -->

### Task Questions

**1. Use the "Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap" file. What is the packet number of the credentials using "HTTP Basic Auth"?**

- **Answer:**

**2. What is the packet number where "empty password" was submitted?**

- **Answer:**

---

## Task 10 – Bonus: Actionable Results

### Key Concepts

<!-- What the Firewall ACL Rules feature does and where to access it -->
<!-- Which firewall platforms Wireshark can generate rules for -->
<!-- Rule scope: IP, port, MAC address combinations, outside interface targeting -->

### Task Questions

**1. Use the "Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap" file. Select packet number 99. Create a rule for "IPFirewall (ipfw)". What is the rule for "denying source IPv4 address"?**

- **Answer:**

**2. Select packet number 231. Create "IPFirewall" rules. What is the rule for "allowing destination MAC address"?**

- **Answer:**

---

## Task 11 – Conclusion

### Key Concepts

<!-- Why Wireshark alone is not sufficient and what IDS/IPS adds to the workflow -->
<!-- Next rooms in the network traffic analysis progression -->

### Task Questions

**1. Read the task above.**

- **Answer:**

---

## References

<!-- Wireshark filter cheat sheet built during this room -->
<!-- ICMP and DNS tunnelling indicators -->
<!-- Log4j detection patterns -->
<!-- FTP response code reference -->
<!-- TLS handshake filter syntax -->