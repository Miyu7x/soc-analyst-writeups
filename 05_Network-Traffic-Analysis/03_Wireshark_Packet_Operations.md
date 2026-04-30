---
title: "Wireshark: Packet Operations"
module: "02_Core-SOC-Solutions"
path: "SOC Level 1 > Network Traffic Analysis"
platform: TryHackMe
tags: [wireshark, packet-analysis, statistics, display-filters, protocol-filters, advanced-filtering, network-traffic]
status: complete
date: 2026-04-28
date_completed: 2026-04-30
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

---

## Task 1 - Introduction

### Key Concepts

Second room in the Wireshark trio. Focuses on packet-level investigation using statistics, protocol details, and advanced display filters.

Prerequisites: Wireshark: The Basics, Networking Fundamentals.

---

## Task 2 - Statistics | Summary

### Key Concepts

The Statistics menu provides multiple options for building an investigation hypothesis before diving into individual packets.

**Statistics > Resolved Addresses**
- Lists IP addresses and DNS hostnames found in the capture
- Hostname info is pulled from DNS answers in the pcap
- Useful for quickly identifying accessed resources

**Statistics > Protocol Hierarchy**
- Tree view of all protocols in the capture with packet counters and percentages
- Shows overall port and service usage
- Right-click any row to apply as a display filter

**Statistics > Conversations**
- Traffic between two specific endpoints
- Five formats: Ethernet, IPv4, IPv6, TCP, UDP

**Statistics > Endpoints**
- Similar to Conversations but shows unique per-field information
- Name Resolution button (lower-left) resolves known MAC addresses to manufacturer names
- IP and port name resolution available but not enabled by default
  - Enable via: Edit > Preferences > Name Resolution > resolve transport names / resolve network IP addresses
- GeoIP mapping available but requires MaxMind DB files
  - Set path via: Edit > Preferences > Name Resolution > MaxMind database directories

> **Version note:** Older Wireshark versions (THM VM) do not have a search bar in the Endpoints window. Workaround: scroll manually using the Country column, or Google the organization name first to know what to look for.

![Resolved addresses example](screenshots/wireshark_resolvedaddresses.png)
![Protocol hierarchy example](screenshots/wireshark_protocolhierarchy.png)
![Conversations example](screenshots/wireshark_conversation.png)
![Endpoints example](screenshots/wireshark_endpoints.png)
![Name resolution menu](screenshots/wireshark_nameresolutionmenu.png)
![Endpoint menu with name resolution](screenshots/wireshark_endpointmenu.png)
![GeoIP location](screenshots/wireshark_geoiplocation.png)
![Endpoints GeoIP view](screenshots/wireshark_endpointsgeoip.png)

### Task Questions

- What is the IP address of the hostname that starts with "bbc"?

  ![](screenshots/wireshark_bbc.png)
  - **Answer: 199.232.24.81**

- What is the number of IPv4 conversations?

  ![](screenshots/wireshark_ipv4conversations.png)
  - **Answer: 435**

- How many bytes (k) were transferred from the "Micro-St" MAC address?

  *Found Micro-ST under non-resolved addresses, then checked Endpoints for the byte count.*

  ![](screenshots/wireshark_findmicrost.png)
  ![](screenshots/wireshark_endpointbytes.png)
  - **Answer: 7474**

- What is the number of IP addresses linked with "Kansas City"?

  ![](screenshots/wireshark_kansascity.png)
  - **Answer: 4**

- Which IP address is linked with "Blicnet" AS Organisation?

  *No search bar in this Wireshark version. Googled Blicnet to confirm it is a Bosnian ISP, then located it in Endpoints using the Country column.*

  ![](screenshots/wireshark_bosnia.png)
  ![](screenshots/wireshark_blicnet.png)
  - **Answer: 188.246.82.7**

---

## Task 3 - Statistics | Protocol Details

### Key Concepts

**Statistics > IPv4 Statistics / IPv6 Statistics**
- Narrows statistics to a specific IP version
- Destinations and Ports sub-view splits traffic by direction -- use this when a question asks for the top *destination* specifically, not the top overall address

**Statistics > DNS**
- Tree view of all DNS packets with rcodes, opcodes, query types, and service stats

**Statistics > HTTP**
- Tree view of HTTP packets with request and response codes and original request paths
- For targeted queries, a display filter is faster than scrolling the tree:
  `http.host == "rad.msn.com" && http.request.method == "GET"`

### Task Questions

- What is the most used IPv4 destination address?

  *Statistics > IPv4 Statistics > Destination and Ports -- not All Addresses, which ranks by combined traffic in both directions. The top result in All Addresses is the biggest overall talker, not necessarily the top destination.*

  ![](screenshots/wireshark_ipv4address.png)
  - **Answer: 10.100.1.33**

- What is the max service request-response time of the DNS packets?

  ![](screenshots/wireshark_dnstime.png)
  - **Answer: 0.467897**

