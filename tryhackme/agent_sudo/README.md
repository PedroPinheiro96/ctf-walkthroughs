# Agent Sudo – TryHackMe

- Difficulty: easy
- Date: 2026-05-11
- Tags: web, ftp, steganography, brute-force, privilege-escalation, sudo, cve, ssh, john, exploit-db

## 🧩 Overview

```
You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.
```

## ⚙️ Tools Used

- Nmap
- Gobuster
- Hydra
- [CyberChef](https://gchq.github.io/CyberChef/)
- [ExploitDB](https://www.exploit-db.com/)
- Exiftool
- 7z
- scp
- base64

## Walkthrough

### Task 1 - Author note

Deploy the machine.

Target IP Address: 10.129.150.50

### Task 2 - Enumerate

Enumerate the machine and get all the important information

#### Q1: How many open ports?

```shell-session
nmap -sS -sV -O -oN scan 10.129.150.50
```

<div align="center"><img src="../attachments/Pasted image 20260511140146.png"/></div>

The scan identified the following:

| Port | Service | Version             |
| ---- | ------- | ------------------- |
| 21   | FTP     | vsftpd 3.0.3        |
| 22   | SSH     | OpenSSH 7.6p1       |
| 80   | HTTP    | Apache httpd 2.4.29 |

```
-> 3
```

#### Q2: How you redirect yourself to a secret page?

After visiting the homepage at `http://10.129.150.50`, I saw the message below:

<div align="center"><img src="../attachments/Pasted image 20260511140329.png"/></div>

```
-> user-agent
```

#### Q3: What is the agent name?

The codename `R` is visible in the homepage message.

I started `Burp Suite`, added the domain to `scope`, and intercepted the `GET` request.

I attempted to brute-force possible codenames by intercepting and modifying the User-Agent header.

I sent the request to `Intruder`, modified the `user-agent` header, selected the `Sniper` attack mode and the `Brute forcer` payload.

<div align="centre"><img src="../attachments/Pasted image 20260511161946.png"/></div>

Except for requests with payloads `R` and `C`, all responses had identical content lengths. This indicates that specific `User-Agent` values expose hidden application functionalities.

<div align="center"><img src="../attachments/Pasted image 20260511142016.png"/></div>

The request with `user-agent: C` contained the `Location` header pointing to `agent_C_attention.php`.

<div align="center"><img src="../attachments/Pasted image 20260511142251.png"/></div>

I opened the server response in the browser and saw the following message:

<div align="center"><img src="../attachments/Pasted image 20260511142036.png"/></div>

The response revealed two key pieces of information:
- Username: `chris`
- The user `chris` has a weak account password. This is useful as it indicates the password may be weak and suitable for dictionary-based attacks.

The request with `user-agent: R` returned a different page.

<div align="center"><img src="../attachments/Pasted image 20260511141844.png"/></div>

Browser view:

<div align="center"><img src="../attachments/Pasted image 20260511141823.png"/></div>

The answer for the question was found on `http://10.129.150.50/agent_C_attention.php`

```
-> chris
```

### Task 3 - Hash cracking and brute-force

Done enumerate the machine? Time to brute your way out.

#### Q1: FTP password

Since it was known that the user `chris` had a weak password, I used `Hydra` to perform a dictionary attack against the `FTP` service.

```shell-session
hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.129.150.50 ftp
```

Hydra successfully recovered the FTP password.

<div align="center"><img src="../attachments/Pasted image 20260511142827.png"/></div>

I successfully authenticated to the FTP server using the `chris` account and password `crystal`. The directory contained 3 files.

<div align="center"><img src="../attachments/Pasted image 20260511143017.png"/></div>

```text
-> crystal
```

#### Q2: Zip file password

After downloading the files, I started by reading the `To_agentJ.txt` file.

<div align="center"><img src="../attachments/Pasted image 20260511143159.png"/></div>

According to the file, `chris`'s password is stored in one of the fake pictures.

I used `exiftool` to extract metadata from both picture files and found out that `cutie.png` contained embedded binary data.

<div align="center"><img src="../attachments/Pasted image 20260511143319.png"/></div>

I extracted the `extracted_at_0x8702.zip` archive from `cutie.png` after uploading it to [CyberChef](https://gchq.github.io/CyberChef/) and using the `Extract Files` recipe.

<div align="center"><img src="../attachments/Pasted image 20260511144148.png"/></div>

`unzip` was not able to extract the file so I used `7z`.
The `ZIP` archive was password-protected, and the password was not yet known.

<div align="center"><img src="../attachments/Pasted image 20260511144725.png"/></div>

To try to crack it, I used `zip2john` to convert the `zip` archive into a format that `john` could use and then used the `rockyou.txt` wordlist to crack the password.

<div align="center"><img src="../attachments/Pasted image 20260511144823.png"/></div>

I successfully extracted `To_agentR.txt` using the password `alien` that `john` cracked.

<div align="center"><img src="../attachments/Pasted image 20260511145148.png"/></div>

```text
-> alien
```

#### Q3: steg password

`To_agentR.txt` mentioned the fake pictures again and also contained an encoded string.
I decoded it using `base64` decoding and the message was decoded to `Area51`.

<div align="center"><img src="../attachments/Pasted image 20260511145402.png"/></div>

```text
-> Area51
```

#### Q4: Who is the other agent(in full name)?

`ExifTool` primarily extracts metadata and may not identify hidden embedded payloads or steganographic content.

To check whether additional data had been hidden in the image files, I used the `steghide` command:

<div align="center"><img src="../attachments/Pasted image 20260511145806.png"/></div>

Using `steghide`, I extracted `message.txt` from `cute-alien.jpg`.

This file contained important information:
- Agent J's name: `james`
- James' passowrd: `hackerrules!`

<div align="center"><img src="../attachments/Pasted image 20260511150107.png"/></div>

```text
-> james
```

#### Q5: SSH password

```text
-> hackerrules!
```

### Task 4 - Capture the user flag

You know the drill.

#### Q1: What is the user flag?

<div align="center"><img src="../attachments/Pasted image 20260511150451.png"/></div>

After connecting to the target machine through `SSH`, I checked if there were any files in the home directory of the user `james` and found `user.txt`.

<div align="center"><img src="../attachments/Pasted image 20260511150506.png"/></div>

```
-> b03d975e8c92a7c04146cfa7a5a313c7
```

#### Q2: What is the incident of the photo called?

To transfer the photo to the attacker machine, I used the `scp` command.

<div align="center"><img src="../attachments/Pasted image 20260511150820.png"/></div>

After transferring it, I searched for the image online and found a related Fox News article.

<div align="center"><img src="../attachments/Pasted image 20260511151006.png"/></div>

```text
-> Roswell alien autopsy
```

### Task 5 - Privilege escalation

#### Q1: CVE number for the escalation

Then, I checked the commands that the user `james` could run as `sudo`.

<div align="center"><img src="../attachments/Pasted image 20260511151532.png"/></div>

I found an exploit on [ExploitDB](https://www.exploit-db.com/) after researching the sudo configuration `(ALL, !root) /bin/bash`:

<div align="center"><img src="../attachments/Pasted image 20260511152301.png"/></div>

```
-> cve-2019-14287
```

#### Q2: What is the root flag?

I copied the exploit from [ExploitDB](https://www.exploit-db.com/) and executed it on the target machine.

<div align="center"><img src="../attachments/Pasted image 20260511152526.png"/></div>

The exploit bypassed the intended sudo restriction and spawned a `root` shell.

<div align="center"><img src="../attachments/Pasted image 20260511152559.png"/></div>

```
-> b53a02f55b57d4439e3341834d70c062
```

#### Q3: (Bonus) Who is Agent R?

```
-> DesKel
```

## 🧠 Lessons Learned

- User-controlled HTTP headers such as `User-Agent` can expose hidden application functionality when improperly validated.
- Metadata analysis and steganography techniques can reveal hidden files and sensitive information embedded within media files.
- Publicly known vulnerabilities such as `CVE-2019-14287` can lead to privilege escalation when systems are misconfigured or unpatched.
- Misconfigured `sudo` permissions can completely undermine privilege separation controls.

## 🛡️ Mitigation

- Avoid relying on obscurity mechanisms such as hidden pages triggered through HTTP headers.
- Enforce strong password policies and implement account lockout protections against brute-force attacks.
- Restrict unnecessary public access to services such as FTP and prefer secure alternatives.
- Keep systems and software updated to patched versions to mitigate known vulnerabilities.
- Follow the principle of least privilege when configuring `sudo` permissions.
- Monitor authentication logs and unusual enumeration activity for signs of brute-force attacks.
