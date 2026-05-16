<img src=screenshots/snort_bar.png width="600">

---
title: Snort
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [snort, IDS, IPS, NIDS, NIPS, packet-logging, pcap, rules, sniffer-mode]
status: in-progress
date: 2026-05-16
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

<img src=screenshots/snort_logo.png width="600">

### Key Concepts

<!-- Q: What kind of system is Snort and who maintains it? -->
<!-- Q: What are Snort's three primary use cases according to the official description? -->

### Task Questions

**1. Read the task above.**

- **Answer:** N/A

---

## Task 2 - Interactive Material and VM

### Key Concepts

<!-- Q: Where are all exercise files located on the VM desktop? -->
<!-- Q: What is the traffic-generator script used for and how do you run it? -->

### Task Questions

**1. Navigate to the Task-Exercises folder and run the command "./.easy.sh" and write the output.**

- **Answer:**

---

## Task 3 - Introduction to IDS/IPS

### Key Concepts

<!-- Q: What is the core difference between IDS and IPS in terms of how they respond to threats? -->
<!-- Q: What does NIDS monitor vs HIDS? -->
<!-- Q: What makes NBA different from NIPS, and what is the baselining risk? -->
<!-- Q: What are the three detection/prevention techniques and how does each work? -->

### IDS Types

| Type | Scope | Action |
|------|-------|--------|
| NIDS | Entire subnet | Generates alert |
| HIDS | Single endpoint | Generates alert |

### IPS Types

| Type | Scope | Action |
|------|-------|--------|
| NIPS | Entire subnet | Terminates connection |
| NBA | Entire subnet | Terminates connection; requires baselining period |
| WIPS | Wireless network | Terminates connection |
| HIPS | Single endpoint | Terminates connection |

### Detection Techniques

| Technique | Approach | Detects Unknown Threats? |
|-----------|----------|--------------------------|
| Signature-Based | Matches known malicious patterns | No |
| Behaviour-Based | Compares normal vs abnormal behaviour | Yes |
| Policy-Based | Compares activity against security policies | No |

### Task Questions

**1. Which IDS or IPS type can help you stop the threats on a local machine?**

- **Answer:**

---

**2. Which IDS or IPS type can help you detect threats on a local network?**

- **Answer:**

---

**3. Which IDS or IPS type can help you detect the threats on a local machine?**

- **Answer:**

---

**4. Which IDS or IPS type can help you stop the threats on a local network?**

- **Answer:**

---

**5. Which described solution works by detecting anomalies in the network?**

- **Answer:**

---

**6. According to the official description of Snort, what kind of NIPS is it?**

- **Answer:**

---

**7. NBA training period is also known as...**

- **Answer:**

---

## Task 4 - First Interaction with Snort

### Key Concepts

<!-- Q: What flag tests a Snort configuration file without processing live traffic? -->
<!-- Q: What does -q do and when would you use it? -->
<!-- Q: What is the significance of being able to point to multiple configuration files with -c? -->

### Parameters

| Parameter | Description |
|-----------|-------------|
| `-V` | Show instance version and build number |
| `-c` | Identify the configuration file to use |
| `-T` | Test the configuration file |
| `-q` | Quiet mode -- suppresses default banner and startup info |

### Task Questions

**1. Run the Snort instance and check the build number.**

- **Answer:**

---

**2. Test the current instance with "/etc/snort/snort.conf" file and check how many rules are loaded with the current build.**

- **Answer:**

---

**3. Test the current instance with "/etc/snort/snortv2.conf" file and check how many rules are loaded with the current build.**

- **Answer:**

---

## Task 5 - Operation Mode 1: Sniffer Mode

### Key Concepts

<!-- Q: What does each sniffer flag show: -v, -d, -e, -X? -->
<!-- Q: What is the difference in output detail between -v alone and -X? -->
<!-- Q: Can sniffer mode flags be combined? -->

### Sniffer Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-v` | Verbose -- display TCP/IP output in console |
| `-d` | Display packet data (payload) |
| `-e` | Display link-layer headers (MAC addresses, frame type) |
| `-X` | Display full packet details in HEX |
| `-i` | Specify a network interface to sniff |

### Task Questions

**1. You can practice the parameter combinations by using the traffic-generator script.**

