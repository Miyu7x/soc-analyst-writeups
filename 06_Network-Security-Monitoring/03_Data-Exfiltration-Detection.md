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

<!-- Q: What is data exfiltration and why is it a primary objective for attackers who have already breached a network? -->
**Data Exfiltration** the act of stealing sensitive data from a network
  - reasons for attackers vary: monitary, political, service disruption, recon...

<!-- Q: What are the four things this room will teach you to do as a SOC analyst? -->
**SOC** analysts are responsible for detecting and stopping attackers before they break in and take off with valuable data
  - understand common methods of data exfil
  - how to detect exfil attempts while perfoming network analysis
  - spot signs of exfil on endpoint devices
  - perform investigations utilizing SIEM to detect exfil channnels

**1. Continue to the next task.**

Answer: N/A

---

## Task 2 - Lab Connection

### Key Concepts

<!-- Q: Where are all the lab files located on the Desktop, and how do you access the Splunk instance? -->
The **Security Information Event Management** we will be using today is SPLUNK
  - to access the exfil logs in the SIEM we can use **index=data_exfil**
<!-- Q: What are the three ways to approach the investigation in this room? -->
**SOCs** should start their investigation looking for:
  - sysmon_logs
  - http_logs
  - dns_logs
    
**1. Connect with the lab.**

Answer: N/A

---

## Task 3 - Data Exfil: Overview, Techniques, and Indicators

### Key Concepts

<!-- Q: What are the five reasons adversaries perform data exfiltration? -->
Threat actors will try to extract data for various reasons"
  - financial gain
  - espionage
  - ransomware
  - disruption
  - persistance and recon
<!-- Q: What are the four phases an attacker moves through before data leaves the network? -->
Attackers have to get through several steps before they can access the dataand then try to exfil
  1. Discover phase, find the files
  2. stage and compress the files
  3. exfil the data
  4. C2 to transfer and confirm success
<!-- Q: Which threat actor used cloud APIs to exfiltrate from managed service providers, and what is that campaign called? -->
This info is on the table
<!-- Q: What log sources would you check for host-based exfiltration indicators, and which Sysmon event IDs are relevant? -->
Host-Based exfilitration would be seen in the sysmon_logs
Network-Based exfil in the http_logs
<!-- Q: Why is single-point detection not enough, and what does cross-source correlation actually mean in practice? -->
Since attackers start their activties earlier than exfil SOCs should be looking not just at endpoints where the data leaves but detective work accross the network

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

**Answer: Network Based**
---

## Task 4 - Detection: Data Exfil through DNS Tunneling

### Key Concepts

<!-- Q: Why is DNS an attractive protocol for data exfiltration, and what ports does it use for different traffic types? -->
**DNS is a prime target for attackers**
  - DNS is typically allowed freely through networks, its an **Always-on** service
  - attackers sumggle bytes inside DNS quesries/responses
  - firewalls and web proxies can miss this
<!-- Q: What six indicators in DNS traffic should make you suspicious of tunneling activity? -->
**Possible Indicators of a DNS Attack**
  - high number of DNS queries sent to a single external domain
  - long subdomains labels or unusual full query names
  - Base32/Base64 patterns in the query name
  - rare record types, large TXT reponses and NULL
  - queries at regular interval, beaconing
<!-- Q: What two confirmed indicators did this lab use to identify the DNS tunneling attempt? -->
<!-- Q: Walk through the Wireshark filter progression used in this task. What does each filter narrow down? -->
**Detecting with Wireshark**
  - **index=data_exfil sourcetype=DNS_logs**
  - wireshark DNS queries with no response **dns.flags.repsonse == 0**
  - wireshark filter for long DNS queries **dns && frame.len > 70**
  - wireshark filter by the name of the suspicious domain **dns && dns.qry.name contains *domainname***
  - SPLUNK filter stats of DNS quesries by source IP **stats count by src_ip**
  - SPLUNK filter by suspicious domains **stats count by query | sort -count**
  - SPLUNK  |filter by subdomain encoding **where len(query) > 30**
