---
title: Detecting Web Attacks
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [web-attacks, log-analysis, wireshark, waf, sqli, brute-force, xss, csrf, blue-team, soc-level-1]
status: in-progress
date: 05-19-2026
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

### Key Concepts

<!-- Q: What are the four main detection methods or topics covered in this room? -->

<!-- Q: What three prerequisite rooms are recommended and what does each one cover? -->

### Task Questions

**1. I understand the learning objectives and am ready to learn about detecting web attacks!**

**Answer:** N/A

---

## Task 2 - Client-Side Attacks

### Key Concepts

<!-- Q: What makes client-side attacks difficult to detect from a SOC perspective? What visibility do analysts have? -->

<!-- Q: What is XSS? Walk through the comment box example in your own words. What could an attacker do with it beyond a harmless pop-up? -->

<!-- Q: What is CSRF? How does it trick the browser? -->

<!-- Q: What is clickjacking? How does it work? -->

<!-- Q: Why does browser-side activity create a blind spot for SOC tools like server logs and network captures? -->

### Task Questions

**1. What class of attacks relies on exploiting the user's behavior or device?**

**Answer:**

---

**2. What is the most common client-side attack?**

**Answer:**

---

## Task 3 - Server-Side Attacks

### Key Concepts

<!-- Q: What is the key advantage defenders have when dealing with server-side attacks vs. client-side attacks? -->

<!-- Q: What is a brute-force attack? What real-world breach is referenced in this task and what was the impact? -->

<!-- Q: What is SQL injection? What is the difference between string concatenation and parameterized queries? What breach is referenced and what was its scale? -->

<!-- Q: What is command injection? Why is it dangerous in terms of permissions? -->

| Attack Type | How It Works | Evidence Left Behind |
|---|---|---|
| Brute-Force | | |
| SQL Injection | | |
| Command Injection | | |

### Task Questions

**1. What class of attacks relies on exploiting vulnerabilities within web servers?**

**Answer:**

---

**2. Which server-side attack lets attackers abuse forms to dump database contents?**

**Answer:**

---

## Task 4 - Log-Based Detection

### Key Concepts

<!-- Q: What are the six fields in a standard web server access log? What does each field reveal to a SOC analyst? -->

| Log Field | Example Indicator |
|---|---|
| Client IP Address | |
| Timestamp and Requested Page | |
| Status Code | |
| Response Size | |
| Referrer | |
| User-Agent | |

<!-- Q: Walk through the three-stage attack sequence shown in the logs. What does each stage look like and what clues does it leave? -->

<!-- Q: What do 200 response codes indicate during a directory fuzz? What does a 302 response code mean in the context of a brute-force? -->

<!-- Q: What are the limitations of access logs? What data is NOT captured, and why does that matter for investigations? -->

<!-- Q: GET vs POST in logs -- what does each capture and what does each miss? -->

### Task Questions

**1. What is the attacker's User-Agent while performing the directory fuzz?**

**Answer:**

---

**2. What is the name of the page on which the attacker performs a brute-force attack?**

**Answer:**

---

**3. What is the complete, decoded SQLi payload the attacker uses on the /changeusername.php form?**

**Answer:**

---

## Task 5 - Network-Based Detection

### Key Concepts

<!-- Q: What advantage does network traffic analysis have over log-based detection? What additional data is visible? -->

<!-- Q: What is the limitation of network-based detection when HTTPS is involved? -->

<!-- Q: Walk through the three-stage attack sequence in Wireshark. What does each stage look like at the packet level? -->

<!-- Q: What Wireshark filter is used in the task? What does the http.user_agent filter show? -->

<!-- Q: What does "Follow HTTP Stream" do in Wireshark and why is it useful for investigations? -->

<!-- Q: What additional detail does network traffic reveal that logs cannot -- use the brute-force and SQLi examples from the task. -->

### Task Questions

**1. What password does the attacker successfully identify in the brute-force attack?**

**Answer:**

---

**2. What is the flag the attacker found in the database using SQLi?**

**Answer:**

---

## Task 6 - Web Application Firewall

### Key Concepts

<!-- Q: How does a WAF differ from reviewing logs or packet captures? What can a WAF do that passive analysis cannot? -->

<!-- Q: What are the four categories of WAF rules? Give an example of each. -->

| Rule Type | Description | Example Use Case |
|---|---|---|
| Block common attack patterns | | |
| Deny known malicious sources | | |
| Custom-built rules | | |
| Rate-limiting and abuse prevention | | |

<!-- Q: What is a challenge-response mechanism in the context of a WAF? When is it preferred over outright blocking? -->

<!-- Q: How do WAFs integrate threat intelligence? What kinds of lists do they maintain and why? -->

<!-- Q: Walk through the sqlmap rule example. Write the logic in your own words. -->

### Task Questions

**1. What do WAFs inspect and filter?**

**Answer:**

---

**2. Create a custom firewall rule to block any User-Agent that matches "BotTHM".**

**Answer:**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Q: Without looking back, summarize the three detection methods covered in this room and one key strength or limitation of each. -->

<!-- Q: How does correlating logs, network traffic, and WAF rules give a more complete picture than any single source alone? -->

### Task Questions

**1. Complete the room and continue on your cyber learning journey!**

**Answer:** N/A
