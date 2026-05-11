---
title: Network Security Essentials
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [network-security, firewall, perimeter, IDS, VPN, SOC, log-analysis]
status: in-progress
date: 
date_completed: 
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

Networks are the backbone of every modern organization
  - servers, workstations, applications and security devices

What a network is and keycomponenets
  - network permieter
  - permitere threats
  - examine firewall logs

![](screenshots/whatsisanetwork.png)
  
    
### Task Questions

**1. Complete the task.**

- **Answer:**

---

## Task 2 - (Room content not provided -- check THM for task title)

### Key Concepts

<!-- -->

### Task Questions

**1. Question?**

- **Answer:**

---

## Task 3 - A Network: Overview

### Key Concepts

A computer network is an organized structure that where network assets connect to talk to each other

![](screenshots/smallenterprise.png)

**Small Enterprise Network**

**Working Endpoints**
  - employees do theiur daily wortk at their pc's or laptops
  - most common entry poinbt for attachers entry point is phishing or malicious downloads
  - endpoints here tend to be less monitored if an attacker gets in here they can move laterally
    - **Importance** endpoint logs reveal malicious processes but networek logs may show C2 conncetions
   
**File and DB servers**  - stortes the business most important data(assets)
  - file servers provide centralized acces to valuable or senttive data
    - **Importance** these servers are trageted fro ransomware, data exfil usually happens here

**Application Servers**
  - web, email, VPN...
  - host comapny website and applications
    - **Impoirtance** these apps are exertnal facing applications this makes them high value targets, attackers are constantly scanning for software vulnerabilitiers
      an entry here usually means gaining foothold into the internal network
        - SQL injection on web apps
        - brute force login on email or VPN
        - suspicious IP interacting with sensitive apps
     
**Active Directory AD and Authetication Servers**
  - AD is the identity backbone of most organizations
  - responsible for managing users, groups, computers, and access rights
  - AD credentials are used to gain access to the organizations services
    - **Importance**
        - AD is the main component that controll all user accounts and systems within the network
        - attackers target the AD for privilege escalation, persistence and lateral movement
        - a single compromised domain admin account can bring down the whole organization
        - authentication logs can point to multiple failed login attempts
        - unusual logins from unknown locations at odd hours
        - accounts accessing systems they normally shouldnt
     
  **Routers & Switches** (Network Infrastructure)
    - routers connects the organizations LAN to the internet
    - siwtches connect devices within the same network (PC to printers, add more examples here)
    - these devices make the circulatory sysrtem of the enterprise
      - **Importance**
          - routers and switches are not directly exposed 
          - if compromised attackers can manipulate network traffic MIM attacks
          - create backdoors for rerouting traffic
          - open hidden channels to the internet

  **Firewalls and Perimeter Devices**
    - firewall is the primary security gateway that controls traffic between the internal trusted network and the untrusted internet
    - firewall inspects incoming and outgoing packets, then decides based on seucirty rules wether to allow it or block it
    - modern firewalls can inspect apps, prevent intrusions,and malware detection
      - **Importance**
        - protects the organization from direct exposure to the internet
        - prevents unauthorized access to internal services(databases and RDP)
        - logs every connection attempt
        - firewall logs are often the earliest indicator of attacks, shows port scans, vruteforce attempts, exploitation attmepmts

  
#### Network Components Summary

| Component | Primary Role | Security Significance |
|---|---|---|
| User Workstations (Endpoints) | Daily employee work | Most common attacker entry point (phishing, malicious downloads); C2 connections may appear in network logs before host logs |
| File & Database Servers | Store and serve critical business data | Prime ransomware and exfiltration targets |
| Application Servers (Web, Email, VPN) | Host externally facing services | High-value targets due to internet exposure; monitor for SQLi, brute-force, suspicious IPs |
| Active Directory / Auth Servers | Identity and access backbone | Controls all user accounts; targeted for privilege escalation, persistence, lateral movement |
| Routers & Switches | Traffic routing and internal connectivity | If compromised: MitM, backdoors, hidden channels |
| Firewalls / Perimeter Devices | Filter traffic between internal and untrusted networks | Logs are earliest indicators of port scans, brute-force, exploitation attempts |

