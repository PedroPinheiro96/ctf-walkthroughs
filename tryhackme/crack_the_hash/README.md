# Crack the Hash – TryHackMe

- Difficulty: easy
- Date: 2026-04-20
- Tags: hash, cracking, hashcat, tryhackme, ctf

## 🧩 Overview

```text
This room contains multiple hashes in different formats and we have to crack them.
```

## ⚙️ Tools Used

- **hashcat** - Hash Cracking tool
- **[hashid](https://www.kali.org/tools/hashid/)** - Hash type identifier
- **rockyou.txt** - Common password wordlist
- **[CrackStation](https://crackstation.net/)** — Online rainbow table lookup
- **[Hashes.com](https://hashes.com/en/tools/hash_identifier)** Online hash identifier

## Walkthrough

### Common Hashcat Modes Used

| Hash Type | Mode |
|---|---|
| MD5 | 0 |
| SHA-1 | 100 |
| SHA-256 | 1400 |
| bcrypt | 3200 |
| SHA512crypt | 1800 |
| HMAC-MD5 | 50 |

### Level 1

#### Q1: 48bb6e862e54f2a795ffc4e541caed4d

```shell-session
HASH: 48bb6e862e54f2a795ffc4e541caed4d

Possible Hashs:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

echo "48bb6e862e54f2a795ffc4e541caed4d" > l1q1.txt

hashcat -m 0 -a 0 l1q1.txt /usr/share/wordlists/rockyou.txt
```

#### Q2: CBFDAC6008F9CAB4083784CBD1874F76618D2A97

```shell-session
HASH: CBFDAC6008F9CAB4083784CBD1874F76618D2A97  

Possible Hashs: 
[+]  SHA-1 
[+]  MySQL5 - SHA-1(SHA-1($pass)) 

echo "CBFDAC6008F9CAB4083784CBD1874F76618D2A97" > l1q2.txt

Command: hashcat -m 100 -a 0 l1q2.txt /usr/share/wordlists/rockyou.txt
```

#### Q3: 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

```shell-session
HASH: 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

Possible Hashs:
[+]  SHA-256
[+]  Haval-256


echo "1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032" > l1q3.txt

hashcat -m 1400 -a 0 l1q3.txt /usr/share/wordlists/rockyou.txt
```

#### Q4: \$2y\$12\$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

To identify this hash, I've used [this hash identifier](https://hashes.com/en/tools/hash_identifier).

```
Type: bcrypt


Bcrypt is intentionally computationally expensive, making brute-force attacks significantly slower. Instead of attempting a local crack, I checked [Crackstation](https://www.crackstation.net/) to determine whether the hash had already been indexed in public rainbow tables and I found a match.
```

#### Q5: 279412f945939ba78ce0758d3fd83daa

```shell-session
HASH: 279412f945939ba78ce0758d3fd83daa

Possible Hashs:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

echo "279412f945939ba78ce0758d3fd83daa" > l1q5.txt

hashcat -m 0 -a 0 l1q5.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 -a 0 l1q5.txt /usr/share/wordlists/rockyou.txt
```

None of the commands I tried returned a match so I assumed that the hash was not in the rockyou.txt list and checked Crackstation.

CrackStation successfully identified the password, confirming that it was not present in the `rockyou.txt` wordlist.

### Level 2

This task increases the difficulty. All of the answers will be in the classic [rock you](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) password list.

#### Q1: F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

```shell-session
HASH: F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

Possible Hashs:
[+]  SHA-256
[+]  Haval-256

echo "F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85" > l2q1.txt

hashcat -m 1400 -a 0 l2q1.txt /usr/share/wordlists/rockyou.txt
```

#### Q2: 1DFECA0C002AE40B8619ECF94819CC1B

```shell-session
HASH: 1DFECA0C002AE40B8619ECF94819CC1B

Possible Hashs:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

echo "1DFECA0C002AE40B8619ECF94819CC1B" > l2q2.txt

hashcat -m 1000 -a 0 l2q2.txt /usr/share/wordlists/rockyou.txt
```

##### Q3: Hash: \$6\$aReallyHardSalt\$6WKUTqzq\.UQQmrm0p\/T7MPpMbGNnzXPMAXi4bJMl9be\.cfi3\/qxIf\.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02\. Salt: aReallyHardSalt

To identify this hash, I've used [this hash identifier](https://hashes.com/en/tools/hash_identifier).
The hash type can also be found in [Hash Examples](https://hashcat.net/wiki/doku.php?id=example_hashes) by looking at the `$6$` prefix.

```shell-session
Type: SHA-512-crypt

echo '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.' > l2q3.txt

hashcat -m 1800 -a 0 l2q3.txt /usr/share/wordlists/rockyou.txt
```

#### Q4: Hash: 1DFECA0C002AE40B8619ECF94819CC1B Salt: Tryhackme

```shell-session
HASH: 1DFECA0C002AE40B8619ECF94819CC1B

Possible Hashs:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

echo "1DFECA0C002AE40B8619ECF94819CC1B:tryhackme" > l2q4.txt

hashcat -m 160 -a 0 l2q4.txt /usr/share/wordlists/rockyou.txt
```

## 🚩 Flags

### Level 1

```text
Q1: easy
Q2: password123
Q3: letmein
Q4: bleh
Q5: Eternity22
```

### Level 2

```text
Q1: paule
Q2: n63umy8lkf4i
Q3: waka99
Q4: 481616481616
```

## 🧠 Lessons Learned

- Correct hash identification is critical before attempting to crack.
- Wordlists like `rockyou.txt` are effective but not exhaustive.
- Some hashes (e.g., bcrypt) are intentionally slow and better approached via lookup tables if possible.
- Salts significantly increase cracking difficulty and require correct hash modes in tools like hashcat.

## 🛡️ Mitigation

- Use strong, unique passwords resistant to dictionary attacks
- Avoid outdated or weak hashing algorithms such as MD5 and SHA-1
- Use slow password hashing algorithms like bcrypt, Argon2, or scrypt
- Always implement unique salts for password storage
- Enforce password complexity and MFA where possible