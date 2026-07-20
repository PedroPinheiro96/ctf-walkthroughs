# Brooklyn Nine Nine – TryHackMe

- Difficulty: easy
- Date: 2026-05-13
- Tags: ctf, linux, brute force, password guessing, privilege escalation, tryhackme

## 🧩 Overview

```text
This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.
```

## ⚙️ Tools Used

- Nmap
- Hydra

## Method 1 Walkthrough

### Task 1 - Deploy and get hacking

Target IP: 10.129.169.208

First, I enumerated the target using `nmap` to identify open ports, running services, and service versions.

<div align="center"><img src="../attachments/Pasted image 20260513120454.png"/></div>

The scan returned the following results:

- The server is running Ubuntu.
- Port 21 (**FTP**) is open and running **vsftp 3.0.3**.
- Port 22 (**SSH**) is open and running **OpenSSH 7.6p1**.
- Port 80 (**HTTP**) is open and running **Apache httpd 2.4.29**.

Since the server exposed an FTP service, I attempted to authenticate using the `anonymous` account.

Anonymous authentication was enabled, allowing access to the FTP server without credentials.

I listed the contents of the FTP directory and discovered `note_to_jake.txt`, which I then downloaded for further analysis.

<div align="center"><img src="../attachments/Pasted image 20260513120524.png"/></div>

The note indicates that Jake uses a weak password.
This suggests that a dictionary attack against the account may be successful.

<div align="center"><img src="../attachments/Pasted image 20260513120819.png"/></div>

Next, I used `Hydra` to perform dictionary attacks against Jake's FTP and SSH accounts.

Hydra did not recover valid FTP credentials, suggesting that either the FTP account used a stronger password or password authentication was not applicable for the exposed service.

<div align="center"><img src="../attachments/Pasted image 20260513125750.png"/></div>

I then targeted the SSH service, where Hydra successfully recovered Jake's password.

SSH: `987654321`

<div align="center"><img src="../attachments/Pasted image 20260513125813.png"/></div>

I connected to the target via SSH using Jake's credentials and listed the contents of the current directory, which was empty.

Then, I enumerated the contents of `/home` and its subdirectories and found the `user.txt` flag in the `/home/holt/` directory.

<div align="center"><img src="../attachments/Pasted image 20260513130023.png"/></div>

I used `cat` to print the contents of the file:

<div align="center"><img src="../attachments/Pasted image 20260513130056.png"/></div>

With the user flag obtained, I proceeded to enumerate potential privilege escalation vectors.

I started by checking which commands the user `jake` could run as `sudo`:

<div align="center"><img src="../attachments/Pasted image 20260513130139.png"/></div>

The output shows that `jake` can execute `/usr/bin/less` with `sudo` without providing a password.

Since `less` can be abused for privilege escalation, I checked [GTFOBins](https://gtfobins.org/gtfobins/less/#shell) and found the following technique:

<div align="center"><img src="../attachments/Pasted image 20260720120838.png"/></div>

I executed the GTFOBins technique on the target, successfully escalating my privileges to `root`.

I checked the `/root/` directory and found the `root.txt` flag.

<div align="center"><img src="../attachments/Pasted image 20260513131808.png"/></div>

## 🧠 Lessons Learned

- Anonymous FTP access can unintentionally expose sensitive information that assists further attacks.
- Weak passwords remain susceptible to dictionary attacks using tools such as Hydra.
- Misconfigured `sudo` permissions can provide straightforward privilege escalation paths.
- GTFOBins is an invaluable resource for identifying legitimate binaries that can be abused for privilege escalation.

## 🛡️ Mitigation

- Disable anonymous FTP access unless it is explicitly required.
- Enforce strong password policies and multi-factor authentication for remote services.
- Restrict SSH access to authorized users and consider rate limiting or fail2ban to mitigate brute-force attacks.
- Apply the principle of least privilege when configuring `sudo`, allowing only the minimum commands required.
- Regularly audit `sudo` rules and system configurations to identify privilege escalation opportunities.