- **Answer:** N/A

---

## Task 6 - Operation Mode 2: Packet Logger Mode

### Key Concepts

<!-- Q: What is the default log output format when using -l and what tools can read it? -->
<!-- Q: What changes when you add -K ASCII and what is the tradeoff vs binary format? -->
<!-- Q: Why does Snort need root/sudo to sniff and how does that affect log file ownership? -->
<!-- Q: How do BPF filters work with the -r parameter? Give an example. -->

### Packet Logger Parameters

| Parameter | Description |
|-----------|-------------|
| `-l` | Logger mode -- set target log directory (default: /var/log/snort) |
| `-K ASCII` | Log packets in ASCII format instead of binary |
| `-r` | Read and replay a saved log file |
| `-n` | Stop after processing N packets |

### Log Format Comparison

| Format | Human-Readable? | Readable by Snort? | Readable by tcpdump/Wireshark? |
|--------|----------------|-------------------|-------------------------------|
| Binary (default) | No | Yes | Yes |
| ASCII (-K ASCII) | Yes | No | No |

### Task Questions

**1. Navigate to folder "145.254.160.237". What is the source port used to connect port 53?**

- **Answer:**

---

**2. Read the snort.log file with Snort; what is the IP ID of the 10th packet?**

- **Answer:**

---

**3. Read the "snort.log.1640048004" file with Snort; what is the referer of the 4th packet?**

- **Answer:**

---

**4. Read the "snort.log.1640048004" file with Snort; what is the Ack number of the 8th packet?**

- **Answer:**

---

**5. Read the "snort.log.1640048004" file with Snort; what is the number of the "TCP port 80" packets?**

- **Answer:**

---

## Task 7 - Operation Mode 3: IDS/IPS Mode

### Key Concepts

<!-- Q: What is the difference between -A console and -A cmg output? -->
<!-- Q: Which alert modes produce console output and which write to file only? -->
<!-- Q: What does -A none do differently from -N? -->
<!-- Q: What parameters activate IPS inline mode and why does it need two interfaces? -->

### IDS/IPS Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-c` | Define the configuration file |
| `-T` | Test the configuration file |
| `-N` | Disable logging |
| `-D` | Background (daemon) mode |
| `-A` | Set alert mode |

### Alert Modes

| Mode | Console Output? | Detail Level |
|------|----------------|--------------|
| `console` | Yes | Fast style -- basic header and rule info |
| `cmg` | Yes | Full header + payload in hex and text |
| `fast` | No | Summary to alert file -- timestamp, IPs, ports |
| `full` | No | All possible information to alert file |
| `none` | No | No alert file generated |

### Task Questions

**1. What is the number of the detected HTTP GET methods?**

- **Answer:**

---

**2. You can practice the rest of the parameters by using the traffic-generator script.**

- **Answer:** N/A

---

## Task 8 - Operation Mode 4: PCAP Investigation

### Key Concepts

<!-- Q: What is the difference between --pcap-list and --pcap-show? -->
<!-- Q: Why would you use --pcap-show when investigating multiple pcap files? -->
<!-- Q: What does running Snort against a pcap without a config file give you vs with one? -->

### PCAP Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-r` / `--pcap-single=` | Read a single pcap file |
| `--pcap-list=""` | Read multiple pcaps (space-separated) |
| `--pcap-show` | Label each pcap in console output during processing |

### Task Questions

**1. Investigate the mx-1.pcap file with the default configuration file. What is the number of the generated alerts?**

- **Answer:**

---

**2. How many TCP Segments are Queued?**

- **Answer:**

---

**3. How many "HTTP response headers" were extracted?**

- **Answer:**

---

**4. Investigate the mx-1.pcap file with the second configuration file. What is the number of the generated alerts?**

- **Answer:**

---

**5. Investigate the mx-2.pcap file with the default configuration file. What is the number of the generated alerts?**

- **Answer:**

---

**6. What is the number of the detected TCP packets?**

- **Answer:**

---

**7. Investigate the mx-2.pcap and mx-3.pcap files with the default configuration file. What is the number of the generated alerts?**

- **Answer:**

---

## Task 9 - Snort Rule Structure

### Key Concepts

