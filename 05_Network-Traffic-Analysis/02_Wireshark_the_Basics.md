---
title: Wireshark The Basics
module: Network Traffic Analysis
path: SOC Level 1
platform: TryHackMe
tags: [Wireshark, packet-analysis, PCAP, display-filters, packet-dissection, OSI, SOC]
status: in-progress
date: 2026-04-27
date_completed:
---


*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7)*

## Task 1 — Introduction

### Key Concepts
<!-- Define Wireshark: open-source, cross-platform, PCAP analysis and live traffic sniffing -->
<!-- Clarify what Wireshark is NOT (not an IDS, does not modify packets) -->
<!-- Two files: http1.pcapng for walkthrough, Exercise.pcapng for questions -->

### Task Questions

- **Which file is used to simulate the screenshots?**
  - **Answer:**

- **Which file is used to answer the questions?**
  - **Answer:**

---

## Task 2 — Tool Overview

### Key Concepts

<!-- Three use cases: troubleshooting, security anomaly detection, protocol investigation -->

#### Wireshark GUI Sections

| Section | Purpose |
|---|---|
| Toolbar | |
| Display Filter Bar | |
| Recent Files | |
| Capture Filter and Interfaces | |
| Status Bar | |

#### Packet Detail Panes

| Pane | What It Shows |
|---|---|
| Packet List Pane | |
| Packet Details Panel | |
| Packet Bytes Pane | |

<!-- Packet colouring: temporary (session only) vs permanent (saved to profile) -->
<!-- Traffic sniffing: blue button = start, red = stop, green = restart -->
<!-- Merge PCAPs: File > Merge; must save merged file before working on it -->
<!-- View file details: Statistics > Capture File Properties (hash, capture time, interface, stats) -->

### Task Questions

- **Read the "capture file comments". What is the flag?**
  - **Answer:**

- **What is the total number of packets?**
  - **Answer:**

- **What is the SHA256 hash value of the capture file?**
  - **Answer:**

---

## Task 3 — Packet Dissection

### Key Concepts

<!-- Packet dissection = protocol dissection: decoding available protocols and fields layer by layer -->

#### OSI Layers in Wireshark Packet Details

| Layer | Wireshark Label | What It Shows |
|---|---|---|
| Layer 1 | Frame / Packet | |
| Layer 2 | Source MAC | |
| Layer 3 | Source IP | |
| Layer 4 | Protocol | |
| Layer 4 cont. | Protocol Errors | |
| Layer 5 | Application Protocol | |
| Layer 5 ext. | Application Data | |

<!-- Clicking a detail in the details pane highlights the corresponding bytes in the bytes pane -->

### Task Questions

- **View packet number 38. Which markup language is used under the HTTP protocol?**
  - **Answer:**

- **What is the arrival date of the packet? (Month/Day/Year)**
  - **Answer:**

- **What is the TTL value?**
  - **Answer:**

- **What is the TCP payload size?**
  - **Answer:**

- **What is the e-tag value?**
  - **Answer:**

---

## Task 4 — Packet Navigation

### Key Concepts

<!-- Packet numbers: unique ID assigned to each packet; helps navigate large captures -->
<!-- Go to Packet: via Go menu or toolbar; supports in-frame tracking and next packet in conversation -->
<!-- Find Packets: Edit > Find Packet; four input types: Display filter, Hex, String, Regex -->
<!-- Search field matters: list pane vs details pane vs bytes pane return different results -->
<!-- Mark Packets: shown in black; lost after closing the capture file (session only) -->
<!-- Packet Comments: persist in the capture file until removed; useful for analyst handoffs -->
<!-- Export Packets: File menu; share only relevant/suspicious packets -->
<!-- Export Objects: extract transferred files from streams; supported protocols: DICOM, HTTP, IMF, SMB, TFTP -->
<!-- Time Display Format: default = Seconds Since Beginning of Capture; UTC preferred for analysis -->

#### Expert Info Severity Levels

| Severity | Colour | Meaning |
|---|---|---|
| Chat | Blue | |
| Note | Cyan | |
| Warn | Yellow | |
| Error | Red | |

#### Expert Info Groups

| Group | Info |
|---|---|
| Checksum | |
| Comment | |
| Deprecated | |
| Malformed | |

### Task Questions

- **Search the "r4w" string in packet details. What is the name of artist 1?**
  - **Answer:**

- **Go to packet 12 and read the packet comments. What is the answer?**
  - **Answer:**

- **There is a ".txt" file inside the capture file. Find the file and read it; what is the alien's name?**
  - **Answer:**

- **Look at the expert info section. What is the number of warnings?**
  - **Answer:**

---

## Task 5 — Packet Filtering

### Key Concepts

<!-- Two filter types: capture filters (limit what is captured) vs display filters (limit what is shown) -->
<!-- Golden rule: "If you can click on it, you can filter and copy it" -->

#### Filtering Methods

| Method | How It Works | Use Case |
|---|---|---|
| Apply as Filter | Right-click a field value; filters immediately | Quick single-value filter |
| Conversation Filter | Filters all packets linked by IP and port | Isolate a full conversation |
| Colourise Conversation | Highlights linked packets without reducing packet count | Visual tracking without filtering |
| Prepare as Filter | Adds query to filter bar without applying it | Build complex filters before executing |
| Apply as Column | Adds a field as a visible column in packet list | Track a value across all packets |
| Follow Stream | Reconstructs application-level data from a stream | Read HTTP/TCP/UDP data as it appeared |

<!-- Follow Stream: server traffic = blue, client traffic = red -->
<!-- Remove stream filter: click X on the right side of the display filter bar -->

#### Common Display Filter Syntax

| Filter | Syntax Example |
|---|---|
| By protocol name | |
| By port (TCP) | |
| By port (UDP) | |
| By IP address | |

### Task Questions

- **Go to packet 4. Apply "Hypertext Transfer Protocol" as a filter. What is the filter query?**
  - **Answer:**

- **What is the number of displayed packets?**
  - **Answer:**

- **Go to packet 33790, follow the HTTP stream. What is the total number of artists?**
  - **Answer:**

- **What is the name of the second artist?**
  - **Answer:**

---

## Task 6 — Conclusion

### Key Concepts
<!-- Wireshark basics complete: navigation, dissection, filtering, packet export -->
<!-- Next room: Wireshark Packet Operations -- deeper packet investigation -->

### Task Questions

- **Proceed to the next room and keep learning!**
  - **Answer:** No answer needed