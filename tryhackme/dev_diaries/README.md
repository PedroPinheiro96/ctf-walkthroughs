# DevDiaries – TryHackMe

- Difficulty: easy
- Date: 2026-05-08
- Tags: osint, git, github, waybackmachine, subdomain-enumeration

## 🧩 Overview

```text
Hunt through online development traces to uncover what was left behind.
```

## ⚙️ Tools Used

-  [WhoIsXML](https://subdomains.whoisxmlapi.com/)

## Walkthrough

### Task 1 - Challenge

We have just launched a website developed by a freelance developer. The source code was not shared with us, and the developer has since disappeared without handing it over.

Despite this, traces of the development process and earlier versions of the website may still exist online.

You are only given the website's primary domain as a starting point: **marvenly.com**

#### Q1: What is the subdomain where the development version of the website is hosted?

To enumerate potential subdomains associated with `marvenly.com`, I used [WhoIsXML](https://subdomains.whoisxmlapi.com/).

<div align="center"><img src="../attachments/Pasted image 20260508181510.png"/></div>

The search returned three subdomains associated with the target domain.

``` text
-> uat-testing.marvenly.com
```

#### Q2: What is the GitHub username of the developer?

Direct access to the discovered subdomains was no longer possible, suggesting the infrastructure had been decommissioned or restricted.

To recover historical content, I searched the archived snapshots available on the [Wayback Machine](https://web.archive.org?utm_source=chatgpt.com).

I was able to find a result for `uat-testing.marvenly.com`.

<div align="center"><img src="../attachments/Pasted image 20260508184330.png"/></div>

The footer of the page revealed the developer's username:
<div align="center"><img src="../attachments/Pasted image 20260508184432.png"/></div>

```text
notvibecoder23
```

#### Q3: What is the developer's email address?

A quick Google search revealed the developer's GitHub page.

<div align="center"><img src="../attachments/Pasted image 20260508184535.png"/></div>

An easy way to look for the email of the developer is by accessing the URL of one of the commits and adding `.patch` to the end of it.

Appending `.patch` to the commit URL exposed the raw commit metadata, including the author email address configured in Git.

```text
https://github.com/notvibecoder23/marvenly_site/commit/7a7090dd0ce6b8932d0c4a44e050e7fa1e0b2edd.patch
```

<div align="center"><img src="../attachments/Pasted image 20260508185305.png"/></div>

```text
freelancedevbycoder23@gmail.com
```

#### Q4: What reason did the developer mention in the commit history for removing the source code?

Reviewing the repository commit history revealed a message indicating that the project had been abandoned due to a payment dispute.

```text
The project was marked as abandoned due to a payment dispute
```

#### Q5: What is the value of the hidden flag?

After examining the commit with commit message `Removed my signature, ready for deployment`, I found the flag:

<div align="center"><img src="../attachments/Pasted image 20260508184807.png"/></div>

```text
-> THM{g1t_h1st0ry_n3v3r_f0rg3ts}
```

## 🧠 Lessons Learned

- Publicly exposed development artefacts can reveal sensitive operational and developer information.
- Historical snapshots and archived content may expose data no longer available on live infrastructure.
- Git commit metadata can unintentionally leak email addresses and internal project details.
- Deleted source code or removed content may still remain accessible through Git history.
- OSINT techniques can be highly effective when correlating data across multiple public sources.

## 🛡️ Mitigation

- Avoid exposing development or staging environments to the public internet.
- Regularly audit public repositories for sensitive information and exposed metadata.
- Use separate email addresses for public development activities where appropriate.
- Remove sensitive data from Git history using repository rewriting techniques when exposure occurs.
- Restrict indexing and archival of sensitive environments where possible.
- Implement proper source code ownership and handover processes for third-party developers.