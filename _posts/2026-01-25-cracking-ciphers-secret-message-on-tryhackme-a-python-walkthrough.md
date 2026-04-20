---
layout: post
title: 'Cracking “Cipher’s Secret Message” on TryHackMe: A Python Walkthrough'
date: 2026-01-25 19:22:27 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "If you are diving into the world of CTFs (Capture The Flags), you will inevitably run into custom Python encryption scripts. Today, we are…"
canonical_url: "https://medium.com/@hemanthakrishnach/cracking-ciphers-secret-message-on-tryhackme-a-python-walkthrough-52830a1e036c"
image: "/assets/images/posts/cracking-ciphers-secret-message-on-tryhackme-a-python-walkthrough/img-000-ecbfb61a.png"
---

If you are diving into the world of CTFs (Capture The Flags), you will inevitably run into custom Python encryption scripts. Today, we are…

---

![image](/assets/images/posts/cracking-ciphers-secret-message-on-tryhackme-a-python-walkthrough/img-000-ecbfb61a.png)

If you are diving into the world of CTFs (Capture The Flags), you will inevitably run into custom Python encryption scripts. Today, we are breaking down the **“Cipher’s Secret Message”** room on TryHackMe. We are given an encrypted string and the source code used to create it, and our job is to reverse the logic to retrieve the flag.

### The Challenge

![image](/assets/images/posts/cracking-ciphers-secret-message-on-tryhackme-a-python-walkthrough/img-001-4ca8f855.png)

We are provided with two key pieces of information:

1. **The Ciphertext:** `a_up4qr_kaiaf0_bujktaz_qm_su4ux_cpbq_ETZ_rhrudm`- **The Encryption Algorithm:**

```python
from secret import FLAGdef enc(plaintext):    return "".join(        chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) + i) % 26 + base)         if c.isalpha() else c        for i, c in enumerate(plaintext)    )with open("message.txt", "w") as f:    f.write(enc(FLAG))
```

The goal is to decode the message.

### Analyzing the Encryption

At first glance, this looks like a Caesar Cipher, but there’s a twist. A standard Caesar Cipher shifts every letter by the *same* amount. Let’s break down the logic inside the list comprehension to see why this is different:

1. `enumerate(plaintext)`: This provides both the character (`c`) and its position (`i`) in the string.- `ord(c) - base`: This converts the character into a 0-25 range (A=0, B=1, etc.).- `+ i`: **This is the key.** The shift amount isn't constant; it increases by 1 for every character position. Index 0 shifts by 0, Index 1 shifts by 1, etc.- `% 26`: This ensures the alphabet wraps around (so 'z' shifted by 1 becomes 'a').

Essentially, this is a **Progressive Shift Cipher**. To decrypt it, we simply need to reverse the math.

### The Solution Script

To reverse the encryption, we need to **subtract** the index (`i`) instead of adding it.

So, just change the + to — in the give script
```less
chr((ord(c) - (base := ord('A') if c.isupper() else ord('a')) + i) % 26 + base)
```

Here is the logic for decryption:

P=(C−Base−i)(mod26)+Base

Below is the corrected Python script to solve the challenge.
```python
def decode(ciphertext): return “”.join( # Subtract ‘i’ instead of adding it to reverse the shift chr((ord(c) — (base := ord(‘A’) if c.isupper() else ord(‘a’)) — i) % 26 + base) if c.isalpha() else c for i, c in enumerate(ciphertext) )# The encrypted message provided in the challengeenc_msg = “a_up4qr_kaiaf0_bujktaz_qm_su4ux_cpbq_ETZ_rhrudm”print(f”Decoded Message: {decode(enc_msg)}”)
```

### How it works:

- We iterate through the ciphertext using `enumerate` to get the index `i`.- We check if the character is a letter (`isalpha()`). If it's a symbol or number (like `_` or `4`), we leave it alone.- We apply the reverse formula: `(Value - Index) % 26`. Python handles negative modulo correctly (e.g., `-1 % 26` becomes `25`), making this very robust.

### The Result

Running the script produces the following output:![image](/assets/images/posts/cracking-ciphers-secret-message-on-tryhackme-a-python-walkthrough/img-002-24ac53d5.png)

### The Final Flag

The challenge instructions ask us to wrap the decoded message within the specific flag format.

### Summary

This challenge is a great introduction to reading Python list comprehensions and understanding modular arithmetic. While standard tools like CyberChef are powerful, writing a quick 5-line script is often the fastest way to solve custom logic puzzles like this one.

**Happy Hacking!**
