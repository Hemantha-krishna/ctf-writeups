---
layout: post
title: 'Lo-Fi - TryHackMe Writeup'
date: 2025-07-04 09:25:20 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Want to hear some lo-fi beats, to relax or study to? We’ve got you covered!"
canonical_url: "https://medium.com/@hemanthakrishnach/lo-fi-tryhackme-writeup-185e8145584c"
image: "/assets/images/posts/lo-fi-tryhackme-writeup/img-000-7decce02.jpg"
---

Want to hear some lo-fi beats, to relax or study to? We’ve got you covered!

---

### Lo-Fi TryHackMe Writeup

Want to hear some lo-fi beats, to relax or study to? We’ve got you covered!

Room: <https://tryhackme.com/room/lofi>![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-000-7decce02.jpg)

### Overview

This room focuses on **Local File Inclusion (LFI)** and **path traversal** vulnerabilities. Below is a summary of my process and findings while exploiting the target.![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-001-6431e94f.png)![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-002-d34baa09.png)

### Initial Setup

- I started the **machine** and the **AttackBox** provided by TryHackMe.- After launching the AttackBox, I navigated to the target IP address in a web browser.

![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-003-d07637b3.jpg)

### Reconnaissance

On visiting the site, I noticed a section titled **“Discography”** with multiple buttons.

- Clicking on each button under **Discography** changed the URL pattern to:- [http://MACHINE\_IP/?page=relax.php](http://machine_ip/?page=relax.phphttp://MACHINE_IP/?page=sleep.php)- [http://MACHINE\_IP/?page=sleep.php](http://machine_ip/?page=relax.phphttp://MACHINE_IP/?page=sleep.php) and so on.

This hinted that the application was including files dynamically via the `page` parameter — a strong indicator of a potential **LFI vulnerability**.![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-004-65f7b3d5.jpg)

### Exploitation: LFI and Path Traversal

Since the room hinted at **file inclusion**, I tested the following payload:

[http://MACHINE\_IP/?page=../../../../etc/passwd](http://machine_ip/?page=../../../../etc/passwd)

✅ **Success!** The contents of `/etc/passwd` were displayed, confirming the presence of a Local File Inclusion vulnerability.![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-005-e8b904a5.jpg)

### Searching for the Flag

Now that I had confirmed LFI, the next step was to **find and read the flag**. I tried exploring directories that might contain flags, such as:![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-006-a1c75fdb.jpg)

Then, I tried:![image](/assets/images/posts/lo-fi-tryhackme-writeup/img-007-06eb3261.jpg)

### 🎉 Flag found!

The flag was successfully displayed in the browser.

### Conclusion

This was a great exercise in exploiting LFI and understanding the importance of secure file inclusion practices. The key steps included:

- Identifying file inclusion via URL parameters- Performing path traversal- Locating sensitive files such as `/etc/passwd` and the flag

#### Thank you for reading my write-up. I hope you found it useful
