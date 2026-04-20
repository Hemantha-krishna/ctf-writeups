---
layout: post
title: 'TryHackMe PWN101 Challenge1 Writeup'
date: 2025-09-07 23:20:51 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
  - ctf
description: "Intermediate level binary exploitation challenges."
canonical_url: "https://medium.com/@hemanthakrishnach/tryhackme-pwn101-challenge1-writeup-1a51e835360f"
image: "/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-000-5563d17a.png"
---

Intermediate level binary exploitation challenges.

---

Intermediate level binary exploitation challenges.![image](/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-000-5563d17a.png)![image](/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-001-f5ef0620.png)

Room link: [https://tryhackme.com/room/pwn101](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa1E4QjJxcHZJZXQxSUxLMGd2cVY5TVBOMGltZ3xBQ3Jtc0ttTURpVTdZSlJsOWpBcUFUNTU3bTg0TTVBM0xqOWw1bXZPbzR3X2Y4SkJkTUdBOFhhNW9SQkVmcWFUcnJsbWNMRGhiYkxLcFFnX05SNkZ2MG5FRzdJMTItdXlNZ0lDN2RIdi0zZmsyM3ZRNmdGbkEtUQ&q=https%3A%2F%2Ftryhackme.com%2Froom%2Fpwn101&v=8FEYdpZdftQ)

### Challenge 1 — pwn101

#### Challenge Overview

The first challenge in the PWN101 series is designed to get us comfortable with the basics of binary exploitation. The target binary is hosted remotely, and we can interact with it over a simple `nc` (netcat) connection.

We’re told the service is listening on **port 9001**, and the hint is a simple string:
```bash
This should give you a start: ‘AAAAAAAAAAA’
```

We connect using:
```bash
nc 10.201.126.158 9001
```

Once inside, we can start sending inputs.

For a quick test, let’s feed it the suggested string: AAAAAAAAAAA. We get![image](/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-002-dcc0497a.png)

#### Overflow Attempt

Instead of a short input, let’s try a **longer string**:![image](/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-003-9f7ae98c.png)

Boom. The binary doesn’t like that very much. And here’s where things get interesting: sending an oversized input actually **triggers a shell**.

This is a textbook case of a **buffer overflow**, where excess input spills into memory space and alters the program’s flow. In this particular challenge, the program is intentionally vulnerable; it drops us straight into a shell when the buffer is exceeded.

#### Getting the Flag

Once we have shell access, grabbing the flag is as easy as:
```bash
cat flag.txt
```
![image](/assets/images/posts/tryhackme-pwn101-challenge1-writeup/img-004-3a1c50a5.png)

That was just the warm-up. The real fun begins as we move further into **PWN101**, where challenges become progressively more complex, requiring deeper knowledge of memory management, registers, and exploitation techniques.

### Thanks for reading, and see you in the next challenge!
