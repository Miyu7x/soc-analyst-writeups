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

---

## Task 1 -- Introduction

Learning Objectives


---

## Task 2 -- Working With Text Logs

Log Format


Working With Logs


Filtering Logs


Discovering Logs


Logging Caveats


Which time server domain did the VM contact to sync its time?

**Answer:**

What is the kernel message from Yama in /var/log/syslog?

**Answer:**

---

## Task 3 -- Authentication Logs

Login and Logout Events


Local and Remote Logins


Cron and Sudo Logins


SSH-Specific Events


Miscellaneous Events


Commands Run With Sudo


| Login Type | Log Service | Key Keywords |
|---|---|---|
| Local (keyboard) | login | session opened/closed |
| Remote SSH | sshd | Accepted / Failed password |
| Remote SMB | smbd | session opened/closed |
| Cron job | CRON | session opened/closed |
| Sudo escalation | sudo | session opened / COMMAND= |

Which IP address failed to log in on multiple users via SSH?

**Answer:**

Which user was created and added to the "sudo" group?

**Answer:**

---

## Task 4 -- Common Linux Logs

Generic System Logs


App-Specific Logs


Bash History


| Log File | Purpose |
|---|---|
| /var/log/kern.log | Kernel messages and errors |
| /var/log/syslog | Consolidated system events |
| /var/log/dpkg.log | Package installs (Debian-based) |
| /var/log/dnf.log | Package installs (RHEL-based) |
| ~/.bash_history | Per-user interactive command history |

According to the VM's package manager logs, which version of unzip was installed?

**Answer:**

What is the flag in one of the users' bash history?

**Answer:**

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
