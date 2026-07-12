---
title: "IP and Domain Threat Intel"
module: "Threat Analysis Tools"
path: "SOC Level 1"
platform: "TryHackMe"
tags:
  - soc
  - blue-team
  - tryhackme
  - threat-intel
  - dns
  - whois
  - ip-enrichment
  - vpn-detection
status: "in-progress"
date: ""
date_completed: ""
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/threat3_intro.png width="1500">
</p>

---

## Task 1 — Introduction

### Learning Objectives
 
- Enrich domains with WHOIS age, DNS records, and TLS details
- Learn the concepts of ASNs and geolocation for SOC triage
- Spot red-flag services using VirusTotal, Shodan, and Censys
- Detect VPN, proxy, and Tor exit nodes with IP2Proxy and Spur
- Correlate signals across sources instead of trusting one verdict

---

## Task 2 — Domain Enrichment

### Resolving DNS Domains

Domain Name Sysytem Protocol - Converts www.x.com to 162.159.140.229. The user types in a human readable address the DNS converts it into an address the machine can read it

### A / AAAA Records

https://www.nslookup.io/
https://dnschecker.org

These websites keep recors of the IP address and where they are hosted. SOCs can check these records to see if the IP could be tied to any **Advanced Persistence Threat** groups.

### TXT Records

TXT Records details the IP addresses: security settings such as SPF and DKIM and its used tools. A legitemate website dfiscloses its security tools.
  - An attack on infrastructure will have an empty TXT records
  - Suspicious Sender Policy Framework records and faked Domain Keys Identified Mail which are often abused by fishing campaigns 

### WHOIS / RDAP

WHOIS is another domain enrichament tool: registration information, expiry dates and the domains age date.
RDAP is going to take over WHOIS in the near future. As modern attacker domains does not stay online for more than a few months, which makes the WHOIS tracker obselete since the malicious domain will appear and be gone within a few days.

### Attack Techniques Using DNS

**Content Delivery Network**
Globally distributed network of proxy servers sits on the edge of the network, so the connection distance is shortened.
  - minimizes the physical distance data must travel to reach its user

**CDN Abuse** attackers will route their malicious traffic through a legitimate CDNs like cloudflare, Akamai of Fastly to hide their origin server



### Typosquatting


### IDN Attacks


### SOC Practice

On June 1, 2026, the SIEM raises a critical alert pointing to the domain purematrixa[.]com.
As an L1 analyst, enrich this indicator by using tools such as nslookup.io(opens in new tab) or whois.domaintools.com(opens in new tab).

IMPORTANT NOTE: Domain intel changes rapidly, so online tools may give you incorrect answers over time. 

To answer the questions, please use the attached PDF reports
However, you are encouraged to try out online tools as well

---

**1.** Which CDN does the purematrixa[.]com domain use?

<p align="center">
<img src=screenshots/threat3_cloudflare.png width="700">
</p>

**Answer: Cloudflare**

---

**2.** According to the report, how old was the domain when SIEM raised an alert?

<p align="center">
<img src=screenshots/threat3_date.png width="700">
</p>

**Answer: 1 days old**

---

## Task 3 — IP Enrichment

### IP Enrichment Within the SOC

When a SIEM or EDR alert pops up, it reveals at least one IP address, more often than not that IP will not seem malicious since most attackers will hide their real IP address.
  - The IP could be from a CDN edge, users  with cloud services...
    - Without IP enrichment as an SOC, you might risk blocking legitimate infrastructure and underreacting by ignoring real malicious servers.
    - SOCs specifically L1s can do some investigation about the IP services
        - https://www.abuseipdb.com/ To see if the IP was involved in any **port scans** or **brute-force** attacks
        - https://www.virustotal.com/ Revelas IP reputation

**Important:** If an IP is not within a **CDN** range, even one Alert with that IP is considered dangerous and worth looking into!

### Autonomous Systems

<p align="center">
<img src=screenshots/threat3_map.png width="700">
</p>

**AS** Autonomous System is a group of IP prefixes that are controlled by a single organization
  - **Residential ASN** Alerts here point to a VPN or compromised device
      - **AS124888** Vodafone
  - **Server Hosting** Risky as it is most widely used by attackers to distrubute malware
      - **AS215439** PLAY2GO
  - **Cloud/CDN ASN** Used by everyday users and attackers, requires further investigation
      - **AS16509** Amazon AWS

        
### Geolocation (GeoIP)

Some websites offer geo location, while city and state are diffcult to track, the country should be avliable for reserach.
  - As an SOC, you may observe an employee logging in from a different IP. Is that employee working remotely? Is there a reason the IP is pinging in a different country?
  - Sometimes SOCs monitor a company in France so a random login from the Phillipines would be unsual.

https://ipinfo.io/
https://iplocation.net/

### SOC Practice

Another alert appears, now pointing to 2[.]58[.]56[.]50, a potential C2 server address.
Use the learned tools and services (e.g., VirusTotal and BPG.Tools) to answer the questions.

IMPORTANT NOTE: Same as with domains, IP ownership changed rapidly.

To answer the questions, please use the attached PDF reports
However, you are encouraged to try out online tools as well

---

**1.** What country does the malicious IP resolve to?

<p align="center">
<img src=screenshots/threat3_netherlands.png width="700">
</p>

**Answer: Netherlands**

---

**2.** Looking at VirusTotal comments, what C2 server is hosted behind the IP?

<p align="center">
<img src=screenshots/threat3_c2server.png width="700">
</p>

**Answer: Remcos**

---

**3.** What Autonomous System does the IP belong to? (Full name)

