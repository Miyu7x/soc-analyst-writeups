---
title: Data Exfiltration Detection
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [data-exfiltration, dns-tunneling, ftp, http, icmp, wireshark, splunk, network-forensics, soc-level-1]
status: in-progress
date: 2026-05-13
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

<!-- Data exfiltration defined: unauthorized transfer of sensitive data out of a network, primary attacker objective post-breach -->
<!-- SOC analyst role: detect and stop exfiltration before data leaves the environment -->
<!-- Room scope: common exfil techniques, network traffic analysis, endpoint indicators, SIEM log correlation -->

**1. Continue to the next task.**

Answer: N/A

---

## Task 2 - Lab Connection

### Key Concepts

<!-- Lab setup: all files in data_exfil folder on Desktop; Splunk pre-ingested at MACHINE_IP:8000 -->
<!-- Three investigation approaches: pcap analysis, raw log file review, Splunk queries -->
<!-- Always set Time Range to All Time; base index = index=data_exfil -->

**1. Connect with the lab.**

Answer: N/A

---

## Task 3 - Data Exfil: Overview, Techniques, and Indicators

### Key Concepts

<!-- Five adversary motivations: financial gain, espionage, ransomware/extortion, disruption/sabotage, persistence/recon -->
<!-- Four common exfil phases: Discovery/Collection > Staging/Compression > Exfiltration Transport > C2 Coordination -->
<!-- Detection requires cross-source correlation: host + network + cloud telemetry, not single-point alerts -->
<!-- SOC L1 triage anchors: source host/user, destination, volume transferred, process identity/command-line, corroborating logs -->

### Threat Actor Reference

| Threat Actor / Campaign | Exfiltration Technique | Description |
|---|---|---|
| APT29 (Cozy Bear) | HTTPS over legitimate domains | Encrypted HTTPS channels used to exfiltrate from government networks |
| FIN7 | HTTP POST to C2 servers | Embedded stolen data in HTTP POST requests to evade detection |
| Lunar Spider (Zloader) | Encrypted C2 channels | Two-month intrusion using encrypted channels and staged exfiltration |
| DarkSide Ransomware | Dual extortion: encryption + exfil | Stole data before encrypting, then threatened public leaks |
| APT10 (Cloud Hopper) | Cloud-to-cloud transfer | Exfiltrated from managed service providers using cloud APIs |

### Techniques and Indicators Reference

| Technique | Examples | Where to Look |
|---|---|---|
| Network-based | HTTP/HTTPS uploads to S3/Azure Blob/webmail, FTP/SFTP/SCP, DNS tunneling, ICMP/covert protocols, custom TCP/UDP | Proxy/web gateway logs (large POSTs), firewall/NGFW flows (high bytes to single IP/ASN), netflow, DNS logs (long hostnames, TXT queries) |
| Host-based | PowerShell/Invoke-WebRequest, rclone, awscli, curl/wget, archive creation (zip/rar/7z), removable USB, ADS/hidden streams | Sysmon/EDR (EventID 1/3/11), Windows Security (4663/4656), auditd/shell history on Linux, removable-media events |
| Cloud exfiltration | S3 PutObject/multipart upload, Azure Blob uploads, GCS objects, Drive/SharePoint external sharing | CloudTrail / Azure Activity / GCP Audit, cloud storage access logs, unusual service-account or IP activity |
| Covert and encoding | DNS tunneling, base64/chunked encoding, steganography into images/audio, low-and-slow file splitting | DNS logs, proxy logs with many small POSTs, correlation of intermittent uploads + suspicious process activity |
| Insider and collaboration tools | Slack/Teams/Dropbox/Google Drive/Box uploads or sharing to external users, compromised employee accounts | Audit logs (share events, file downloads), mail logs |

**1. Exfiltrating the data through HTTP comes under which technique?**

Answer:

---

## Task 4 - Detection: Data Exfil through DNS Tunneling

### Key Concepts

<!-- DNS tunneling: encodes data inside DNS query/response; abuses the fact that DNS is almost always allowed outbound -->
<!-- DNS uses UDP port 53 for queries; TCP port 53 for zone transfers or large responses -->
<!-- Two confirmed lab indicators: large number of DNS requests with no response + abnormally large query length -->
<!-- Multiple internal hosts can be compromised simultaneously; all tunnel to one external domain -->