<!-- Endpoint logs reveal malicious processes; network logs may show C2 first -->
<!-- AD is the crown jewel -- a single compromised domain admin can bring down the enterprise -->
<!-- Monitor auth logs: failed logins, unusual IPs, off-hours access, accounts accessing unexpected systems -->

### Task Questions

**1. Continue to the Next task.**

- **Answer:**

---

## Task 4 - Network Visibility

### Key Concepts

**Network Visibility** is crucial in cybersecurity
  - monitor and understand whats hgappening accross your network
  - as an SOC it is crucial to know that **"You cant defend what you cant see"**
  - a set tools that build a bigger picture of incidents

![](screenshots/hostcentric.png)
**Host-Centric**
  - generated by individual devices
  - what is happening inside the machine

![](screenshots/networkcentric.png)
**Network-Centric**
  - generated by network appliences
  - what is happening outside the devices
  - firewalls, IDS/IPS, routers and switches, web proxies, VPN

#### Log Source Comparison

| Category | Generated By | What It Shows | Best For |
|---|---|---|---|
| Host-Centric | Endpoints, servers, laptops | Logons, process creation, service starts, failed auth, AV/EDR alerts | Understanding direct impact on a specific machine |
| Network-Centric | Firewalls, IDS/IPS, routers, proxies, VPN | Src/dst IPs, ports, protocols, allow/block actions | Revealing recon, lateral movement, C2, data exfiltration |

<!-- Host logs = what happened inside a room -->
<!-- Network logs = who entered and left the building -->
<!-- Correlation of both = complete incident timeline -->

#### Key Network-Centric Log Sources

| Source | What It Logs | Primary Value |
|---|---|---|
| Firewalls | Every connection allowed or denied | First place to look for unauthorized connection attempts |
| IDS/IPS | Signature matches and behavioral anomalies | Detecting active attacks in real time |
| Routers/Switches | NetFlow / traffic flow summaries | High-level overview: who talked to whom, how much data |
| Web Proxies | Every URL visited by users | Web-based threats, policy violations, exfiltration |
| VPN | Who connected, from where, at what time | Monitoring remote access security |

### Task Questions

**1. Continue to the next task.**

- **Answer:**

---

## Task 5 - Network Perimeter

### Key Concepts

**Network Perimeter**
  - boundary that separates an organization internal trsuted network and external untrusted zone the internet
  - internal where the organbization business system lives
  - external the internet full of potential threats
  - perimeter is the controlled endpoint through which all traffic passes

**The Perimeter**
  - made of devices on the edge of the network
  - also includes vritual gateways, cloud connections, remote access points
  - gatekeeper: controls what goes in and out of network
  - not a single device, a conjuction of security controls and network components
  - first line of defense and where SOCs firstly detect malicious activity
  - weak or misconfigured perimeter can lead to exposed services RDP, MySQL, SMB, brute force attacks, exfiltration
      - **Importance**
          - reviewing firewall logs
          - identify scanning attempts
          - flagging unsual traffic
          - understand what should and should not be exposed at thje perimeter

#### Perimeter Components

| Component | Role |
|---|---|
| Firewalls | Filter traffic between internal and external networks |
| Routers / Gateways | Route traffic and enforce access rules |
| DMZ (Demilitarized Zone) | Buffer segment hosting public-facing servers (web, mail, VPN) |
| Remote Access Gateways / VPNs | Secure entry points for remote employees |

<!-- Weak or misconfigured perimeters allow: exposed service exploitation (RDP, MySQL, SMB), recon/scanning, brute-force, data exfiltration -->
<!-- SOC analyst role at the perimeter: review firewall logs, identify scans/brute-force, flag unusual outbound traffic -->

### Task Questions

**1. Continue to the next task.**

- **Answer:**

---

## Task 6 - Network Perimeters: Monitoring and Protecting

### Key Concepts

