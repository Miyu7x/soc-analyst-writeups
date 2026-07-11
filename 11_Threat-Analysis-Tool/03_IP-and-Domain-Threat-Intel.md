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

## Task 1 — Introduction

### Learning Objectives


### Prerequisites


### Practice


**Answer the questions below**

---

**1.** All set to begin.

**Answer:** N/A

---

## Task 2 — Domain Enrichment

### Resolving DNS Domains


### A / AAAA Records


### TXT Records


### WHOIS / RDAP


### Attack Techniques Using DNS


### CDN Abuse


### Typosquatting


### IDN Attacks


### SOC Practice


**Answer the questions below**

---

**1.** Which CDN does the purematrixa[.]com domain use?

**Answer:**

---

**2.** According to the report, how old was the domain when SIEM raised an alert?

**Answer:**

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
