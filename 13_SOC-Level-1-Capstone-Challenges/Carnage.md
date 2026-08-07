---
title: Carnage
module: Network Security and Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [traffic-analysis, wireshark, pcap, http, tls, cobalt-strike, c2, malspam, dfir]
status: Completed
date: 2026-08-05
date_completed: 2026-08-05
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/carnage.png width="1500">
</p>
---

## Task 1: Scenario

Eric Fischer, Purchasing Department at Bartell Ltd, received an email from a known contact with a Word document attachment. He opened it and clicked **Enable Content**. The SOC endpoint agent alerted on suspicious outbound connections from his workstation. The pcap was pulled from the network sensor and handed to me for analysis.

**Task:** investigate the packet capture and uncover the malicious activity.

> Credit: pcap captured and shared by Brad Duncan, malware-traffic-analysis.net
> Safety note: do not directly interact with any domain or IP in this challenge.

**Setup notes:**

- Wireshark time display set to `View > Time Display Format > UTC Date and Time of Day`
- Why: the room asks for absolute timestamps, and the default "seconds since beginning of capture" will not match the answer format.
- Victim host is `10.9.23.102`. Every outbound connection from that IP is in scope.

---

## Task 2: Traffic Analysis

<p align="center">
<img src=screenshots/carnage1.png width="700">
</p>

### Q1. What was the date and time for the first HTTP connection to the malicious IP? (answer format: yyyy-mm-dd hh:mm:ss)

The first outbound connection is the anchor for the whole timeline. Everything after it is post-compromise.

Set the time format first, otherwise the answer will not match the required format:

*View -> Time Display Format -> UTC Date and Time of Day*

Then filter for outbound requests and take the earliest one:

```
http.request.method == GET
```

<p align="center">
<img src=screenshots/carnage-q1.png width="700">
</p>

**Answer: 2021-09-24 16:44:38**

---

### Q2. What is the name of the zip file that was downloaded?

Same packet as Q1. The very first HTTP request in the capture is the download itself, so the filename is in the request URI.

<p align="center">
<img src=screenshots/carnage-q2.png width="700">
</p>

**Answer: documents.zip**

---

### Q3. What was the domain hosting the malicious zip file?

The packet list column only shows the URI path, not the host. The full URL lives in the packet details pane:

*Hypertext Transfer Protocol > Host*

HTTP/1.1 sends the hostname in the `Host:` header rather than in the request line, which is why it is not visible in the summary column.

<p align="center">
<img src=screenshots/carnage-q3.png width="700">
</p>

**Answer: attirenepal.com**

---

### Q4. Without downloading the file, what is the name of the file in the zip file?

Follow the TCP stream. Zip archives store filenames in the **local file header as plain text**, even though the file contents are compressed. So the name is readable in the raw stream without ever extracting anything.

<p align="center">
<img src=screenshots/carnage-q4.png width="700">
</p>

**Answer: chart-1530076591.xls**

---

### Q5. What is the name of the webserver of the malicious IP from which the zip file was downloaded?

In the HTTP response packet, packet details pane:

*Hypertext Transfer Protocol > Server*

<p align="center">
<img src=screenshots/carnage-q5.png width="700">
</p>

**Answer: LiteSpeed**

---

### Q6. What is the version of the webserver from the previous question?

Two headers in the same response, and they describe different layers:

- `Server:` gives **LiteSpeed**, the web server, with no version string
- `X-Powered-By:` gives **PHP/7.2.34**, the application layer running on top

The room expects the `X-Powered-By` value here even though, strictly speaking, that is the scripting engine version and not the web server version. Worth noting so I do not second-guess it next time.

<p align="center">
<img src=screenshots/q6.png width="700">
</p>

**Answer: PHP/7.2.34**

---

### Q7. Malicious files were downloaded to the victim host from multiple domains. What were the three domains involved with this activity?

The room's hint says check HTTPS traffic and narrow to 16:45:11 through 16:45:30.

TLS is encrypted, but the **Client Hello is not**. It carries the SNI extension, which is the hostname the client is asking for in cleartext. That is what makes this answerable at all:

```
tls.handshake.extensions_server_name
```

In that 29-second window the host reaches five domains:

| Packet | Time | Destination | Domain | Verdict |
|---|---|---|---|---|
| 2427 | 16:45:11.840716 | 148.72.192.206 | finejewels.com.au | malicious |
| 2646 | 16:45:17.228469 | 13.69.109.131 | self.events.data.microsoft.com | legitimate Windows telemetry |
| 2909 | 16:45:20.389994 | 20.54.36.229 | client.wns.windows.com | legitimate Windows notification service |
| 3009 | 16:45:21.314012 | 210.245.90.247 | thietbiagt.com | malicious |
| 3229 | 16:45:25.731116 | 148.72.53.144 | new.americold.com | malicious |

The two Microsoft domains are normal background noise from any Windows host. Knowing what normal looks like is what makes the other three stand out.

