---
title: Splunk - Data Manipulation
module: Advanced Splunk
path: SOC Level 2
platform: TryHackMe
tags: [splunk, data-parsing, props-conf, transforms-conf, field-extraction, event-breaking, data-masking, pci-dss, regex]
status: In Progress
date: 2026-10-26
date_completed:
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/splunk4-intro.png width="1500">
</p>

---

# TASK 1 - Introduction

- This room is about what happens to data BEFORE I search it in Splunk
- Raw logs are useless until Splunk knows where each event starts and ends, and which pieces are fields
- Clean parsing is what lets me correlate across sources, spot anomalies, and build detections that actually fire
- Goals: learn how Splunk ingests and parses data, work with the core config files, extract and mask fields, and get accurate search results

---

# TASK 2 - Scenario Briefing

- I'm a SOC analyst at the MSSP Cybertees Ltd.
- A client needs event data from a custom source ingested into Splunk
- My job is to configure Splunk so it parses and transforms those logs correctly
- Four problems to fix:
  - Event breaking, so each event is split correctly
  - Multi-line parsing, for events that span more than one line
  - Masking sensitive data, to stay PCI DSS compliant
  - Field extraction, pulling out the fields the client cares about

---

## Lab Access

- Splunk lives in the default `/opt` directory
- The scripts I'll ingest are in `/home/ubuntu/Downloads/scripts`
- Switch to root with `sudo su` before doing anything
- If Splunk freezes, run `/opt/splunk/bin/splunk restart` as root and give it a few minutes

---

## Finn's Directions

1. Start the lab machine and wait for the split screen terminal
2. Switch to root:
   ```bash
   sudo su
   ```
3. Look at what lives in `/opt`:
   ```bash
   ls -la /opt
   ```
4. Look at the scripts folder. Use `-l` so you can see the permissions column. An `x` in it means the file is executable:
   ```bash
   ls -l /home/ubuntu/Downloads/scripts
   ```

---

## 1. What is the full file path to the Splunk directory?



<p align="center">
<img src=screenshots/splunk4-q1.png width="700">
</p>

**Answer:**

---

## 2. How many executables are available in the `/scripts` directory?



<p align="center">
<img src=screenshots/splunk4-q2.png width="700">
</p>

**Answer:**

---

# TASK 3 - How Splunk Processes Data

## Determining Your Format

- Splunk handles CSV, JSON, XML, and raw text
- Knowing the data's structure tells me which sourcetype to pick or build, so the right parsing rules get applied

---

## Identifying the Source Type

- `sourcetype` is one of the most important settings at ingestion
- It tells Splunk how to handle timestamps, line breaks, and field extractions for that data
- Splunk ships with built-in sourcetypes, and I can create custom ones

---

## Setting Up Inputs

- `inputs.conf` = WHAT to ingest, where it comes from, how to collect it, and its basic metadata (`sourcetype`, `host`, `source`)
- Two locations to know:
  - `/opt/splunk/etc/system/default` is Splunk's defaults. Don't edit, updates overwrite it
  - `/opt/splunk/etc/system/local` is where custom changes go. It takes priority over default
- Apps I create get their own config files too

```bash
[monitor:///path/to/your/data]
disabled = false
index = your_index
sourcetype = your_sourcetype
```

---

## Configuring Processing Properties

- `props.conf` = HOW to parse a specific sourcetype or source
- The stanza name matches the sourcetype set in `inputs.conf`

```bash
[your_sourcetype]
SHOULD_LINEMERGE = false
TIME_FORMAT = %Y-%m-%d %H:%M:%S
```

---

## Defining Field Extractions

- I can define custom field extractions right in `props.conf` with `EXTRACT-` plus a regex
- Next task covers a cleaner file for this

```bash
[your_sourcetype]
EXTRACT-fieldname1 = regular_expression1
EXTRACT-fieldname2 = regular_expression2
```

---

## Using the Splunk UI

- The web UI can do a lot of the same things: sourcetypes, field extractions, lookups, parsing options
- This room sticks to editing config files from the command line

---

## Finn's Directions

1. This is a reading task, no lab work
2. Both answers live in the Setting Up Inputs section above

---

