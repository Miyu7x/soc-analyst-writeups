---
title: Network Traffic Basics
module: Network Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [NTA, network-analysis, TCP/IP, Wireshark, DNS-tunneling, packet-capture, SOC]
status: in-progress
date: 2026-04-25
date_completed:
---
*By [Renata](https://github.com/Miyu7x) | [TryHackMe](https://tryhackme.com/p/Miyu7)*

## Task 1 — Introduction
### Key Concepts

Wireshark is a traffic analysis tool, **NAT or Network Traffic Analysis** is process that combines 
	- logs
	- deep packet inspection
	- network flow statistics 
In order to have complete of whats communicated inside the network


---

## Task 2 — What is the Purpose of Network Traffic Analysis?

### Key Concepts
DNS Tunneling and Beaconing 

Example of firewall log of unusual number of DNS queries are coming from a a host named WIN-016 with IP 192.168.1.16

![Example of DNS log](screenshots/dns_queries_firewall.png)

Log includes:
	- Query and querytype 
	- Subdomain and top level domain 
	**(use abuseDB and VirusTotal to check for malicious domain)**
	- Host IP, whos sending out the queries
	- 
### Task Questions

1. **What is the name of the technique used to smuggle C2 commands via DNS?**
   - **Answer: DNS Tunneling**

---

## Task 3 — What Network Traffic Can We Observe?

### Key Concepts

Logs include bits and pieces of the header but not the full packet details

![Thm TCP/IP model](screenshots/thm_tcp_ip_model.png)
#### TCP/IP Layer Breakdown

| Layer       | What Logs Capture                | What Logs Miss              | Attack Detection Use Case |
| ----------- | -------------------------------- | --------------------------- | ------------------------- |
| Application | Header information               | application data or payload |                           |
| Transport   | source, destination ports, flags | all other fields?           |                           |
| Internet    |                                  |                             |                           |
| Link        |                                  |                             |                           |

**Application**
	- **GET** Request header client is requesting a file suspicious_package.zip

![](screenshots/thm_getrequest.png)

	- Response shows 200 code, which means it was accepted, shows file name, but not file contents

![](screenshots/thmget_response-1.png)

**Transport**
	- Application data and header are segmented and encapsulated into smaller pieces here at the transport layer
	- Source, destination ports and flags

Firewall log entry

![697](screenshots/thmfirewall_logentry.png)

Example of firewall logs including sequence numbers in the header
	- Far apart sequence numbers should warrant immediate investigation
	- first 3 lines show normal TCP 3-way handshake
	- lines 4 and 5 legitimate data transfer
	- line 6 shows a massive jump in **sequence number** as another source tries to inject itself into the session: **Session Hijacking**

![](screenshots/thm_sessionhijack_example.png)

**Internet**
	-Transport layer sends down a segment
		- **Internet layer** also adds its header
			- **Example**: mailing a large document. If the document is too thick to fit in one envelope, you split it into multiple envelopes and label each one so the recipient knows how to reassemble them in the right order.
			- MTU is basically the maximum size a single packet is allowed to be on a given network. A common MTU size is 1500 bytes on Ethernet. So if the internet layer receives a segment from transpot that is, say, 4000 bytes, it can't send that as one piece -- it's too big. So it breaks it into fragments (smaller chunks that fit within the MTU limit), and slaps a header on each fragment that contains info like:
				- what the original packet ID was (so the receiver knows which fragments belong together)
				- the fragment offset (so the receiver knows what order to reassemble them in)
				- a "more fragments" flag (so the receiver knows when the last piece has arrived)

**Example**an attacker can create tiny fragments to evade the IDS or mess up the reassembly of fragments by using overlapping byte ranges: **Fragmentation**

![](screenshots/thm_ids_log.png)

**Link**
	- Internet layer finishes encapsulation
	- IP Packet is sent to **Link Layer**
	- Link layer add its own header
		- Source
		- Destination MAC address
		Example: host with IP 192.168.1.200 is replying to each ARP request with the same MAC
		
![](screenshots/arp_spoof_example.png)
		
### Task Questions

1. **What is the size of the ZIP attachment included in the HTTP response? (in bytes)**
   - **Answer: 10485760 *

2. **Which attack do attackers use to try to evade an IDS?**
   - **Answer: Fragmentation**

3. **What field in the TCP header can we use to detect session hijacking?**
   - **Answer: Sequence Number**

---

## Task 4 — Network Traffic Sources and Flows

### Key Concepts

How does Network Flow and what are its sources?

2 sources of network flow
- Intermediary
- Endpoint
	- North-South: traffic that exits or enters the LAN and passes thru the firewall
	- East-West: Traffic that stays within the LAN(LAN on the cloud as-well)

#### Sources

| Category     | Description                           | Examples                                                           |
| ------------ | ------------------------------------- | ------------------------------------------------------------------ |
| Intermediary | in the middle                         | firewalls, switches, web proxies, IDS, IPS, routers, access points |
| Endpoint     | where all traffic originates and ends | servers, devices, vms, cloud resources, mobile phones, tablets     |

#### Flows

| Flow Type   | Direction                       | Common Protocols                                                   |
| ----------- | ------------------------------- | ------------------------------------------------------------------ |
| North-South | from LAN to WAN                 | HTTPS, DNS, SSH, VPN, SMTP, RDP                                    |
| East-West   | within the LAN, moves laterally | file shares, router, application communication, backup, monitoring |

**HTTPS Flow Example**
![](screenshots/thm_flow_example.png)

**External DNS Flow Example**
![](thm_dns_flowexample.png)

**SMB with Kerberos**? what does it really mean
![](screenshots/thm_smb_kerberos_flow.png)
### Task Questions

1. **Which category of devices generates the most traffic in a network?**
   - **Answer:**

2. **Before an SMB session can be established, which service needs to be contacted first for authentication?**
   - **Answer: Kerberos**

3. **What does TLS stand for?**
   - **Answer: Transport Layer Security**

---

## Task 5 — How Can We Observe Network Traffic?

### Key Concepts

**Observing a Network**
- Combination of multiple information sources
	- logs
	- full packet capture
	- network statistics
**Tools**
- Wireshark
- TVPdump
- IPS/IDS like Snort, Suricata and zeek

**Full Packet Capture**
- physical network tap
- port mirroring copies packets from one port on an intermediary device to another that is attached to
- full packet capture takes up a lot of storage: 1gbos line for 24 hours = 10 TB of storage!
  
  **Physical Network tap**
	- offers zero performance reduction when capturing large amounts of traffic

 ![](screenshots/physical_network_tap.png)

**Port Mirroring**

![](screenshots/thm_port_mirroring.png)
#### Collection Methods

| Method | How It Works | Performance Impact | Notes |
|---|---|---|---|
| Network TAP | | | |
| Port Mirroring (SPAN) | | | |

#### Network Statistics Protocols

**Network Statistics** is done collecting network metadata of the flowing data great for:
- number of DNS requests
- C2 traffic
- data exfiltration
- lateral movement

![](screenshots/thm_network_statisticsexample.png)


| Protocol | Origin | Key Feature |
|---|---|---|
| NetFlow | | |
| IPFIX | | |

<!-- Use cases: C2 traffic, data exfiltration, lateral movement detection -->

#### Analysis Tools
<!-- Wireshark, TCPdump, Snort, Suricata, Zeek -->

![](screenshots/networktraffic_scenario.png)

![](screenshots/malicious_psdownload.png)

![](tap_theConnection.png)
### Task Questions

![Place the TAP on the Web Proxy](screenshots/first_tap.png)

1. **What is the flag found in the HTTP traffic in scenario 1?**
![](screenshots/networktap_flag.png)
   - **Answer: THM{FoundTheMalware}**

1. **What is the flag found in the DNS traffic in scenario 2?**
   
![](screenshots/dns_tap.png)

![](screenshots/dns_tap_flagfind.png)

![](screenshots/dnstxt_flag.png)  
   - **Answer: THM{C2CommandFound}**

---

## Task 6 — Conclusion

### Key Concepts
<!-- Next step: Wireshark basics room -->

### Task Questions

1. **I am ready to do some traffic analysis!**
   - **Answer:** No answer needed