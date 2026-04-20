---
layout: post
title: 'ItsyBitsy TryHackMe Writeup'
date: 2025-07-06 11:36:07 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Put your ELK knowledge together and investigate an incident."
canonical_url: "https://medium.com/@hemanthakrishnach/itsybitsy-tryhackme-writeup-d6a6755db7d8"
image: "/assets/images/posts/itsybitsy-tryhackme-writeup/img-000-ce344e9a.png"
---

Put your ELK knowledge together and investigate an incident.

---

Put your ELK knowledge together and investigate an incident.![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-000-ce344e9a.png)

Room: <https://tryhackme.com/room/itsybitsy>![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-001-8bca365c.png)

### Scenario

During routine SOC monitoring, an IDS alert flagged potential C2 activity involving a user named **Browne** from the HR department. The alert indicated that a file with a malicious pattern (`THM:{________}`) may have been accessed. Our task is to investigate one week’s worth of HTTP connection logs—available via the `connection_logs` index in Kibana—to uncover what happened.

### 1. How many events were returned for the month of March 2022?

- Head over to the **Discover** tab in Kibana.- Set the time range: **From**: March 1, 2022, **To**: March 31, 2022- The total number of events is displayed at the top.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-002-e183c6af.jpg)

### 2. What is the IP associated with the suspected user in the logs?

- Searching for “**Browne**” yielded no direct hits.- However, only **two source IPs** are active in the logs.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-003-5ced827d.jpg)

- Upon investigating:- The **first IP** showed normal behavior.- The **second IP** was accessing **pastebin.com** over **port 80 —**highly suspicious.- Click the **“+” icon** beside the suspicious IP to filter the logs.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-004-87765403.jpg)

### 3. The user’s machine used a legit windows binary to download a file from the C2 server. What is the name of the binary?

- With the suspicious IP filtered, look into the **user\_agent** field.- This reveals the **Windows binary** used to make the HTTP request.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-005-ad24b215.jpg)

### 4. The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?

- From the logs, check the **host** field.- The infected machine accessed **pastebin.com**, a known file-sharing and code-pasting service often abused by attackers.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-006-218f2c7a.jpg)

### What is the full URL of the C2 to which the infected host is connected?

With the **host** and **uri** fields in hand:

- Host: `pastebin.com`- URI: `/yTg0Ah6a`

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-007-884064a5.jpg)

So, our URL will be pastebin.com/yTg0Ah6a

### A file was accessed on the filesharing site. What is the name of the file accessed?

- Visit the full URL in your browser.- The page displays the file name and its contents.

![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-008-e53f4211.jpg)

### The file contains a secret code with the format THM{\_\_\_\_\_}.

From the content of the file, extract the flag.![image](/assets/images/posts/itsybitsy-tryhackme-writeup/img-009-903021c1.jpg)

### Conclusion

This was a fantastic mini-challenge that put ELK stack skills to the test. It simulated a real-world SOC investigation and highlighted the importance of correlation between logs and known threat behaviors.

### Thank you for reading my write-up. I hope you found it useful