<!-- Q: Which Splunk query actually surfaces the suspicious queries, and why does query length matter? -->

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
**IMPORTANT Note: Suspicious traffic will look like Tunneling so we would filter by query len**

**1. What is the suspicious domain receiving the DNS traffic?**

<img src=screenshots/dnsexfilexample.png width"600">
*The question mentions a domain, I used SPLUNK and applied the filter to sort by query and count the requests*
**stats count by query | sort -count**

**Answer: tunnelcorp.net**
---

**2. How many suspicious traffic/logs related to DNS tunneling were observed?**

<img src=screenshots/wiresharktunnelcorp.png width"600">
*In this question I tried the filter contain tunnelcorp and wasnt getting a result, I figured I was using the wrong tool, so i examined the dns log on wireshark instead*
**dns && dns.qry.name contains tunnelcorp**

**Answer: 315**
---

**3. Which local IP sent the maximum number of suspicious requests?**

<img src=screenshots/tunnelcorplen.png width"600">
*Here they are looking specifically for suspicious traggic only*
**where len (query) > 30**

**Answer: 192.168.1.103**
---

## Task 5 - Detection: Data Exfil through FTP

### Key Concepts

<!-- Q: Why is FTP particularly dangerous from a credentials standpoint, and what makes it useful to attackers? -->
FTP is often used by attacker for Exfil because its a  transferring of files between a client and server over TCP/IP network
  - they use compromised credentials, service accounts and user creds
  - non standard ports or tunneling
  - detected via packet inspections, server logs and SSH metadata and network flow/size/pattern analysis
<!-- Q: What do the FTP commands STOR and RETR each tell you about what the attacker was doing? -->
**Indicators of Attack**
   - USER and PASS commands
   - STOR for uploading amd RETR for downloads
   - repeated or large transfers
   - data channel openings on PASV ports along with large payloads
<!-- Q: What filter progression did this task use in Wireshark, and what did each step reveal? -->
**Wireshark FTP filters**
**ftp || ftp data**
  - filter login attempts with USER/PASS **ftp.request.command == "USER" || ftp.request.command == "PASS"**
  - filter by filenames or credentials **ftp contains "STOR"
      - right click to follow TCP stream
  - filter out by file extensions PDF, csv, TXT... **ftp contains "csv"**
  - filter out by traffic with large pauload size **ftp && frame.len > 90**
      - follow stream
<!-- Q: What made the guest account activity suspicious in this lab specifically? -->

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

<img src=screenshots/wiresharkftpguest.png width="600">
*Filtered by ftp contains "guest"*

**Answer: 5**
---

**2. Apply the filter; what is the name of the customer-related file exfiltrated from the root account?**

<img src=screenshots/filterbyroot.png width="600">
*I orginally thought about filtering for files, but you would need to know the exact file type, so I filtered by root instead and inspected the packets*

**Answer: customer_data.xlsx**
---

**3. Which internal IP was found to be sending the largest payload to an external IP?**

<img src=screenshots/largeguestpayload.png width="600">
*Filtered this one by length **ftp && frame.len >90** and sorted highest to lowest*

**Answer: 192.168.1.105**
---

**4. What is the flag hidden inside the FTP stream transferring the CSV file to the suspicious IP?**

<img src=screenshots/thmftpexfil.png width="600">
**Answer: THM{ftp_exfil_hidden_flag}**
---

## Task 6 - Detection: Data Exfil via HTTP

### Key Concepts

<!-- Q: Why does HTTP exfiltration blend in so well with normal traffic, and what makes it hard to detect? -->
Attackers use **HTTP** to exfil data because it blends in with normal web traffic
  - HTTP traffic passes by firewalls and proxies, it can be obfuscated(encrypted, ecoded, tunneled)
