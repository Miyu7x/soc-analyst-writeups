---
title: "Wireshark: Packet Operations"
module: "02_Core-SOC-Solutions"
path: "SOC Level 1 > Network Traffic Analysis"
platform: TryHackMe
tags: [wireshark, packet-analysis, statistics, display-filters, protocol-filters, advanced-filtering, network-traffic]
status: in-progress
date: 2026-04-28
date_completed:
---

_Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)_


## Task 1 - Introduction

### Key Concepts

Investigate packets and events of interest inside them with Wireshark
    - apply adavanced filters
    - view statistics and protocol details
---

## Task 2 - Statistics | Summary

### Key Concepts

Menu offers multiple statistics options for invesyihation
    - avaliable protocols
    - endpoints
    - conversations
    - some protocol specific details(DHCP, DNS, HTTP)

**Resolved Addresses**
    - identify IP addresses
    - DNS names 

![Resolved address example](screenshots/wireshark_resolvedaddresses.png)

**Protocol Hierarchy**
    - breaks down all avaliable protocols and capture the file anmd helps analusts view the procols ina tree format
    - packet counters and percentages
    - overall usage of ports and services and focus on the event of interest

![Protocol hierarchy example](screenshots/wireshark_protocolhierarchy.png)

**Coneverstaions**
    - represent traffic between two specific endpoints
    - view them in 5 base formats(ethernet. IPv4, IPv6, TCP and UDP)

![Conversations example](screenshots/wireshark_conversation.png)

**Endpoints**
    - similar to conversation
    - this option though provides unique information for a single information field(ethernet. IPv4, IPv6, TCP and UDP)
    - resoilves known manufacturers MAC address into human readable form
        - activate this under **name resolution** button on the lower left corner of the endpoints window

![](screenshots/wireshark_endpoints.png)

Name resolutions is not limited to MAC addresses 
    - provides IP and port name reasolutions
    - **not** enbaled by default
    - ** Edit - Preferences - Name Resolution menu**
        - enable resolve transport names
        - enable resolve netowrk IP addresses

![Name resolution menu enable IP and transport names](screenshots/wireshark_nameresolutionmenu.png)

Endpoint menu with name resolution

![Endpoint menu view with name resoution](screenshots/wireshark_endpointmenu.png)

Wireshark also provides IP and geolocation mapping that helps analysts identify the maps source and destinatio addresses
    - not activated by default
    - needs supplemental data like the GeoIP database 
    - must download and indicate path 

![](screenshots/wireshark_geoiplocation.png)    

Example of Endpoints and GeoIP view

![](screenshots/wireshark_endpointsgeoip)

### Task Questions

- What is the IP address of the hostname that starts with "bbc"?
    
    - **Answer:**
- What is the number of IPv4 conversations?
    
    - **Answer:**
- How many bytes (k) were transferred from the "Micro-St" MAC address?
    
    - **Answer:**
- What is the number of IP addresses linked with "Kansas City"?
    
    - **Answer:**
- Which IP address is linked with "Blicnet" AS Organisation?
    
    - **Answer:**

---

## Task 3 - Statistics | Protocol Details

### Key Concepts

<!-- IPv4 and IPv6 Statistics: narrows statistics to a specific IP version; Statistics > IPvX Statistics --> <!-- DNS statistics: tree view of DNS packets; rcodes, opcodes, query types, service stats; Statistics > DNS --> <!-- HTTP statistics: tree view of HTTP packets; request and response codes, original requests; Statistics > HTTP -->

### Task Questions

- What is the most used IPv4 destination address?
    
    - **Answer:**
- What is the max service request-response time of the DNS packets?
    
    - **Answer:**
- What is the number of HTTP Requests accomplished by "rad[.]msn[.]com"?
    
    - **Answer:**

---

## Task 4 - Packet Filtering | Principles

### Key Concepts

<!-- Two filter types: Capture Filters (set before capture, not changeable) vs Display Filters (changeable during capture) --> <!-- Capture filter syntax: Scope (host/net/port/portrange), Direction (src/dst), Protocol (ether/ip/tcp/udp) --> <!-- Example capture filter: tcp port 80 --> <!-- Display filter syntax: supports 3000+ protocols; example: tcp.port == 80 -->

#### Comparison Operators

