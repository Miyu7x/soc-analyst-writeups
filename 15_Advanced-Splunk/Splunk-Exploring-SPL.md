---
title: "Splunk: Exploring SPL"
module: Advanced Splunk
path: SOC Level 2
platform: TryHackMe
tags: [splunk, spl, siem, log-analysis, anomaly-detection]
status: In Progress
date: 2026-10-14
date_completed: 
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/splunk-exploring-spl-intro.png width="1500">
</p>

---

# TASK 1 - Introduction

- Splunk is the SIEM, SPL (Search Processing Language) is how you actually get answers out of it
- This room builds SPL up in layers: search operators, filtering, structuring, transforming, then anomaly detection
- Most of the work happens in the `windowslogs` index, the anomaly hunting uses `vpnlogs`
- Prereqs are Introduction to SIEM and Splunk: Basics
- Time range goes to **All time** before every search, or the results come back thin

---

# TASK 2 - Search & Reporting

<p align="center">
<img src=screenshots/splunke-t2.png width="700">
</p>

- Search & Reporting is the default app on the Splunk home page, it is where every search lives
- Four parts to know:
  - **Search Head:** where the query goes
  - **Time Picker:** sets the time window for the search
  - **Search History:** every query I have already run
  - **Data Summary:** the hosts, sources, and sourcetypes available

---

## Your First Search

- An index is a Splunk database, a container that keeps related data together
- First query, time range set to All time:

```
index=windowslogs
```

- `index=windowslogs` and `index = windowslogs` both work, spaces around `=` do not matter

---

## Fields Sidebar

- Left panel of the search page, split into two sections:
  - **Selected Fields:** the default extracted fields. Click any field and toggle Selected to add it
  - **Interesting Fields:** everything else Splunk pulled out of the events
- `#` means the field holds numbers, `α` means it holds text
- The count next to a field is how many events contain it
- Clicking a field gives quick top values without writing a single query

---

## 1. Submit your first query for All Time: index=windowslogs. How many total events do you see?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q1.png width="700">
</p>

**Answer: 12256**

---

## 2. After you submit your first query, look in the Fields sidebar. Which SourceIP has recorded the most amount of events?

If we search interesting fields we can see that we are missing two key components of an investigation: SourceIP and DestinationIP. At the bottom of the Interesting Fields there is an option to add more fields. 

<p align="center">
<img src=screenshots/splunk-exploring-spl-q2.png width="700">
</p>

<p align="center">
<img src=screenshots/splunk-exploring-spl-q21.png width="700">
</p>


**Answer: 172.90.12.11**

---

## 3. How many events appear on 04/15/2022 from 08:05 AM to 08:06 AM?

We had previously set the time param,eters for all time, but for this question we can adjust the date and time to before 04/15/2022 08:06:000

<p align="center">
<img src=screenshots/splunk-exploring-spl-q3.png width="700">
</p>

**Answer: 134**

---

# TASK 3 - Search Operators

- SPL combines commands, functions, and operators to filter, transform, and format logs
- Filtering by field only works when the events are parsed into fields first

---

## Free Text Search

- Simplest search there is, index plus a keyword:

```
index=windowslogs alice
```

- Case-insensitive, matches any event containing the word
- Best move when I do not know the field names yet, or I am hunting one unique keyword

---

## Search Operators

- Operators are the building blocks of every query: they filter, remove, and refine results
- Three families: relational, logical, and wildcard

---

## Relational Operators

- Compare two expressions: equal, not equal, greater than, less than

| Operator | Example | Explanation |
|---|---|---|
| Equals = | UserName=Mark | Search for all events in which the field name UserName is equal to Mark |
| Not Equal To != | UserName!=Mark | Search for all events in which the field name UserName is not equal to Mark |
| Less Than < | Age<10 | The field Age has a value of less than 10 |
| Less Than or Equal To <= | Age<=10 | The field Age has a value of less than or equal to 10 |
| Greater Than > | Outbound_Traffic>50 | The Outbound_Traffic field value is greater than 50 |
| Greater Than or Equal To >= | Outbound_Traffic>=50 | The Outbound_Traffic field value is greater than or equal to 50 |

- Filtering out SYSTEM noise:

```
index=windowslogs AccountName!=SYSTEM
```

---

## Logical Operators

- Connect or modify conditions, they work on true/false

| Operator | Example | Explanation |
|---|---|---|
| NOT | NOT UserName=* | Returns events where UserName field does not exist. Don't confuse it with != operator |
| AND | UserName=David AND IPAddress=10.10.10.10 | Returns all events in which the UserName field is equal to David and the IPAddress field is equal to10.10.10.10 |
| OR | UserName=David OR UserName=John | Returns all events in which the UserName field is equal to David or John |
| IN | UserName IN(David, John) | A more convenient alternative to the OR keyword, especially for long lists. |

