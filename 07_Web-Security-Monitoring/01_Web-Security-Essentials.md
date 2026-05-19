---
title: Web Security Essentials
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [web-security, waf, cdn, antivirus, web-server, blue-team, soc-level-1]
status: completed 
date: 05-19-2026
date_completed: 05-19-2026
---

<img src=screenshots/web_intro.png width="1500">

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

### Key Concepts

**Web Security Essentials**
<!-- Q: What are the three components of any web service that this room will focus on protecting? -->
- Web apps are common targets for attackers
- Web apps are always online and exposed
- Web apps are often connected to back-end systems like databases which hold critical data
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
- 1990s: desktop applications featured limited speed and connectivity
- 2000s: more web apps available -- email, social media, and banking
- 2010s: massive rise in cloud computing and Software as a Service (SaaS)
<!-- Q: What security advantages do web apps lose compared to traditional desktop apps? -->
As the number of web apps skyrockets, each one is an opportunity for an attacker to exploit.
- Vulnerable web apps are usually the first stage in a larger attack sequence
- **Web App Owners** are responsible for the security of their users' data

| As a Web App Owner | As a Web App User |
|---|---|
| App is always online and must be secured 24/7 | Data is stored in the app, potentially insecurely |
| Anyone worldwide can access the app at any time | Once browser is breached, all accounts are at risk |
| Hard to keep up with constantly emerging threats | A breach can result in identity theft or financial loss |
| Responsible for securing users' data | Privacy can be permanently compromised |

<!-- Q: What happened in the Equifax 2017 breach? What vulnerability was exploited and what was the impact? -->
**Web Servers Can Also Be Vulnerable**
  - The 2017 Equifax breach was due to an Apache vulnerability that was not patched
  - Attackers used the web server vulnerability to gain access to the internal database that stored customers' information
<!-- Q: What happened in the Capital One 2019 breach? What misconfiguration caused it and what was exposed? -->
**Web Applications Such as WAF**
  - If misconfigured, will leave an entire network vulnerable
  - This allowed attackers to breach the systems and gain access to the company's infrastructure and databases
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
  - Your browser sends a request to a web server
  - The server processes the request and verifies access
  - Returns the response to the user -- the response can be a webpage or an image

**Attackers Can Abuse the Request-Response Cycle**
  - Overwhelming servers with requests
  - Bypassing access controls
  - Tricking the server into executing harmful commands
<!-- Q: What are the three main components of any web service? What does each one do? -->
**3 Components of Web Service**
  - Application: code, images, and styles that dictate how the website works
  - Web Server: hosts the application, listens for requests and returns a response to the user
  - Host Machine: the underlying operating system (Linux or Windows) that runs the web server and the application

<p align="center">
<img src=screenshots/web_components.png width="400">
</p>

| Component | Role |
|---|---|
| Application | The code, images, styles, and icons - how the site looks and functions |
| Web Server | Hosts the application; listens for requests and returns responses |
| Host Machine | The underlying OS (Linux/Windows) that runs the web server and application |

<!-- Q: Why are web servers a common attacker target? What makes them different from internal systems? -->
**Web Servers**
<p align="center">
<img src=screenshots/web_servers.png width="400">
</p>
  - Returns responses to the user upon receiving a request
  - Publicly exposed -- the front of websites and apps
  - On at all times, which makes them an attractive attack vector for hackers
<!-- Q: What are the three most common web servers? What is each one known for or associated with? -->
**Common Web Servers**
  - Apache: hosts simple websites and blogs, most commonly WordPress
  - Nginx: hosts high-performance web apps such as Netflix, GitHub, and Airbnb
  - Internet Information Services (IIS): Microsoft-developed, hosts enterprise environments

| Web Server | Known For / Used By |
|---|---|
| Apache | Most popular; commonly hosts WordPress sites |
| Nginx | High-performance apps; used by Netflix, Airbnb, GitHub |
| IIS | Microsoft-developed; common in enterprise environments |

### Task Questions

**1. What does your web browser send to a server to receive a web page?**

**Answer: Request**

---

**2. What web server is most commonly used to host WordPress websites?**

**Answer: Apache**

---

**3. What do we call the OS and environment that runs the web server and application?**

**Answer: Host Machine**

---

## Task 4 - Protecting the Web

### Key Concepts

<!-- Q: What is the difference between visibility and mitigation in the context of security measures? -->
**How Do We Protect Web Servers?**
  - Some tools provide visibility: what's happening on the network
  - Other tools provide mitigation: actively stop or limit an attack
<!-- Q: Map each protection to its layer. Which controls protect the application? The web server? The host machine? -->
**Protecting the Application**
  - Secure coding: avoid insecure functions, ensure proper error handling, remove sensitive information
  - Input validation and sanitization: validate and sanitize user input to prevent injection attacks
  - Access control: restrict access based on user roles

**Protecting the Web Server**
  - Logging: keep a detailed record of all web requests with access logs
  - Web Application Firewall (WAF): filter and block harmful traffic based on defined rules
  - Content Delivery Network (CDN): reduce direct exposure to your server and use integrated WAFs

**Protecting the Host Machine**
  - Least privilege: use low-privilege users for services
  - System hardening: disable unnecessary services and close unused ports
  - Antivirus: add endpoint-level protection that blocks known malware

**Strong Authentication and Patch Management** are the best security tips for protection.

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
**Logging and Security**
  - Provides detailed information on every interaction with the server
  - Fields include: client IP address, timestamp, requested page or data, response status from the server, and user agent
