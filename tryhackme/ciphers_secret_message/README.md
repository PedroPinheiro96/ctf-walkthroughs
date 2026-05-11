# Cipher's Secret Message – Platform

- Difficulty: easy
- Date: 2026-05-08
- Tags: crypto

## 🧩 Overview

```
Sharpen your cryptography skills by analyzing code to get the flag.
```

## ⚙️ Tools Used

- Python

## Walkthrough

### Task 1 - Cipher's Secret Message

One of the Ciphers' secret messages was recovered from an old system alongside the encryption algorithm, but we are unable to decode it.

**Order:** Can you help void to decode the message?

**Message** : a_up4qr_kaiaf0_bujktaz_qm_su4ux_cpbq_ETZ_rhrudm

**Encryption algorithm** :

```python
from secret import FLAG

def enc(plaintext):
    return "".join(
        chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) + i) % 26 + base) 
        if c.isalpha() else c
        for i, c in enumerate(plaintext)
    )

with open("message.txt", "w") as f:
    f.write(enc(FLAG))
```

Note: Wrap the decoded message within the flag format **THM{}**

The encryption function applies a **position-dependent Caesar cipher** to each alphabetical character in the plaintext.

```python
def enc(plaintext):    
	return "".join(
	    chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) + i) % 26 + base)
	    if c.isalpha() else c
	    for i, c in enumerate(plaintext)    )
```

#### Step-by-step:

- The function iterates over each character in the plaintext using `enumerate`, which provides both the character and its index `i`.
- It checks whether the character is alphabetical:
    - If not, it is returned unchanged.
- For alphabetical characters:
    - It determines the ASCII base:
        - `A` for uppercase letters
        - `a` for lowercase letters
    - It converts the character into a 0–25 alphabet index using `ord(c) - base`.
    - It adds the character's position index `i`, introducing a progressive shift.
    - The result is wrapped using modulo 26 to stay within the alphabet range.
    - Finally, it is converted back to a character using `chr()`.

Each character is shifted by a value equal to its position index in the string.

To reverse the encryption, the same logic is applied but the shift direction is inverted.

Instead of adding the index `i`, we subtract it:

```python
chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) - i) % 26 + base)
```

The following script can be used to decrypt the ciphertext:

```python
cipher = "a_up4qr_kaiaf0_bujktaz_qm_su4ux_cpbq_ETZ_rhrudm"

def dec(ciphertext):  
    return "".join(  
        chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) - i) % 26 + base)  
        if c.isalpha() else c  
        for i, c in enumerate(ciphertext)  
    )  
  
print(f"Decrypted message: {dec(cipher)}")
```

## 🚩 Flag

```
-> THM{a_sm4ll_crypt0_message_to_st4rt_with_THM_cracks}
```

## 🧠 Lessons Learned

- Index-based transformations (such as position-dependent shifts) weaken classical ciphers by introducing deterministic patterns that can be exploited.
- Understanding how plaintext is transformed step-by-step is often sufficient to derive the inverse operation without brute forcing.

## 🛡️ Mitigation

- Avoid implementing custom cryptographic algorithms for securing data.
- Use well-established cryptographic libraries instead of homegrown encryption schemes.
- Do not rely on obscurity or simple transformations for security.
- Ensure proper cryptographic review of any encryption logic before deployment.