- `AND` is implied, so both of these return the same thing:

```
index=windowslogs AccountName!=SYSTEM AND AccountName=James
index=windowslogs AccountName!=SYSTEM AccountName=James
```

- `NOT UserName=*` and `UserName!=x` are not the same: NOT catches events where the field is missing entirely

---

## Wildcards and CIDR Search

- Partial matches and whole subnets in one shot

| Symbol | Example | Explanation |
|---|---|---|
| * | status=*fail* | This will return all events that have status field set to failed, failure, appfail, etc. |
| * | DestinationIp=172.* | This will return all events that contain values like DestinationIp=172.90.0.0.1 or DestinationIp=172.18.5.22 |
| N/A | DestinationIp=172.18.0.0/16 | This will return all events where DestinationIp field is within the 172.18.0.0/16 subnet |

---

## Order of Evaluation

- How Splunk reads a query decides what comes back, quotes and parentheses control it

---

## Quotes

- Quotes turn text into one exact value, word order matters
- `failed login` without quotes means both words, any order
- `"failed login"` means that exact phrase
- Quotes also escape operators, so `"TO BE OR NOT TO BE"` searches the phrase instead of running OR and NOT

---

## Parentheses

- Splunk evaluates OR before AND, so a query without parentheses can quietly mean something else

| Your Search | How Splunk Evaluates It |
|---|---|
| index=windowslogs alice AND bob OR charlie (Implicit search, no parentheses) | index=windowslogs alice AND (bob OR charlie) (Mistake! Splunk evaluated OR before AND) |
| index=windowslogs (alice AND bob) OR charlie (Explicit search with parentheses) | index=windowslogs (alice AND bob) OR charlie (Correct! The results match your requirements) |

- Group conditions with parentheses every time, never trust the implicit order

---

## 1. How many events in the windowslogs index have an EventID field value equal to 4624?

For our search this is a simple EventID=4624.

<p align="center">
<img src=screenshots/splunk-exploring-spl-q4.png width="700">
</p>

**Answer: 26**

---

## 2. How many events are observed with the DestinationIp = 172.18.39.6 and DestinationPort = 135?

To search for a specific IP and Port we can use AND.
index="windowslogs" DestinationIp="172.18.39.6" AND DestinationPort="135"

<p align="center">
<img src=screenshots/splunk-exploring-spl-q5.png width="700">
</p>

**Answer: 4**

---

## 3. Use the query `index=windowslogs Hostname=Salena.Adam DestinationIp=172.18.38.5` Which SourceIp returns the highest count?

The source IP that returned the highest count was 172.90.12.11, with 17 hits and made up about 89% of the traffic.
<p align="center">
<img src=screenshots/splunk-exploring-spl-q6.png width="700">
</p>

**Answer: 172.90.12.11**

---

## 4. How many events are returned when you search the term cyber*?

For this one, Splunk lets us just search for the term as is.
index="windowslogs" cyber*

<p align="center">
<img src=screenshots/splunk-exploring-spl-q7.png width="700">
</p>

**Answer: 12256**

---

## 5. Which operator is given the lowest priority in Splunk searches?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q8.png width="700">
</p>

**Answer: AND**

---

# TASK 4 - Filtering Results

- Thousands of logs a minute, no filters means drowning
- Commands chain with a pipe `|`, each one hands its output to the next

---

## Useful Filtering Commands

- `fields`, `dedup`, `rename`, and `regex` narrow results down to what actually matters

---

## Fields

- Keeps or drops fields from the results. `-` excludes, `+` includes but is optional

```
index=windowslogs | fields host User SourceIp
```

- Lifesaver when an event has hundreds of fields and I only care about three

---

## Dedup

- Removes duplicate values, one event per unique value

```
index=windowslogs
| fields EventID User Image Hostname SourceIp
| dedup SourceIp
```

- Seven unique SourceIps means seven events back
- Cleans up noisy sources, Microsoft 365 can fire 50 events for a single activity

---

## Rename

- Changes a field name in the results, cleaner for screenshots in a SOC report

```
index = windowslogs
| fields EventID User Image Hostname SourceIp
| rename User as Employee
```

- Also flattens JSON/XML subfields so I stop typing the prefix:

```
index=jsondata
| rename request.* as *
```

- `request.path` becomes `path`, `request.ip` becomes `ip`

---

## Regex

- Filters by pattern instead of exact keyword. Splunk uses PCRE

```
index = windowslogs | regex Image = "\.exe$"
```

- `$` anchors the match to the end of the string, so only values ending in `.exe`
- Irreplaceable for custom or badly parsed data sources

---

## 1. Use the fields command to highlight Domain, SourceProcessId, and TargetProcessId. Which SourceProcessId has the highest value?

