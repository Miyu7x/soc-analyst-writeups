---
title: Linux Logging for SOC
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, logging, auth-log, auditd, syslog, bash-history, runtime-monitoring]
status: in-progress
date: 2026-06-26
date_completed:
---
 
*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*
<p align="center">
<img src=screenshots/linux_intro.png width="1500">
</p>
---

## Task 1 -- Introduction

### Learning Objectives

Linux authentication, runtime, and system logs: how to view them directly on the host and how Linux logs are sent to SIEM.

---

## Task 2 -- Working With Text Logs

### Log Format

The log formatting will be viewed on Linux servers without a GUI, no desktop logging.

### Working With Logs

Unlike Windows logs, Linux logs are plain .txt files.
 - Logs can be viewed with any text editor
    - This means that the logs will be unstructured because they have no formatting rules, which makes them hard to read
    - Most Linux logs are located in the /var/log folder
    - /var/log/syslog logs various system events

---

**Syslog File Content Example**

root@thm-vm:~$ cat /var/log/syslog | head
[...]
2025-08-13T13:57:49.388941+00:00 thm-vm systemd-timesyncd[268]: Initial clock synchronization to Wed 2025-08-13 13:57:49.387835 UTC.
2025-08-13T13:59:39.970029+00:00 thm-vm systemd[888]: Starting dbus.socket - D-Bus User Message Bus Socket...
2025-08-13T14:02:22.606216+00:00 thm-vm dbus-daemon[564]: [system] Successfully activated service 'org.freedesktop.timedate1'
2025-08-13T14:05:01.999677+00:00 thm-vm CRON[1027]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
[...]

---

### Filtering Logs

There are thousands of events on a syslog, but only a few are useful for SOC.
 - It's essential to filter and narrow down as much as possible
 - For example: using the **grep** command to filter for the "CRON" keyword would yield only the **cronjob** logs

---

**CRON Log Filtering Example**
# Or "grep -v CRON" to exclude "CRON" from results
root@thm-vm:~$ cat /var/log/syslog | grep CRON
2025-08-13T14:17:01.025846+00:00 thm-vm CRON[1042]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
2025-08-13T14:25:01.043238+00:00 thm-vm CRON[1046]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
2025-08-13T14:30:01.014532+00:00 thm-vm CRON[1048]: (root) CMD (date > mycrondebug.log)

---

### Discovering Logs

Searching for **all users logins** you can filter them with grep and keywords such as "login", "auth", or "session"

---

**Example of Filtering by User Login**

# List what's logged by your system (/var/log folder) 
root@thm-vm:~$ ls -l /var/log
drwxr-xr-x  2 root      root               4096 Aug 12 16:41 apt
drwxr-x---  2 root      adm                4096 Aug 12 12:40 audit
-rw-r-----  1 syslog    adm               45399 Aug 13 15:05 auth.log
-rw-r--r--  1 root      root            1361277 Aug 12 16:41 dpkg.log
drwxr-sr-x+ 3 root      systemd-journal    4096 Oct 22  2024 journal
-rw-r-----  1 syslog    adm              214772 Aug 13 13:57 kern.log
-rw-r-----  1 syslog    adm              315798 Aug 13 15:05 syslog
[...]

# Search for potential logins across all logs (/var/log)
root@thm-vm:~$ grep -R -E "auth|login|session" /var/log
[...]

---

### Logging Caveats

Linux allows you to easily change log format, verbosity, and storage location

---

1. Which time server domain did the VM contact to sync its time?

   <p align="center">
<img src=screenshots/linux_domain.png width="700">
</p>
Used the command: ubuntu@thm-vm:~$ cat /var/log/syslog | grep timesync

**Answer: ntp.ubuntu.com**

---

2. What is the kernel message from Yama in /var/log/syslog?

   <p align="center">
<img src=screenshots/linux_mindful.png width="700">
</p>
Command: cat /var/log/syslog | grep "Yama" to filter out by the specific word "Yama"

**Answer: Becoming mindful.**

---

## Task 3 -- Authentication Logs

Linux's most useful log file can be found: /var/log/auth.log
 - Stores authentication events, management events, and launched sudo commands

