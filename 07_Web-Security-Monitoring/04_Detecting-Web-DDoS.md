---
title: Detecting Web DDoS
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [ddos, dos, web-security, log-analysis, splunk, waf, cdn, soc-level-1]
status: completed
date: 2026-05-21
date_completed: 2026-05-22
---

<p align="center">
  <img src=screenshots/web_ddosintro.png width="1500">
</p>

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

**Denial-of-Service (DoS)** attacks take many forms, but their main goal is to disrupt or completely block access to a website or web service. Attacks can be launched against different layers of the OSI model -- this room focuses on the **application layer (Layer 7)**.

**1. I understand the learning objectives and am ready to embark on a Denial-of-Service adventure!**

**Answer:** N/A

---

## Task 2 - DoS and DDoS Attacks

**DoS** - Denial-of-Service
<p align="center">
  <img src=screenshots/web_dosintro.png width="500">
</p>

- Overwhelms websites or apps so users cannot log in, shop, or access services
- Targets the application layer (Layer 7) of the OSI model
- Limited to a single machine and a single internet connection

**DDoS** - Distributed Denial-of-Service
<p align="center">
  <img src=screenshots/web_ddosintro2.png width="500">
</p>

- An advanced form of DoS -- instead of one machine, it leverages a botnet
- **Botnet**: an army of compromised devices (computers, IoT devices, servers) under attacker control
- Far more effective than a basic DoS because it is not capped by one machine's resources

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

Attackers have a variety of motivations behind DoS and DDoS campaigns. The consequences can be severe -- lost revenue, frustrated users, and lasting reputational damage.

| Motive | Description | Example Scenario |
|---|---|---|
| Financial Loss | Disrupt services to stop or reduce sales and revenue | Flooding an e-commerce site during peak holiday sales |
| Extortion | Demand payment to stop a current attack | Threatening a bank with a ransom DDoS |
| Hacktivism | Disruption for social or political protest | Attacking government websites during election season |
| Distraction | Redirect defenders' attention while other attacks take place | Launching a DDoS while attacking other infrastructure |
| Competition | Disrupt a rival's service to drive up costs or gain market share | A competitor launches a DDoS during a product launch |
| Denial of Wallet | Force the victim to rack up service usage costs | Attackers repeatedly access AWS S3 data, generating costs per request |
| Reputational Damage | Cause customers to lose trust in a company | Game servers crashing during launch day |

**In the Wild**
- **2015 BBC attack**: taken offline on New Year's Eve by the group New World Hacking, who claimed it was simply a test of their capabilities
- **2023 Microsoft attack**: Anonymous Sudan used HTTP flooding and Slowloris to disrupt Azure, OneDrive, and Outlook

**1. Which attacker motive aims to make customers lose confidence in a company?**

**Answer: Reputational Damage**

---

**2. Which motive most likely drove the 2023 DDoS attack against Microsoft?**

**Answer: Hacktivism**

---

## Task 4 - Log Analysis

Web server logs (Apache, NGINX, Microsoft IIS) are a valuable source of evidence when investigating denial-of-service attacks. Analysts should look for multiple layered signals that together form a picture of a DDoS attempt -- a single indicator alone may not be conclusive.

**Indicators of DoS and DDoS Attacks**

| Indicator | Example | Description |
|---|---|---|
| High Request Rate | 10.10.10.100 -> 1000 GET /login | A resource-heavy page flooded with requests to overwhelm authentication |
| Odd User-Agents | curl/7.6.88 -> /index repeatedly | Spoofed or unusual User-Agents pointing to automated attacks |
| Geographic Anomalies | IP addresses from around the world | Globally distributed botnet traffic from unexpected regions |
| Burst Timestamps | 50 requests in 1 second -> /search | Unnatural traffic spikes pointing to automation |
| Server Errors (5xx) | Spike of 503 Service Unavailable | Resources maxed out, service struggling under attack traffic |
| Logic Abuse | GET /products?limit=999999 | Queries crafted to force the server to load massive data sets |

Attackers target resource-intensive endpoints and may use the same **User-Agent** string across requests, or rotate User-Agents to blend in with legitimate traffic.

**Commonly Targeted Endpoints**

| Endpoint | Reason |
|---|---|
| /login | Involves authentication processes |
| /search | Requires complex database queries |
| /api endpoints | Critical for dynamic content delivery |
| /register or /signup | Requires database writes and validation |
| /contact or /feedback | Requires database entries and can trigger email notifications |
| /cart or /checkout | Requires session management, inventory checks, and payment processing |

