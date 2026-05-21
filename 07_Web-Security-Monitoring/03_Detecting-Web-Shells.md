---
title: Detecting Web Shells
module: Web Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [web-shells, detection, log-analysis, apache, auditd, wireshark, file-system-analysis, network-traffic, php, wordpress]
status: completed
date: 05-20-2026
date_completed: 05-20-2026
---

<p align="center">
  <img src=screenshots/web_shells.png width="1500">
</p>

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

---

## Task 1 - Introduction

<!-- Q: What is a web shell and why is it relevant to a SOC Analyst specifically? -->

**1. I understand the learning objectives and am ready to embark on a web shell adventure.**

**Answer:** N/A

---

## Task 2 - Web Shell Overview

Web shells are software programs or scripts that run on a web server, allowing attackers to execute commands, maintain persistence, and retain remote access through the browser.

In order to execute a web shell, an attacker typically needs one of the following:
- a file upload vulnerability
- a misconfiguration
- prior access to the system

A common Windows Microsoft Exchange vulnerability involves uploading or modifying an existing **.aspx** file.

Web shells serve two roles in the attack chain:
- **Initial access** via file upload vulnerabilities -- MITRE ATT&CK T1190: Exploit Public-Facing Application
- **Persistence** via MITRE ATT&CK T1505.003: Server Software Component: Web Shell

**1. Which MITRE ATT&CK Persistence sub-technique are web shells associated with?**

<p align="center">
  <img src=screenshots/web_mitreattck.png width="500">
</p>

**Answer: T1505.003**

---

**2. What file extension is commonly used for web shells targeting Microsoft Exchange?**

**Answer: .aspx**

---

## Task 3 - Anatomy of a Web Shell

Web shells work by abusing legitimate system execution functions. Common PHP functions exploited include:
- `shell_exec()`
- `exec()`
- `system()`
- `passthru()`

<!-- Q: Walk through the logic of the simple PHP web shell step by step. What does each component do? -->

**How a Simple PHP Web Shell Works**

If the shell sits at `http://10.10.10.100/uploads/awebshell.php`, an attacker appends `?cmd=whoami` to interact with it. Here is what happens under the hood:

1. **Parameter check** -- the shell checks the URL for a `cmd` parameter
2. **Store the command** -- the value (e.g., `whoami`) gets stored in the variable `$cmd`
3. **Execute** -- `shell_exec($cmd)` runs the command on the web server
4. **Display output** -- the terminal result gets printed back to the browser
5. HTML wraps all of the above, providing a text box for input and an execute button

<!-- Q: How do you URL-encode a command for use in a web shell via curl? Give an example. -->

**Accessing a Web Shell via CLI**

Web shells can be accessed through the browser or via `curl`. Since URLs do not recognize spaces, commands must be URL-encoded. CyberChef can assist with this.

```bash
curl "http://10.10.10.100:8080/files/awebshell.php?cmd=ls%20-la"
```

- `ls` -- lists directory contents
- `-l` -- long format: file names, permissions, owner, group, file size, timestamp
- `-a` -- all files, including hidden files

**1. Access the shell and determine which account you have access to by running the `whoami` command.**

<p align="center">
  <img src=screenshots/web_wwwdata.png width="500">
</p>

*`curl "http://10.10.10.100:8080/files/awebshell.php?cmd=whoami"`*

**Answer: www-data**

---

**2. List the directory contents and find the flag using the `ls` and `cat` commands.**

<p align="center">
  <img src=screenshots/web_lscatflag.png width="500">
</p>

*`curl "http://10.65.170.122:8080/files/awebshell.php?cmd=cat%20flag.txt"`*

**Answer: THM{W3b_Sh3ll_Usag3}**

---

## Task 4 - Log-Based Detection

Web shells produce web server logs. Each log entry typically contains:
- client IP
- remote log name
- authenticated user
- timestamp
- request line
- status code
- response size
- referrer
- user-agent

<p align="center">
  <img src=screenshots/web_logexample.png width="500">
</p>

**Unusual Web Log Patterns**

- Repeated GET requests -- attacker scanning for a valid upload location
- POST requests to a valid location immediately following repeated GETs
- Repeated GET or POST requests to the same file -- possible web shell interaction
- PUT -- uploading a web shell
- DELETE -- cleaning up evidence
- OPTIONS -- performing reconnaissance
- HEAD -- detecting files