**Log File Format Example**  
   <p align="center">n 
<img src=screenshots/linux_logformatexample.png width="700">
</p>

### Login and Logout Events

Each successful logon and logoff is logged, this log also includes: 
 - Sessions opened and closed
    - command: **cat /var/log/auth.log | grep -E 'session opened|session closed'**
 - Cron and Sudo logins
    - command: **cat /var/log/auth.log | grep -E 'session opened|session closed'**
 - Succesful and Failed SSH logins
    - command: **cat /var/auth.log | grep "sshd" | grep -E 'Accepted|Failed'**
 - User management events(useradd, pass change, escalation...)
    - command: **cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['**
 - Any command ran with **sudo**
    - command: **cat /var/log/auth.log | grep -E 'COMMAND='**
 - /var/log/auth.log

**Examples:**

**Local and Remote Logins**
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed'
# Local, on-keyboard login and logout of Bob (login:session)
2025-08-02T16:04:43 thm-vm login[1138]: pam_unix(login:session): session opened for user bob(uid=1001) by bob(uid=0)
2025-08-02T19:23:08 thm-vm login[1138]: pam_unix(login:session): session closed for user bob
# Remote login examples of Alice (via SSH and then SMB)
2025-08-04T09:09:06 thm-vm sshd[839]: pam_unix(sshd:session): session opened for user alice(uid=1002) by alice(uid=0)
2025-08-04T12:46:13 thm-vm smbd[1795]: pam_unix(samba:session): session opened for user alice(uid=1002) by alice(uid=0)

**Cron and Sudo Logins**
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed'
# Traces of some cron job launch running as root (cron:session)
2025-08-06T19:35:01 thm-vm CRON[41925]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2025-08-06T19:35:01 thm-vm CRON[3108]: pam_unix(cron:session): session closed for user root
# Carol running "sudo su" to access root (sudo:session)
2025-08-07T09:12:32 thm-vm sudo: pam_unix(sudo:session): session opened for user root(uid=0) by carol(uid=1003)

**SSH Specific Events**
root@thm-vm:~$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'
# Common SSH log format: <is-successful> <auth-method> for <user> from <ip>
2025-08-07T11:21:25 thm-vm sshd[3139]: Failed password for root from 222.124.17.227 port 50293 ssh2
2025-08-07T14:17:40 thm-vm sshd[3139]: Failed password for admin from 138.204.127.54 port 52670 ssh2
2025-08-09T20:30:51 thm-vm sshd[1690]: Accepted publickey for bob from 10.19.92.18 port 55050 ssh2: <key>
   
**User Management Events**
root@thm-vm:~$ cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['
2023-02-01T11:09:55 thm-vm passwd[644]: password for 'ubuntu' changed by 'root'
2025-08-07T22:11:11 thm-vm userdel[1887]: delete user 'oldbackdoor'
2025-08-07T22:11:29 thm-vm useradd[1878]: new user: name=backdoor, UID=1002, GID=1002, shell=/bin/sh
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to group 'sudo'
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to shadow group 'sudo'

**Commands Run With Sudo**
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'COMMAND='
2025-08-07T11:21:49 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/systemctl stop edr
2025-08-07T11:23:18 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/ufw status numbered
2025-08-07T11:23:33 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/su

---

| Login Type | Log Service | Key Keywords |
|---|---|---|
| Local (keyboard) | login | session opened/closed |
| Remote SSH | sshd | Accepted / Failed password |
| Remote SMB | smbd | session opened/closed |
| Cron job | CRON | session opened/closed |
| Sudo escalation | sudo | session opened / COMMAND= |

---

1. Which IP address failed to log in on multiple users via SSH?

   <p align="center">n 
<img src=screenshots/linux_logformatexample.png width="700">
</p>
To find the specific failed ssh logs only command: **cat /var/log/auth.log | grep "sshd" | grep -E 'Failed'**

**Answer: 10.14.94.82**

---

2. Which user was created and added to the "sudo" group?

   <p align="center">n 
<img src=screenshots/linux_xerxes.png width="700">
</p>
To find user management events we can filter the log by: **cat /var/log/auth.log | grep -E '(useradd|usermod)\[**

**Answer: xerxes**

---

## Task 4 -- Common Linux Logs

### Generic System Logs

Linux logs offer a variety of other information:
 - Kernel messages and errors
    - /var/log/kern.log
 - Various Linux events
    - /var/log/syslog
 - Package manager logs
    - /var/log/dpkg.log
 - Package manager logs
    - /var/log/dnf.log

### App-Specific Logs

App-specific logs help SOCs monitor specific programs, such as database, mail, container, and web server logs.

**Example of Nginx Web Server Logs**
root@thm-vm:~$ cat /var/log/nginx/access.log
# Every log line corresponds to a web request to the web server
10.0.1.12 - - [11/08/2025:14:32:10 +0000] "GET / HTTP/1.1" 200 3022
10.0.1.12 - - [11/08/2025:14:32:14 +0000] "GET /login HTTP/1.1" 200 1056
10.0.1.12 - - [11/08/2025:14:33:09 +0000] "POST /login HTTP/1.1" 302 112
10.0.4.99 - - [11/08/2025:17:11:20 +0000] "GET /images/logo.png HTTP/1.1" 200 5432
10.0.5.21 - - [11/08/2025:17:56:23 +0000] "GET /admin HTTP/1.1" 403 104

### Bash History

An interesting log source is bash history; every command is logged the moment you press enter.
 - Bash history is stored in memory and, written to ~/.bash_history once the user logs out
 - This is not very useful, though, as attackers can simply hide their commands and avoid being logged completely.

**Bash History File and Command Example**
ubuntu@thm-vm:~$ cat /home/ubuntu/.bash_history
echo "hello" > world.txt
nano /etc/ssh/sshd_config
sudo su
ubuntu@thm-vm:~$ history
1 echo "hello" > world.txt
2 nano /etc/ssh/sshd_config
3 sudo su
4 ls -la /home/ubuntu
5 cat /home/ubuntu/.bash_history
6 history


| Log File | Purpose |
|---|---|
| /var/log/kern.log | Kernel messages and errors |
| /var/log/syslog | Consolidated system events |
| /var/log/dpkg.log | Package installs (Debian-based) |
| /var/log/dnf.log | Package installs (RHEL-based) |
| ~/.bash_history | Per-user interactive command history |

---

1. According to the VM's package manager logs, which version of unzip was installed?

   <p align="center">n 
<img src=screenshots/linux_unzip.png width="700">
</p>

**Answer: 6.0-28ubuntu4.1**

---

2. What is the flag in one of the users' bash history?

    <p align="center">n 
<img src=screenshots/linux_notetoremember.png width="700">
</p>
Ran the command:
ubuntu@thm-vm:~$ cat /home/ubuntu/.bash_history
sudo su
sudo su
sudo su
sudo apt install zip unzip
passwd

From this output we can see the user was in root, and there is no trace of a THM flag. Escalated privileges to root with **sudo su** in the bash I utilized the arrow keys to see if any commands were stored in memory and observed the flag. 

**Answer: THM{note_to_remember}**

---

## Task 5 -- Runtime Monitoring

Runtime Monitoring


System Calls


Which Linux system call is commonly used to execute a program?

**Answer:**

Can a typical program open a file or create a process bypassing system calls? (Yea/Nay)

**Answer:**

---

## Task 6 -- Using Auditd

Audit Daemon


Using Auditd


File Events


Auditd Alternatives


| auditd Field | Meaning |
|---|---|
| pid / ppid | Process ID and Parent Process ID |
| auid | Audit user -- original login account |
| uid | User who ran the command (may differ after sudo/su) |
| tty | Session identifier |
| exe | Absolute path to the executed binary |
| key | Tag from auditd rule for filtering |

When was the secret.thm file opened for the first time? (MM/DD/YY HH:MM:SS)

**Answer:**

What is the original file name downloaded from GitHub via wget?

**Answer:**

Which network range was scanned using the downloaded tool?

**Answer:**

---

## Task 7 -- Conclusion

Key Takeaways


**Answer:** N/A