## 1. Which configuration file would you edit to instruct Splunk on what data to ingest?



<p align="center">
<img src=screenshots/splunk4-q3.png width="700">
</p>

**Answer:**

---

## 2. What is the full path to Splunk's default configuration files?



<p align="center">
<img src=screenshots/splunk4-q4.png width="700">
</p>

**Answer:**

---

# TASK 4 - Exploring Configuration Files

## Configuration Files

- Splunk runs on a set of config files that control collection, parsing, transforming, and searchability
- Never edit `/opt/splunk/etc/system/default`. This room works in `/local`
- `transforms.conf` defines transformations and enrichments on indexed events, like pulling a `username` field out with a regex

```bash
[extract_username]
REGEX = User\s+(\S+)\s+logged
FORMAT = username::$1
```

- `fields.conf` defines how custom or extracted fields behave. It references the field name created in `transforms.conf`

```bash
[username]
INDEXED = true
```

- `indexes.conf` manages indexes: storage, retention, access control

```bash
[my_index]
homePath = $SPLUNK_DB/my_index/db
coldPath = $SPLUNK_DB/my_index/colddb
thawedPath = $SPLUNK_DB/my_index/thaweddb
maxTotalDataSizeMB = 100000
```

- `outputs.conf` sets where indexed data gets sent, like a remote indexer or third-party system

```bash
[tcpout:group1]
server = 10.10.10.100:9997
```

---

## Stanzas In Splunk

- Config files are split into stanzas
- Every stanza starts with a label in square brackets `[]` that says what the settings below it apply to
- What a stanza represents depends on the file:
  - An index in `indexes.conf`
  - A sourcetype in `props.conf`
  - A field transformation in `transforms.conf`
  - An input source in `inputs.conf`
  - An output group in `outputs.conf`

---

## Finn's Directions

1. Reading task, no lab work
2. Mental model before the questions, this is how the files hand off to each other:
   - `inputs.conf` says what to grab and stamps it with a `sourcetype`
   - `props.conf` finds that `sourcetype` by its stanza name and says how to parse it
   - `transforms.conf` holds a named regex that `props.conf` can call
   - `fields.conf` takes the field NAME from `FORMAT` and makes it an indexed field
3. For question 2, look at the `FORMAT` line. The field name is the part before `::`. Then look at how the `username` example above turned into a `fields.conf` stanza

---

## 1. Which Splunk configuration file is used to define output destinations for indexed data?



<p align="center">
<img src=screenshots/splunk4-q5.png width="700">
</p>

**Answer:**

---

## 2. Take a look at the example `transforms.conf` entry below. What line would you add to `fields.conf` to begin defining the custom field?

```ini
[extract_sourceip]
REGEX = User\s+(\S+)\s+logged
FORMAT = SourceIP::$1
```



<p align="center">
<img src=screenshots/splunk4-q6.png width="700">
</p>

**Answer:**

---

# TASK 5 - Creating a Splunk App

- Splunk comes with default apps. Search & Reporting is the one I use most
- Splunkbase is the app marketplace, online and inside Splunk, with apps from Splunk, vendors, and the community

---

## Firing Up Splunk

- Start Splunk from `/opt/splunk/bin` with `./splunk start`
- Dashboard is at `MACHINE_IP:8000`

---

## Creating Your Application

- Manage Apps cogwheel shows every installed app
- From there I can Browse more apps, Install app from file, or Create app
- I created an app called `DataApp` and saved it
- New apps get placed in `/opt/splunk/etc/apps`

---

## Getting Familiar With the App Directory

- `ls /opt/splunk/etc/apps` lists every app, and `DataApp` shows up first
- Inside `DataApp` there are four folders:

| Folder | Purpose |
|---|---|
| bin | Contains custom scripts or executables used by the app |
| default | Stores the app's default configuration files |
| local | Stores modified configuration files |
| metadata | Contains permission metadata for the app |

---

## Generating Logs With Python

- Scripts go in `bin`
- I made a one-line Python script, `samplelogs.py`, that prints a sample log line, and tested it with `python3`

---

## Creating Inputs

- Splunk needs to know where the script lives, so I created `inputs.conf` in the app's `/local`
- This runs the script every 5 seconds and sends output to `main` with sourcetype `testing` and host `test`

