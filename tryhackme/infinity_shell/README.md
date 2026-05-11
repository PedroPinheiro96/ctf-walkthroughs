# Infinity Shell – TryHackMe

- Difficulty: easy
- Date: 2026-05-09
- Tags: forensics, linux

## 🧩 Overview

```text
Cipher’s legion of bots has exploited a known vulnerability in our web application, leaving behind a dangerous web shell implant. Investigate the breach and trace the attacker's footsteps!

Note: Click the **Start Machine** button to spawn the Virtual Machine.
```

## ⚙️ Tools Used

- Linux commands

## Walkthrough

<div align="center"><img src="../attachments/Pasted image 20260509201753.png"/></div>

First, I started by checking the error logs. `error.log` was empty but `error.log.1` had interesting information.

<div align="center"><img src="../attachments/Pasted image 20260509205559.png"/></div>

The filename `image.php` immediately appeared suspicious because executable PHP scripts should not normally exist inside an image directory.

Next, I read the contents of `/var/www/html/CMSsite-master/img/image.php` and found that it was a PHP script to spawn a web shell.

<div align="center"><img src="../attachments/Pasted image 20260509205835.png"/></div>

The web shell accepts Base64-encoded commands through the `query` parameter and executes them using the `system()` function.

Then, I checked the access logs in `/var/log/apache2/`.
The `other_vhosts_access.log.1` file contained all the logs related to the web server.

The access logs revealed the attacker's activity sequence.

The threat actor first accessed `/index.html`, then browsed to `/CMSsite-master`, where the application redirected them to `/CMSsite-master/`.

Shortly afterwards, requests were made to `register.php`, indicating that the attacker created an account on the platform. Following successful registration, the attacker accessed the administrative area at `/CMSsite-master/admin/`.

Multiple requests were then sent to:

  ```text
  /CMSsite-master/admin/profile.php?section=not_cipher
  ```

Immediately after these requests, the attacker accessed:

```text
/CMSsite-master/img/image.php
```

This strongly suggests that the `profile.php` endpoint was vulnerable to file upload or remote file inclusion, allowing the attacker to place the malicious PHP web shell inside the `/img/` directory.
After that, the user sent multiple requests that contained `base64` encoded strings.

<div align="center"><img src="../attachments/Pasted image 20260509210936.png"/></div>

Several requests contained Base64-encoded payloads submitted to the web shell for command execution. One of the decoded payloads revealed the challenge flag.

<div align="center"><img src="../attachments/Pasted image 20260509211010.png"/></div>

## 🚩 Flag

```text
THM{sup3r_34sy_w3bsh3ll}
```

## 🧠 Lessons Learned

- Web server logs can be used to reconstruct attacker behaviour and identify the sequence of compromise.
- Unusual files inside web-accessible directories (e.g. PHP files in image folders) are strong indicators of web shell activity.
- Base64 encoding is commonly used by attackers to obfuscate commands sent to web shells.
- Monitoring file uploads and administrative functionality is critical for detecting post-exploitation activity.

## 🛡️ Mitigation

- Restrict file uploads to approved extensions and validate file content server-side.
- Disable execution of PHP scripts inside upload and image directories.
- Implement proper access controls and input validation on administrative endpoints.
- Monitor web server logs for suspicious requests, encoded payloads, and abnormal file access patterns.
- Deploy a Web Application Firewall (WAF) to detect and block web shell activity.