```
index="windowslogs" 
| fields Domain SourceProcessId TargetProcessId
```

<p align="center">
<img src=screenshots/splunk-exploring-spl-q9.png width="700">
</p>

**Answer: 9496**

---

## 2. Try out this query `index=windowslogs | regex TargetObject="Manager$"`. Which TargetObject field value contains the highest number of results?

- Ran regex query, clicked on TargetObjects and found the top value = HKLM\\SOFTWARE\\Microsoft\\SecurityManager.
- This jumps out right away, any security related registry being touched raises red flags.
- Did some further digging and everything seemed normal.
- Image: C:\windows\system32\svchost.exe is normal path.
- EventType: CreateKey, this is important nothing seems deleted or out of place.
- Conclusion: Normal Windows service activity.

<p align="center">
<img src=screenshots/splunk-exploring-spl-q10.png width="700">
</p>

**Answer: HKLM\SOFTWARE\Microsoft\SecurityManager**

---

# TASK 5 - Structuring Results

- Raw results are overwhelming, structuring commands order and format them so the important stuff is up front

---

## Table Command

- Picks the fields I want and lays them out clean, perfect for timelines and host or user investigations

```
index=windowslogs | table _time EventID Hostname SourceName
```

---

## Useful Structuring Commands

| Command | Example | Explanation |
|---|---|---|
| head | index=windowslogs \| head 20 | Returns the first (newest) 20 events. Useful to speed up the search if you don't need complete results |
| tail | index=windowslogs \| tail 20 | Returns the last (oldest) 10 events. Useful to speed up the search if you don't need complete results |
| sort | index=windowslogs \| sort User | This will sort the logs in alphabetical order based on the field User |
| reverse | index=windowslogs \| reverse | This command reverses the order of events (descending order) |

---

## Timelining With Table

- `table` plus `reverse` rebuilds what happened on a host, oldest first

```
index = windowslogs Hostname = Salena.Adam
| table _time Hostname EventID Category
| reverse
```

- From there I can cut system noise or add more columns

---

## Subsearches

- Sysmon EventID 1 shows the process, but it does not log LogonType
- Security EventID 4624 has the LogonType, so the two get correlated through `LogonId`
- Remote session is LogonType 3/10, a service is LogonType 5

