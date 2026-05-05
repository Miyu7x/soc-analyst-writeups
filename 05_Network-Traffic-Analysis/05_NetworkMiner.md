---
title: NetworkMiner
module: Network Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [network-forensics, pcap, NFAT, traffic-analysis, credential-extraction, OS-fingerprinting]
status: in-progress
date: 
date completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Room Introduction

### Key Concepts

NetworkMiner is open soruce Network Forensics Analysis Tool (NAFT)
  - Windows, Linux, macOS, FREEBSD
  - 2 ways to use it: passive sniffer that fingerprints hots, sessions and open ports without sending any traffic or: PCAP pareser that reassembles files and ceritficates captured from traffic 
  - supported data types: live traffic, traffic captures, log files
---

## Task 2 - NetworkMiner in Forensics

**NetworkMiner** is a great tool to provide quick and useful hints on where to start your inverstigation
  - context of captured hosts: IP and MAC addresses, hostnames and OS information
  - list of potential attack indicators: traffic spikes or port scans
  - tools and toolkits used to perform potential attacks

    
### Key Concepts

<!-- Three main data types in network forensics: live traffic, traffic captures, log files -->
<!-- NetworkMiner surfaces quick context: host IP/MAC, OS info, hostnames -->
<!-- Useful for spotting anomalies, traffic spikes, port scans, and toolkits used -->
<!-- Handles both PCAP files and live traffic -->

---

## Task 3 - What is NetworkMiner?

### Key Concepts

Pros and Cons of NetworkMiner

**Pros:** 
OS fingerprinting
Easy file extraction
Credential grabbing
Clear text keyword parsing
Overall overview

**Cons:**
Not useful in active sniffing
Not useful for large pcap investigation
Limited filtering
Not built for manual traffic investigation

### Feature Comparison

| Feature | NetworkMiner | Wireshark |
|---|---|---|
| Purpose | Quick overview, mapping, extraction | In-depth analysis |
| OS Fingerprinting | Yes | No |
| Filtering Options | Limited | Yes |
| Protocol Analysis | No | Yes |
| Payload Analysis | No | Yes |
| Statistical Analysis | No | Yes |
| Host Categorisation | Yes | No |
| Parameter/Keyword Discovery | Yes | Manual |

---

## Task 4 - Tool Overview 1

### Key Concepts

NetworkMiner is considered and initial investigation tool for gravving low hanging fruit and traffic overview
  - right click on case panel to view metadata

**Credentials Menu**



### Task Questions - mx-3.pcap

**1. What is the total number of frames?**

- **Answer:**

**2. How many IP addresses use the same MAC address with host 145.253.2.203?**

- **Answer:**

**3. How many packets were sent from host 65.208.228.223?**

- **Answer:**

**4. What is the name of the webserver banner under host 65.208.228.223?**

- **Answer:**

### Task Questions - mx-4.pcap

**5. What is the extracted username for the 02694W-WIN10 host?**

- **Answer:**

**6. What is the extracted password for the user logged into the 02694W-WIN10 host? Enter the full NTLM hash.**

- **Answer:**

---

## Task 5 - Tool Overview 2

### Key Concepts

<!-- Files tab: frame number, filename, extension, size, src/dst address, src/dst port, protocol, timestamp, reconstructed path, details -->
<!-- Images tab: hovering shows source/destination address and file path -->
<!-- Parameters tab: parameter name, value, frame number, src/dst host, src/dst port, timestamp, details -->
<!-- Keywords tab: must reload case files after updating search keywords; searches all processed pcap data -->
<!-- Messages tab: frame number, src/dst host, protocol, sender, receiver, timestamp, size; supports attachments -->
<!-- Anomalies tab: detects EternalBlue exploit and spoofing attempts; NetworkMiner is NOT a dedicated IDS -->

### Task Questions - mx-7.pcap

**1. What is the name of the Linux distro mentioned in the file associated with frame 63075?**

- **Answer:**

**2. What is the header of the page associated with frame 75942?**

- **Answer:**

**3. What is the source address of the image "ads.bmp.2E5F0FD9[1].bmp"?**

- **Answer:**

**4. What is the frame number of the possible TLS anomaly?**

- **Answer:**

### Task Questions - mx-9.pcap

**5. Which platform sent an email with the subject starting with "You have more"?**

- **Answer:**

**6. What is the email address of Branson Matheson?**

- **Answer:**

---

## Task 6 - Version Differences

### Key Concepts

<!-- v1.6 vs v2.7 are the two versions installed in the VM -->
<!-- MAC address conflict detection: available in v2+ only -->
<!-- Frame processing with detailed frame data: available in v1.6 only -->
<!-- Cleartext data tab (all extracted cleartext in one view): available in v1.6 only -->
<!-- Parameter processing is more extensive in v2+ -->
<!-- Packet details (sent/received breakdown): more detailed in v1.6 -->

### Version Feature Matrix

| Feature | v1.6 | v2.7 |
|---|---|---|
| MAC address conflict detection | No | Yes |
| Frame processing tab | Yes | No |
| Cleartext data tab | Yes | No |
| Extensive parameter processing | No | Yes |
| Detailed packet breakdown | Yes | No |

### Task Questions

**1. Which version can detect duplicate MAC addresses?**

- **Answer:**

**2. Which version can handle frames?**

- **Answer:**

**3. Which version can provide more details on packet details?**

- **Answer:**

---

## Task 7 - Exercises

### Key Concepts

<!-- Use case1.pcap with v1.6 for frame and packet detail questions -->
<!-- Use case2.pcap for USB, image, credential, and DNS questions -->
<!-- OS fingerprinting visible under Hosts tab -->
<!-- Byte counts for specific ports found in host communication details -->
<!-- Sequence numbers found in frame detail view (v1.6) -->
<!-- Content types counted from Parameters or Files tabs -->

### Task Questions - case1.pcap

**1. What is the full OS name of the host 131.151.37.122?**

- **Answer:**

**2. How many bytes were sent by the client (*.32.91) through port 1065?**

- **Answer:**

**3. How many bytes were sent back by the server (*.37.122) through port 143?**

- **Answer:**

**4. What is the sequence number of frame number 9?**

- **Answer:**

**5. What is the number of the detected "content types"?**

- **Answer:**

### Task Questions - case2.pcap

**6. What is the USB product's brand name?**

- **Answer:**

**7. What is the name of the phone model?**

- **Answer:**

**8. What is the source IP of the fish image?**

- **Answer:**

**9. What is the password of the homer.pwned.se@gmx.com?**

- **Answer:**

**10. What is the DNS query of frame 62001?**

- **Answer:**

---

## Task 8 - Conclusion

### Key Concepts

<!-- Do not use NetworkMiner as a primary sniffer -->
<!-- Use for quick traffic overview before moving to Wireshark or tcpdump -->
<!-- Next rooms: Wireshark, Snort, Brim -->