<p align="center">
<img src=screenshots/carnage-q7.png width="700">
</p>

**Answer: finejewels.com.au, thietbiagt.com, new.americold.com**

---

### Q8. Which certificate authority issued the SSL certificate to the first domain from the previous question?

The server's certificate is sent in cleartext during the handshake, before encryption starts. Follow the TCP stream on the finejewels.com.au conversation and the issuer string is readable:

```
GoDaddy.com, Inc.1-0+..U...$http://certs.godaddy.com/repository/1301..U...*Go Daddy Secure Certificate Authority - G20..
```

To isolate just the certificate packet instead of scrolling the stream:

```
tls.handshake.type == 11
```

<p align="center">
<img src=screenshots/carnage-q8.png width="700">
</p>

**Answer: GoDaddy**

---

### Q9. What are the two IP addresses of the Cobalt Strike servers? Use VirusTotal (the Community tab) to confirm if IPs are identified as Cobalt Strike C2 servers. (answer format: enter the IP addresses in sequential order)

This was a hard one. Filtering on `http.request` surfaces two suspicious destinations:

**208.91.128.6**

- POST requests
- The IP itself comes back clean on VirusTotal, but the domain `maldivehost.net` scores 6/91

**185.106.96.158**

- GET requests
- **Beaconing pattern:** a consistent 5-second interval between requests, with each full conversation completing in about 40 milliseconds. Regular timing plus tiny sessions is the signature of an automated check-in, not a human browsing.
- Suspicious base64-looking encoded URI
- The Host header claims `ocsp.verisign.com`, which comes back clean on VirusTotal

That last point is the trap. **Searching the domain got me nowhere, so I searched the IP instead**, which came back 6/91.

<p align="center">
<img src=screenshots/carnage-ips.png width="700">
</p>

The VirusTotal Community tab on 185.106.96.158 confirms Cobalt Strike C2 and names the second IP as well, which matches what I already had in the conversations list.

<p align="center">
<img src=screenshots/carnage-flanders.png width="700">
</p>
<p align="center">
<img src=screenshots/carnage-q9.png width="700">
</p>

**Answer: 185.106.96.158, 185.125.204.174**

---

### Q10. What is the Host header for the first Cobalt Strike IP address from the previous question?

*Packet details pane > Hypertext Transfer Protocol > Host*, then right-click and **Apply as Column** so it is visible in the packet list from here on.

This is why Q9 was hard. Cobalt Strike malleable C2 profiles let the operator **set an arbitrary Host header** to impersonate a trusted service. Verisign OCSP is a certificate revocation checker, so traffic to it looks routine and would sail past a domain-based blocklist. The IP is the real indicator, not the name.

<p align="center">
<img src=screenshots/q10.png width="700">
</p>

**Answer: ocsp.verisign.com**

---

### Q11. What is the domain name for the first IP address of the Cobalt Strike server? You may use VirusTotal to confirm if it's the Cobalt Strike server (check the Community tab).

New filter learned today. `dns.a` matches the **A record in a DNS answer**, so this asks Wireshark to show the DNS response where that IP was the answer. It works backwards from IP to the name that resolved to it:

```
dns.a == 185.106.96.158
```

<p align="center">
<img src=screenshots/q11.png width="700">
</p>

**Answer: survmeter.live**

---

### Q12. What is the domain name of the second Cobalt Strike server IP? You may use VirusTotal to confirm if it's the Cobalt Strike server (check the Community tab).

Same filter, second IP.

```
dns.a == 185.125.204.174
```

<p align="center">
<img src=screenshots/q12.png width="700">
</p>

**Answer: securitybusinpuff.com**

---

### Q13. What is the domain name of the post-infection traffic?

Already noted in Q9. `208.91.128.6` was the destination receiving POST requests, and its domain `maldivehost.net` scores 6/91 on VirusTotal even though the IP itself comes back clean.

POST traffic outbound is the direction that matters. GET pulls data in, POST pushes data out.

<p align="center">
<img src=screenshots/q13.png width="700">
</p>

**Answer: maldivehost.net**

---

### Q14. What are the first eleven characters that the victim host sends out to the malicious domain involved in the post-infection traffic?

Filter for outbound POSTs:

```
http.request.method == POST
```

A 25-second interval between each request, all to the same encoded URI:

```
5208  2021-09-24 16:54:38.381382  10.9.23.102 -> 208.91.128.6  HTTP  265
maldivehost.net  POST /zLIisQRWZI9/AjlCfX9lc2V+fGV7e34= HTTP/1.1
```

The first path segment is the eleven characters. The trailing `=` on the second segment is base64 padding, so that portion is encoded data rather than a real path.

<p align="center">
<img src=screenshots/q14.png width="700">
</p>

**Answer: zLIisQRWZI9**

---

### Q15. What was the length for the first packet sent out to the C2 server?

Same POST filter, sorted by time to get the earliest.