```
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

- The subsearch inside `[ ]` runs first: renames `TargetLogonId` to `LogonId` and saves LogonId, LogonType, IpAddress in a temporary lookup
- The main search then finds Sysmon EventID 1 events and adds LogonType and IpAddress wherever the LogonId matches
- Powerful but slow on big data sets, `stats` and `eval` usually do the same job faster
- Enriching data source A with fields from data source B is exactly when subsearch plus `join` earns its spot

---

## 1. Build a table that highlights the EventID, AccountName, and AccountType fields. Which AccountName appears first in your results?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q11.png width="700">
</p>

**Answer:**

---

## 2. Append the above query to include the reverse command. Which EventID appears first?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q12.png width="700">
</p>

**Answer:**

---

## 3. Use the query below to build a timeline of events. What password was given to the user A1berto?

```
index=windowslogs EventID=1
| table _time ParentProcessId ProcessId ParentCommandLine CommandLine
| reverse
```

<p align="center">
<img src=screenshots/splunk-exploring-spl-q13.png width="700">
</p>

**Answer:**

---

# TASK 6 - Transforming Commands

- Transforming commands turn raw events into counts, stats, and visuals
- Searches that use them are called transforming searches

---

## General Transformational Commands

| Command | Example | Explanation |
|---|---|---|
| top | index=windowslogs \| top User limit=5 | Returns the ten most frequent values of the field specified. A numerical value can be included with limit= to reduce or expand the results |
| rare | index=windowslogs \| rare User limit=5 | Returns the ten least frequent values of the field specified. A numerical value can be included with limit= to reduce or expand the results |

---

## Highlight

- Visually marks field values and keywords in the raw events

```
index=windowslogs | highlight User EventID Image "Process accessed"
```

- Switch the view from **List** to **Raw** or nothing shows up highlighted

---

## Stats

- Counts, sums, averages, the big summarizer

| Command Function | Example | Explanation |
|---|---|---|
| Average | stats avg(ProcessCount) | Calculates the average value of the chosen field |
| Max | stats max(Price) | Returns the maximum value of the chosen field |
| Min | stats min(UserAge) | Returns the minimum value of the chosen field |
| Sum | stats sum(Cost) | Returns the sum of the chosen field values |
| Count | stats count by SourceIp | Returns the number of occurrences of the chosen field |

```
index=windowslogs | stats count by EventID | sort EventID
```

---

## Chart

- Same functions as `stats`, but the output feeds straight into a visualization

```
index=windowslogs | chart count by User
```

---

## Timechart

- Shows how data changes over time, peaks and spikes jump right out

```
index=windowslogs Image!="" | timechart span=30m count by Image limit=5
```

- `Image!=""` drops the empty Image values, `span=30m` sets 30-minute buckets, `limit=5` keeps the top five processes

---

## Data Enrichment and Field Manipulation

- Adding context Splunk does not have on its own: location, external tables, new fields

---

## IP Location

- Adds City, Region, and Country to an IP using Splunk's built-in geolocation tables

```
index=windowslogs | iplocation SourceIp | stats count by Country
```

---

## Lookup

- Matches a field against a CSV or lookup table to pull in extra data

```
index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole
| stats count by Hostname UserRole
```

- Here the CSV maps each Hostname to an employee role in `UserRole`

---

## Eval

- Creates new fields, modifies existing ones, does math right inside the search

```
index=windowslogs
| eval LogonTypeDesc = case(LogonType == 3, "Network Logon", LogonType == 5, "Service")
| stats count by LogonType LogonTypeDesc
```

- LogonType 3 becomes "Network Logon", LogonType 5 becomes "Service"
- A bare number in a report means nothing to a manager, a readable label does

---

## 1. Use the top command to query the Image field. Which Image field value has the most occurrences?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q14.png width="700">
</p>

**Answer:**

---

## 2. Try out the iplocation command with the SourceIp field. Which Region do the IP addresses in your events originate from?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q15.png width="700">
</p>

**Answer:**

---

## 3. Try out this lookup query. Which Image field value has the highest RiskScore?

```
index=windowslogs
| lookup image_riskscore Image OUTPUT RiskScore
| stats count by Image RiskScore
| sort - RiskScore
```

<p align="center">
<img src=screenshots/splunk-exploring-spl-q16.png width="700">
</p>

**Answer:**

---

# TASK 7 - Anomaly Detection

---

## Anomaly Detection

- Big data set, plain field stats show nothing weird, so the outliers have to be calculated
- Example: 2,000 VPN logins with only time, username, source IP, and source country

---

## Detecting Outliers by Country

- A US user logging in from the US is normal. An EU user logging in from the US might be an intrusion
- `eventstats` works like `stats` but keeps the raw events for the next step
- `where` is like `search`, just more powerful

| Line / Command | Result |
|---|---|
| Line 2: Counts total logins by user | For kbrown user, logins_by_user=200 |
| Line 3: Counts logins by user and source country | For kbrown and Austria, logins_by_user_country=1 |
| Line 4: Evaluates frequency of logins per country | For kbrown and Austria, country_freq=0.005 |
| Line 5: Includes only rare user-country pairs | Note: The sensitivity (0.1) is called the threshold |
| Line 6: Shows the outliers as a table | Only 2 outliers out of 2,000 login events! |

```
index=vpnlogs
| eventstats count as logins_by_user by user 
| eventstats count as logins_by_user_country by user src_country 
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```

- One login from a rare country is a strong signal, either a VPN or a breach

---

## Detecting Outliers by Hour

- Harder than country: time zones and personal habits vary, some people log in at night on purpose
- Three numbers per user:
  - **typical_hour:** their average login hour, for example 13:30 UTC
  - **stdev_hour:** how predictable they are. 0 is perfectly predictable, 2 means ± 2 hours
  - **zscore:** how many standard deviations the login sits from their normal. 3 means three times more anomalous than usual
- jsmith normally logs in around 13:30 with low deviation, then logs in at 18:30. Worth a look

```
index=vpnlogs
| eval hour=tonumber(strftime(_time, "%H")) + tonumber(strftime(_time, "%M"))/60
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user
| eval zscore=abs(hour - typical_hour) / stdev_hour
| where zscore > 3
| eval hour=round(hour, 2), typical_hour=round(typical_hour, 2)
| eval stdev_hour=round(stdev_hour, 2), zscore=round(zscore, 2)
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
| sort - hour_zscore
```

---

## ML and Impossible Travel

- Bigger detections like Impossible Travel run on these same basic commands
- The recipe is SPL, some math, and threat context from `iplocation` or `lookup` tables
- `fit` and `apply` bring machine learning in, training on my data so future outliers get caught better

---

## 1. Run the first anomaly detection query (index=vpnlogs). Which other user is marked as an outlier?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q17.png width="700">
</p>

**Answer:**

---

## 2. Which country is anomalous for the user from Q1? Answer Example: CN

<p align="center">
<img src=screenshots/splunk-exploring-spl-q18.png width="700">
</p>

**Answer:**

---

## 3. Run the second anomaly detection query (index=vpnlogs). Which user suspiciously logged in at 3 AM?

<p align="center">
<img src=screenshots/splunk-exploring-spl-q19.png width="700">
</p>

**Answer:**

---

# TASK 8 - Conclusion

- 

