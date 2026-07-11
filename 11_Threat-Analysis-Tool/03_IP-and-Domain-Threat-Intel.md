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
<img src=screenshots/shadow_intro.png width="1500">
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
Globally distributed network of proxy servers, sits on the edge of the network so the connection distance is shortned.
  - minimizes physical distance data must travel to reach its user

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
<img src=screenshots/shadow_cloudflare.png width="700">
</p>

**Answer: Cloudflare**

---

**2.** According to the report, how old was the domain when SIEM raised an alert?

<p align="center">
<img src=screenshots/shadow_date.png width="700">
</p>

**Answer: 1 days old**

---

## Task 3 — IP Enrichment

### IP Enrichment Within the SOC


### Autonomous Systems


### Geolocation (GeoIP)


### SOC Practice


**Answer the questions below**

---

**1.** What country does the malicious IP resolve to?

**Answer:**

---

**2.** Looking at VirusTotal comments, what C2 server is hosted behind the IP?

**Answer:**

---

**3.** What Autonomous System does the IP belong to? (Full name)

**Answer:**

---

**4.** What two tags does BPG.Tools attribute to the ASN? (Tag1, Tag2)

**Answer:**

---

## Task 4 — Service Exposure

### Exposed Services


### Shodan Reconnaissance


### Censys Search


### TLS Certificates


### SOC Practice


**Answer the questions below**

---

**1.** Refer to the attached downloadable files in Task 1. What remote access service is exposed?

**Answer:**

---

**2.** How many ports have been identified as open on the server?

**Answer:**

---

**3.** One of the exposed services leaks an active C2 server! What is the name of that C2? (E.g., Cobalt Strike)

**Answer:**

---

**4.** For how many days is the C2 server's certificate valid?

**Answer:**

---

## Task 5 — VPN Detection

### VPN Detection


### IP2Proxy and Spur


### SOC Analyst Workflow


**Answer the questions below**

---

**1.** A London-based user logs in from a London IP address. You can confirm the IP belongs to a Mullvad VPN provider. Would you raise the alarm and prioritize the alert (Yea/Nay)?

**Answer:**

---

**2.** Same scenario, but the IP doesn't match any VPN providers. Would you close the alert as a False Positive then? (Yea/Nay)

**Answer:**

---

## Task 6 — Challenge

### Challenge


**Answer the questions below**

---

**1.** What IP does the domain resolve to?

**Answer:**

---

**2.** What cloud provider did the attacker use? (E.g., Amazon Cloud)

**Answer:**

---

**3.** What country is the malicious server located in? (E.g., France)

**Answer:**

---

**4.** When was the malicious domain name created? (E.g., 24.05.2026)

**Answer:**

---

**5.** According to the exposed service, what is the attack server's OS?

**Answer:**

---

## Task 7 — Conclusion

**Answer the questions below**

---

**1.** Complete the room!

**Answer:** N/A

---
