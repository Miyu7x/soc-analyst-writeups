---
title: Snort
module: Network Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [snort, IDS, IPS, NIDS, NIPS, packet-logging, pcap, rules, sniffer-mode]
status: completed
date: 2026-05-16
date_completed: 2026-05-16
---

<img src=screenshots/snort_bar.png width="1800">

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

<p align="center">
<img src=screenshots/snort_logo.png width="400">
</p>

### Key Concepts

<!-- Q: What kind of system is Snort and who maintains it? -->
**Snort is a network IDS**
- NIDS/NIPS -- Network Intrusion Detection System and Network Intrusion Prevention System

<!-- Q: What are Snort's three primary use cases according to the official description? -->

### Task Questions

**1. Read the task above.**

- **Answer:** N/A

---

## Task 2 - Interactive Material and VM

### Key Concepts

<!-- Q: Where are all exercise files located on the VM desktop? -->
**Task Exercises folder on the desktop**
- Exercise files live under `/etc/snort`
- `traffic-generator.sh` simulates traffic for Snort since the VM is offline -- run with `sudo ./traffic-generator.sh`
- Wait until the end of script execution before stopping Snort

<!-- Q: What is the traffic-generator script used for and how do you run it? -->

### Task Questions

**1. Navigate to the Task-Exercises folder and run the command "./.easy.sh" and write the output.**

<img src=screenshots/snort_tooeasy.png width="400">

- **Answer: Too Easy!**

---

## Task 3 - Introduction to IDS/IPS

<img src=screenshots/snort_idsvsips.png width="400">

### Key Concepts

<!-- Q: What is the core difference between IDS and IPS in terms of how they respond to threats? -->
**Intrusion Detection System (IDS)**
- **Passive** monitoring solution for detecting malicious activities, abnormal incidents, and policy violations
- Generates alerts for each suspicious event
- 2 types:
  - **NIDS** -- monitors traffic flow across the entire subnet; if a signature is identified it creates an alert
  - **HIDS** -- monitors traffic flow from a single endpoint; produces an alert

**Intrusion Prevention System (IPS)**
- **Active** monitoring solution for stopping, preventing, and terminating malicious activities upon detection
- 4 types:
  - **NIPS** -- monitors network traffic flow; terminates the connection if a signature is identified
  - **NBA** (Network Behaviour Analysis) -- behaviour-based traffic monitoring; terminates connection if an anomaly is detected; compares known/normal behaviour with unknown/abnormal behaviour; helps detect previously unknown threats; requires a training period called **baselining**
  - **WIPS** (Wireless Intrusion Prevention System) -- monitors traffic flow from wireless networks
  - **HIPS** -- actively **protects** traffic flow from a single **endpoint** (local machine)

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

- **Answer: HIPS**

---

**2. Which IDS or IPS type can help you detect threats on a local network?**

- **Answer: NIDS**

---

**3. Which IDS or IPS type can help you detect the threats on a local machine?**

- **Answer: HIDS**

---

**4. Which IDS or IPS type can help you stop the threats on a local network?**

- **Answer: NIPS**

---

**5. Which described solution works by detecting anomalies in the network?**

- **Answer: NBA**

---

**6. According to the official description of Snort, what kind of NIPS is it?**

- **Answer: full-blown**

---

**7. NBA training period is also known as...**

- **Answer: baselining**

---

## Task 4 - First Interaction with Snort

### Key Concepts

<!-- Q: What flag tests a Snort configuration file without processing live traffic? -->
Snort can be started directly from the CLI
- `-V` -- info about your instance
- `-T` -- tests configuration
- `-c` -- identifies the config file (snort.conf)
- The configuration file holds rules, plugins, detection mechanisms, default actions, and output settings
- Only one config file can be used at runtime

<!-- Q: What does -q do and when would you use it? -->
`-q` suppresses the default Snort banner and initial setup info

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

<img src=screenshots/snort_149.png width="400">

- **Answer: 149**

---

**2. Test the current instance with "/etc/snort/snort.conf" file and check how many rules are loaded with the current build.**

<img src=screenshots/snort_totalrules.png width="400">

`sudo snort -c /etc/snort/snort.conf -T`

- **Answer: 4151**

---

**3. Test the current instance with "/etc/snort/snortv2.conf" file and check how many rules are loaded with the current build.**

<img src=screenshots/snort_1rule.png width="400">

- **Answer: 1**

---

## Task 5 - Operation Mode 1: Sniffer Mode

<img src=screenshots/snort_sniffer.png width="400">

### Key Concepts

<!-- Q: What does each sniffer flag show: -v, -d, -e, -X? -->
Just like tcpdump, Snort uses various flags to view different data about the packets it ingests.

Run `sudo ./traffic-generator.sh` to simulate traffic for Snort to sniff.

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

<img src=screenshots/snort_loggermode.png width="400">