<!-- Q: Walk through the four-step log example in the room. What HTTP methods are used and what does each one mean? -->
**Logging Methods for HTTP**
  - GET requests: the user wants to open tryhackme.com
  - POST requests: the user submitted their login info to the server

**GET/POST Log Example**
<p align="center">
  <img src=screenshots/web_getpostlog.png width="400">
</p>
<!-- Q: How can access logs help an analyst reconstruct an attack sequence? What would a malicious version of that four-step log look like? -->

### Task Questions

**1. What cyber security concept involves stopping or limiting damage from threats?**

**Answer: Mitigation**

---

**2. What security control involves ensuring all software and components are up to date?**

**Answer: Patch Management**

---

## Task 5 - Defense Systems

### Key Concepts

<!-- Q: What is a CDN and how does it work at a high level? What is the role of the origin server vs. edge servers? -->
**Content Delivery Network**
  - Stores and serves content from servers closer to the user requesting it to reduce response time
  - Think: one centralized server with multiple servers deployed on the edges to supply different parts of the world
  - Aside from speed, CDNs also help raise security:
      - IP masking: hides the origin server's IP address
      - Distributed Denial of Service (DDoS) protection: CDNs absorb high volumes of traffic, making DDoS attacks less effective
      - Enforced HTTPS: encrypted communication via Transport Layer Security (TLS) by default on most CDNs
      - Integrated WAF: many CDNs include built-in web application firewalls

  <p align="center">
  <img src=screenshots/web_cdnlocations.png width="400">
  </p>

| CDN Security Benefit | Attacker Technique It Counters |
|---|---|
| IP Masking | Direct targeting of the origin server |
| DDoS Protection | Volumetric denial-of-service attacks |
| Enforced HTTPS | Cleartext interception / man-in-the-middle |
| Integrated WAF | Web application attacks (SQLi, XSS, etc.) |

<!-- Q: What is a WAF and how does it decide what to block? Use the bouncer analogy to explain it in your own words. -->
**Web Application Firewall (WAF)**
  - Inspects incoming HTTP traffic and allows or blocks it depending on defined rules
  - Think of it as a bouncer at a club -- checks your ID and either lets you in or kicks you out
  - Cloud-based: sits in front of the web server
  - Host-based: software deployed inside the web server, offers per-application control
  - Network-based: physical or virtual appliance on the perimeter of the network (best suited for enterprise environments)
  - Inspects HTTP requests to detect:
      - Anomalies
      - Known attacks
      - Suspicious patterns

| WAF Type | Where It Sits | Best For |
|---|---|---|
| Cloud-based (Reverse Proxy) | In front of the web server | Easy deployment, great scalability |
| Host-based | Directly on the web server | Per-application control and customization |
| Network-based | Physical/virtual appliance at network perimeter | Enterprise environments |

| WAF Feature | Detection Method | Example from Room |
|---|---|---|
| Signature-Based Detection | Matches known attack patterns or payloads | User-Agent matching a known tool: `sqlmap/1.8.1` |
| Heuristic-Based Detection | Analyzes context and behavior of requests | Long query string with special characters: `search?q=%3Cscript%20(1)` |
| Anomaly & Behavioral Analysis | Flags deviations from normal traffic behavior | Single IP making repeated login attempts in a short period |
| Location & IP Reputation Filtering | Uses location and threat intel to block IPs | Request from an IP outside normal business area |

<!-- Q: What is AV primarily designed to protect? Why is it not a blanket solution for web attacks? -->
**Antivirus**
  - Safeguards endpoints (desktops, laptops, servers) from malicious files and programs
  - Signature-based detection requires knowledge of attack patterns or known malware in order to prevent them
  - Works best in combination with other security measures
<!-- Q: What is the difference between what a WAF blocks and what an AV catches? Where do they overlap? -->

### Task Questions

**1. Which type of Web Application Firewall operates by running on the same system as the application itself?**

**Answer: Host-Based**

---

**2. Which common WAF detection technique works by matching incoming requests against known malicious patterns?**

**Answer: Signature-Based**

---

## Task 6 - Practice Scenario

### Key Concepts

**Secure-A-Site**

### Task Questions

**1. What flag did you receive for securing the Web Application?**
<p align="center">
<img src=screenshots/web_webapp.png width="200">
</p>
<p align="center">
<img src=screenshots/web_webapp2.png width="200">
</p>
<p align="center">
<img src=screenshots/web_webapp3.png width="200">
</p>

**Answer: THM{web_app_secured!}**

---

**2. What flag did you receive for securing the Web Server?**
<p align="center">
<img src=screenshots/web_webserver.png width="200">
</p>
<p align="center">
<img src=screenshots/web_webserver2.png width="200">
</p>
<p align="center">
<img src=screenshots/web_webserver3.png width="200">
</p>

**Answer: THM{server_security_expert!}**

---

**3. What flag did you receive for securing the Host Machine?**
<p align="center">
<img src=screenshots/web_host.png width="200">
</p>
<p align="center">
<img src=screenshots/web_host2.png width="200">
</p>
<p align="center">
<img src=screenshots/web_host3.png width="200">
</p>

**Answer: THM{the_final_security_layer!}**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Q: Without looking back, summarize the three-layer web security model covered in this room. What protects each layer? -->

<!-- Q: How does this room connect to your broader SOC analyst role? What would you actually use from this room on the job? -->

### Task Questions

**1. Complete the room and continue on your cyber learning journey!**

**Answer:** N/A
