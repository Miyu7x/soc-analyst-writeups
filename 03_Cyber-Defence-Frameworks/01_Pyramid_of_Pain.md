---
title: Pyramid of Pain
module: Cyber Defence Frameworks
path: SOC Level 1
platform: TryHackMe
tags:
  - soc
  - pyramid-of-pain
  - threat-intelligence
  - ioc
  - ttps
  - mitre-attck
status: in-progress
date: 2026-04-16
date_completed: 2026-04-16
---

## Task 1 - Introduction

**Pyramid of Pain**
![Pyramid of Pain](screenshots/T1_pyramidofpain.png)

### Key Concepts
<!-- What is the Pyramid of Pain and why does it matter for SOC work? -->
**IOCs** or Indicators of Compromise is a compilation of threat intelligence SOCs have collected. This threat intelligence is then categorized by severity from high at the top to low on the bottom. 
When a threat arrises an SOC can determine what they should be hunting, which is big game and not low hanging fruit such as changed changed hases and IP addresses to the **important stuff** such as high impact defections like **Tactics Terchiniques and Procedures**
<!-- Which security platforms/solutions apply this model? -->
Security Platforms that have taken up the Pyramid of Pain 
- Cisco security
- SentinelOne
- SOCRadar
<!-- Who is the target audience for this framework? -->
The Pyramid of Pain is essential knowledge for:
- SOC analysts
- Threat Hunters
- Incident Responders

### Task Questions

1. Read the above.
   - **Answer: Check**

---

## Task 2 - Hash Values (Trivial)

### Key Concepts
<!-- What is a hash value and what role does it play in threat identification? -->
**Hash Value**
- Fixed Length
- Unique to its own data
- Result of a hashing algorithm 

A **hash** is not considered cyptographically secure if two files have the same hash

SOC anlysts use hash values to:
- gain knowledge into a malware sample
- understand a malicious or suspicious file
- its a method for uniquely identifying and referencing a malicious artifact

| Algorithm | Hash Size | Status |
|-----------|-----------|--------|
| MD5 | 128-bit | Not cryptographically secure |
| SHA-1 | 160-bit | Deprecated (NIST, 2011) |
| SHA-256 | 256-bit | Current standard |
**Attackers** can modify a single bit of a file and the hash would be completely different 
- Malware has many variations and instances
- **Threat Hunting** using hashes as your **Indicator of Compromise or IOC** can become difficult
### Task Questions

1. Analyse the report associated with the hash `b8ef959a9176aef07fdca8705254a163b50b49a17217a4ff0107487f59d4a35d`. What is the filename of the sample?
   ![File name for the hash is Sales Receipt 5606](screenshots/T2_salesreceipt_hash.png)
   - **Answer: Sales_Receipt 5606.xls**

---

## Task 3 - IP Address (Easy)

### Key Concepts
**IP addresses** in the pyramid of pain are indicated with the color green
SOC analysts can block IPs which is great for base security but attackers can simply get a new IP address amking the it a **trivial security** method

**Fast Flux is a DNS techinique** used by botnets that use compromised hosts that act as proxies
- Hides phishing
- web proxying
- malware delivery
- network has multiple IP addresses associated with a domain
- constantly changing



### Task Questions

1. What is the first IP address the malicious process (PID 1632) attempts to communicate with?
![First malicious IP communication ](T3_firstmalicious_ip.png)
   - **Answer: 50.87.136.52**

2. What is the first domain name the malicious process (PID 1632) attempts to communicate with?
   - **Answer: craftingalegacy.com**

---

## Task 4 - Domain Names (Simple)

### Key Concepts
<!-- What makes domain names more painful for attackers to change vs. IP addresses? -->
**Domain Name System**
Unlike IP addresses attackers cannot simply change a **domain** name, they would have to purchase it which makes it a little more difficult for attackers.

![Jamn and punycode example](screenshots/T4_jamf_punycode.png)

**The adıdas.de** would have a Punycode of **http://xn--addas-o4a.de/**

Attackers love to hide malicious domains under shortned URL's
- bit.ky
- goo.gl
- ow.ly
- x.co

![Examples of shortned URLs](screenshots/T4_shortned_urls.png)

**Detect Malicious Domains**
- Logs
- Web server logs

**Any.run** is a sandboxing service that SOC analysts can use to review HTTP requests, or DNS requests 

**HTTP Requests Tab**
- Shows recorded HTTP requests
**Connections**
- Shows any communication made since the detonation of the sample
**DNS Requests**
- Shows DNS requests made 

### Task Questions

1. Go to the any.run report and provide the first suspicious domain request you are seeing.
![First suspicious domain request](screenshots/T4_q1_craftingalegacy.png)
   - **Answer: craftingalegacy.com**

2. What term refers to an address used to access websites?
   - **Answer: Domain Name**

3. What type of attack uses Unicode characters in the domain name to imitate a known domain?
   - **Answer: PunyCode Attack**

1. Provide the redirected website for the shortened URL using a preview: `https://tinyurl.com/bw7t8p4u`
![Redirect Tiny URL](T4_q4_redirect_tinyurl.png)
   - **Answer: https://tryhackme.com**