### DNS Tunneling IoAs

| Indicator | What to Look For |
|---|---|
| Query volume | Many DNS queries to a single external domain, far above baseline |
| Query length | Long subdomain labels or full query names > 60-100 characters |
| Encoding patterns | High-entropy or base64/base32-like patterns (mixed case, digits, `-`, `=`) |
| Record types | Rare types such as TXT or NULL; many large TXT responses |
| Response behavior | Frequent NXDOMAIN; TCP or large UDP fragments for DNS |
| Timing | Evenly spaced queries (beaconing behavior) |

### Wireshark Commands

```
# Isolate all DNS traffic
dns

# Show DNS queries with no response
dns.flags.response == 0

# Find long queries (suspicious subdomain lengths)
dns && frame.len > 70

# Filter on the identified suspicious domain
dns && dns.qry.name contains "<SUSPICIOUS_DOMAIN>"
```

### Splunk Commands

```
# Baseline: all DNS logs
index=data_exfil sourcetype=DNS_logs

# Query count per source IP
index="data_exfil" sourcetype="DNS_logs" | stats count by src_ip

# Query frequency sorted by count
index="data_exfil" sourcetype="dns_logs" | stats count by query | sort -count

# Filter on long query names (> 30 chars)
index="data_exfil" sourcetype="DNS_logs" | where len(query) > 30
```

**1. What is the suspicious domain receiving the DNS traffic?**

Answer:

---

**2. How many suspicious traffic/logs related to DNS tunneling were observed?**

Answer:

---

**3. Which local IP sent the maximum number of suspicious requests?**

Answer:

---

## Task 5 - Detection: Data Exfil through FTP

### Key Concepts

<!-- FTP: plaintext protocol; attackers abuse via compromised creds, misconfigured servers, ephemeral accounts -->
<!-- Credentials transmitted in cleartext; USER and PASS visible directly in TCP stream -->
<!-- STOR = upload (attacker sending data out), RETR = download from server -->
<!-- Non-standard ports and tunneling used to blend FTP with other traffic -->

### FTP IoAs

| Indicator | What to Look For |
|---|---|
| Credentials | USER and PASS commands in cleartext; weak or guest accounts |
| Upload commands | STOR repeated or with large transfers; RETR for staged data retrieval |
| Data connections | Large payloads on ephemeral PASV ports to unusual external IPs |
| Timing | Transfers outside business hours |
| File types | Sensitive extensions in STOR commands: pdf, csv, txt |

### Wireshark Commands

```
# Isolate all FTP control and data traffic
ftp || ftp-data

# Show only login attempts
ftp.request.command == "USER" || ftp.request.command == "PASS"

# Look for upload commands
ftp contains "STOR"

# Filter on CSV file transfers specifically
ftp contains "csv"

# Identify large payload transfers
ftp && frame.len > 90

# Right-click any packet > Follow > TCP Stream to inspect content in cleartext
```

**1. How many connections were observed from the guest account?**

Answer:

---

**2. Apply the filter; what is the name of the customer-related file exfiltrated from the root account?**

Answer:

---

**3. Which internal IP was found to be sending the largest payload to an external IP?**

Answer:

---

**4. What is the flag hidden inside the FTP stream transferring the CSV file to the suspicious IP?**

Answer:

---

## Task 6 - Detection: Data Exfil via HTTP

### Key Concepts

<!-- HTTP exfil blends with normal web traffic; hard to detect without filtering on size and destination -->
<!-- Adversaries adapt: low-and-slow, encoding, chunked/multipart, HTTPS tunneling, staging via cloud -->
<!-- Correlation approach: match Splunk finding to pcap entry, then inspect HTTP stream for exfiltrated content -->
<!-- Detection requires both SIEM log analysis and pcap confirmation to be conclusive -->

### HTTP Exfil Techniques

| Technique | Description |
|---|---|
| POST uploads | Bulk data sent to attacker-controlled hosts or cloud storage in POST request bodies |
| GET with encoded data | Small chunks in query strings or path segments (low-and-slow) |
| Common services / CDN | Exfiltration disguised as uploads to popular or attacker-controlled subdomains |
| Custom headers | Data placed in headers (e.g., `X-Data: <base64>`) to bypass string-based DLP |
| Chunked / multipart | Large payloads split across multiple requests to avoid size thresholds |
| HTTPS/TLS tunneling | Encrypted channel hides payload; requires TLS inspection, SNI analysis, or metadata detection |
| Staging via cloud | Attacker uploads to Dropbox/GitHub/Gist and fetches externally |

