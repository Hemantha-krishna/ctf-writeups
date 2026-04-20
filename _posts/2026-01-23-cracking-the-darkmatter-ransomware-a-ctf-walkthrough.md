---
layout: post
title: 'Cracking the DarkMatter Ransomware: A CTF Walkthrough'
date: 2026-01-23 03:45:28 +0000
categories: [cybersecurity]
tags:
  - writeup
  - ctf
description: "In the world of cybersecurity Capture The Flags (CTFs), few things are as satisfying as turning a hacker’s own cryptography against them…"
canonical_url: "https://medium.com/@hemanthakrishnach/cracking-the-darkmatter-ransomware-a-ctf-walkthrough-bd731bb74f57"
image: "/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-000-755a7838.png"
---

In the world of cybersecurity Capture The Flags (CTFs), few things are as satisfying as turning a hacker’s own cryptography against them…

---

In the world of cybersecurity Capture The Flags (CTFs), few things are as satisfying as turning a hacker’s own cryptography against them. In this walkthrough, we’ll tackle a scenario involving the “DarkMatter Gang” ransomware. We will analyze the encryption artifacts, exploit weak RSA keys, and recover our data without paying a single coin.![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-000-755a7838.png)

Room link: <https://tryhackme.com/room/hfb1darkmatter>

![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-001-a78861d2.png)

### Phase 1: The Incident

The scenario begins with a compromised machine. Upon accessing the system, we are greeted with a chilling ransom note:![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-002-daba25ca.png)

The attackers demand a payment of **0.5 BTC** to a specific wallet address to prevent permanent data loss.

nvestigating the `/tmp` directory reveals the aftermath of the attack. We identify several critical files left behind by the ransomware:

- `dock-replace.log`- `encrypted_aes_key.bin`- `public_key.txt`

The presence of a public key suggests the malware utilizes asymmetric encryption (likely RSA) to secure the symmetric key (AES) used on the files.

### Phase 2: Cryptographic Reconnaissance

To defeat the encryption, we first need to understand the keys. We display the contents of `public_key.txt`:
```ini
n=340282366920938460843936948965011886881e=65537
```
![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-003-04154012.png)

Here we see the standard RSA public key components: the modulus (n) and the public exponent (e).

### The Vulnerability

RSA security relies on the difficulty of factoring the modulus (n) back into its two prime components (p and q). If n is small enough or has been generated using weak primes, we can derive the private key (d).

### Phase 3: Cracking the Key

#### 1. Factorization

Since the provided modulus n is relatively small, we check [**FactorDB**](https://factordb.com/), a database of known factorizations. Entering our n value yields an immediate result, providing us with the two prime numbers:

- **p:** 18446744073709551533- **q:** 18446744073709551557

![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-004-2aa72cfc.png)

#### 2. Calculating the Private Key

With p, q, and e in hand, we have everything required to mathematically reconstruct the private decryption key. We use an online RSA Cipher tool (like [dCode.fr](https://www.dcode.fr/rsa-cipher)) to perform the calculation.

We input the following values into the RSA Decoder:

- **Modulus (N):** 340282366920938460843936948965011886881- **Public Exponent (E):** 65537- **Prime Factor (P):** 18446744073709551533- **Prime Factor (Q):** 18446744073709551557

The tool computes the private key (d):

`196442361873243903843228745541797845217`![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-005-9bce82de.png)

### Phase 4: Decryption and Recovery

Now that we have the private key, we return to the ransomware interface and paste the calculated d value (`19644...`) into the prompt.![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-006-46f01dc2.png)

#### The Loot

With the file system unlocked, we can inspect the sensitive data the attackers tried to hold hostage. We open `student_grades.docx`.

Among these entries, we locate the string “THM,” confirming we have successfully retrieved the CTF flag.![image](/assets/images/posts/cracking-the-darkmatter-ransomware-a-ctf-walkthrough/img-007-dccb8cdf.png)

### Conclusion

This challenge serves as a perfect example of why key size matters in cryptography. While the math behind RSA is solid, utilizing a modulus that can be easily factored destroys the security integrity of the system. In a real-world scenario, keys are 2048-bit or 4096-bit, making this factorization attack computationally infeasible, but in the world of CTFs, checking for “small n” is always a great first step.

#### Thank you for reading, I hope you find it helpful!