- What is the number of HTTP Requests accomplished by "rad[.]msn[.]com"?

  *HTTP Statistics tree works but display filter is faster and more precise.*

  `http.host == "rad.msn.com" && http.request.method == "GET"`

  ![](screenshots/wireshark_httpnumber.png)
  ![](screenshots/wireshark_httpgetfilter.png)
  - **Answer: 39**

---

## Task 4 - Packet Filtering | Principles

### Key Concepts

Two filter types in Wireshark:

- **Capture Filters** -- set before capture, cannot be changed during capture, saves only specific traffic
- **Display Filters** -- applied during or after capture, reduces visible packets for investigation, supports 3000+ protocols

Capture filter syntax uses byte offsets, hex values, and boolean operators. Display filter syntax uses protocol dot-notation and is far more powerful.

#### Comparison Operators

| English | C-Like | Description | Example |
|---------|--------|-------------|---------|
| eq | == | Equal | `ip.src == 10.10.10.100` |
| ne | != | Not equal -- **deprecated**, use `!()` instead | `!(ip.src == 10.10.10.100)` |
| gt | > | Greater than | `ip.ttl > 250` |
| lt | < | Less than | `ip.ttl < 10` |
| ge | >= | Greater than or equal to | `ip.ttl >= 0xFA` |
| le | <= | Less than or equal to | `ip.ttl <= 0xA` |

> **Why avoid `!=`:** For fields that can have multiple values in one packet (like ports), `!=` returns true if *any* value does not match -- which can include packets that do have the unwanted value. `!(field == value)` negates the entire expression cleanly and is always the safer choice. Wireshark flags `!=` with a yellow warning for this reason.

#### Logical Operators

| English | C-Like | Description | Example |
|---------|--------|-------------|---------|
| and | && | Logical AND | `ip.src == 10.10.10.100 && tcp.port == 80` |
| or | \|\| | Logical OR | `ip.src == 10.10.10.100 \|\| ip.src == 10.10.10.111` |
| not | ! | Logical NOT | `!(ip.src == 10.10.10.222)` |

Filter toolbar color codes: green = valid, red = invalid, yellow = warning (works but unreliable, rewrite it).

---

## Task 5 - Packet Filtering | Protocol Filters

### Key Concepts

#### IP Filters

| Filter | Description |
|--------|-------------|
| `ip` | All IP packets |
| `ip.addr == 10.10.10.111` | All packets containing this IP (either direction) |
| `ip.addr == 10.10.10.0/24` | All packets in this subnet |
| `ip.src == 10.10.10.111` | Packets originating from this IP |
| `ip.dst == 10.10.10.111` | Packets sent to this IP |

> `ip.addr` is direction-agnostic. Use `ip.src` / `ip.dst` when direction matters.

#### TCP and UDP Filters

| Filter | Description |
|--------|-------------|
| `tcp.port == 80` | All TCP packets on port 80 |
| `tcp.srcport == 1234` | TCP packets originating from port 1234 |
| `tcp.dstport == 80` | TCP packets sent to port 80 |
| `udp.port == 53` | All UDP packets on port 53 |
| `udp.srcport == 1234` | UDP packets originating from port 1234 |
| `udp.dstport == 5353` | UDP packets sent to port 5353 |

> HTTP always runs over TCP. Filtering `http` already implies TCP -- adding `tcp` is redundant unless narrowing by port specifically. HTTP/3 is the exception (runs over QUIC/UDP) but Wireshark treats it as a separate protocol.

#### HTTP and DNS Filters

| Filter | Description |
|--------|-------------|
| `http` | All HTTP packets |
| `http.response.code == 200` | HTTP 200 responses |
| `http.request.method == "GET"` | HTTP GET requests |
| `http.request.method == "POST"` | HTTP POST requests |
| `dns` | All DNS packets |
| `dns.flags.response == 0` | DNS requests only |
| `dns.flags.response == 1` | DNS responses only |
| `dns.qry.type == 1` | DNS type A records |

> Display Filter Expression menu (Analyze > Display Filter Expression) lists all supported protocol fields, accepted value types, and predefined values -- useful when you do not know the exact field name.

### Task Questions

- What is the number of IP packets?

  `ip`

  ![](screenshots/wireshark_ippackets.png)
  - **Answer: 81420**

- What is the number of packets with a "TTL value less than 10"?

  `ip.ttl < 10`

  ![](screenshots/wiresharkLlessthan10.png)
  - **Answer: 66**

- What is the number of packets which use "TCP port 4444"?

  `tcp.port == 4444`

  ![](screenshots/wireshark_tcp4444.png)
  - **Answer: 632**

- What is the number of "HTTP GET" requests sent to port "80"?

  `tcp.port == 80 && http.request.method == "GET"`

  ![](screenshots/wireshark_httptcp80.png)
  - **Answer: 527**