---

## Task 5 - Host Artifacts (Annoying)

### Key Concepts
<!-- What are host artifacts and what forms do they take? -->
**Host artifcats** are traces"crumbs" left behind by the attacker that they visited your system
- registry values
- suspicious process execution
- attack patterns
- Indicators of Compromise
-  files dropped in weird locations

If the attacker gets caught at this stage they have to get a new toolkit, its not like changing the hash or getting a new IP address


### Task Questions

1. A process named `regidle.exe` makes a POST request to an IP address based in the United States on port 8080. What is the IP address?
   ![Post request on port 8080](screenshots/T5_q2_httprequest.png)
   - **Answer: 96.126.101.6**

1. The actor drops a malicious executable (EXE). What is the name of this executable?
![Find the executable dropped by the attacker](screenshots/T5_q3_executablejugk.png)
   - **Answer: G_jugk.exe**

1. Look at the Virustotal report. How many vendors determine this host to be malicious?
![How many vendors found this URL malicious](screenshots/T5_q5_9vendors.png)
   - **Answer: 9**

---

## Task 6 - Network Artifacts (Annoying)

### Key Concepts
**Network Artifacts**
**User-Agent** can be found in the headers of Internet traffic, those contain the OS and the broswer the attacker is using
**C2 communication** or "phone home" is how the attacker communicates with the infected machine
Because malware often hits specific **URIs** its often predictable. Example: POST to something like /api/v1/checkin every 60 seconds.
<!-- How can custom User-Agent strings be used to detect adversary activity? -->
<!-- What tools are used to analyze network artifacts in PCAPs? -->

### Task Questions

1. What browser uses the User-Agent string shown in the screenshot above?
   ![Web Browser](screenshots/T6_q1_internetexplorer.png)
   - **Answer: Internet Explorer**

1. How many POST requests are in the screenshot from the pcap file?
   ![How many POST requests](screenshots/T5_q2_howmanypost.png)
   - **Answer: 6**

---

## Task 7 - Tools (Challenging)

### Key Concepts

If an attacker gets caught here with their tool, its basically game over for that tool. The attacker would have to either make a new tool or learn a new tool, which ix incredibly time consuming

**SOC Prime Threat Detection Marketplace**
As an SOC its important to have as many tools on our belt as we can,  a great resource where SOCs share their detection rules: https://tdm.socprime.com/active-threats/
 
 Resources for samples, malicious feeds and YARA results:
 - **MalwareBazaar** https://bazaar.abuse.ch/
 - **Malshare** https://malshare.com/

For **Similarity Analysis Fuzzy Hashing** which does just as it says performs simillarities analysis between files to detect minor differences based on the **fuzzy hash values**
	 

### Task Questions

1. Provide the method used to determine similarity between the files.
   - **Answer: Fuzzy Hashing**

1. Provide the alternative name for fuzzy hashes without the abbreviation.
   ![](T7_q2_fuzzyhashing.png)
   - **Answer: Context Triggered Piecewise Hashes*

---

## Task 8 - TTPs (Tough)

### Key Concepts
**TTPs Tactics Techniques and Procedures**
- **ENTIRE** MITRE ATT&CK Matrix. This means the everystyep of the entire process;
	- starting from phishing attempts to persistence and data exfiltration

**Example:** if you could detect a Pass-the-Hash(opens in new tab) attack using Windows Event Log Monitoring and remediate it, you would be able to find the compromised host very quickly and stop the lateral movement inside your network.
	The attacker now has 2 options:
			- Start from 0, with more research, training and newly configured tool
			- Give up!
### Task Questions

1. Navigate to the ATT&CK Matrix webpage. How many techniques fall under the Exfiltration category?
   ![MITRE ATT&CK Overview](screenshots/T8_q1_mittreattck.png)
   - **Answer: 9**

1. Chimera is a China-based hacking group active since 2018. What is the name of the commercial remote access tool they use for C2 beacons and data exfiltration?
   ![Chimera Group](screenshots/T8_q2_chimerac2.png)
   - **Answer: Cobalt Strike**

---

## Task 9 - Practical: The Pyramid of Pain

### Key Concepts
<!-- Apply each IOC type to its correct tier on the pyramid -->
<!-- Review the logic behind each placement before submitting -->

### Task Questions

1. Complete the static site. What is the flag?
   ![Answer for the Pyramid of Pain](T9_pyramidanswer.png)
   - **Answer: THM{PYRAMIDS_COMPLETE}**

---

## Task 10 - Conclusion

### Key Concepts

The job of stopping an attacker is not easy, but the sooner we catch them the more evidence we have for an attacker thats bad news, those traces and crumbs they leave will make sure that they cannot use them again and if they try it will be spotted right away!

Exploring https://tdm.socprime.com/active-threats/ has to be one of the coolest take aways from a THM room so far since i started this journey, its fascinating
![Exploring SOCprime](screenshots/T10_socprime_firstlook.png)



### Task Questions

1. Read the above.
   - **Answer: Check**

---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*