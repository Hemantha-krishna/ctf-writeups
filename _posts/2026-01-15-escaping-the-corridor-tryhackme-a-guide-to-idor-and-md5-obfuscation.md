---
layout: post
title: 'Escaping the Corridor - TryHackMe: A Guide to IDOR and MD5 Obfuscation'
date: 2026-01-15 21:21:44 +0000
categories: [cybersecurity]
tags:
  - tryhackme
description: "Can you escape the Corridor?"
canonical_url: "https://medium.com/@hemanthakrishnach/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation-57895514ef63"
image: "/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-000-e46fc86e.png"
---

Can you escape the Corridor?

---

### Escaping the Corridor — TryHackMe Room Walkthrough

Can you escape the Corridor?![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-000-e46fc86e.png)

Room link: <https://tryhackme.com/room/corridor>

### Escaping the Corridor: A Guide to IDOR and MD5 Obfuscation

In the world of Capture The Flag (CTF) challenges, sometimes the most complex-looking URLs are hiding very simple logic. This write-up explores the “Corridor” room on TryHackMe, a challenge designed to test your ability to spot Insecure Direct Object Reference (IDOR) vulnerability.

### The Challenge

Upon entering the “Corridor,” you are placed in a strange hallway and asked to find your way back. The room explicitly hints that you should examine URL endpoints, noting that the hexadecimal values found there look “an awful lot like a hash”.

The objective is to uncover website locations you were not expected to access by manipulating these values.

### Step 1: Reconnaissance and Pattern Recognition

As I navigated the site, I noticed the URL structure changed as I entered different “rooms.”![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-001-c3005fa0.png)

For example, one URL ended in the hash: `8f14e45fceea167a5a36dedd4bea2543`![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-002-ae16f2e9.png)

Suspecting this was a hash, I utilized a cracking tool to identify the algorithm and the plaintext value.

- **The Hash:** `8f14e45fceea167a5a36dedd4bea2543`.- **The Algorithm:** The tool identified it as **MD5**.- **The Result:** The hash cracked instantly to the number **7**.

![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-003-5e17d30b.png)

### Step 2: The Logic Gap

This revelation confirmed the IDOR vulnerability. The website organizes rooms numerically, but instead of using `id=7`, it uses `id=md5(7)`.

I observed that the visible doors in the corridor are numbered from 1 to 13. To capture the flag, I needed to find a room number that existed on the server but wasn’t listed in the UI.

### Step 3: exploiting the Vulnerability

My first instinct was to check the next sequential number.

- **Input:** `14`.- **MD5 Hash:** `aab3238922bcc25a6f606eb525ffdc56`.

![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-004-d7eb8f3a.png)

- **Result:** When I navigated to this hash, the server returned a **404 Not Found**.

![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-005-eab65aa2.png)

Since moving *forward* (to 14) didn’t work, I decided to move *backward*. In array indexing and computer science, counting often starts at zero.

I used an MD5 encrypter to generate the hash for the number **0**:

- **Input:** `0`.- **MD5 Hash:** `cfcd208495d565ef66e7dff9f98764da`.

![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-006-eed49f5f.png)

I manually constructed the malicious URL by appending this new hash to the base IP address:

`http://<MACHINE_IP>/cfcd208495d565ef66e7dff9f98764da`

### Conclusion

The exploit was successful. Navigating to the hash for room “0” bypassed the restricted user interface and revealed the hidden page containing the flag.![image](/assets/images/posts/escaping-the-corridor-tryhackme-a-guide-to-idor-and-md5-obfuscation/img-007-fe0c15b5.png)

#### Thank you for reading my write-up. I hope you found it helpful.
