---
title: Alert Triage With Elastic
module: SIEM Triage for SOC
path: SOC Level 1
platform: TryHackMe
tags: [siem, elastic, kibana, alert-triage, proxylogon, windows-logging, sysmon]
status: Completed
date: 2026-07-21
date_completed: 2026-07-21
---
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/elastic_intro.png width="1500">
</p>

---

## TASK 1 - Introduction

SomeCorp's infrastructure triggered a string of SOC alerts. Using Kibana, this investigation correlates web logs, Windows Security logs, Sysmon, and PowerShell script block logging to confirm a ProxyLogon-based breach, an Administrator logon outside business hours, a backdoor account creation, and privilege escalation via group membership changes.

---

## TASK 2 - Scenario Briefing

Our team manages servers, applications, and network infrastructure for several small-business clients. SomeCorp, one of those clients, has triggered multiple SOC alerts. As the on-call analyst, the job is to use the available logs, dashboards, and tools to investigate the activity, decide whether it's malicious, and reconstruct the attack sequence.

Data view: `Alert Triage With Elastic` | Time range: Entire data range | Starting query: `_index:weblogs`

---

**How many logs are available for analysis within the entire time range?**

<p align="center">
<img src=screenshots/elastic_total.png width="700">
</p>

As an SOC we would typically filter our data surrounding the time we think an attack might be taking place. Since this is an isolated exercise we can select the Entire Data Range and see 1,467 events. In a real case scenario, Entire Data Range would probably pull hundreds of thousands of events.

**Answer:** 1,467

---

**What is the field value for the `client.ip` in the `weblogs` index?**

<p align="center">
<img src=screenshots/elastic_clientipel.png width="700">
</p>

If we receive various alerts from a single IP in quick succession, and that IP is making various requests to your web server, this is a red flag and should be investigated further.

**Answer:** 203.0.113.55

---

## TASK 3 - Investigating Web Attacks

Query: `_index:weblogs and client.ip:203.0.113.55 and http.request.method:POST`
Table fields: `client.ip`, `user.agent`, `http.request.method`, `url.path`, `http.response.status_code`

Follow-up query: `_index:weblogs and client.ip:203.0.113.55 and http.request.method:GET and errorEE.aspx` (sorted Old-New)

---

**How many `POST` requests did the IP address `203.0.113.55` make to `proxyLogon.ecp`?**

<p align="center">
<img src=screenshots/elastic_proxylogon.png width="700">
</p>

POST requests to /ecp/proxyLogon.ecp should be lighting alarm bells in your SOC brain — ProxyLogon (CVE-2021-26855) is an exploit that attempts to take over a Microsoft Exchange Server.

**Answer:** 3

---

**Which `user.agent` paired with the IP address `203.0.113.55` made the `POST` requests?**

<p align="center">
<img src=screenshots/elastic_useragent.png width="700">
</p>

Seeing `python-requests/2.25.1` as an SOC is another major red flag — this outdated user agent is well known for automated attacks, vulnerability scanning, and aggressive scraping.

**Answer:** python-requests/2.25.1

---

**How many logs contain the `cmd=` query parameter in the `url.path` field?**

<p align="center">
<img src=screenshots/elastic_urlpath.png width="700">
</p>

If the attacker finds the web server vulnerable, they'll try to run commands through the URL. Any sign of `cmd=` in the logs, paired with an outdated user agent and contact with `proxyLogon.ecp`, warrants a full hands-on investigation. The available fields section wasn't clear enough on its own, so I used the query: `_index:weblogs and url.path:*cmd=*`

**Answer:** 20

---

**Which command was run utilizing `errorEE.aspx` on `Jul 20, 2025 @ 04:45:50.000`?**

<p align="center">
<img src=screenshots/elastic_hostname.png width="700">
</p>

The attacker is performing recon, running a command to learn more about the host machine: `/errorEE.aspx?cmd=hostname`

**Answer:** hostname

---

## TASK 4 - Uncovering Account Activity

Query: `@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator`
Table fields: `winlog.event_id`, `host.name`, `winlog.event_data.TargetUserName`, `winlog.logon.type`, `winlog.event_data.IpAddress`

Follow-up query: `@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:1 and user.name:Administrator`
Table fields: `user.name`, `process.parent.name`, `process.command_line`

Follow-up query: `@timestamp >= "2025-07-20T05:13:10.000" and winlog.channel:Security and winlog.task:User Account Management`
Table fields: `winlog.event_id`, `winlog.task`, `message`

---

**What is the `winlog.record_id` of the Administrator 4624 logon event?**

<p align="center">
<img src=screenshots/elastic_admin.png width="700">
</p>

