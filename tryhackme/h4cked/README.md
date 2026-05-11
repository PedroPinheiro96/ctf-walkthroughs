# h4cked – TryHackMe

- Difficulty: easy
- Date: 2026-05-04
- Tags: wireshark, network traffic, pcap, ftp

## 🧩 Overview

```text
Find out what happened by analysing a .pcap file and hack your way back into the machine
```

## ⚙️ Tools Used

- Wireshark
- Nmap
- Hydra
- Netcat

## Walkthrough

### Task 1

It seems like our machine got hacked by an anonymous threat actor. However, we are lucky to have a .pcap file from the attack. Can you determine what happened? Download the .pcap file and use Wireshark to view it.

#### Q1: It Seems like Our Machine Got Hacked by an Anonymous Threat Actor. However, We Are Lucky to Have a .pcap File from the Attack. Can You Determine what Happened? Download the .pcap File and Use Wireshark to View It.

```text
-> N/A
```

#### Q2: The Attacker is Trying to Log into a Specific Service. What Service is This?

<div align="center"><img src="../attachments/Pasted image 20260504040539.png"/></div>

```text
-> By examining the .pcap file, it's possible to see multiple connections to port 21.
Port 21 is used by FTP (File Transfer Protocol).
```

#### Q3: There is a Very Popular Tool by Van Hauser Which Can Be Used to Brute Force a Series of Services. What is the Name of This Tool?

```text
-> Hydra
```

#### Q4: The Attacker is Trying to Log on with a Specific Username. What is the Username?

Right-click on the first packet -> Follow -> TCP Stream

<div align="center"><img src="../attachments/Pasted image 20260504032036.png"/></div>

```text
-> Jenny
```

#### Q5: What is the User's Password?

Keep following the TCP Stream until a successful message is seen. The user logs in on Stream 7.

<div align="center"><img src="../attachments/Pasted image 20260504032132.png"/></div>

```text
-> password123
```

#### Q6: What is the Current FTP Working Directory after the Attacker Logged In?

Stream 16:

<div align="center"><img src="../attachments/Pasted image 20260504032237.png"/></div>

```text
-> /var/www/html
```

#### Q7: The Attacker Uploaded a Backdoor. What is the Backdoor's Filename?

Stream 16:

<div align="center"><img src="../attachments/Pasted image 20260504032306.png"/></div>

```text
-> shell.php
```

#### Q8: The Backdoor Can Be Downloaded from a Specific URL, as it is Located inside the Uploaded File. What is the Full URL?

Stream 18:

<div align="center"><img src="../attachments/Pasted image 20260504032422.png"/></div>

```text
-> http://pentestmonkey.net/tools/php-reverse-shell
```

#### Q9: Which Command Did the Attacker Manually Execute after Getting a Reverse Shell?

Stream 20:

<div align="center"><img src="../attachments/Pasted image 20260504032507.png"/></div>

```text
-> whoami
```

#### Q10: What is the Computer's Hostname?

Stream 20: Possible to see on the prompt line.

<div align="center"><img src="../attachments/Pasted image 20260504032604.png"/></div>

```text
-> wir3
```

#### Q11: Which Command Did the Attacker Execute to Spawn a New TTY Shell?

Stream 20: The attacker upgraded the non-interactive shell into a fully interactive TTY using Python's `pty.spawn()` method.

<div align="center"><img src="../attachments/Pasted image 20260504032641.png"/></div>

```shell-session
-> python3 -c 'import pty; pty.spawn("/bin/bash")'
```

#### Q12: Which Command Was Executed to Gain a Root Shell?

Stream 20. The attacker checks which commands jenny can run as sudo and finds out that all the commands can be run as sudo.

<div align="center"><img src="../attachments/Pasted image 20260504032735.png"/></div>

```shell-session
-> sudo su
```

#### Q13: The Attacker Downloaded Something from GitHub. What is the Name of the GitHub Project?

Stream 20:

<div align="center"><img src="../attachments/Pasted image 20260504032906.png"/></div>