**Monitoring the Perimeter**
  - firewalls, IDS/IPS
  - detect misconfigs
  - logs

**Example of Port Scanning while the Firewall blocks it**
  - same extrenal IP 203.0.113.10 trying to connect to multiple ports on the same internal machine in quick succession
  - classic example of port scans 
![](screenshots/firewalllogexample.png)

**Example of a Web Server attack an SQL Injections**
  - filter by block to immediately find threats
  - SQL shows the attacker trying to dump DB info
  - XSS alert in an attempt to inject malicious script
  - Directory Taversal slert shows attempt to read sensitve server files
  - the log shows that the WAF successfully identified and blocked multiple web attacks
![](screenshots/waflogexample.png)

**VPN Brute force example of password guessing**
  - log is flooded with FAILD_AUTH and SUCCESS_AUTH
  - massive amounts of failurtes is a red flag
![](screenshots/vpnlogexample.png)

#### Attack Pattern Recognition

| Pattern | What It Looks Like | Attack Type |
|---|---|---|
| Repetition from one source to many destinations | Same IP hitting many ports quickly | Port scanning / reconnaissance |
| Repetition from one source to one destination | Same IP flooding one service with auth attempts | Brute-force |
| Traffic at perfect, regular intervals | Internal host calling out on a schedule | Malware C2 beaconing |

<!-- Context is key: IDS tells you WHY something was flagged, not just that it was blocked -->
<!-- Perimeter logs location: /Perimeter_logs/task6/ on Desktop -->

#### Scenario Summary

| Scenario | Log Source | Key Indicator | Verdict |
|---|---|---|---|
| Port Scanning | Firewall | Same external IP hitting multiple ports on one host in rapid succession | Classic port scan -- attacker mapping open services |
| Web Server Attacks | WAF | action=BLOCK entries with attack_type (SQLi, XSS, Directory Traversal) | Active web attack; filter for BLOCK to find threats |
| VPN Brute-Force | VPN Gateway | Mass FAILED_AUTH from single source IP against multiple usernames | Brute-force credential attack against VPN |

### Task Questions

**1. Examine the firewall logs. Which IP address is performing the port scan?**

![](screenshots/scanningports.png)
*The IP 203.0.113.10 is scanning multiple ports*
- **Answer: 203.0.113.10**

**2. In the WAF Logs, which single source IP is responsible for all the blocked web attacks?**

![](screenshots/waflogexample2.png)
*Searched for BLOCK connections*
- **Answer: 198.51.100.12**

**3. In the VPN logs, how many brute-force attempts failed?**

![](screenshots/failedauth.png)
*Use the CLI here to filter these txt logs, the log is long so reading a txt file is not the appropriate work flow for an SOC.
cd into Desktop/Perimeter_logs/task6 then **grep "FAILED_AUTH" vpn_logs.txt | wc -l** then pipe the command to count the words by lines
- **Answer: 90**

**4. Which suspicious IP address was found attempting the brute-force attack against the VPN gateway?**

*Its the IP on the FAILED_AUTH from the above question*
- **Answer: 45.137.22.13**

---

## Task 7 - Perimeter Logs: Investigating the Breach

### Key Concepts

 Incident scenario: Initech Corp, mid-sized financial services company
 - One month of perimeter logs to review: firewall.log, ids_alerts.log, vpn_auth.log
 - Log location: ~/Desktop/Perimeter_logs/challenge/ 
 - Two analysis methods: manual CLI or Splunk (index="network_logs")

#### Investigation Workflow Summary

| Phase | Log Source | Command / Method | What to Look For |
|---|---|---|---|
| Reconnaissance | firewall.log | grep "BLOCK" then count by source IP | External IP with most BLOCK hits = scanner |
| Exploitation Confirmation | firewall.log | grep suspicious IP + grep "ALLOW" | ALLOW entries to internal hosts = successful access |
| VPN Brute-Force | vpn_auth.log | grep FAIL, count by source IP | IP with most FAILs followed by SUCCESS = attacker |
| Lateral Movement | firewall.log | grep compromised internal IP + ALLOW | Internal IP scanning SMB/RDP/SSH across other hosts |
| IDS Correlation | ids_alerts.log | grep compromised IP, filter by SMB | ET EXPLOIT MS-SMB Lateral Movement alerts |
| C2 Beaconing | ids_alerts.log | grep C2 | Regular beacons to external IP on port 4444 |
| Data Exfiltration | firewall.log + ids_alerts.log | grep compromised IP, look for outbound HTTP POST | Large POST uploads to external IP on 80/8080 |