### HTTP IoAs

| Indicator | What to Look For |
|---|---|
| Request size | Unusually large HTTP POST bodies to external or unexpected hosts |
| Destination reputation | Requests to low-reputation or rarely-seen-in-baseline domains |
| Beaconing + upload pattern | Frequent small requests to same host followed by large uploads |
| Transfer encoding | Chunked or multipart transfers composing a larger file across requests |

### Splunk Commands

```
# Baseline: all HTTP logs
index="data_exfil" sourcetype="http_logs"

# Filter on POST requests only
index="data_exfil" sourcetype="http_logs" method=POST

# Average, max, and min bytes sent per domain, sorted by count
index="data_exfil" sourcetype="http_logs" method=POST | stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain | sort - count

# Isolate large POST payloads (filter out benign traffic)
index="data_exfil" sourcetype="http_logs" method=POST bytes_sent > 600 | table _time src_ip uri domain dst_ip bytes_sent | sort - bytes_sent
```

### Wireshark Commands

```
# Isolate all HTTP traffic
http

# Filter on POST requests only
http.request.method == "POST"

# Large POST frames (initial threshold)
http.request.method == "POST" and frame.len > 500

# Tighten threshold to isolate the exfil entry
http.request.method == "POST" and frame.len > 750

# Right-click result > Follow > HTTP Stream to inspect exfiltrated content
```

**1. Which internal compromised host was used to exfiltrate this sensitive data?**

Answer:

---

**2. What's the flag hidden inside the exfiltrated data?**

Answer:

---

## Task 7 - Detection: Data Exfiltration via ICMP

### Key Concepts

<!-- ICMP tunneling: data encoded (base64/hex) into echo request/reply payloads; allowed through most firewalls, low inspection -->
<!-- Normal ping frame is ~74 bytes total; anything over 100 bytes is suspicious and warrants payload inspection -->
<!-- Detection is straightforward compared to DNS/HTTP: frame size anomaly is the primary signal -->
<!-- Bursts of large ICMP with no other app traffic from same host = strong indicator -->

### ICMP Tunneling Techniques

| Technique | Description |
|---|---|
| Echo request/reply tunneling | Data encoded (base64/hex) into ICMP type 8 payloads; remote server collects and decodes |
| Custom ICMP types/codes | Uncommon ICMP types or non-zero codes to evade signature-based detection |
| Fragmentation and reassembly | Large payloads split across multiple packets |
| Encryption/obfuscation | Encrypted or base64-encoded payloads appear as random data in payload bytes |

### ICMP IoAs

| Indicator | What to Look For |
|---|---|
| Session persistence | Ongoing ICMP to external host not used for legitimate monitoring |
| Payload size | ICMP payloads > 64 bytes; total frame.len > 100 (normal ping ~74 bytes) |
| Payload content | High-entropy data or base64/hex patterns in the Data field |
| Traffic isolation | Bursts of ICMP immediately followed by no other legitimate app traffic from same host |
| Timing | Evenly spaced ICMP packets with similar payload sizes (beaconing) |

### Wireshark Commands

```
# Isolate all ICMP traffic
icmp

# Filter on echo requests only (type 8)
icmp.type == 8

# Flag unusually large payloads (normal ping ~74 bytes; > 100 is suspicious)
icmp.type == 8 and frame.len > 100

# Inspect payload: select packet > expand ICMP layer in packet details > check Data field
```

**1. What is the flag found in the exfiltrated data through ICMP?**

Answer:

---

## Task 8 - Conclusion

### Key Concepts

<!-- Room covered: exfil fundamentals, Cyber Kill Chain (Actions on Objectives), FTP/HTTP/DNS/ICMP detection -->
<!-- Detection is network-centric in this room; future rooms will expand to additional channels and log sources -->
<!-- No single-point detection is sufficient; effective SOC work requires cross-source correlation -->

**1. Continue to complete the room.**

Answer: N/A

---
