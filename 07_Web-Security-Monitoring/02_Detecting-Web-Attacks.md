---
title: Detecting Web Attacks
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [web-attacks, log-analysis, wireshark, waf, sqli, brute-force, xss, csrf, blue-team, soc-level-1]
status: completed
date: 05-19-2026
date_completed: 05-19-2026
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

### Key Concepts

<!-- Q: What are the four main detection methods or topics covered in this room? -->
**Web Attacks** are one of the most common ways attackers gain entry into systems
  - Because websites and web apps sit in front of databases and back-end infrastructure
<!-- Q: What three prerequisite rooms are recommended and what does each one cover? -->

### Task Questions

**1. I understand the learning objectives and am ready to learn about detecting web attacks!**

**Answer:** N/A

---

## Task 2 - Client-Side Attacks

<p align="center">
<img src=screenshots/web_client.png width="400">
</p>

### Key Concepts

<!-- Q: What makes client-side attacks difficult to detect from a SOC perspective? What visibility do analysts have? -->
**Client-Side Attacks**
  - Relies on attackers abusing weaknesses in: user behavior and user devices
  - Exploits vulnerabilities in browsers
  - Tricks users into performing actions to give attackers access to accounts and valuable data stored on the user's device
  - Occurs inside the user's system
  - The attack targets the manipulated environment (the website), so there is no suspicious HTTP traffic or logs for SOCs to investigate

**Number of Web Apps Is Rising**
  - Companies are integrating more third-party plugins, which broadens the attack surface
<!-- Q: What is XSS? Walk through the comment box example in your own words. What could an attacker do with it beyond a harmless pop-up? -->

<p align="center">
<img src=screenshots/web_xss.png width="400">
</p>

**Cross-Site Scripting (XSS)**
  - Most common client-side attack
  - An attacker injects a script into an input field; when a user visits that trusted site, the script runs inside their browser
<!-- Q: What is CSRF? How does it trick the browser? -->
**Cross-Site Request Forgery (CSRF)**
  - The browser is tricked into sending unauthorized requests on behalf of the trusted user
<!-- Q: What is clickjacking? How does it work? -->

### Task Questions

**1. What class of attacks relies on exploiting the user's behavior or device?**

**Answer: Client-Side**

---

**2. What is the most common client-side attack?**

**Answer: XSS**

---

## Task 3 - Server-Side Attacks

<p align="center">
<img src=screenshots/web_ssrf.png width="400">
</p>

<p align="center">
<img src=screenshots/web_serverside.png width="400">
</p>

### Key Concepts

<!-- Q: What is the key advantage defenders have when dealing with server-side attacks vs. client-side attacks? -->
**Server-Side Attacks**
  - Exploit vulnerabilities in: the web server, the application's code, or the back end
  - Client-side attacks manipulate a user's interaction with a site; server-side attacks happen inside the systems themselves
  - Server-side attacks leave a trail of evidence for SOCs to follow

**Common Server-Side Attacks**
  - Brute-force: repeatedly tries different usernames and passwords to gain access to an account
  - SQL Injection (SQLi): attacks the database that sits behind the website by injecting SQL code into an unsanitized input field
  - Command Injection: the website takes user input and passes it directly to the system without checking it, and the server just runs the commands
<!-- Q: What is command injection? Why is it dangerous in terms of permissions? -->

| Attack Type | How It Works | Evidence Left Behind |
|---|---|---|
| Brute-Force | Automated tool sends repeated login attempts with different credential combinations | Repeated POST requests in quick succession in access logs |
| SQL Injection | Attacker injects SQL code into an unsanitized input field to manipulate the database query | SQLi payloads visible in query strings in logs; full payload and results visible in packet captures |
| Command Injection | Attacker injects OS commands into user input that is passed directly to the system without validation | Unusual system-level commands appearing in request parameters |

### Task Questions

**1. What class of attacks relies on exploiting vulnerabilities within web servers?**