```
http.request.method == POST
```

Interpretation, not fact: the exfiltration appears to be broken into small chunks on a fixed interval rather than one large transfer, which is consistent with trying to stay under volume-based alerting.

<p align="center">
<img src=screenshots/q15.png width="700">
</p>

**Answer: 281**

---

### Q16. What was the Server header for the malicious domain from the previous question?

Follow the TCP stream of any POST packet to 208.91.128.6 and read the response headers.

<p align="center">
<img src=screenshots/q16.png width="700">
</p>

**Answer: Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4**

---

### Q17. The malware used an API to check for the IP address of the victim's machine. What was the date and time when the DNS query for the IP check domain occurred? (answer format: yyyy-mm-dd hh:mm:ss UTC)

`dns.qry.name` matches the name being asked for. `contains` does a substring match rather than an exact one, which is what makes a guess like this work:

```
dns.qry.name contains "api"
```

Malware checks its own public IP for geolocation and to detect whether it is running in a sandbox or a cloud range.

<p align="center">
<img src=screenshots/q17.png width="700">
</p>

**Answer: 2021-09-24 17:00:04**

---

### Q18. What was the domain in the DNS query from the previous question?

Same packet as Q17.

<p align="center">
<img src=screenshots/q18.png width="700">
</p>

**Answer: api.ipify.org**

---

### Q19. Looks like there was some malicious spam (malspam) activity going on. What was the first MAIL FROM address observed in the traffic?

First I tried `smtp contains "@"`, which was not useful. Then I tried the command keyword instead:

```
smtp contains "mail"
```

49 events. What stands out, filtered by source IP, is that the destination IPs cluster in the `185.` range. Attackers commonly buy adjacent blocks, so neighbouring IPs are a grouping signal.

The precise filter for this, worth using next time:

```
smtp.req.command == "MAIL"
```

<p align="center">
<img src=screenshots/q19.png width="700">
</p>

**Answer: farshin@mailfa.com**

---

### Q20. How many packets were observed for the SMTP traffic?

Filter on the protocol alone and read the count from the status bar at the bottom right, which shows **Displayed** out of total.

```
smtp
```

<p align="center">
<img src=screenshots/q20.png width="700">
</p>

**Answer: 1439**

---

## IOC Summary

| Type | Indicator | Context |
|---|---|---|
| IP | 10.9.23.102 | Victim host, Eric Fischer's workstation |
| IP | 185.106.96.158 | Cobalt Strike C2, resolves to survmeter.live |
| IP | 185.125.204.174 | Cobalt Strike C2, resolves to securitybusinpuff.com |
| IP | 208.91.128.6 | Post-infection POST traffic, maldivehost.net |
| IP | 148.72.192.206 | finejewels.com.au |
| IP | 210.245.90.247 | thietbiagt.com |
| IP | 148.72.53.144 | new.americold.com |
| Domain | attirenepal.com | Hosted documents.zip |
| Domain | finejewels.com.au | Second-stage payload delivery |
| Domain | thietbiagt.com | Second-stage payload delivery |
| Domain | new.americold.com | Second-stage payload delivery |
| Domain | survmeter.live | Cobalt Strike C2 |
| Domain | securitybusinpuff.com | Cobalt Strike C2 |
| Domain | maldivehost.net | Post-infection exfiltration |
| Domain | api.ipify.org | Legitimate service abused for victim IP lookup |
| Host header | ocsp.verisign.com | Spoofed in Cobalt Strike traffic to impersonate Verisign OCSP |
| File | documents.zip | Initial download from attirenepal.com |
| File | chart-1530076591.xls | Macro document inside the zip |
| URI | /zLIisQRWZI9/ | First path segment of the exfiltration POST |
| Email | farshin@mailfa.com | First MAIL FROM in the malspam traffic |
| Certificate | Go Daddy Secure Certificate Authority - G2 | Issuer for finejewels.com.au |

---

## Notes and open questions

- Timeline: first HTTP download 16:44:38, second-stage domains 16:45:11 to 16:45:25, C2 POST traffic from 16:54:38, IP check at 17:00:04. Roughly ten minutes from click to established C2.
- The `ocsp.verisign.com` Host header is the lesson from this room. A clean domain reputation means nothing when the operator picks the name. Pivot to the IP.
- Two independent beaconing tells on 185.106.96.158: fixed 5-second interval, and ~40ms full conversations. Either alone is weak, together they are strong.
- Not verified: whether the `.xls` macro is what fetched the second-stage payloads, or whether something else did. Would need the file itself, not just the pcap.
- Not verified: what was actually exfiltrated. The POST bodies are base64-looking but I did not decode them.
- Not verified: the SMTP traffic's relationship to the infection. 1439 packets of outbound mail from a workstation is abnormal, but I did not confirm the host was the sender rather than a relay.
- Open question: three domains served second-stage payloads within 14 seconds. Redundancy in case one was already blocked, or three different payloads?