- What is the number of type A DNS Queries?

  `dns.qry.type == 1 && dns.flags.response == 0 && !llmnr`

  *THM hint only mentioned filtering LLMNR but did not include `dns.flags.response == 0` in the question. Without the response flag you get both queries and responses (106 packets). Without the LLMNR exclusion the count is still 3 digits. The full filter above is required to reach the expected answer. In Windows environments LLMNR noise is almost always present in a pcap -- excluding it when counting DNS queries is standard practice.*

  ![](screenshots/wireshark_dnsbadquestion.png)
  - **Answer: 51**

---

## Task 6 - Advanced Filtering

### Key Concepts

#### Advanced Operators and Functions

| Filter | Type | Description | Example |
|--------|------|-------------|---------|
| `contains` | Comparison | Case-sensitive substring search on a specific field | `http.server contains "Apache"` |
| `matches` | Comparison | Case-insensitive regex pattern search | `http.host matches "\.(php\|html)"` |
| `in` | Set membership | Match a field against a set of values | `tcp.port in {80 443 8080}` |
| `upper()` | Function | Convert field to uppercase before comparison | `upper(http.server) contains "APACHE"` |
| `lower()` | Function | Convert field to lowercase before comparison | `lower(http.server) contains "apache"` |
| `string()` | Function | Convert non-string field to string for regex matching | `string(frame.number) matches "[13579]$"` |

> **`contains` is case-sensitive.** If unsure of capitalization, wrap in `lower()` and search lowercase. `lower()` and `upper()` only normalize case -- they do not change other characters. `"Microsoft-IIS"` requires the hyphen regardless of case wrapping because the hyphen is part of the actual server header value, not a syntax choice.

> **When you do not know the exact server string:** Start broad with `lower(http.server) contains "microsoft"`, click a result packet, expand HTTP in the packet details pane under the response, and read the exact Server field value. Then tighten the filter. This approach works for any unknown field value.

> **`matches` vs `contains`:** Use `contains` for a fixed substring. Use `matches` when you need a regex pattern such as multiple options, wildcards, or anchors. Example with odd frame numbers: `string(frame.number) matches "[13579]$"`

> **`string()` use case:** Converts numeric fields like `ip.ttl` or `frame.number` into strings so regex operators can be applied. Example for even TTL numbers: `string(ip.ttl) matches "[02468]$"` -- the `$` anchors to the last digit.

> **`in` operator use case:** Replaces chaining multiple `||` conditions. `tcp.port in {3333 4444 9999}` is cleaner and less error-prone than `tcp.port == 3333 || tcp.port == 4444 || tcp.port == 9999`.

**Bookmarks and Filter Buttons**
- Save frequently used filters as bookmarks via the star icon in the filter toolbar
- Create one-click filter buttons for fast reuse during investigations

**Profiles**
- Edit > Configuration Profiles (or click Profile in the status bar)
- Create separate profiles per investigation type with their own coloring rules and filter buttons
- Switch profiles without losing your default setup

### Task Questions

- Find all Microsoft IIS servers. What is the number of packets that did not originate from "port 80"?

  `lower(http.server) contains "microsoft-iis" && !(tcp.port == 80)`

  *`contains` is case-sensitive so `lower()` normalizes the field first. The hyphen is required -- it is part of the actual server header string. Using `!(tcp.port == 80)` instead of `!= 80` avoids the deprecated operator inconsistency.*

  ![](screenshots/wireshark_microsoftiis.png)
  - **Answer: 21**

- Find all Microsoft IIS servers. What is the number of packets that have "version 7.5"?

  `http.server matches "7.5"`

  ![](screenshots/wireshark_serverversion.png)
  - **Answer: 71**

- What is the total number of packets that use ports 3333, 4444 or 9999?

  `tcp.port in {3333 4444 9999}`

  ![](screenshots/wireshark_tcpportin.png)
  - **Answer: 2235**

- What is the number of packets with "even TTL numbers"?

  `string(ip.ttl) matches "[02468]$"`

  *`ip.ttl` is numeric. `string()` converts it so `matches` can apply a regex. `[02468]$` anchors to the last digit -- any TTL ending in an even digit.*

  ![](screenshots/wireshark_stringconversion.png)
  - **Answer: 77289**

- Change the profile to "Checksum Control". What is the number of "Bad TCP Checksum" packets?

  `tcp.checksum.status == "Bad"`

  *Switch profile via Edit > Configuration Profiles > Checksum Control. Bad checksum packets appear highlighted in red. The checksum status field is visible in the TCP layer of the packet details pane. Confirmed by `TCP CHECKSUM INCORRECT` label in the status bar when clicking a flagged packet.*

  ![](screenshots/wireshark_badchecksum.png)
  - **Answer: 34185**

- Use the existing filtering button to filter the traffic. What is the number of displayed packets?

  *The Checksum Control profile includes a pre-built filter button. Click it to apply.*

  ![](screenshots/wireshark_bookmarked.png)
  - **Answer: 261**

---

## Task 7 - Conclusion

### Key Concepts

Next room: Wireshark: Traffic Analysis -- applying statistics and filters to investigate suspicious traffic activity in realistic scenarios.