Once an attacker gets into the system, they'll go after privileged accounts. This 4624 event — outside of work hours, with a RemoteInteractive logon type and a public IP in the 203.x.x.x range — means the attacker has taken the next step in the attack.

**Answer:** 17166

---

**What is the `process.pid` of the Sysmon 1 event that occurred on Jul 20, 2025 @ 05:11:27.996?**

<p align="center">
<img src=screenshots/elastic_sysmon.png width="700">
</p>

Five seconds later, the attacker creates a process (Sysmon Event ID 1) on the same host, `winserv2019.some.corp`.

**Answer:** 964

---

**What is the `winlog.event_id` for the new user account being created?**

**Answer:** 4720

---

**What is the name of the new user account?**

<p align="center">
<img src=screenshots/elastic_svchost.png width="700">
</p>

Attackers often aim for persistence by creating accounts that sound like legitimate service processes.

**Answer:** svc_backup

---

## TASK 5 - Exposing Command Execution

Query: `@timestamp >= "2025-07-20T05:13:15" and process.parent.name:cmd.exe and user.name:Administrator`
Table fields: `process.command_line`, `process.name`, `process.parent.name`

Correlated query: `@timestamp >= "2025-07-20T05:13:15" and (winlog.event_id:4732 or process.parent.name:cmd.exe)`

### PowerShell Usage

Query: `@timestamp >= "2025-07-20T05:13:15" and event.module:powershell and event.code:4104` (field added: `powershell.file.script_block_text`, sorted Old-New)

### No Alert Created

Query: `process.name: "Rar.exe"`

---

**What command does the attacker use to add the new account to the "Remote Desktop Users" group?**

<p align="center">
<img src=screenshots/elastic_example8.png width="700">
</p>

The attacker adds the account they created to Remote Desktop Users, which gets them persistence. I used the query: `process.command_line:*group*`

**Answer:** net localgroup "Remote Desktop Users" svc_backup /add

---

**What is the `winlog.record_id` of the `4732` Security event when the attacker adds the user to the Administrator group?**

<p align="center">
<img src=screenshots/elastic_group.png width="700">
</p>

Worth noting: the attacker used PowerShell here to add the user to the Admin group.

**Answer:** 17254

---

**What PowerShell command did the attacker run on `Jul 20, 2025 @ 05:16:14.628`?**

<p align="center">
<img src=screenshots/elastic_group.png width="700">
</p>

Needed the `message` and `powershell.file.script_block_text` fields in the table to see what `process.command_line` alone didn't show.

**Answer:** net group "Domain Admins" /domain

---

**What is the name of the archive that the attacker creates using the `Rar.exe` executable?**

<p align="center">
<img src=screenshots/elastic_filename.png width="700">
</p>

The attacker staged their haul in `C:\Temp\finance_it_archive.rar`, run from the `svc_backup` account:

```
Process Create: RuleName: - UtcTime: 2025-07-20 05:17:55.918 ProcessGuid: {356535D0-7C03-687C-FB00-000000006600} ProcessId: 4140 Image: C:\Program Files\WinRAR\Rar.exe FileVersion: 7.12.0 Description: Command line RAR Product: WinRAR Company: Alexander Roshal OriginalFileName: - CommandLine: "C:\Program Files\WinRAR\rar.exe" a -hpSpring2025! -m5 C:\Temp\finance_it_archive.rar C:\Users\asmith\Documents\* C:\IT\Admin\Scripts\* CurrentDirectory: C:\Users\svc_backup.SOME\ User: SOME\svc_backup LogonGuid: {356535D0-7B1F-687C-AA50-090000000000} LogonId: 0x950aa TerminalSessionId: 2 IntegrityLevel: Medium Hashes: SHA256=BDBE2BDE697C45E02902F19415D7AF07DC871F30A3AC84FB552657AEDDC38714 ParentProcessGuid: {356535D0-7BB5-687C-F900-000000006600} ParentProcessId: 4640 ParentImage: C:\Windows\System32\cmd.exe ParentCommandLine: "C:\Windows\system32\cmd.exe" ParentUser: SOME\svc_backup
```

**Answer:** finance_it_archive.rar

---

## TASK 6 - Conclusion

The investigation ties together a full attack chain: exploitation of ProxyLogon on the Exchange server, web shell recon via `errorEE.aspx`, an off-hours Administrator RDP logon from the same external IP, creation of the `svc_backup` backdoor account, its addition to Remote Desktop Users and the local Administrators group, PowerShell-based domain recon, and finally staging stolen files into a password-protected RAR archive. Every alert in this room escalates to a True Positive, and the case should be handed off with the full timeline attached.