```text
-> Reptile
```

#### Q14: The Project Can Be Used to Install a Stealthy Backdoor on the System. It Can Be Very Hard to Detect. What is This Type of Backdoor Called?

```text
-> Rootkit
```

### Task 2

#### Q1: The Attacker Has Changed the User's Password! Can You Replicate the Attacker's Steps and Read the flag.txt? The Flag is Located in the /root/Reptile Directory. Remember, You Can Always Look back at the .pcap File if Necessary. Good Luck!

First let's confirm which ports are open and what services are running on the target by running nmap.

```shell-session
nmap -sS -sV -oN nmap_scan 10.128.137.224
```

<div align="center"><img src="../attachments/Pasted image 20260504033325.png"/></div>

FTP is running. Let's use Hydra to run a dictionary attack to guess the password:

#### Q2: Run Hydra (or Any Similar tool) on the FTP Service. The Attacker Might Not Have Chosen a Complex Password. You Might Get Lucky if You Use a Common Word List.

```shell-session
hydra -l jenny -P /usr/share/wordlists/rockyou.txt ftp://10.128.137.224
```

<div align="center"><img src="../attachments/Pasted image 20260504033527.png"/></div>

```text
-> The password for the jenny account is 987654321
```

I was able to log in as `jenny` with the password `987654321`.

<div align="center"><img src="../attachments/Pasted image 20260504033858.png"/></div>

#### Q3: Change the Necessary Values inside the Web Shell and Upload it to the Webserver

I've found a `shell.php` file in the directory and downloaded it.

<div align="center"><img src="../attachments/Pasted image 20260504034253.png"/></div>

Then I've opened the file and updated the values of `$ip` and `$port` to the IP address of my machine and port 33333.

<div align="center"><img src="../attachments/Pasted image 20260504034554.png"/></div>

#### Q4: Create a Listener on the Designated Port on Your Attacker Machine. Execute the Web Shell by Visiting the .php File on the Targeted Web Server.

I've saved the file and uploaded it to the FTP server. I've also started a listener on port `13131`

```shell-session
nc -lnvp 13131
```

<div align="center"><img src="../attachments/Pasted image 20260504034857.png"/></div>

After uploading the modified web shell, I executed it by visiting `http://10.128.137.224/shell.php` and the reverse shell was executed, which returned a non-interactive reverse shell connection.

<div align="center"><img src="../attachments/Pasted image 20260504035059.png"/></div>

I verified that Python 3 was installed on the target system and run a python script to upgrade to an interactive shell:

```shell-session
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<div align="center"><img src="../attachments/Pasted image 20260504035702.png"/></div>

#### Q5: Become Root!

I've switched to the `jenny` user account and checked which commands I could run as sudo and saw that I could run all the commands as sudo.
Then I run `sudo su` to switch to the root account.

<div align="center"><img src="../attachments/Pasted image 20260504040118.png"/></div>

#### Q6: Read the flag.txt File inside the Reptile Directory

<div align="center"><img src="../attachments/Pasted image 20260504040243.png"/></div>

## 🚩 Flag

```
ebcefd66ca4b559d17b440b6e67fd0fd
```

## 🧠 Lessons Learned

- PCAP analysis can reconstruct an attacker's full intrusion workflow
- FTP traffic exposes credentials in plaintext when unencrypted
- Reverse shells can often be upgraded into fully interactive TTY sessions
- Misconfigured sudo permissions can lead to immediate privilege escalation
- Rootkits are designed to maintain stealthy persistence on compromised systems

## 🛡️ Mitigation

- Disable or restrict FTP in favour of secure alternatives such as SFTP
- Enforce strong password policies to prevent brute-force attacks
- Implement account lockout and rate limiting protections
- Monitor for suspicious authentication attempts
- Restrict sudo privileges using the principle of least privilege
- Deploy endpoint monitoring to detect reverse shells and rootkits
- Use encrypted protocols to prevent credential exposure in network traffic