<!-- Splunk query: index="network_logs" -->
<!-- CLI workflow: grep -> cut -> sort -> uniq -c to count and rank -->

### Task Questions

*I tried connecting to SPLUNK via the THM VM but no success so well explore these logs thru the CLI*

**head example.log** gives us information on the headers
![](screenshots/headfirewall.png)


**Reconnaissance** we can search the logs for blocked requests in the firewall logs
**cat firewall.log | grep "BLOCK" | head**
![](screenshots/reconexample.png)

If we would like to filter by **IP** and maximum **BLOCK** requests
  - The pattern grep --> cut --> sort --> uniq -c --> head is a template 
**cat firewall.log | grep "BLOCK" | cut -d' ' -f5 | cut -d: -f1 | sort -nr | uniq -c**

*I researched the flags here to see exactly what each meant*
![](screenshots/flagsexaplained.png)

**Allowed Requests** has the firewall allowed any requests by the suspicious(REDACTED) IP
**cat firewall.log | grep [REDACTED] | grep "ALLOW"**
![](screenshots/allowedredacted.png)

**VPN and Brute Force in logs** failed arrempts are a red flag
**cat vpn_auth.log | grep FAIL | cut -d' ' -f3 | sort -nr | uniq -c**
![](screenshots/vpnlogs2.png)

Narrow down by the suspisicous IP
**cat example.log | grep IPnumber**

**Lateral Movement** filter to discover lateral movement
**cat example.log | grep IPnumber | grep "ALLOW" | head**

**C2 Beacvoning** after detecting lateral movement we should check for C2 communication
**cat example.log | grep C2 | head**

**Find Suspicious IP resonsible for C2 beaconing**
**cat example.log | grep -n IPnumber | cut -d' ' -f6,7,8,9,10,19,22,23 | head -n 15**
![](screenshots/confirmedc2example.png)
This shows the infected host performing more suspicious activities
*Flags broken down for the above example*
![](screenshots/flagsexample2.png)

**Internal Network is fully compromised** the malicious IP is acting as a C2 server receiving compromised information from the victim host
**cat example.log | grep -n IPnumber | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head**
![](screenshots/fullycompromisedexample.png)

**Data Exfiltration Example**
**cat example.log | grep -n IPnumber | cut -d' ' -f5,6,7  |   uniq  | sort**
![](screenshots/exfilexamplelog.png)

**1. Examine the firewall logs. What external IP performed the most reconnaissance?**

- **Answer:**

**2. In the firewall log, which internal host was targeted by scans?**

- **Answer:**

**3. Which username was targeted in VPN logs?**

- **Answer:**

**4. What internal IP was assigned after successful VPN login?**

- **Answer:**

**5. Which port was used for lateral SMB attempts?**

- **Answer:**

**6. In the IDS logs, which host beaconed to the C2?**

- **Answer:**

**7. During the investigation, which IP was observed to be associated with C2?**

- **Answer:**

**8. Which host showed the exfiltration attempts?**

- **Answer:**

---

## Task 8 - Conclusion

### Key Concepts

<!-- Enterprise network = ecosystem: firewalls, AD, app servers, file/DB servers, endpoints, WAPs -->
<!-- Perimeter = boundary between trusted and untrusted networks -->
<!-- Attackers probe perimeter constantly: port scans, brute-force, exploits against exposed services -->
<!-- SOC analyst role: recognize normal vs suspicious traffic, escalate, understand component roles -->

### Task Questions

**1. Complete the room.**

- **Answer:**