**Answer: Server-Side**

---

**2. Which server-side attack lets attackers abuse forms to dump database contents?**

**Answer: SQLi**

---

## Task 4 - Log-Based Detection

### Key Concepts

<!-- Q: What are the six fields in a standard web server access log? What does each field reveal to a SOC analyst? -->
**Log Format**
  1. Client IP Address: known malicious IP or outside of expected geo range
  2. Timestamp and Requested Page: requests made at unusual hours or repeated in a short period of time
  3. Status Code: repeated 404 responses indicating a page could not be found
  4. Response Size: significantly smaller or larger than normal response sizes
  5. Referrer: referring pages that do not fit normal site navigation
  6. User-Agent: outdated browser versions or common attack tools such as sqlmap and wpscan

<p align="center">
<img src=screenshots/web_logformatexample.png width="400">
</p>

| Log Field | Example Indicator |
|---|---|
| Client IP Address | Known malicious IP or outside expected geo range |
| Timestamp and Requested Page | Requests at unusual hours or repeated in a short period |
| Status Code | Repeated 404s indicate page not found; 302 indicates a successful redirect after brute-force |
| Response Size | Significantly smaller or larger than normal |
| Referrer | Referring pages that do not fit normal site navigation |
| User-Agent | Outdated browser versions or attack tools (sqlmap, wpscan, FFUF) |

<!-- Q: Walk through the three-stage attack sequence shown in the logs. What does each stage look like and what clues does it leave? -->
**Three-Stage Attack Sequence**

1 - Attacker tests directories and forms with a directory fuzz; 200 response codes signify valid finds for the attacker
<p align="center">
<img src=screenshots/web_fuzz.png width="400">
</p>
<!-- Q: What do 200 response codes indicate during a directory fuzz? What does a 302 response code mean in the context of a brute-force? -->
2 - Attacker exploits login.php with repeated POST requests in quick succession; a 302 Found response indicates a successful brute-force login
<p align="center">
<img src=screenshots/web_post2.png width="400">
</p>

3 - Attacker has gained access to the account and attempts SQLi payloads `' OR '1'='1` and `1' OR 'a'='a` on the /search form
<p align="center">
<img src=screenshots/web_sqli.png width="400">
</p>
<!-- Q: What are the limitations of access logs? What data is NOT captured, and why does that matter for investigations? -->
**Logs Do NOT Always Capture the Full Contents of a Request**
  - The body of POST requests is not recorded -- credentials and payloads submitted in forms are not visible in access logs
<!-- Q: GET vs POST in logs -- what does each capture and what does each miss? -->
  - GET requests may log full paths and query strings depending on server configuration
  - POST requests show that the request occurred but not what was submitted

### Task Questions

**1. What is the attacker's User-Agent while performing the directory fuzz?**
<p align="center">
<img src=screenshots/web_fuffv2.png width="400">
</p>

**Answer: FFUF v2.1.0**

---

**2. What is the name of the page on which the attacker performs a brute-force attack?**

**Answer: /login.php**

---

**3. What is the complete, decoded SQLi payload the attacker uses on the /changeusername.php form?**
<p align="center">
<img src=screenshots/web_sqlidecoded.png width="400">
</p>
<p align="center">
<img src=screenshots/web_sqlidecoded2.png width="400">
</p>

*URL-encoded payload decoded using CyberChef*

**Answer: %' OR '1'='1**

---

## Task 5 - Network-Based Detection

### Key Concepts

<!-- Q: What advantage does network traffic analysis have over log-based detection? What additional data is visible? -->
**Network Analysis** gives the SOC a broader picture than logs alone
  - Captures raw data exchanged between client and server -- full requests and responses
  - Inspecting packets reveals underlying transport protocols and application data