| Request Method | Normal Usage | Possible Abuse |
|---|---|---|
| GET | Retrieve a resource | Recon or interacting with a web shell |
| POST | Submit data to the server | Upload or interact with a web shell |
| PUT | Upload or replace a file | Upload a web shell |
| DELETE | Remove a resource | Cleanup |
| OPTIONS | Request supported methods | Reconnaissance |
| HEAD | Returns headers only | Detect files |

**Response Codes to Remember**
- **200 OK**
- **404 Not Found**

<p align="center">
  <img src=screenshots/web_logpatternexample.png width="500">
</p>

**Additional Suspicious Log Indicators**
- Altered User-Agents: e.g., `Mozilla/4.0+(+Windows+NT+5.1)` truncated to `Mozilla/4.0`
- Blacklisted User-Agents: `curl/1.XX.X` or `wget/1.XX.X`
- External/outside-network IP addresses
- Long query strings
- Encoded query strings: `?query=whoami` becomes `?query=d2hvYW1p`
- Missing referrer -- can indicate direct access rather than normal browsing behavior

**Linux and Auditd**

`auditd` determines what gets logged in `audit.log`. The following command searches for logs matching a web shell rule:

```bash
ausearch -k web_shell
```

**1. What is the part of the URL that associates values to parameters and can be a valuable indicator of web shell activity?**

**Answer: query strings**

---

**2. What auditd syscall would confirm that a file was written to disk following a suspicious POST request to /upload.php?**

**Answer: creat**

---

## Task 5 - Beyond Logs

A web shell needs to be stored somewhere on the server. Common default web root locations:
- Apache: `/var/www/html/`
- Nginx: `/usr/share/nginx/html/`

Attackers may also target subdirectories like `/uploads`, `/images`, `/admin`, or `/tmp`.

**Suspicious File Indicators**
- `.php` and `.jsp` files in unexpected locations
- Double extensions: `image.jpg.php`

**Useful Wireshark Filters**

| Filter | Purpose |
|---|---|
| `http.request.method == "METHOD"` | Find repeated or unusual request methods |
| `http.request.uri contains ".php"` | Locate suspicious or modified PHP files |
| `http.user_agent` | Identify unusual or outdated User-Agents |

**Useful File System Commands**

```bash
# Find .php files modified within a date range
find /var/www -type f -name "*.php" -newerct "YYYY-MM-DD" ! -newerct "YYYY-MM-DD"

# Search for suspicious function calls recursively
grep -r "eval(" /path/to/directory
```

**1. What command would you use to locate .php files in the /var/www/ directory?**

**Answer: find /var/www/ -type f -name "*.php"**

---

**2. Which Wireshark filter would you use to search specifically for PUT requests?**

**Answer: http.request.method == "PUT"**

---

## Task 6 - Investigation

Investigate `/var/log/apache2/access.log` for:
- `.php` file references
- unusual response codes
- `grep` to filter relevant entries

**1. Which IP address likely belongs to the attacker?**

<p align="center">
  <img src=screenshots/web_shadyshell.png width="500">
</p>

**Answer: 203.0.113.66**

---

**2. What is the first directory that the attacker successfully identifies?**

<p align="center">
  <img src=screenshots/web_shadyshell.png width="500">
</p>

**Answer: /wordpress**

---

**3. What is the name of the .php file the attacker uses to upload the web shell?**

<p align="center">
  <img src=screenshots/web_uploadform.png width="500">
</p>

**Answer: upload_form.php**

---

**4. What is the first command run by the attacker using the newly uploaded web shell?**

<p align="center">
  <img src=screenshots/web_uploadform.png width="500">
</p>

```bash
cat /var/log/apache2/access.log | grep "cmd="
```

**Answer: whoami**

---

**5. After gaining access via the web shell, the attacker uses a command to download a second file onto the server. What is the name of this file?**

<p align="center">
  <img src=screenshots/web_linpeas.png width="500">
</p>

*Filtered with `grep "cmd="`*

**Answer: linpeas.sh**

---

**6. The attacker has hidden a secret within the web shell. Use `cat` to investigate the web shell code and find the flag.**

<p align="center">
  <img src=screenshots/web_catflag.png width="500">
</p>

```bash
cd /var/www/html/wordpress/wp-content/uploads
ls
cat shadyshell.php
```

**Answer: THM{W3b_Sh3ll_Int3rnals}**

---

## Task 7 - Conclusion

<!-- Q: Summarize the full detection workflow covered in this room: logs, file system, network, and behavioral indicators. -->
<!-- Q: What combination of tools gives the most complete picture of a web shell attack? -->
<!-- Q: How does correlating multiple log sources change the quality of a detection? -->

**1. Complete the room and continue on your cyber learning journey!**

**Answer:** N/A

---