<!-- Q: What are the seven techniques adversaries use to abuse HTTP for exfiltration? -->
**Techniques Used by Attackers**
  - **POST** uploads to external servers: bulk data is sent to attacker-cpntrolled hosts or cloud storage in POST request bodies
  - **GET** requests with encoded data: they squeeze small chunks into query strings or path segments
  - **Common Services**
  - **Custom headers** data is placed inside headers ExampleData: <base64>
  - **Chunked transfer/multipart** large payloads are split into multiple reuqests to avoid size trashhold
  - **HTTPS/TLS tunneling** encrypted channel hides the payload
  - **Staging via cloud services** the attacker uploads to Dropbox/Git/Gist
<!-- Q: Why did the task increase the frame.len threshold from 500 to 750 in Wireshark, and what did that achieve? -->
We can increase frame len in wireshark to cut out more noise from 500 to 750
<!-- Q: How did the Splunk investigation and the Wireshark pcap analysis complement each other in this lab? -->
<!-- Q: What is a low-and-slow exfiltration approach and why is it harder to catch than a single large transfer? -->
**Logs in SPLUNK**
  - filter to get an overview of the avarage amount of bytes sentout to various domains
    **method=POST | stats count avg(bytes_sent) max(bytes_sent) by domain | sort - count**
  - filter POST requests with large payloads
    **method=POST bytes_sent > 600 | table_time src_ip uri domain dst_ip bytes_sent | sort - bytes_sent**

**Logs in Wireshark**
  - GET or POST requests: **http.request.method == "POST"**
  - **and fram.len > 500** increase to ;750 to cut out noise

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

<img src=screenshots/packetbytesthmflag.png width="600">
*Filtered by POST looking for large payloads **http.request.method == "POST" and frame.len > 750**
Answer:

---

**2. What's the flag hidden inside the exfiltrated data?**

<img src=screenshots/packetbytesthmflag2.png width="600">
*Im not sure how they wanted us to find the answer but my eyes are always reading the packet bytes, so..*
Answer: THM{http_raw_3xf1ltr4t10n_succ3ss}

---

## Task 7 - Detection: Data Exfiltration via ICMP

### Key Concepts

<!-- Q: Why do attackers use ICMP for exfiltration, and what property of the protocol makes it attractive? -->
**ICMP is commonly used for exfil for being comonly allowed thru firewalls**

**Attackers and ICMP**
  - ICMP echo(type 8)/reply(type 0) tunneling: they place encoded chunks of files in ICMP payloads
  - Uncommon ICMP types
  - Fragmentation and reassmbly: latge payloads split accross multiple packets
  - ebncrption and obfuscation
  - large frame.len and icmp.payload
  - regular timing

Wireshark ICMP Logs
  - filter by ICMP Echo reuqests **icmp.type == 8**
  - filter by frame lenghth ** frame.len > 100**
      - normal packets range around 74 bytes total, anything over 100 is suspiciousic
<!-- Q: What is the normal frame size of a ping, and at what frame.len does ICMP traffic become suspicious? -->
<!-- Q: What are the four techniques attackers use to abuse ICMP specifically? -->
<!-- Q: Why is ICMP exfiltration detection simpler than DNS or HTTP tunneling detection? -->

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

<img src=screenshots/icmplogflag.png width="600">
*Same on this one I think they might have wanted the FLAG a different way but it was visible right there on the packet bytes*

**Answer: THM{1cmp_3ch0_3xf1ltr4t10n_succ3ss}**
---

## Task 8 - Conclusion

### Key Concepts

<!-- Q: What four areas did this room cover from a defender's perspective? -->
<!-- Q: What does "Actions on Objectives" mean in the context of the Cyber Kill Chain, and where does exfiltration fit? -->
<!-- Q: What channels and log sources did this room NOT cover that future rooms will expand on? -->

**1. Continue to complete the room.**

Answer: N/A

---