**Example of a Highly Condensed Log Snippet**
<p align="center">
  <img src=screenshots/web_logsnippet.png width="500">
</p>

1. **Normal user traffic**: every few seconds, a user requests a page and receives a response as expected
2. **DoS attack**: begins at timestamp 10:01:10 -- IP address 203.0.113.55 sends repeated GET requests to /login.php
3. **Web server down**: users are requesting pages and receiving 503 Service Unavailable responses

---

**1. What is the attacker's IP address?**

<p align="center">
  <img src=screenshots/web_accesslogexample.png width="500">
</p>

*Opened the access log and examined it -- IP 203.12.23.195 had 92 repeated requests, a clear sign of automated attack traffic.*

**Answer: 203.12.23.195**

---

**2. Which page is repeatedly targeted by the attacker's requests?**

<p align="center">
  <img src=screenshots/web_loginpage.png width="500">
</p>

*Filtered by `uri_path` to identify the most targeted endpoint.*

**Answer: /login**

---

**3. After the attack, what error code do legitimate users receive?**

<p align="center">
  <img src=screenshots/web_statuscode.png width="500">
</p>

**Answer: 503**

---

## Task 5 - Leveraging SIEMs

Digging through raw logs on the command line is not efficient at scale. **SIEM** (Security Information and Event Management) platforms make log analysis far more effective by combining multiple log sources and allowing analysts to filter by useful fields such as IP address, User-Agent, or response code.

**Splunk timechart example** (visualizing requests over a 10-minute window):

```
timechart span=1min count by uri limit=5
```

- `timechart` -- creates a time-based chart, bucketing events across time
- `span=1min` -- each data point represents 1 minute of events
- `count` -- counts how many log events fall into each time bucket
- `by uri` -- groups the count by URI, so you see a separate line per endpoint
- `limit=5` -- caps output to the top 5 URIs to keep the chart readable

"Give me a per-minute count of requests, broken out by the top 5 most requested URIs, displayed as a time chart." In a DDoS investigation, this is where you see one URI spiking hard while the others stay flat -- that's your targeted endpoint.

<p align="center">
  <img src=screenshots/web_timechartexample.png width="500">
</p>

---

**1. What was the most frequently requested `uri`?**

**Answer: /search**

---

**2. Which `clientip` made the most requests to the target `uri`?**

<p align="center">
  <img src=screenshots/web_clientipuri.png width="500">
</p>

**Answer: 203.0.113.7**

---

**3. How many IP addresses were part of the botnet that attacked your website?**

*The screenshot above shows both the IP with the highest request count and the total number of IPs making requests to the target URI.*

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

*Paginated through the results starting at page 1. On page 8, the last legitimate IP visible was 10.10.0.27 -- after that, the attacker IPs and botnet addresses (starting with 203.0.113.x) begin to appear.*

**Answer: 10.10.0.27**

---

## Task 6 - Defense

Attackers constantly look for weak points to exploit, but defenders have tools at every layer to keep services resilient. A strong defense combines secure development practices with network and infrastructure controls.

**Application-Level Defenses**

| Defense Type | Description |
|---|---|
| Input Validation | Validates and sanitizes user input to prevent malformed or resource-intensive queries |
| CAPTCHA | Challenge requiring the user to solve a puzzle, blocking automated bots |
| JavaScript Challenge | Runs quietly in the background to confirm human traffic -- automated tools typically fail |

**Network and Infrastructure Defenses**

| Defense Type | Description |
|---|---|
| CDN | Caches and serves content from edge servers closest to users, absorbing DDoS traffic before it hits the origin |
| Load Balancing | Distributes traffic across multiple servers so no single server is overloaded |
| WAF | Inspects incoming traffic and blocks or challenges requests based on rules and threat intelligence |

**Bypassing Defenses**
Attackers can attempt to bypass CDN caching by appending random query parameters (e.g. `/products?a=abcd`), forcing the origin server to respond directly. Rotating User-Agents and spoofing referrer headers are also used to evade WAF rules.

**Large-Scale Mitigation**
- Google mitigated a DDoS peaking at **398 million requests per second** in 2023
- Cloudflare claims the largest ever recorded, peaking at **11.5 Tbps**, lasting only 35 seconds

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