<!-- Q: What is the limitation of network-based detection when HTTPS is involved? -->
**HTTPS Limitation**
  - HTTPS traffic is encrypted, so packet payloads are not readable without access to the decryption keys
  - WAFs can decrypt TLS traffic; passive packet capture alone cannot
<!-- Q: Walk through the three-stage attack sequence in Wireshark. What does each stage look like at the packet level? -->

<!-- Q: What Wireshark filter is used in the task? What does the http.user_agent filter show? -->
**Wireshark Filters**
  - `ip.dst == 10.10.20.200 && http.user_agent` -- filter by destination IP and User-Agent
  - `http` -- show only HTTP traffic
  - `http.user_agent contains "sqlmap"` -- filter for requests made by sqlmap
<!-- Q: What does "Follow HTTP Stream" do in Wireshark and why is it useful for investigations? -->
**Follow HTTP Stream**
  - Reconstructs the full request and response between client and server
  - Can reveal submitted credentials, POST body data, and SQLi results -- information that access logs cannot capture
<!-- Q: What additional detail does network traffic reveal that logs cannot -- use the brute-force and SQLi examples from the task. -->

### Task Questions

**1. What password does the attacker successfully identify in the brute-force attack?**
<p align="center">
<img src=screenshots/web_astrongpass.png width="400">
</p>

**Answer: astrongpassword123**

---

**2. What is the flag the attacker found in the database using SQLi?**
<p align="center">
<img src=screenshots/web_dumpthedb.png width="400">
</p>

*Filter used: `http.user_agent contains "sqlmap"`*

**Answer: THM{dumped_the_db}**

---

## Task 6 - Web Application Firewall

<p align="center">
<img src=screenshots/web_wafexample.png width="400">
</p>

### Key Concepts

<!-- Q: How does a WAF differ from reviewing logs or packet captures? What can a WAF do that passive analysis cannot? -->
**WAF**
  - First line of defense for websites and web applications
  - Inspects and filters web requests based on configured rules -- what gets allowed in or blocked
  - Unlike Wireshark, a WAF can decrypt TLS/HTTPS traffic
  - Malicious bot traffic makes up 37% of global web traffic
      - CAPTCHA challenge-response can be used instead of blocking outright, reducing the chance of blocking legitimate users
  - Modern WAFs include pre-built rule sets designed to mitigate OWASP Top 10 attacks
      - Leverages threat intelligence feeds to automatically block requests from known malicious IPs
      - Receives regular updates to combat new and emerging threats, including recently discovered CVEs
<!-- Q: What are the four categories of WAF rules? Give an example of each. -->
**4 Rule Types**
  - Block common attack patterns
  - Deny known malicious sources
  - Custom-built rules
  - Rate-limiting and abuse prevention

**Example Rule -- Blocking User-Agent "sqlmap":**
```
If User-Agent contains "sqlmap" then BLOCK
```

| Rule Type | Description | Example Use Case |
|---|---|---|
| Block common attack patterns | Blocks known malicious payloads and indicators | Block requests with User-Agent matching `sqlmap` |
| Deny known malicious sources | Uses IP reputation, threat intel, or geo-blocking to stop risky traffic | Block IPs from recent botnet campaigns |
| Custom-built rules | Tailored rules for a specific application's needs | Allow only GET/POST requests to /login |
| Rate-limiting and abuse prevention | Limits request frequency to prevent abuse | Limit login attempts to 5 per minute per IP |

### Task Questions

**1. What do WAFs inspect and filter?**

**Answer: Web Requests**

---

**2. Create a custom firewall rule to block any User-Agent that matches "BotTHM".**

**Answer: IF User-Agent CONTAINS "BotTHM" THEN BLOCK**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Q: Without looking back, summarize the three detection methods covered in this room and one key strength or limitation of each. -->

<!-- Q: How does correlating logs, network traffic, and WAF rules give a more complete picture than any single source alone? -->

### Task Questions

**1. Complete the room and continue on your cyber learning journey!**

**Answer:** N/A