<p align="center">
<img src=screenshots/threat3_c2server.png width="700">
</p>
BPG.tools provides us with the complete AS names fo r IP addresses.

**Answer: 1337 Services GmbH**

---

**4.** What two tags does BPG.Tools attribute to the ASN? (Tag1, Tag2)

<p align="center">
<img src=screenshots/threat3_asntags.png width="700">
</p>
 Check under the tab Originated by: AS210558

**Answer: Service Hosting, Tor Services**

---

## Task 4 — Service Exposure

### Exposed Services

Exposed services on IP addresses could mean, an attacker got in by an exposide service, or the exposed serivice is being used as jump point.

### Shodan Reconnaissance

Shodan is powerful **recon** tool for IP analysis. 
  - Indexes internet connected devices and services
  - Provides detailed information on ports, running services amd system configurations
  - Service banners provides information service version, an old verision could mean a vulnerable version

https://www.shodan.io/

### Censys Search

Blue Teams can find useful information on Censys, as it provides exposed services on **non-standard** ports

https://search.censys.io/

### TLS Certificates

HTTPS the S here means the traffic is secured, but who ssigned these secure cetificates?
**TLS** Trabsport Layer Security, if an IP has a self signed certificate that might be worth inverstigating, as well as new certifcates.
  - Self-certificates could point to a prgogram behind HTTPS like Pfsense

https://crt.sh/ Shows Info on YLS certificates, website wont even load most of the time due to access volume
https://www.sslshopper.com/ssl-checker.html


### SOC Practice

This time, you are asked to help with a deeper investigation of 64[.]89[.]160[.]44.
The IP is clearly malicious, but the CTI team wants to get more context on the IP's role.

IMPORTANT NOTE: Shodan and Censys save information for just a few weeks.

To answer the questions, please use the attached screenshots available as downloadable files in Task 1
However, you are encouraged to try out online tools as well


---

**1.** Refer to the attached downloadable files in Task 1. What remote access service is exposed?

<p align="center">
<img src=screenshots/threat3_rdp.png width="700">
</p>
We observe several exposed services:
Services (8)
135 / DCERPC
139 / NETBIOS
445 / SMB
1000 / UNKNOWN
3389 / RDP
27015 / VALVE
7777 / UNKNOWN
27383 / VALVE

**Answer: RDP**

---

**2.** How many ports have been identified as open on the server?

<p align="center">
<img src=screenshots/threat3_ports.png width="700">
</p>

**Answer: 9**

---

**3.** One of the exposed services leaks an active C2 server! What is the name of that C2? (E.g., Cobalt Strike)

<p align="center">
<img src=screenshots/threat3_async.png width="700">
</p>

**Answer: AsyncRAT**

---

**4.** For how many days is the C2 server's certificate valid?

<p align="center">
<img src=screenshots/threat3_expiration.png width="700">
</p>

**Answer: 3935**

---

## Task 5 — VPN Detection

### VPN Detection

A Microsoft Alert outside working hours might not look too suspicious as attackers made sure to change their location, and identification...
  - An SOC should varify this IP is real and not hiding behind a VPN service

### IP2Proxy and Spur

These tools are critical for detecting VPN, proxy and Tor exit nodes

https://spur.us/
https://www.ip2proxy.com/

### SOC Analyst Workflow

**Investigate the domain**
  - legit or typosquatting?
  - is it a known malicious domain?
  - when was it registered, if its new its suspicious

**Investigate the IP**
  - Is it a CDN range? If yes then investigate further
  - Reputation information of the IP from our investigation tools?
  - Location, is it VPN does it match the alert
  - What is the Autonomous Server it belongs to? Noting that  employees should not be logging in from cloud services

    
---

**1.** A London-based user logs in from a London IP address. You can confirm the IP belongs to a Mullvad VPN provider. Would you raise the alarm and prioritize the alert (Yea/Nay)?

**Answer: Yea**

---

**2.** Same scenario, but the IP doesn't match any VPN providers. Would you close the alert as a False Positive then? (Yea/Nay)

**Answer: Nay**

---

## Task 6 — Challenge

### Challenge

An APT group has just struck your company, and the Incident Response team is working through it. As an L1 analyst, you've volunteered to help and have been tasked with gathering everything you can on a malicious domain and identifying the infrastructure the malware relies on. The domain is:

raytracingengine[.]com
NOTE: Start with the online tools covered earlier. If they return nothing useful, which is likely, since attacker infrastructure is short-lived and often already taken down within days, fall back to the attached incident report for exported enrichment data.

---

**1.** What IP does the domain resolve to?

<p align="center">
<img src=screenshots/threat3_racing.png width="700">
</p>

**Answer: 35.188.105.97**

---

**2.** What cloud provider did the attacker use? (E.g., Amazon Cloud)

<p align="center">
<img src=screenshots/threat3_googlecloud.png width="700">
</p>
Use AbuseIPDB to search for information on the IP 35.188.105.97 and well find the AS proivider.

**Answer: Google Cloud**

---

**3.** What country is the malicious server located in? (E.g., France)

<p align="center">
<img src=screenshots/threat3_unitedstates.png width="700">
</p>

**Answer: United States**

---

**4.** When was the malicious domain name created? (E.g., 24.05.2026)

<p align="center">
<img src=screenshots/threat3_created.png width="700">
</p>

**Answer: 21.02.2026**

---

**5.** According to the exposed service, what is the attack server's OS?

<p align="center">
<img src=screenshots/threat3_linux.png width="700">
</p>

**Answer: Linux**

---

## Task 7 — Conclusion

**Answer the questions below**

---

**1.** Complete the room!

**Answer:** N/A

---
