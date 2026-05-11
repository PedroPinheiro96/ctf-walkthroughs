# TakeOver – TryHackMe

- Difficulty: easy
- Date: 2026-04-29
- Tags: enumeration

## 🧩 Overview

```text
Hello there,  
  
I am the CEO and one of the co-founders of futurevera.thm. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.  

Recently blackhat hackers approached us saying they could takeover and are asking us for a big ransom. Please help us to find what they can takeover.  
  
Our website is located at https://futurevera.thm

Hint: Don't forget to add the MACHINE_IP in /etc/hosts for futurevera.thm ; )
```

## ⚙️ Tools Used

- Nmap
- Gobuster

## Walkthrough

Let's start by running nmap to see what ports are open:

```shell-session
nmap -sS -sV 10.130.171.91
```

Ports 80 (HTTP) and 443 (HTTPS) are open.

```shell-session
root@ip-10-130-64-57:~# nmap -sS -sV 10.130.171.91
Starting Nmap 7.80 ( https://nmap.org ) at 2026-04-30 01:24 BST
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 10.130.171.91
Host is up (0.0030s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.62 seconds
```

Then we use Gobuster for vhost subdomain enumeration:

```shell-session
root@ip-10-130-64-57:~# gobuster vhost -u "http://futurevera.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://futurevera.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: portal.futurevera.thm Status: 200 [Size: 69]
Found: payroll.futurevera.thm Status: 200 [Size: 70]
Progress: 114532 / 114533 (100.00%)
===============================================================
Finished
===============================================================
```

Since the discovered virtual hosts were not publicly resolvable, I mapped them locally inside `/etc/hosts` to interact with them directly.

```shell-session
nano /etc/hosts
10.130.171.91   futurevera.thm portal.futurevera.thm payroll.futurevera.thm support.futurevera.thm blog.futurevera.thm
```

Both pages display the message "payroll.futurevera.thm is only available via internal VPN".

Let's run `Gobuster` but this time on the `HTTPS` domain (we need to add the `-k` flag to skip the `TLS verification`):

```shell-session
root@ip-10-130-64-57:~# gobuster vhost -u "https://futurevera.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             https://futurevera.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: blog.futurevera.thm Status: 421 [Size: 408]
Found: support.futurevera.thm Status: 421 [Size: 411]
Progress: 19983 / 19984 (99.99%)
===============================================================
Finished
===============================================================
```

`blog.futurevera.thm` appeared to be a normal page with no immediately interesting functionality.
The TLS certificate for `blog.futurevera.thm` did not expose any additional useful information.

Inspecting the TLS certificate for `support.futurevera.thm` revealed an additional `Subject Alternative Name (SAN)`:

```shell-session
Subject Alt Names
DNS Name    secrethelpdesk934752.support.futurevera.thm 
```

This exposed a hidden internal hostname that was not discovered during initial enumeration.

After adding it to `/etc/hosts` and navigating to it, the flag is shown in the URL:

<div align="center"><img src="../attachments/Pasted image 20260430014608.png"/></div>

## 🚩 Flag

```text
flag{beea0d6edfcee06a59b83fb50ae81b2f}
```

## 🧠 Lessons Learned

- **VHOST enumeration can reveal assets that DNS won't**
	Using **Gobuster** in `vhost` mode uncovered hosts like `portal` and `payroll` even though they may not have been publicly resolvable via DNS. This reinforces that relying only on DNS enumeration can leave gaps.
- **Always test both HTTP and HTTPS separately**
	The HTTP scan revealed some hosts, but switching to HTTPS exposed additional ones (`blog`, `support`). Different configurations (and even misconfigurations) often exist between ports 80 and 443.
- **TLS certificates can leak hidden subdomains**
    Inspecting the certificate for `support.futurevera.thm` revealed a **Subject Alternative Name (SAN)** entry:
    `secrethelpdesk934752.support.futurevera.thm`
    This is a classic example of how certificates can unintentionally expose internal or obscure hostnames.
- **Host header + /etc/hosts manipulation is essential**
    Manually adding discovered subdomains to `/etc/hosts` allows you to properly interact with virtual hosts that aren't publicly resolvable.

## 🛡️ Mitigation

- Avoid exposing sensitive or internal hostnames in TLS certificate SAN fields
- Restrict access to internal services using proper network segmentation
- Remove unused virtual hosts and stale DNS records
- Monitor for unintended hostname disclosure through certificates
- Use separate certificates for internal and external infrastructure where possible