### Key Concepts

Snort can act as a sniffer and log the sniffed packets via logger mode.
- Default log output directory is `/var/log/snort`
- Dumped in tcpdump format by default
- Read logs back with: `sudo tcpdump -r snort.log.1640048004 -ntc 10`

`sudo snort -dev -l .` will log traffic and save it in the current directory
- Log files are named like `snort.log.187636754` (timestamp suffix varies per capture)

<!-- Q: Why does Snort need root/sudo to sniff and how does that affect log file ownership? -->
Snort requires sudo privileges to run, so logged traffic is owned by root.

<!-- Q: How do BPF filters work with the -r parameter? Give an example. -->

**-K ASCII** logs packets in ASCII format
- Folders are named by IP address (e.g. `142.250.187.110`)
- Read into the IP folder with `sudo ls ./142.250.187.110/`
- Files inside are human-readable (e.g. `TCP:3371-80`, `UDP:3009-53`)
- Binary format is not human-readable; ASCII makes logs accessible without Snort

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

<img src=screenshots/snort_port53.png width="400">

- **Answer: 3009**

---

**2. Read the snort.log file with Snort; what is the IP ID of the 10th packet?**

<img src=screenshots/snort_first10.png width="400">

`sudo snort -r snort.log.1640048004 -n 10`
*Shows just the first 10 packets*

- **Answer: 49313**

---

**3. Read the "snort.log.1640048004" file with Snort; what is the referer of the 4th packet?**

<img src=screenshots/snort_referer.png width="400">

`sudo snort -Xr snort.log.1640048004 -n 4`
- `-X` -- show full packet details in HEX
- `-r` -- read packet mode
- `-n 4` -- first 4 packets only

- **Answer: http://www.ethereal.com/development.html**

---

**4. Read the "snort.log.1640048004" file with Snort; what is the Ack number of the 8th packet?**

<img src=screenshots/snort_tcp.png width="400">

`sudo snort -Xr snort.log.1640048004 tcp -n 8`
*Filtered by TCP since an ACK number points to the three-way handshake -- protocol filter goes after the log file name*

- **Answer: 0x38AFFFF3**

---

**5. Read the "snort.log.1640048004" file with Snort; what is the number of the "TCP port 80" packets?**

<img src=screenshots/snort_port53.png width="400">

`sudo snort -Xr snort.log.1640048004 'tcp and port 80'`

- **Answer: 41**

---

## Task 7 - Operation Mode 3: IDS/IPS Mode

<img src=screenshots/snort_operationmode.png width="400">

### Key Concepts

**Snort in IDS/IPS Mode**
- Allows traffic management using **user-defined rules**
- NIDS/NIPS depend on rule configurations
- `-A` sets the alert mode; default is `full` which shows all possible alert info
  - `fast` -- displays alert message, timestamp, source IP, destination IP, and port numbers
  - `console` -- fast style alerts on the console screen
  - `cmg` -- basic header details with payload in hex and text
  - `none` -- disables alerting
- `-N` disables logging entirely

**Example ICMP rule:**

`alert icmp any any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)`

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

<img src=screenshots/snort_getmethod.png width="400">

`sudo snort -c /etc/snort/snort.conf -A full -l .`
*Once you stop the instance the log summary appears in the console*

- **Answer: 2**

---

**2. You can practice the rest of the parameters by using the traffic-generator script.**

- **Answer:** N/A

---

## Task 8 - Operation Mode 4: PCAP Investigation

<img src=screenshots/snort_operation.png width="400">

### Key Concepts

**Snort PCAP read/investigate mode**
- Process pcap files with Snort
- Snort provides default traffic statistics with alerts depending on the ruleset

<!-- Q: What is the difference between --pcap-list and --pcap-show? -->
**`--pcap-list="filename.pcap filename2.pcap"`**
- Read multiple specific pcap files in one command

<!-- Q: Why would you use --pcap-show when investigating multiple pcap files? -->
**`--pcap-show`**
- Labels each pcap file in the output so you know which alerts came from which file -- essential when working with multiple pcaps

<!-- Q: What does running Snort against a pcap without a config file give you vs with one? -->

**Command breakdown:**

`sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show`

- `-c /etc/snort/snort.conf` -- use this config file
- `-q` -- quiet mode, no Snort startup banner
- `--pcap-list="icmp-test.pcap http2.pcap"` -- read these two pcap files
- `-A console` -- output alerts to the console in fast style
- `--pcap-show` -- labels each pcap file so alerts are traceable to their source

### PCAP Mode Parameters

| Parameter | Description |
|-----------|-------------|
| `-r` / `--pcap-single=` | Read a single pcap file |
| `--pcap-list=""` | Read multiple pcaps (space-separated) |
| `--pcap-show` | Label each pcap in console output during processing |

### Task Questions

