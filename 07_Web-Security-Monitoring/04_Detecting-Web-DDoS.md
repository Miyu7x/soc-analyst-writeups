---
title: Detecting Web DDoS
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [ddos, dos, web-security, log-analysis, splunk, waf, cdn, soc-level-1]
status: completed
date: 2026-05-21
date_completed:
---

<p align="center">
  <img src=screenshots/web_ddosintro.png width="1500">
</p>

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

**DDoS** attacks happen in many forms:
  - main goal is to block kweb service or block access to a website
  - DDoS attacks can be launched in different layers of the OSI model, but this lab focuses on the **application layer**

**1. I understand the learning objectives and am ready to embark on a Denial-of-Service adventure!**

Answer: N/A

---

## Task 2 - DoS and DDoS Attacks

**DoS** Denial-of-Service 
<p align="center">
  <img src=screenshots/web_dosintro.png width="500">
</p>
  - overwhelms websites or apps to the user access, they cant login, shop and, access services
  - the lab explores attacks at the application layer 7 of the OSI model
  - relies on a single machine and a single internet connection

**DDoS** Dsitributed Denial-of-Service
<p align="center">
  <img src=screenshots/web_ddosintro2.png width="500">
</p>
  - an davanced form of DoS instead of one machine one internet connection, it uses botnets 
  - botnets(army of compromised devices under attacker control)
  
| DoS Attack Type | Description |
|---|---|
| Slowloris | Sending many partial HTTP requests to tie up server resources |
| HTTP Flood | Sending a large number of HTTP requests to overwhelm the server |
| Cache Bypass | Bypassing CDN edge servers and forcing the origin server to respond |
| Oversized Query | Forcing the server to process large, resource-intensive requests |
| Login/Form Abuse | Overloading authentication logic with login attempts or password resets |
| Faulty Input Validation Abuse | Exploiting poorly designed input handling |

**1. What class of attack relies on disrupting the availability of a web service?**
 
**Answer: Denial-of-Service**
 
---
 
**2. What do we call the network of compromised machines that attackers use to launch DDoS attacks?**
 
**Answer: Botnet**
 
---
 
## Task 3 - Attack Motives
 <p align="center">
  <img src=screenshots/web_attackmotive.png width="500">
</p>

Attackers have various different motivations to coordinate  attcaks.  
 
| Motive | Description | Example Scenario |
|---|---|---|
| Financial Loss | Disrupt services to stop or reduce sales and revenue | Flooding an e-commerce site during peak holiday sales |
| Extortion | Demand payment to stop a current attack | Threatening a bank with a ransom DDoS |
| Hacktivism | Disruption for social or political protest | Attacking government websites during election season |
| Distraction | Redirect defenders' attention while other attacks take place | Launching a DDoS while attacking other infrastructure |
| Competition | Disrupt a rival's service to drive up costs or gain market share | A competitor launches a DDoS during a product launch |
| Denial of Wallet | Force the victim to rack up service usage costs | Attackers repeatedly access AWS S3 data, generating costs per request |
| Reputational Damage | Cause customers to lose trust in a company | Game servers crashing during launch day |
 
**1. Which attacker motive aims to make customers lose confidence in a company?**
 
**Answer: Reputational Damage**
 
---
 
**2. Which motive most likely drove the 2023 DDoS attack against Microsoft?**
 
**Answer: Hacktivism**
 
---
 
## Task 4 - Log Analysis

 **Logs** offers a valuable source of evidence when investivgating DoS
   - All major web services Apache, Nginx, Microsoft IIS records their web requests

  **Indicators of DoS and DDoS attcks**
 
| Indicator | Example | Description |
|---|---|---|
| High Request Rate | 10.10.10.100 -> 1000 GET /login | A resource-heavy page flooded with requests to overwhelm authentication |
| Odd User-Agents | curl/7.6.88 -> /index repeatedly | Spoofed or unusual User-Agents pointing to automated attacks |
| Geographic Anomalies | IP addresses from around the world | Globally distributed botnet traffic from unexpected regions |
| Burst Timestamps | 50 requests in 1 second -> /search | Unnatural traffic spikes pointing to automation |
| Server Errors (5xx) | Spike of 503 Service Unavailable | Resources maxed out, service struggling under attack traffic |
| Logic Abuse | GET /products?limit=999999 | Queries crafted to force the server to load massive data sets |

**SOCs** should look for multiple layered signals that form a picture of a DDoS attempt
  - attackers will target resource-intensive endpoints
      - with same **User-Agent** string or a change it up to seem legitimate
 
**Commonly targeted endpoints:**
 
