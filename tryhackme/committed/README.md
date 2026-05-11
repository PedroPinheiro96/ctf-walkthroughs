# Committed – TryHackMe

- Difficulty: Easy
- Date: 19/04/2026
- Tags: git, version control, forensics

## 🧩 Overview

```text
This challenge demonstrates how sensitive information can remain exposed in Git commit history even after being removed in later revisions. The objective is to analyze the repository history and recover hidden credentials.
```

## ⚙️ Tools Used

- Git

## Walkthrough

### Task 1 - Committed

Oh no, not again! One of our developers accidentally committed some sensitive code to our GitHub repository. Well, at least, that is what they told us... the problem is, we don't remember what or where! Can you track down what we accidentally committed?

First, I started by unzipping the archive:

```
unzip committed.zip
```

I changed to that directory and ran `ls -la` to see if I could find the `.git` directory.

<div align="center"><img src="../attachments/Pasted image 20260508170149.png"/></div>

This confirmed the presence of `Git` metadata. Then I ran `git status` to see if there were any staged files and the current branch.

<div align="center"><img src="../attachments/Pasted image 20260508165216.png"/></div>

As it's possible to see above, the current branch is `master`. This also confirms that we are in a `Git` tracked directory.

The presence of Git metadata and successful execution of Git commands confirmed this was a Git repository. This means it is possible to analyse the `git` history.

Then, I enumerated available branches (local and remote) using:

```shell-session
git branch -a
```

<div align="center"><img src="../attachments/Pasted image 20260508165319.png"/></div>

The task mentions that a developer accidentally committed some sensitive code to the GitHub repository. I checked the last commits with `git log`.

```shell-session
git log --oneline --all
```

<div align="center"><img src="../attachments/Pasted image 20260508165450.png"/></div>

One of the commits that stands out is:
- `c56c470 Oops`

The commit message `Oops` suggested a corrective commit likely intended to fix a previous mistake.

To inspect the changes introduced by the suspicious commit, I used `git show` and the hash of the commit before `Oops`:

<div align="center"><img src="../attachments/Pasted image 20260508165623.png"/></div>

```shell-session
git show 3a8cc16
```

The command returned the file differences and the flag was on it:

<div align="center"><img src="../attachments/Pasted image 20260508170011.png"/></div>

It was possible to return the previous content of the file because `Git` maintains a complete history of the repository.

The credential exposure occurred because sensitive information had been committed to the repository before being removed in a later revision.

## 🧠 Lessons Learned

- Git preserves full historical snapshots of a repository, including deleted or modified sensitive data.
- Removing secrets in later commits does not eliminate them from Git history.
- Commit messages such as "fix", "oops", or "remove credentials" are strong indicators of sensitive historical changes.
- Git history should always be reviewed when auditing repositories for secrets.

## 🛡️ Mitigation

- Never commit sensitive data (passwords, API keys, tokens) to version control systems.
- Use secret management tools (e.g. environment variables, vault systems).
- Implement pre-commit hooks to detect secrets before committing.
- If secrets are exposed, rewrite history rather than deleting in a new commit.