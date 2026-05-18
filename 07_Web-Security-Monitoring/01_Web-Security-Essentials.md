---
title: Web Security Essentials
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [web-security, waf, cdn, antivirus, web-server, blue-team, soc-level-1]
status: in-progressTHM 
date: 
date_completed: 
---

<img src=screenshots/web_intro.png width="1500">

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

### Key Concepts

**Web Security Essentials**
<!-- Q: What are the three components of any web service that this room will focus on protecting? -->
- web apps are common tragets for attackers
- web apps are always online and exposed
- web apps are often connected to back-end systems like databases which will hold critical data
<!-- Q: Why are web applications considered high-value targets from an attacker's perspective? -->

<!-- Q: What two prerequisite rooms does this room build on, and what does each one cover? -->

### Task Questions

**1. I understand the learning objectives and am ready to learn about web security!**

**Answer:** N/A

---

## Task 2 - Why Web?

<p align="center">
<img src=screenshots/web_saas.png width="400">
</p>

### Key Concepts

**Shift from Desktop to the Web**
<!-- Q: What drove the shift from desktop to web applications over the past few decades? Walk through the timeline: 1990s, 2000s, 2010s, today. -->
- 1990s desktop applications featured limited speed and connectivity
- 2000s more web apps avliable, email, social media and, banking
- 2010s sees a massive rise in cloud computing and software as a service or **SaaS**
<!-- Q: What security advantages do web apps lose compared to traditional desktop apps? -->
As the number of web apps skyrockets each one of those is an oppurtinity for an attacker to exlpoit
- vulnerable web apps are usally the first stage in a larger attack seuqence
- **Web App Owners** are responsibility for the security of its users data

| As a Web App Owner | As a Web App User |
|---|---|
| App is always online and must be secured 24/7 | Data is stored in the app, potentially insecurely |
| Anyone worldwide can access the app at any time | Once browser is breached, all accounts are at risk |
| Hard to keep up with constantly emerging threats | A breach can result in identity theft or financial loss |
| Responsible for securing users' data | Privacy can be permanently compromised |

<!-- Q: What happened in the Equifax 2017 breach? What vulnerability was exploited and what was the impact? -->
**Web Servers can also be Vunerable**
  - 2017s Equifax breach was due to an Apache vulnerability that was not patched
  - attackers used the web server vulnerability to gain access to the internal database that stored constumers information
<!-- Q: What happened in the Capital One 2019 breach? What misconfiguration caused it and what was exposed? -->
**Web Applications such as WAF**
  - if misconfigured will leave an intire network vulnerable
  - this allowed attackers to breach the systems and gain access to the companys infrastructure and databases
<!-- Q: What do both breaches have in common in terms of attack path? -->

### Task Questions

**1. Have applications shifted from desktop to web over the past couple of decades (Yea/Nay)?**

**Answer: Yea**

---

**2. Who is ultimately responsible for ensuring the security of users' data within a web application?**

**Answer: Web App Owner**

---

## Task 3 - Web Infrastructure

### Key Concepts

<!-- Q: Describe the request-response cycle in your own words. What happens from the moment you type a URL to when you see a webpage? -->
**What happens when you try to connect to a website?**
  - your browser sends a request to a web server
  - server processes the request, verfiies access
  - returns the response to the user
<!-- Q: What are the three main components of any web service? What does each one do? -->

| Component | Role |
|---|---|
| Application | The code, images, styles, and icons - how the site looks and functions |
| Web Server | Hosts the application; listens for requests and returns responses |
| Host Machine | The underlying OS (Linux/Windows) that runs the web server and application |

<!-- Q: Why are web servers a common attacker target? What makes them different from internal systems? -->

<!-- Q: What are the three most common web servers? What is each one known for or associated with? -->

| Web Server | Known For / Used By |
|---|---|
| Apache | Most popular; commonly hosts WordPress sites |
| Nginx | High-performance apps; used by Netflix, Airbnb, GitHub |
| IIS | Microsoft-developed; common in enterprise environments |

### Task Questions

**1. What does your web browser send to a server to receive a web page?**

**Answer:**

---

**2. What web server is most commonly used to host WordPress websites?**

**Answer:**

---

**3. What do we call the OS and environment that runs the web server and application?**

**Answer:**

---

## Task 4 - Protecting the Web

### Key Concepts

<!-- Q: What is the difference between visibility and mitigation in the context of security measures? -->

<!-- Q: Map each protection to its layer. Which controls protect the application? The web server? The host machine? -->

