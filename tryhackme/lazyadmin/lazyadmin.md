# LazyAdmin – TryHackMe

- Difficulty: easy
- Date: 2026-05-06
- Tags: tryhackme, ctf, enumeration

## 🧩 Overview

```text
Easy linux machine to practice your skills
```

## ⚙️ Tools Used

- Gobuster
- Nmap
- Hashcat
- [Hash Identifier](https://hashes.com/en/tools/hash_identifier)
- Searchsploit

## Walkthrough

### Task 1 - Lazy Admin

#### Q1: What is the User Flag?

We start by scanning the server for open ports and services.

```shell-session
nmap -sS -sV -O -oN scan 10.113.142.91
```

<div align="center"><img src="attachments/Pasted image 20260506133734.png"/></div>

By the scan results, we can see:
- The server is running **Ubuntu**
- Port 22 (**SSH**) is open -> **OpenSSH 7.2p2**
- Port 80 (**HTTP**) is open -> **Apache httpd 2.4.18**

Accessing port 80 shows the default Apache page.

<div align="center"><img src="attachments/Pasted image 20260506133826.png"/></div>

Next, I've used `gobuster` to enumerate directories.

```shell-session
gobuster dir -u 10.113.142.91 -w /usr/share/wordlists/dirb/common.txt -r
```

<div align="center"><img src="attachments/Pasted image 20260506133931.png"/></div>

This reveals a `/content` directory.

<div align="center"><img src="attachments/Pasted image 20260506133952.png"/></div>

This page gives us a bit more context but there isn't anything else other than informational text.

I continued enumeration against the `/content` directory:

```shell-session
gobuster dir -u http://10.113.142.91/content -w /usr/share/wordlists/dirb/common.txt -r
```

This returns a few more directories:

<div align="center"><img src="attachments/Pasted image 20260506134054.png"/></div>

`http://10.113.142.91/content/as` is a login page:

<div align="center"><img src="attachments/Pasted image 20260506134144.png"/></div>

`http://10.113.142.91/content/inc` contains multiple files and directories.
An interesting directory is `mysql_backup`.

<div align="center"><img src="attachments/Pasted image 20260506134623.png"/></div>

Inside the `mysql_backup` directory there's a `mysql_backup*` file.

<div align="center"><img src="attachments/Pasted image 20260506134604.png"/></div>

After downloading it, I found an `INSERT INTO` statement that contained credentials:

<div align="center"><img src="attachments/Pasted image 20260506135745.png"/></div>

- Admin user: `manager`
- Hashed password: `42f749ade7f9e195bf475f37a44cafcb`

An online hash identifier identifies it as an `MD5` hash.

<div align="center"><img src="attachments/Pasted image 20260506122940.png"/></div>

Next, I've used `Hashcat` to crack the password hash.

```shell-session
hashcat -m 0 -a 0 42f749ade7f9e195bf475f37a44cafcb /usr/share/wordlists/rockyou.txt
```

After a while, `Hashcat` returns the cracked hash **Password123**:

<div align="center"><img src="attachments/Pasted image 20260506140245.png"/></div>

We should now be able to log in into SweetRice's admin panel:

- Account: `manager`
- Password: `Password123`

<div align="center"><img src="attachments/Pasted image 20260506140457.png"/></div>

It worked!

<div align="center"><img src="attachments/Pasted image 20260506140604.png"/></div>

The admin dashboard disclosed that the target was running SweetRice `1.5.1`.

Since the CMS version was exposed, I searched for known public exploits targeting `SweetRice 1.5.1` using `Searchsploit`.

<div align="center"><img src="attachments/Pasted image 20260506141319.png"/></div>

Let's try to upload a PHP reverse shell to the target.

First, I copied a PHP reverse shell from `/usr/share/webshells/php/php-reverse-shell.php` to `/tml/lazyadmin/rev_shell.php5` and updated the variables `$ip` and `$port` to match my attacker machine
.

<div align="center"><img src="attachments/Pasted image 20260506144727.png"/></div>

Next, I downloaded the second exploit returned by searchsploit: `php/webapps/40716.py`.

<div align="center"><img src="attachments/Pasted image 20260506145011.png"/></div>

After downloading it, I've executed it with `python3 40716.py` and entered the target machine IP and the admin credentials that I obtained earlier.

<div align="center"><img src="attachments/Pasted image 20260506145855.png"/></div>

The exploit abused an unrestricted file upload vulnerability in SweetRice, allowing arbitrary PHP files to be uploaded to the web server.

We can view the file by navigating to `http://10.113.142.91/content/attachment/`.

<div align="center"><img src="attachments/Pasted image 20260506150003.png"/></div>

Now that the reverse shell has been uploaded, we need to start a listener on the attacker machine on the port specified on `rev_shell.php5`.

```shell-session
nc -lnvp 1337
```

After starting the listener, I executed the `rev_shell.php5` file by accessing the endpoint on the browser.

Executing the uploaded PHP shell triggered a reverse shell connection back to my Netcat listener.

<div align="center"><img src="attachments/Pasted image 20260506150331.png"/></div>

The first thing I did was check which commands can be run as root by `www-data`:

<div align="center"><img src="attachments/Pasted image 20260506150911 1.png"/></div>

This means we can run `/usr/bin/perl /home/itguy/backup.pl` as root and without a password. This might be important for later.

Let's see what's inside `/home/itguy`.

<div align="center"><img src="attachments/Pasted image 20260506151008.png"/></div>

`user.txt` is in the directory. That's one of the flags!
Next, I've printed out its value:

<div align="center"><img src="attachments/Pasted image 20260506151106.png"/></div>

#### Q2: What is the Root Flag?

Let's go back to the commands the commands `wwww-data` can run as sudo:

- `/usr/bin/perl /home/itguy/backup.pl`

The file is a Perl script.
First, let's see what it does:

<div align="center"><img src="attachments/Pasted image 20260506151501.png"/></div>

This script invokes `sh` and passes `/etc/copy.sh` as the script to run.

<div align="center"><img src="attachments/Pasted image 20260506152207.png"/></div>

Others can **read, write and execute** it. This is a critical misconfiguration.

Next, I checked what is the content of `/etc/copy.sh`

<div align="center"><img src="attachments/Pasted image 20260506152027.png"/></div>

This is a backdoor. Not important for the challenge.

Let's spawn an interactive shell by editing `/etc/copy.sh`.
To do this, let's add the path `/bin/bash` to the file.

<div align="center"><img src="attachments/Pasted image 20260506152510.png"/></div>

Now, let's run it by calling `backup.pl` as sudo.

<div align="center"><img src="attachments/Pasted image 20260506152801.png"/></div>

We are now `root`!

Since `/etc/copy.sh` was executed as root through the Perl script, appending `/bin/bash` caused a root shell to spawn when the script executed.

I've moved to the home directory and checked its contents.
The file `root.txt` is there.

<div align="center"><img src="attachments/Pasted image 20260506152923.png"/></div>

## 🚩 Flag

```

```

## 🧠 Lessons Learned

- **Always enumerate thoroughly**
    Hidden directories and backup files often contain sensitive data.
- **Weak password storage is dangerous**
    Using unsalted MD5 makes password cracking trivial.
- **Check file permissions carefully**
    World-writable files in privileged execution paths are a major risk.
- **Sudo misconfigurations are critical**
    Allowing script execution as root without restrictions can lead to full compromise.
- **Look for chaining opportunities**
    No single issue led to root—multiple small weaknesses combined did.

## 🛡️ Mitigation

- **Secure file permissions**
    - Avoid world-writable files, especially files that can be run as root
    - Apply least privilege (`chmod 600/700` where appropriate)
- **Harden sudo configurations**
    - Avoid allowing scripts to run as root
- **Protect sensitive files**
    - Never expose backup files via web directories
    - Store backups securely and outside web root
- **Use strong password hashing**
    - Replace MD5 with modern algorithms
- **Monitor for backdoors**
    - Regularly audit scripts and cron jobs
    - Use intrusion detection systems
- **Patch and update software**
    - Keep CMS platforms (like SweetRice) up to date
