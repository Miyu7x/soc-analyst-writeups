---
title: NetworkMiner
module: Network Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [network-forensics, pcap, NFAT, traffic-analysis, credential-extraction, OS-fingerprinting]
status: completed
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

Three main data types in network forensics: live traffic, traffic captures, log files
Handles both PCAP files and live traffic

---

## Task 3 - What is NetworkMiner?

### Key Concepts

NetworkMiner has a sniffing feature but it is not inteded to be used as a sniffer
  - only valaibale on windows
  - not as reliable as other feauture
  - not a dedicated sniffer tool like Wireshark and tcpdumpg

NetworkMiner is great for quick packet parsing and processing
  - grabs the low hanging fruit before a deep dive

**Suggested Work Flow**
  - record traffic for offline analysis
  - quickly overview the pcap with NetworkMiner
  - do a deep analysis with Wireshark

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
  - drag and drop files
  - tools: clears the dash and removes captured data
  - help menu: provides infro on the current version and updates
  - case panel: shows list of investigated pcap files, right click on case panel to view metadata
  - hosts: IP address, MAC address, OS type, open ports, sent/received packets, incoming/outgoing session and, host details
    - OS fingerprinting uses the Satori GitHub repo and p0f, and the MAC address DB uses mac-ages Github-repo
  - sessions: frame number, client and server address
  - source and destination port
  - protocol
  - start time
      - search for keywords inside frames: "ExactPhrase" "AllWords" "AnyWord" "RegExe"
    -DNS: frame number, timestamp, client and server, source/destination ports, IP TTL, DNS time, trnasaction ID and type, DNS query and answer, Alexa Top1m
  - credentials: kerberos hashes, 




### Task Questions - mx-3.pcap

**1. What is the total number of frames?**

![](screenshots/networkminer_metadata.png)
*Followed the Case Panel and right clik to view Metada*
- **Answer: 460**

**2. How many IP addresses use the same MAC address with host 145.253.2.203?**

![](screenshots/networkminer_samemac.png)
- **Answer: 2**

**3. How many packets were sent from host 65.208.228.223?**

![](screenshots/networkminer_sentpackets.png)
- **Answer: 72**

**4. What is the name of the webserver banner under host 65.208.228.223?**

![](screenshots/networkminer_webserver.png)
- **Answer: Apache**

### Task Questions - mx-4.pcap

**5. What is the extracted username for the 02694W-WIN10 host?**

![](screenshots/networkminer_username.png)
*Credentials*
- **Answer: #B/Administrator**

**6. What is the extracted password for the user logged into the 02694W-WIN10 host? Enter the full NTLM hash.**

![](screenshots/networkminer_passwords.png)
*Right-click to copy password*
- **Answer: $NETNTLMv2$#B$136B077D942D9A63$FBFF3C253926907AAAAD670A9037F2A5$01010000000000000094D71AE38CD60170A8D571127AE49E00000000020004003300420001001E003000310035003600360053002D00570049004E00310036002D004900520004001E0074006800720065006500620065006500730063006F002E0063006F006D0003003E003000310035003600360073002D00770069006E00310036002D00690072002E0074006800720065006500620065006500730063006F002E0063006F006D0005001E0074006800720065006500620065006500730063006F002E0063006F006D00070008000094D71AE38CD601060004000200000008003000300000000000000000000000003000009050B30CECBEBD73F501D6A2B88286851A6E84DDFAE1211D512A6A5A72594D340A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E00360036002E0033003600000000000000000000000000**

---

## Task 5 - Tool Overview 2

### Key Concepts

NetworkMiner has some features only avaliable to premium
   - OSINT hash lookup and sample submission
   - search bar
   - right click allows in depth views of open files and folders
   - messages menu: extracted emails, chats and messages


### Task Questions - mx-7.pcap

**1. What is the name of the Linux distro mentioned in the file associated with frame 63075?**

![](screenshots/networkminer_63075.png)
*Filtered for the frame number 63075 under a couple different tabs and found it under files)
- **Answer: CentOS**

**2. What is the header of the page associated with frame 75942?**

![](screenshots/networkminer_header.png)
- **Answer: Password-Ned AB**

**3. What is the source address of the image "ads.bmp.2E5F0FD9[1].bmp"?**

![](screenshots/networkminer_imgip.png)
*Searched for the file name*
- **Answer: 80.239.178.187**

**4. What is the frame number of the possible TLS anomaly?**

![](screenshots/networkminer_anomaly.png)
- **Answer: 36255**

### Task Questions - mx-9.pcap

**5. Which platform sent an email with the subject starting with "You have more"?**

![](screenshots/networkminer_facebook.png)
- **Answer: Facebook**

**6. What is the email address of Branson Matheson?**

![](screenshots/networkminer_branson.png)
- **Answer: branson@sandsite.org**

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

*We used the version 2.7 for the duplicate MAC address question*
- **Answer: 2.7**

**2. Which version can handle frames?**

- **Answer: 1.6**

**3. Which version can provide more details on packet details?**

- **Answer: 1.6**

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

![](screenshots/networkminer_windows.png)
*I was a bit lost on this one, I checked all the tabs for anything that would match OS host and nothing seemed to fit*
- **Answer: Windows - Windows NT 4**

**2. How many bytes were sent by the client (*.32.91) through port 1065?**

![](screenshots/networkminer_192.png)
- **Answer: 192**

**3. How many bytes were sent back by the server (*.37.122) through port 143?**

![](screenshots/networkminer_20769.png)
- **Answer: 20769**

**4. What is the sequence number of frame number 9?**

![](screenshots/networkminer_sequence.png)
- **Answer: 2AD77400**

**5. What is the number of the detected "content types"?**

![](screenshots/networkminer_contenttype.png)
*Parameters tab then rearrange parameter-name and wer can see the Content-Type*
- **Answer: 2**

### Task Questions - case2.pcap

**6. What is the USB product's brand name?**

![](screenshots/networkminer_product.png)
- **Answer: ASIX**

**7. What is the name of the phone model?**

![](screenshots/networkminer_lumia.png)
*The question is as vague and the hint Investigate the files and images. No need for external research. You manually check the images and there are a few different Lumia models and phones here. Vague question*
- **Answer: Lumia 535**

**8. What is the source IP of the fish image?**

![](screenshots/networkminer_fish.png)
- **Answer: 50.22.95.9**

**9. What is the password of the homer.pwned.se@gmx.com?**

![](sceenshots/networkminer_passw.png)
- **Answer: spring2015**

**10. What is the DNS query of frame 62001?**

![](screenshots/networkminer_pop.png)
- **Answer: popgmxcom*

---

## Task 8 - Conclusion

### Key Concepts

<!-- Do not use NetworkMiner as a primary sniffer -->
<!-- Use for quick traffic overview before moving to Wireshark or tcpdump -->
<!-- Next rooms: Wireshark, Snort, Brim -->