| Layer | Security Control | What It Does |
|---|---|---|
| Application | Secure Coding | Avoids insecure functions, handles errors properly, removes sensitive info from code |
| Application | Input Validation & Sanitization | Validates and sanitizes user input to prevent injection attacks |
| Application | Access Control | Restricts access based on user roles |
| Web Server | Logging | Keeps a detailed record of all web requests via access logs |
| Web Server | WAF | Filters and blocks harmful traffic based on defined rules |
| Web Server | CDN | Reduces direct server exposure and provides integrated WAF and DDoS protection |
| Host Machine | Least Privilege | Uses low-privilege users for services to limit blast radius |
| Host Machine | System Hardening | Disables unnecessary services and closes unused ports |
| Host Machine | Antivirus | Endpoint-level protection that blocks known malware |

<!-- Q: What fields are captured in a web server access log? Why does each field matter to a SOC analyst? -->

<!-- Q: Walk through the four-step log example in the room. What HTTP methods are used and what does each one mean? -->

<!-- Q: How can access logs help an analyst reconstruct an attack sequence? What would a malicious version of that four-step log look like? -->

### Task Questions

**1. What cyber security concept involves stopping or limiting damage from threats?**

**Answer:**

---

**2. What security control involves ensuring all software and components are up to date?**

**Answer:**

---

## Task 5 - Defense Systems

### Key Concepts

<!-- Q: What is a CDN and how does it work at a high level? What is the role of the origin server vs. edge servers? -->

<!-- Q: List the four security benefits of a CDN. For each one, explain the attacker technique it defends against. -->

| CDN Security Benefit | Attacker Technique It Counters |
|---|---|
| IP Masking | Direct targeting of the origin server |
| DDoS Protection | Volumetric denial-of-service attacks |
| Enforced HTTPS | Cleartext interception / man-in-the-middle |
| Integrated WAF | Web application attacks (SQLi, XSS, etc.) |

<!-- Q: What is a WAF and how does it decide what to block? Use the bouncer analogy to explain it in your own words. -->

<!-- Q: What are the three types of WAF deployments? Where does each one sit in the network? When would you use each? -->

| WAF Type | Where It Sits | Best For |
|---|---|---|
| Cloud-based (Reverse Proxy) | In front of the web server | Easy deployment, great scalability |
| Host-based | Directly on the web server | Per-application control and customization |
| Network-based | Physical/virtual appliance at network perimeter | Enterprise environments |

<!-- Q: Walk through the four WAF detection methods. For each one, describe how it works and give the example from the room. -->

| WAF Feature | Detection Method | Example from Room |
|---|---|---|
| Signature-Based Detection | Matches known attack patterns or payloads | User-Agent matching a known tool: `sqlmap/1.8.1` |
| Heuristic-Based Detection | Analyzes context and behavior of requests | Long query string with special characters: `search?q=%3Cscript%20(1)` |
| Anomaly & Behavioral Analysis | Flags deviations from normal traffic behavior | Single IP making repeated login attempts in a short period |
| Location & IP Reputation Filtering | Uses location and threat intel to block IPs | Request from an IP outside normal business area |

<!-- Q: What is AV primarily designed to protect? Why is it not a blanket solution for web attacks? -->

<!-- Q: What is the difference between what a WAF blocks and what an AV catches? Where do they overlap? -->

### Task Questions

**1. Which type of Web Application Firewall operates by running on the same system as the application itself?**

**Answer:**

---

**2. Which common WAF detection technique works by matching incoming requests against known malicious patterns?**

**Answer:**

---

## Task 6 - Practice Scenario

### Key Concepts

<!-- Q: What changes did you make to secure the Web Application layer? What vulnerability or misconfiguration did each fix address? -->

<!-- Q: What changes did you make to secure the Web Server layer? -->

<!-- Q: What changes did you make to secure the Host Machine layer? -->

<!-- Q: Looking at all three layers together, which fix felt most impactful from a SOC analyst perspective and why? -->

### Task Questions

**1. What flag did you receive for securing the Web Application?**

**Answer:**

---

**2. What flag did you receive for securing the Web Server?**

**Answer:**

---

**3. What flag did you receive for securing the Host Machine?**

**Answer:**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Q: Without looking back, summarize the three-layer web security model covered in this room. What protects each layer? -->

<!-- Q: How does this room connect to your broader SOC analyst role? What would you actually use from this room on the job? -->

### Task Questions

**1. Complete the room and continue on your cyber learning journey!**

**Answer:** N/A