|English|C-Like|Description|
|---|---|---|
|eq|==|Equal|
|ne|!=|Not equal|
|gt|>|Greater than|
|lt|<|Less than|
|ge|>=|Greater than or equal to|
|le|<=|Less than or equal to|

#### Logical Operators

|English|C-Like|Description|
|---|---|---|
|and|&&|Logical AND|
|or|\||Logical OR|
|not|!|Logical NOT|

<!-- Filter toolbar: green = valid, red = invalid, yellow = warning (works but unreliable) --> <!-- Note: use !(value) style rather than !=value for consistent results -->

---

## Task 5 - Packet Filtering | Protocol Filters

### Key Concepts

#### IP Filters

|Filter|Description|
|---|---|
|`ip`|Show all IP packets|
|`ip.addr == 10.10.10.111`|All packets containing this IP|
|`ip.addr == 10.10.10.0/24`|All packets in this subnet|
|`ip.src == 10.10.10.111`|Packets originating from this IP|
|`ip.dst == 10.10.10.111`|Packets sent to this IP|

<!-- ip.addr is direction-agnostic; ip.src/ip.dst filter by direction -->

#### TCP and UDP Filters

|Filter|Description|
|---|---|
|`tcp.port == 80`|All TCP packets on port 80|
|`tcp.srcport == 1234`|TCP packets from port 1234|
|`tcp.dstport == 80`|TCP packets to port 80|
|`udp.port == 53`|All UDP packets on port 53|
|`udp.srcport == 1234`|UDP packets from port 1234|
|`udp.dstport == 5353`|UDP packets to port 5353|

#### HTTP and DNS Filters

|Filter|Description|
|---|---|
|`http`|All HTTP packets|
|`http.response.code == 200`|HTTP 200 responses|
|`http.request.method == "GET"`|HTTP GET requests|
|`http.request.method == "POST"`|HTTP POST requests|
|`dns`|All DNS packets|
|`dns.flags.response == 0`|DNS requests|
|`dns.flags.response == 1`|DNS responses|
|`dns.qry.type == 1`|DNS A records|

<!-- Display Filter Expression menu (Analyse > Display Filter Expression): shows all fields, value types, and predefined values for any protocol -->

### Task Questions

- What is the number of IP packets?
    
    - **Answer:**
- What is the number of packets with a "TTL value less than 10"?
    
    - **Answer:**
- What is the number of packets which use "TCP port 4444"?
    
    - **Answer:**
- What is the number of "HTTP GET" requests sent to port "80"?
    
    - **Answer:**
- What is the number of type A DNS Queries?
    
    - **Answer:**

---

## Task 6 - Advanced Filtering

### Key Concepts

#### Advanced Operators and Functions

|Filter|Type|Description|Example|
|---|---|---|---|
|`contains`|Comparison|Case-sensitive substring search on a specific field|`http.server contains "Apache"`|
|`matches`|Comparison|Case-insensitive regex pattern search|`http.host matches "\.(php\|html)"`|
|`in`|Set membership|Match against a set of values|`tcp.port in {80 443 8080}`|
|`upper()`|Function|Convert string field to uppercase before comparison|`upper(http.server) contains "APACHE"`|
|`lower()`|Function|Convert string field to lowercase before comparison|`lower(http.server) contains "apache"`|
|`string()`|Function|Convert non-string field to string|`string(frame.number) matches "[13579]$"`|

<!-- Bookmarks: save filters for reuse via the star icon in the filter toolbar --> <!-- Filter buttons: one-click apply for saved filters --> <!-- Profiles: Edit > Configuration Profiles or status bar > Profile; use separate profiles per investigation case for different coloring rules and filter buttons -->

### Task Questions

- Find all Microsoft IIS servers. What is the number of packets that did not originate from "port 80"?
    
    - **Answer:**
- Find all Microsoft IIS servers. What is the number of packets that have "version 7.5"?
    
    - **Answer:**
- What is the total number of packets that use ports 3333, 4444 or 9999?
    
    - **Answer:**
- What is the number of packets with "even TTL numbers"?
    
    - **Answer:**
- Change the profile to "Checksum Control". What is the number of "Bad TCP Checksum" packets?
    
    - **Answer:**
- Use the existing filtering button to filter the traffic. What is the number of displayed packets?
    
    - **Answer:**

---

## Task 7 - Conclusion

### Key Concepts

<!-- Next room: Wireshark: Traffic Analysis - investigating suspicious traffic activities -->
