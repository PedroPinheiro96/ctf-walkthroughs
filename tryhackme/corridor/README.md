# Corridor – TryHackMe

- Difficulty: Easy
- Date: 2026-04-24
- Tags: IDOR

## 🧩 Overview

```text
You have found yourself in a strange corridor. Can you find your way back to where you came?  

In this challenge, you will explore potential vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a _hash_, don't they?). This could help you uncover website locations you were not expected to access.
```

## ⚙️ Tools Used

- [CrackStation](https://crackstation.net/)
- [CyberChef](https://gchq.github.io/CyberChef/)

## Walkthrough

### Initial Enumeration

The description mentions a website so I navigated to `http://10.113.167.239:80`.

The website shows an image of 13 doors.

<div align="center"><img src="../attachments/Pasted image 20260508172525.png"/></div>

I right-clicked on the page to view its source code and noticed that the `img` had an `image map` that had what seemed to be `MD5` hashes for `alt text` and `href`.

<div align="center"><img src="../attachments/Pasted image 20260508172627.png"/></div>

### Identifying the Hash Pattern

The value appeared to be a hash, so I used [CrackStation](https://crackstation.net/) to confirm it.
As expected, CrackStation confirmed it was a hash and returned the `MD5` hash type.

<div align="center"><img src="../attachments/Pasted image 20260508172801.png"/></div>

I've tried the URL with each of the 13 hashes but each discovered endpoint returned an identical webpage.

<div align="center"><img src="../attachments/Pasted image 20260508172830.png"/></div>

### Exploiting the IDOR

Since the existing rooms followed a predictable `MD5`-based naming scheme, I tested whether additional resources could be accessed by generating `MD5` hashes for values outside the visible range.

First, I computed the hash value of `0` on [CyberChef](https://gchq.github.io/CyberChef/).

<div align="center"><img src="../attachments/Pasted image 20260508173150.png"/></div>

```text
cfcd208495d565ef66e7dff9f98764da
```

After generating the hash, I navigated directly to the corresponding endpoint.

```text
http://10.113.167.239/cfcd208495d565ef66e7dff9f98764da
```

<div align="center"><img src="../attachments/Pasted image 20260508173312.png"/></div>

The endpoint returned a previously inaccessible page containing the flag.

The application relied on predictable `MD5` hashes as direct object references without validating authorization. By generating additional hashes manually, it was possible to access unintended resources.

This is an example of an `IDOR (Insecure Direct Object Reference)` vulnerability.

The vulnerability existed because the application relied on predictable MD5-based resource references without implementing proper access restrictions.

## 🚩 Flag

```shell-session
flag{2477ef02448ad9156661ac40a6b8862e}
```

## 🧠 Lessons Learned

- Predictable identifiers can lead to insecure direct object reference (IDOR) vulnerabilities
- Hashes should not be treated as secure access controls
- Source code inspection can reveal hidden application logic
- Enumeration of URL patterns is an important part of web application testing

## 🛡️ Mitigation

- Implement proper authorization checks for all resources
- Avoid exposing direct object references in URLs
- Do not rely on hashes alone for access control
- Use unpredictable identifiers such as UUIDs
- Restrict access to sensitive endpoints server-side
