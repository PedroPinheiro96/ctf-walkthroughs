# OhSINT – TryHackMe

- Difficulty: easy
- Date: 2026-05-10
- Tags: osint, reconnaissance, metadata

## 🧩 Overview

```
Are you able to use open source intelligence to solve this challenge?
```

## ⚙️ Tools Used

- Exiftool
- [CyberChef](https://gchq.github.io/CyberChef/)
- [Wigle](https://wigle.net/)

## Walkthrough

### Task 1 - OhSINT

What information can you possibly get with just one image file?

#### Q1: What is this user's avatar of?

First, I used `exiftool` to extract metadata from the `WindowsXP.jpg` file.

<div align="center"><img src="../attachments/Pasted image 20260510161623.png"/></div>

The metadata revealed multiple pieces of information about the file and its creator.
It's also possible to see the author of the file - `OWoodflint`.

Searching the username `OWoodflint` across public platforms revealed associated `GitHub`, `Twitter/X`, and `WordPress` accounts.``

Only the `Twitter/X` profile had a profile picture.

<div align="center"><img src="../attachments/Pasted image 20260510172817.png"/></div>

```text
-> cat
```

#### Q2: What city is this person in?

The `README.md` file in the `Github` profile showed that the user is from `London`.

<div align="center"><img src="../attachments/Pasted image 20260510164044.png"/></div>

```text
-> London
```

#### Q3: What is the SSID of the WAP he connected to?

One of the posts on `Twitter/X` contained a `BSSID`.

<div align="center"><img src="../attachments/Pasted image 20260510164011.png"/></div>

A `BSSID` is the unique `MAC (Media Access Control)` address assigned to a wireless access point.

[Wigle](https://wigle.net/) contains a huge database of wireless endpoints information such as `BSSID` and `SSID`.

Querying the recovered `BSSID` in `Wigle` returned the associated wireless network:

<div align="center"><img src="../attachments/Pasted image 20260510173610.png"/></div>

```text
-> UnileverWiFi
```

#### Q4: What is his personal email address?

`Git` commits contain metadata about the author of the commit.
I visited the URL of the latest commit and appended `.patch` to the URL.

```
https://github.com/OWoodfl1nt/people_finder/commit/433f4d949ebf424817da0f09820dea5e63766cbf.patch
```

<div align="centre"><img src="Pasted image 20260511163222.png"/></div>

```text
->  OWoodflint@gmail.com
```

#### Q5: What site did you find his email address on?

``` text
-> github
```

#### Q6: Where has he gone on holiday?

The latest blog post revealed the user's current travel location.

<div align="center"><img src="../attachments/Pasted image 20260510162600.png"/></div>

```
-> New York
```

#### Q7: What is the person's password?

After inspecting the WordPress blog's source code, I was able to find what seemed to be a password.

<div align="center"><img src="../attachments/Pasted image 20260510184045.png"/></div>

```text
-> pennyDr0pper.!
```

## 🧠 Lessons Learned

- Image metadata (`EXIF`) may expose usernames, software information, timestamps, and other identifying details.
- Reusing usernames across multiple platforms makes profile correlation significantly easier for attackers.
- Wireless network information such as `BSSID` values can be correlated with public databases to identify physical locations and associated networks.
- Git commit metadata may unintentionally expose sensitive information such as personal email addresses.
- Information disclosed across multiple platforms can be combined to build a detailed profile of a target.
- Sensitive data may still be exposed in webpage source code even when it is not visible in the rendered page.

## 🛡️ Mitigation

- Strip metadata from images before publishing them online.
- Avoid reusing the same usernames across multiple platforms and services.
- Use separate email addresses for public development activity and personal communication.
- Regularly audit Git repositories and commit history for exposed personal information.
- Avoid embedding sensitive information, credentials, or hidden comments in webpage source code.
- Limit publicly accessible personal information across social media and blogging platforms.