```bash
[script:///opt/splunk/etc/apps/DataApp/bin/samplelogs.py]
INDEX = main
SOURCETYPE = testing
HOST = test
INTERVAL = 5
```

- Restart Splunk after any config change

---

## Verifying Log Ingestion

- Inside DataApp, search `index = main` with time range set to All time (real time)
- The sample log lines show up as events

---

## Finn's Directions

1. Start Splunk:
   ```bash
   cd /opt/splunk/bin
   ./splunk start
   ```
2. Open a browser to `MACHINE_IP:8000` (swap in the lab machine's real IP from the THM page)
3. In Splunk click the Manage Apps cogwheel next to Apps on the left
4. Count the apps on this page BEFORE creating anything new. That's question 1, so screenshot it now
5. Click Create app, fill in the name `DataApp` and the other fields, click Save
6. On the Apps page, find DataApp and click Launch App
7. Back in the terminal, confirm the app folder exists and look at its structure:
   ```bash
   ls /opt/splunk/etc/apps
   cd /opt/splunk/etc/apps/DataApp
   pwd
   ls
   ```
   `pwd` prints the full path you're standing in
8. Make the log script:
   ```bash
   cd bin
   echo 'print("This is a sample log...")' > samplelogs.py
   python3 samplelogs.py
   ```
9. Create `inputs.conf` in `/local`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/inputs.conf
   ```
   Paste the stanza from Creating Inputs. Save with `Ctrl+O`, `Enter`, exit with `Ctrl+X`
10. Restart Splunk:
    ```bash
    /opt/splunk/bin/splunk restart
    ```
11. In DataApp, search `index = main`, set time range to All time (real time)

---

## 1. When Splunk is first installed, how many default applications are available?



<p align="center">
<img src=screenshots/splunk4-q7.png width="700">
</p>

**Answer:**

---

## 2. What is the full path to your newly created application directory?



<p align="center">
<img src=screenshots/splunk4-q8.png width="700">
</p>

**Answer:**

---

# TASK 6 - Configuring Event Boundaries

- Event breaking = splitting raw data into individual events so Splunk knows where one ends and the next begins

---

## Understanding the Events

- This task uses the `vpnlogs` executable, copied into the app's `bin`
- Each line has three pieces I care about:
  - User: who connected
  - Server: which server
  - Action: CONNECT or DISCONNECT

```
User: John Doe, Server: Server B, Action: DISCONNECT
User: Bob Johnson, Server: Server A, Action: CONNECT
User: John Doe, Server: Server D, Action: CONNECT
User: Emily Davis, Server: Server C, Action: CONNECT
User: Bob Johnson, Server: Server B, Action: CONNECT
User: Alice Smith, Server: Server C, Action: DISCONNECT
User: John Doe, Server: Server D, Action: CONNECT
User: Alice Smith, Server: Server C, Action: CONNECT
User: Michael Brown, Server: Server B, Action: DISCONNECT
```

---

## Ingesting VPN Logs

- Same process as before, add a stanza to `inputs.conf` in `/local` and restart

```bash
[script:///opt/splunk/etc/apps/DataApp/bin/vpnlogs]
INDEX = main
SOURCETYPE = vpn_logs
HOST = vpn_server
INTERVAL = 5
```

---

## Checking Out the Logs

- Searching `index = main sourcetype = vpn_logs` shows the logs coming in
- Problem: Splunk can't find the boundaries, so each event holds ten VPN lines lumped together

---

## Fixing Event Boundaries

- `props.conf` controls processing, so the fix goes there
- Every event ends in CONNECT or DISCONNECT, so that's the boundary
- Tested `(CONNECT|DISCONNECT)` on regex101 against the sample lines

---

## Creating Your Configuration File

- Created `props.conf` in `/local`:

```bash
[vpn_logs]
SHOULD_LINEMERGE = false
MUST_BREAK_AFTER = (CONNECT|DISCONNECT)
```

- Stanza name matches the sourcetype
- `SHOULD_LINEMERGE = false` because each line is one event
- `MUST_BREAK_AFTER` starts a new event after CONNECT or DISCONNECT

---

## Verifying the Changes

- Same search, All time (real time)
- Each log line is now its own event

---

## Finn's Directions

1. Copy the executable into the app:
   ```bash
   cp /home/ubuntu/Downloads/scripts/vpnlogs /opt/splunk/etc/apps/DataApp/bin/
   ```
2. Run it and LOOK at the structure first, before writing any regex:
   ```bash
   cd /opt/splunk/etc/apps/DataApp/bin
   ./vpnlogs
   ```
3. Add the VPN stanza to the BOTTOM of the existing `inputs.conf`. Leave the samplelogs stanza where it is:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/inputs.conf
   ```
4. Restart:
   ```bash
   /opt/splunk/bin/splunk restart
   ```
5. In DataApp search, All time (real time):
   ```
   index = main sourcetype = vpn_logs
   ```
   Confirm the broken events, ten lines stuck together. Good before/after screenshot
6. Go to regex101.com, paste `(CONNECT|DISCONNECT)` in the expression box and the sample lines in the test string box
7. Create `props.conf`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/props.conf
   ```
   Paste the `[vpn_logs]` stanza from Creating Your Configuration File
8. Restart, then run the same search again. One line per event now

---

## 1. Which configuration file is used to specify parsing rules?



<p align="center">
<img src=screenshots/splunk4-q9.png width="700">
</p>

**Answer:**

---

## 2. What regex pattern is used in the case above to break the events?



<p align="center">
<img src=screenshots/splunk4-q10.png width="700">
</p>

**Answer:**

---

# TASK 7 - Parsing Multi-Line Events

- Some log sources write one event across multiple lines, and Splunk has settings for that

---

## Understanding the Events

- This task uses `authentication_logs`, copied into the app's `bin`
- Each event spans two lines and includes the auth type, user, machine, timestamp, department, and a verdict

```bash
[Authentication]: A login attempt was observed from the user Johny Bil and machine Nepture
at: Fri Nov 28 09:06:53 2025 which belongs to the Development department. The login attempt looks Normal.
```

---

## Ingesting Authentication Logs

- New stanza in `inputs.conf` in `/local`, then restart

```bash
[script:///opt/splunk/etc/apps/DataApp/bin/authentication_logs]
index = main
sourcetype = auth_logs
host = auth_server
interval = 5
```

---

## Checking Out the Logs

- Searching `index = main sourcetype = auth_logs` shows them ingesting
- Problem: Splunk splits each event into two separate events

---

## Fixing Event Boundaries

- Every event starts with `[Authentication]`, so that's the anchor
- New stanza in `props.conf`:

```bash
[auth_logs]
SHOULD_LINEMERGE = true
BREAK_ONLY_BEFORE = \[Authentication\]
```

- `SHOULD_LINEMERGE = true` lets Splunk merge multiple lines into one event
- `BREAK_ONLY_BEFORE` starts a new event right before `[Authentication]`
- Brackets are escaped with `\` because `[ ]` mean something special in regex

---

## Verifying the Changes

- Same search, All time (real time)
- Each two-line log is now one event

---

## Finn's Directions

1. Copy and run it to see the structure:
   ```bash
   cp /home/ubuntu/Downloads/scripts/authentication_logs /opt/splunk/etc/apps/DataApp/bin/
   /opt/splunk/etc/apps/DataApp/bin/authentication_logs
   ```
2. Add the auth stanza to the bottom of `inputs.conf`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/inputs.conf
   ```
3. Restart:
   ```bash
   /opt/splunk/bin/splunk restart
   ```
4. Search, All time (real time), and confirm the split events:
   ```
   index = main sourcetype = auth_logs
   ```
5. Add the `[auth_logs]` stanza to the bottom of `props.conf`. Leave `[vpn_logs]` alone:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/props.conf
   ```
6. Restart and search again
7. Key contrast for your notes: Task 6 breaks AFTER the end of an event, this task breaks BEFORE the start of one

---

## 1. Which configuration setting was used to ensure Splunk breaks the event boundary before the specified regex pattern?



<p align="center">
<img src=screenshots/splunk4-q11.png width="700">
</p>

**Answer:**

---

## 2. Which regex pattern did you use in your `props.conf` file to specify where Splunk should break the events?



<p align="center">
<img src=screenshots/splunk4-q12.png width="700">
</p>

**Answer:**

---

# TASK 8 - Masking Sensitive Data

- Masking fields like credit card numbers keeps us compliant with PCI DSS and HIPAA
- Splunk can mask and anonymize fields

---

## Understanding the Events

- This task uses `purchase-details`, copied into the app's `bin`
- Logs show the user and the full credit card number on every purchase

```
User Emma made a purchase with credit card 5555-5555-5555-5555.
User Sophia made a purchase with credit card 6011-1111-1111-1117.
User Michael made a purchase with credit card 5555-5555-5555-5555.
User David made a purchase with credit card 6011-1234-5678-9012.
User Olivia made a purchase with credit card 6011-1234-5678-9012.
```

---

## Ingesting Purchase Logs

- New stanza in `inputs.conf`

```bash
[script:///opt/splunk/etc/apps/DataApp/bin/purchase-details]
index = main
sourcetype = purchase_logs
host = order_server
interval = 5
```

---

## Setting Up Event Boundaries

- Set the boundary first this time
- Every line ends with 4 digits and a period, so `\d{4}\.` marks the end
- `\d` = any digit, `{4}` = four times, `\.` = a literal period

---

## Altering Processing Properties

- New stanza in `props.conf`:

```bash
[purchase_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = \d{4}\.
```

---

## Checking Out the Logs

- Searching `index = main sourcetype = purchase_logs` shows the events with correct boundaries
- Problem: full card numbers are visible to anyone who searches

---

## Masking Credit Card Information

- `SEDCMD` works like the Unix `sed` command. It does regex find-and-replace on the raw data before it's indexed
- Added this line to the `[purchase_logs]` stanza to mask the last 12 digits:

```bash
[purchase_logs]
SHOULD_LINEMERGE = true
MUST_BREAK_AFTER = \d{4}\.
SEDCMD-cc = s/-\d{4}-\d{4}-\d{4}/-XXXX-XXXX-XXXX/g
```

- `SEDCMD-cc` is the setting, `cc` is a name I choose
- `s/pattern/replacement/g` finds `-` plus three groups of 4 digits and swaps them for `-XXXX-XXXX-XXXX`, on every match

---

## Verifying the Changes

- Same search, All time (real time)
- Card numbers now show only the first 4 digits

---

## Finn's Directions

1. Copy and run it to see the structure:
   ```bash
   cp /home/ubuntu/Downloads/scripts/purchase-details /opt/splunk/etc/apps/DataApp/bin/
   /opt/splunk/etc/apps/DataApp/bin/purchase-details
   ```
2. Add the purchase stanza to the bottom of `inputs.conf`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/inputs.conf
   ```
3. On regex101, test `\d{4}\.` against the sample lines. It should highlight the last 4 digits plus the period on each line
4. Add the `[purchase_logs]` stanza (WITHOUT the SEDCMD line yet) to the bottom of `props.conf`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/props.conf
   ```
5. Restart, then search, All time (real time):
   ```
   index = main sourcetype = purchase_logs
   ```
   Screenshot the unmasked cards for your before/after
6. Open `props.conf` again, add the `SEDCMD-cc` line inside the `[purchase_logs]` stanza, restart
7. Search again
8. Heads up: events indexed BEFORE the SEDCMD stay unmasked. Masking only applies to new data coming in. That's why real time matters here, watch the newest events

---

## 1. Which payment security standard did you adhere to by masking sensitive customer credit card information?



<p align="center">
<img src=screenshots/splunk4-q13.png width="700">
</p>

**Answer:**

---

## 2. What is the full value of the SEDCMD you used in props.conf?



<p align="center">
<img src=screenshots/splunk4-q14.png width="700">
</p>

**Answer:**

---

# TASK 9 - Extracting Custom Fields

- Splunk auto-extracts fields for lots of known log sources, but not custom ones like our VPN logs
- The VPN events have three fields to pull out: User, Server, Action

---

## Creating a Regex Pattern

- Built one regex that captures all three fields into groups
- Tested on regex101 against the sample VPN lines

```
User:\s(.+?),\sServer:\s(.+?),\sAction:\s(\w+)
```

- `\s` = a space, `(.+?)` = grab anything, as little as possible, until the next comma, `(\w+)` = grab the word

---

## Configuring Transformations

- Created `transforms.conf` in `/local`:

```bash
[vpn_custom_fields]
REGEX = User:\s(.+?),\sServer:\s(.+?),\sAction:\s(\w+)
FORMAT = User::$1 Server::$2 Action::$3
WRITE_META = true
```

- Stanza name is the extraction's name
- `FORMAT` maps capture group 1, 2, 3 to field names
- `WRITE_META = true` writes the fields into the index

---

## Updating the Processing Properties

- Added one line to the `[vpn_logs]` stanza in `props.conf` that calls `vpn_custom_fields` by name:

```bash
[vpn_logs]
SHOULD_LINEMERGE = false
MUST_BREAK_AFTER = (CONNECT|DISCONNECT)
TRANSFORM-vpn = vpn_custom_fields
```

---

## Configuring Fields

- Created `fields.conf` in `/local` so Splunk treats the extracted values as indexed, searchable fields:

```bash
[User]
INDEXED = true

[Server]
INDEXED = true

[Action]
INDEXED = true
```

---

## Verifying Field Extraction

- Searching `index = main sourcetype = vpn_logs` now shows User, Server, and Action as fields
- Events are searchable by those fields

---

## Purchase Log Field Extraction

- My turn: extract `User` and `CC_Number` from the purchase logs
- Regex given by the room:

```
^User\s+([A-Za-z]+(?:\s+[A-Za-z]+)*?)\s+made a purchase with credit card\s+([0-9]{4}(?:-XXXX){3}|(?:\d{4}[\s-]?){3}\d{4})\.
```

- Steps: `transforms.conf`, then `props.conf`, then `fields.conf`, then restart

---

## Finn's Directions

**Part A: VPN fields**

1. Test the regex on regex101 with the VPN sample lines. Check the Match Information panel on the right, you should see Group 1, 2, 3 per line
2. Create `transforms.conf`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/transforms.conf
   ```
   Paste the `[vpn_custom_fields]` stanza
3. Open `props.conf` and add the transform line inside `[vpn_logs]`:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/props.conf
   ```
4. Create `fields.conf` and paste the three stanzas:
   ```bash
   nano /opt/splunk/etc/apps/DataApp/local/fields.conf
   ```
5. Restart, then search `index = main sourcetype = vpn_logs`. Check the left Fields sidebar for User, Server, Action
6. Troubleshooting: if the fields never show up, check the props line. Splunk's official setting is spelled `TRANSFORMS-vpn` with an S. Fix the spelling, restart, check new events

**Part B: Purchase fields, same three files, same pattern**

7. Add this to the bottom of `transforms.conf`:
   ```bash
   [purchase_custom_fields]
   REGEX = ^User\s+([A-Za-z]+(?:\s+[A-Za-z]+)*?)\s+made a purchase with credit card\s+([0-9]{4}(?:-XXXX){3}|(?:\d{4}[\s-]?){3}\d{4})\.
   FORMAT = User::$1 CC_Number::$2
   WRITE_META = true
   ```
8. Add this line inside the `[purchase_logs]` stanza in `props.conf`:
   ```bash
   TRANSFORMS-purchase = purchase_custom_fields
   ```
9. Add this to the bottom of `fields.conf`. `[User]` already exists from Part A, so only CC_Number is new:
   ```bash
   [CC_Number]
   INDEXED = true
   ```
10. Restart, then search, All time (real time):
    ```
    index = main sourcetype = purchase_logs
    ```
11. Click `User` and then `CC_Number` in the left Fields sidebar. The popup shows the values and how many there are

---

## 1. After performing field extraction on the purchase logs, how many User field values exist within the event data?



<p align="center">
<img src=screenshots/splunk4-q15.png width="700">
</p>

**Answer:**

---

## 2. How many unique masked CC_Number field values were extracted from the events?



<p align="center">
<img src=screenshots/splunk4-q16.png width="700">
</p>

**Answer:**

---

# TASK 10 - Conclusion