**1. Investigate the mx-1.pcap file with the default configuration file. What is the number of the generated alerts?**

<img src=screenshots/snort_mx1pcap.png width="400">

- **Answer: 170**

---

**2. How many TCP Segments are Queued?**

<img src=screenshots/snort_tcpsegment.png width="400">

- **Answer: 18**

---

**3. How many "HTTP response headers" were extracted?**

<img src=screenshots/snort_httpheaders.png width="400">

- **Answer: 3**

---

**4. Investigate the mx-1.pcap file with the second configuration file. What is the number of the generated alerts?**

<img src=screenshots/snort_conf2.png width="400">

`sudo snort -c /etc/snort/snortv2.conf -A full -l . -r mx-1.pcap`

- **Answer: 68**

---

**5. Investigate the mx-2.pcap file with the default configuration file. What is the number of the generated alerts?**

<img src=screenshots/snort_defaultconf.png width="400">

`sudo snort -c /etc/snort/snort.conf -A full -l . -r mx-2.pcap`

- **Answer: 340**

---

**6. What is the number of the detected TCP packets?**

<img src=screenshots/snort_tcppackets.png width="400">

- **Answer: 82**

---

**7. Investigate the mx-2.pcap and mx-3.pcap files with the default configuration file. What is the number of the generated alerts?**

<img src=screenshots/snort_2pcaps.png width="400">

`sudo snort -c /etc/snort/snort.conf -A full -l . --pcap-list="mx-2.pcap mx-3.pcap"`

- **Answer: 1020**

---

## Task 9 - Snort Rule Structure

<img src=screenshots/snort_rules.png width="400">

### Key Concepts

<img src=screenshots/snort_rulesexample.png width="400">

Rules are required to detect sophisticated attacks.

<!-- Q: What are the four parts of a Snort rule header in order? -->
**Rule example:**

`alert icmp [192.168.1.0/24, 10.1.1.0/24] any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)`

**4 parts of a Snort rule header:**
- Action: `alert`, `log`, `drop`, `reject`
- Protocol
- Source
- Destination
- Rule options

<!-- Q: What four protocols does Snort 2 support in rule headers? -->
Snort 2 supports only 4 protocols: `IP`, `TCP`, `UDP`, `ICMP`

<!-- Q: What is the difference between -> and <> direction operators? -->
**Direction operator** indicates the traffic flow to be filtered by Snort
- `->` source to destination flow
- `<>` bidirectional flow

<!-- Q: What SID range must user-created rules use? -->
**Snort Rule IDs (SIDs) come with a predefined scope:**
- less than 100 -- reserved rules
- 100-999,999 -- rules that come with the build
- 1,000,000 and above -- rules created by the user

<!-- Q: What does rev track and when must you update it? -->
Each rule has a unique **rev** number
- Makes it easy for analysts to track rule history
- Acts as an indicator of how many times the rule has been modified

<!-- Q: What is the difference between content, nocase, and fast_pattern? -->
- `content` -- matches a specific payload pattern (e.g. HTTP GET)
- `nocase` -- disables case sensitivity for the content match
- `fast_pattern` -- prioritises content search to speed up payload inspection

<!-- Q: What does sameip detect and why would that be suspicious? -->
**`sameip`** filters for packets where the source and destination IP are identical -- a red flag for spoofed or loopback-style traffic

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
| `sid` | Rule ID -- user rules must be 1,000,000 or above |
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

<img src=screenshots/snort_newrule.png width="400">

`sudo gedit local.rules`

`alert ip any any <> any any (msg:"ID TEST"; id:35369; sid:100001; rev:1;)`

*Filtering by the IP ID field -- protocol set to `ip` since ID is an IP header field, not TCP specific*

- **Answer: TIMESTAMP REQUEST**

---

**2. Create a rule to filter packets with Syn flag and run it against the given pcap file. What is the number of detected packets?**

<img src=screenshots/snort_tcpflag.png width="400">

`alert tcp any any <> any any (msg:"FLAG TEST"; flags:S; sid:100001; rev:1;)`

- **Answer: 1**

---

**3. Write a rule to filter packets with Push-Ack flags and run it against the given pcap file. What is the number of detected packets?**

<img src=screenshots/snort_pushack.png width="400">

`alert tcp any any <> any any (msg:"FLAG TEST"; flags:PA; sid:100001; rev:1;)`

- **Answer: 216**

---

**4. Create a rule to filter UDP packets with the same source and destination IP and run it against the given pcap file. What is the number of packets that show the same source and destination address?**

<img src=screenshots/snort_sameip.png width="400">

`alert udp any any <> any any (msg:"SAME-IP TEST"; sameip; sid:100001; rev:1;)`

- **Answer: 7**

---

**5. An analyst modified an existing rule successfully. Which rule option must the analyst change after the implementation?**

- **Answer: rev**

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
