# RootMe – TryHackMe

- Difficulty: easy
- Date: 2026-05-08
- Tags: linux, ctf, nmap, python

## 🧩 Overview

```text
A ctf for beginners, can you root me?
```

## ⚙️ Tools Used

- Nmap
- Gobuster
- [GTFOBins](https://gtfobins.org/)
- Python

## Walkthrough

### Task 1 - Deploy the machine

Connect to TryHackMe network and deploy the machine.

### Task 2 - Reconnaissance

First, let's get information about the target.

#### Q1: Scan the machine, how many ports are open?

I began by enumerating the server using Nmap to look for open ports and running services:

```shell-session
nmap -sS -sV -O 10.112.172.227 -oN scan
```

<div align="center"><img src="../attachments/Pasted image 20260508134841.png"/></div>

The scan identified the following:
- The server is running `OpenSSH 8.2p1` on port 22 (**SSH**).
- Port 80 (**HTTP**) is open and is running `Apache httpd 2.4.41`

```text
-> 2
```

#### Q2: What version of Apache is running?

```text
-> 2.4.41
```

#### Q3: What service is running on port 22?

```text
-> ssh
```

#### Q4: Find directories on the web server using the Gobuster tool.

To perform directory enumeration, `gobuster` was used with the `directory-list-2.3-medium.txt` wordlist.

```text
gobuster dir -u 10.112.172.227 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r
```

<div align="center"><img src="../attachments/Pasted image 20260508135528.png"/></div>

#### Q5: What is the hidden directory?

```text
/panel/
```

### Task 3 - Getting a shell

Find a form to upload and get a reverse shell, and find the flag.

#### Q1: user.txt

The `panel` directory lets users upload files to the server. This can be used to upload malicious payloads. One example is a `reverse shell` payload to get a shell on the target machine.

<div align="center"><img src="../attachments/Pasted image 20260508135621.png"/></div>

First, I edited the the `php-reverse-shell.php` located at `/usr/share/webshells/php-reverse-shell.php` with the IP of my machine and the port I wanted the reverse shell to connect back to.

<div align="center"><img src="../attachments/Pasted image 20260508140201.png"/></div>

Then, I uploaded the `php-reverse-shell.php` located at `/usr/share/webshells/php-reverse-shell.php`.

<div align="center"><img src="../attachments/Pasted image 20260508135939.png"/></div>

That displayed a red button with text in Portuguese that means "PHP is not allowed!".

<div align="center"><img src="../attachments/Pasted image 20260508135952.png"/></div>

I renamed the reverse shell file to `php-reverse-shell.php5` and tried to upload it again.

<div align="center"><img src="../attachments/Pasted image 20260508135839.png"/></div>

This time a green button was displayed. The text means "The archive has been successfully uploaded!".

The sanitization was not properly configured and I was able to upload the file.

Now that the reverse shell has been uploaded to the target machine, I configured a listener on port `11111`. To execute the reverse shell, I navigated to `http://10.112.172.227/uploads/php-reverse-shell.php5`.

<div align="center"><img src="../attachments/Pasted image 20260508140426.png"/></div>

The payload was successfully executed and connected back to my machine, returning a non-interactive shell.

Python is usually used to escalate privileges. I confirmed it was installed and run the following line to execute python code:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<div align="center"><img src="../attachments/Pasted image 20260508141504.png"/></div>

After spawning a `bash` shell with `Python`, I stabilised the session as it's seen below.

<div align="center"><img src="../attachments/Pasted image 20260508141553.png"/></div>

I was now able to use all the features of the `bash` shell and it would be easier to perform post-exploitation actions.

After navigating to the home directory of the `www-data` user, I was able to find the first flag `user.txt`.

<div align="center"><img src="../attachments/Pasted image 20260508141634.png"/></div>

```text
-> THM{y0u_g0t_a_sh3ll}
```

### Task 4 - Privilege Escalation

Now that we have a shell, let's escalate our privileges to root.

#### Q1: Search for files with SUID permission, which file is weird?

`SUID` is a special type of permissions that enables users to execute files as the user who owns the file. This can be used by malicious actors to escalate privileges.

The numeric representation of `SUID` is 4. The `find` command can look for files that have the `SUID` bit set:

```shell-session
find / -perm -4000 2>/dev/null
```

<div align="center"><img src="../attachments/Pasted image 20260508144737.png"/></div>

This is bad practice.

Python should not have the `SUID` bit set because any user who runs `/usr/bin/python2.7` will be able to run it with `root` privileges (because the program is owned by `root`).

```text
-> python
```

#### Q2: Find a form to escalate your privileges.

[[|GTFOBins"/></div> contains a list of unix executables that can be used to bypass local security restrictions in misconfigured systems.

I found the following executable to escalate privileges using python with the `SUID` bit set.

<div align="center"><img src="../attachments/Pasted image 20260508151239.png"/></div>

```shell-session
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

This python process is going to be executed with `root` privileges due to the `SUID` bit set on `/usr/bin/python2.7` and it being owned by `root`.
- `/bin/sh` is the executable being replaced into the current process. (It's important to maintain the current process because we can use the `-p` flag)
- `sh` is the process name as seen by the system.
- `-p` ensures the shell does not drop elevated **effective privileges** inherited from `SUID` execution. `root` privileges in this case because `/usr/bin/python2.7` is owned by root and has the `SUID` set.

This Python code replaces the current process with `/bin/sh`. The `-p` flag is important here because we need to preserve the **effective privileges (SUID privileges)** of the user that is calling it. 

In this case it is `root` because that is the user that owns the `/usr/bin/python2.7` program and the `SUID` bit is set. This means that when `www-data` runs the code, it will be executed as `root`.

Privilege Escalation was accomplished by running the executable and I had now access to a `root` shell.

```shell-session
-> /usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

<div align="center"><img src="../attachments/Pasted image 20260508152933.png"/></div>

#### Q3: root.txt

After enumerating the home directory of the `root` user, I found the last flag `root.txt`.

<div align="center"><img src="../attachments/Pasted image 20260508153003.png"/></div>

```shell-session
-> THM{pr1v1l3g3_3sc4l4t10n}
```

## 🧠 Lessons Learned

- File upload vulnerabilities can lead directly to remote code execution if server-side validation is insufficient.  
- SUID misconfigurations on powerful binaries (such as Python) can lead to full privilege escalation.  
- Directory enumeration is critical for discovering hidden attack surfaces such as admin panels or upload endpoints.  
- Shell stabilization techniques (PTY spawning) significantly improve post-exploitation usability and control.  
- Tools like GTFOBins are valuable for identifying privilege escalation paths from misconfigured binaries.

## 🛡️ Mitigation

- Restrict file uploads and enforce strict server-side validation (file type, extension, and content inspection).  
- Disable execution of uploaded files in web-accessible directories.  
- Avoid assigning SUID to interpreters or high-level scripting languages (e.g., Python, Perl, Bash).  
- Regularly audit the system for SUID binaries using tools like `find / -perm -4000`.  
- Follow least privilege principles for file permissions and service accounts.  
- Keep web applications and server software updated to patch known vulnerabilities.  
- Monitor web server logs for suspicious upload and execution activity.