<!-- Q: What are the four parts of a Snort rule header in order? -->
<!-- Q: What four protocols does Snort 2 support in rule headers? -->
<!-- Q: What is the difference between -> and <> direction operators? -->
<!-- Q: What SID range must user-created rules use? -->
<!-- Q: What does rev track and when must you update it? -->
<!-- Q: What is the difference between content, nocase, and fast_pattern? -->
<!-- Q: What does sameip detect and why would that be suspicious? -->

### Rule Actions

| Action | Description |
|--------|-------------|
| `alert` | Generate alert and log the packet |
| `log` | Log the packet only |
| `drop` | Block and log the packet |
| `reject` | Block, log, and terminate the packet session |

### Direction Operators

| Operator | Meaning |
|----------|---------|
| `->` | Source to destination only |
| `<>` | Bidirectional |

### General Rule Options

| Option | Description |
|--------|-------------|
| `msg` | Alert message shown in console/log |
| `sid` | Rule ID -- user rules must be >= 1,000,000 |
| `reference` | CVE or external link |
| `rev` | Revision number -- increment after every rule modification |

### Payload Detection Options

| Option | Description |
|--------|-------------|
| `content` | Match ASCII or HEX string in payload (case-sensitive) |
| `nocase` | Disable case sensitivity for content match |
| `fast_pattern` | Prioritise this content for initial match when multiple content options exist |

### Non-Payload Detection Options

| Option | Description |
|--------|-------------|
| `id` | Filter by IP ID field |
| `flags` | Filter by TCP flags: F (FIN) S (SYN) R (RST) P (PSH) A (ACK) U (URG) |
| `dsize` | Filter by packet payload size |
| `sameip` | Match when source IP equals destination IP |

### Task Questions

**1. Write a rule to filter IP ID "35369" and run it against the given pcap file. What is the request name of the detected packet?**

- **Answer:**

---

**2. Create a rule to filter packets with Syn flag and run it against the given pcap file. What is the number of detected packets?**

- **Answer:**

---

**3. Write a rule to filter packets with Push-Ack flags and run it against the given pcap file. What is the number of detected packets?**

- **Answer:**

---

**4. Create a rule to filter UDP packets with the same source and destination IP and run it against the given pcap file. What is the number of packets that show the same source and destination address?**

- **Answer:**

---

**5. An analyst modified an existing rule successfully. Which rule option must the analyst change after the implementation?**

- **Answer:**

---

## Task 10 - Snort2 Operation Logic: Points to Remember

### Key Concepts

<!-- Q: What are the five main components of Snort's architecture and what does each do? -->
<!-- Q: What are the three Snort rule tiers and what distinguishes each? -->
<!-- Q: What is the difference between PCAP and Afpacket DAQ modes? -->
<!-- Q: What are HOME_NET and EXTERNAL_NET used for in snort.conf? -->

### Snort Main Components

| Component | Role |
|-----------|------|
| Packet Decoder | Collects and prepares packets for pre-processing |
| Pre-processors | Arranges and modifies packets for the detection engine |
| Detection Engine | Applies rules, dissects and analyzes packets |
| Logging and Alerting | Generates logs and alerts |
| Outputs and Plugins | Output integrations and rule management plugins |

### Rule Tiers

| Tier | Cost | Registration | Update Frequency |
|------|------|-------------|-----------------|
| Community Rules | Free | Not required | - |
| Registered Rules | Free | Required | 30-day delay on subscriber rules |
| Subscriber Rules | Paid | Required | Twice weekly (Tue and Thu) |

### DAQ Modules

| Module | Mode | Description |
|--------|------|-------------|
| PCAP | Sniffer (default) | Passive packet capture |
| Afpacket | Inline IPS | Active inline mode for packet dropping |
| Dump | Testing | Testing mode for inline and normalisation |

### Key Config Variables

| Variable | Purpose |
|----------|---------|
| `HOME_NET` | The network being protected |
| `EXTERNAL_NET` | External network -- set to `any` or `!$HOME_NET` |
| `RULE_PATH` | Path to rule files |

### Task Questions

**1. Read the task above.**

- **Answer:** N/A

---

## Task 11 - Conclusion

### Task Questions

**1. Read the task above.**

- **Answer:** N/A