| Endpoint | Reason |
|---|---|
| /login | Involves authentication processes |
| /search | Requires complex database queries |
| /api endpoints | Critical for dynamic content delivery |
| /register or /signup | Requires database writes and validation |
| /contact or /feedback | Requires database entries and can trigger email notifications |
| /cart or /checkout | Requires session management, inventory checks, and payment processing |

**Example of Highly Condensed Log Snippet**
 <p align="center">
  <img src=screenshots/web_logsnippet.png width="500">
</p>
  1. Normal USer Traffic: every few seconds a user requests a page and receives a repsonse as expected
  2. **DoS Attack**: begins at timestamp 10:01:10 with IP address of 203.0.113.55, sends repetaed GET requests to /login.php
  3. Web Server Down: Users are requesting pages and receiving a 503 service unavaliable reposnse

 
**1. What is the attacker's IP address?**

  <p align="center">
  <img src=screenshots/web_accesslogexample.png width="500">
</p>
*I opened the log in Splunk and examined it, we can see the IP 203.12.23.195 had 92 repeated requests*
**Answer: 203.12.23.195**
 
---
 
**2. Which page is repeatedly targeted by the attacker's requests?**

 <p align="center">
  <img src=screenshots/web_loginpage.png width="500">
</p>
*Filter by uri_path*
**Answer: /login**
 
---
 
**3. After the attack, what error code do legitimate users receive?**

  <p align="center">
  <img src=screenshots/web_statuscode.png width="500">
</p>
**Answer: 503**
 
---
 
## Task 5 - Leveraging SIEMs
 
Digging thru raw logs on the command line is not productive. 
  - SIEM Security Information and Event Management platforms makes reading through logs much easier
  - combines multiple logs and we can filter them by useful fields

**Example**: Splunk offers views in timecharts this example shows a timespan of requests(Visualization)
  - between 2:35 and 2:45
  - **timechart span=1min count by uri limit=5**
  - "Give me a per-minute count of requests, broken out by the top 5 most requested URIs, displayed as a time chart."
  <p align="center">
  <img src=screenshots/web_timechartexample.png width="500">
</p>

 
**1. What was the most frequently requested `uri`?**
 n=
**Answer:**
 
---
 
**2. Which `clientip` made the most requests to the target `uri`?**
   <p align="center">
  <img src=screenshots/web_clientipuri.png width="500">
</p>

**Answer: 203.0.113.7**
 
---
 
**3. How many IP addresses were part of the botnet that attacked your website?**

 *The above screenhsot privides the IP that made the most request and also the total number of IPs that made reuqests*
**Answer: 60**
 
---
 
**4. Which `useragent` was most commonly used by the attacking traffic?**
    
    <p align="center">
  <img src=screenshots/web_javaagent.png width="500">
</p>

**Answer: Java/1.8.0_181**
 
---
 
**5. Use the `timechart` command to visualize the requests. What is the peak number of requests made per second during the attack?**

   <p align="center">
  <img src=screenshots/web_timechart.png width="500">
</p>
**Answer: 207**
 
---
 
**6. Which legitimate (non-attacking) `clientip` received the first `503` response status post-attack?**

   <p align="center">
  <img src=screenshots/web_lastip503.png width="500">
</p>
*I started on page 1 and checked every page to see where the attacker IP shows up, page 8 last IP was 10.10.0.27 and page 8 we can start seeing the attackers IPs and botnet 203.0.113.26*
**Answer: 10.10.0.27**
 
---
 
## Task 6 - Defense

 Secure websites start with secure code
   - sanitized input boxes
   - CAPTCHA checks prevents automated access by bots
   - Load-balanicng is crucial to distribute traffic accross servers to prevent over load
 
| Defense Type | Description |
|---|---|
| Input Validation | Validates user input to prevent malformed or resource-intensive queries |
| CAPTCHA | Challenge requiring the user to solve a puzzle, blocking bots |
| JavaScript Challenge | Background check confirming human traffic, fails automated tools |
| CDN | Caches and serves content from edge servers, absorbs DDoS traffic |
| Load Balancing | Distributes traffic across servers so no single server is overloaded |
| WAF | Inspects and filters traffic using rules and threat intelligence |
 
**1. What type of security challenge blocks bots by asking users to solve a simple puzzle?**
 
**Answer: CAPTCHA**
 
---
 
**2. Which CDN feature spreads traffic across multiple servers to prevent overload?**
 
**Answer: Load-balancing**
 
---
 
## Task 7 - Conclusion
 
**1. Complete the room and continue on your cyber learning journey!**
 
**Answer:** N/A
 
---
 
