---
layout: post
title: 'Juicy Details TryHackMe Writeup'
date: 2025-07-27 13:35:27 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "A popular juice shop has been breached! Analyze the logs to see what had happened…"
canonical_url: "https://medium.com/@hemanthakrishnach/juicy-details-tryhackme-writeup-66f454c2b34b"
image: "/assets/images/posts/juicy-details-tryhackme-writeup/img-000-eb4c3aae.png"
---

A popular juice shop has been breached! Analyze the logs to see what had happened…

---

A popular juice shop has been breached! Analyze the logs to see what had happened…![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-000-eb4c3aae.png)

Room: <https://tryhackme.com/room/juicydetails>![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-001-07c79e92.png)

### **Introduction**

You were hired as a SOC Analyst for one of the biggest Juice Shops in the world and an attacker has made their way into your network.

Your tasks are:

- Figure out what techniques and tools the attacker used- What endpoints were vulnerable- What sensitive data was accessed and stolen from the environment

An IT team has sent you a **zip file** containing logs from the server. Download the attached file, type in “I am ready!” and get to work! There’s no time to lose!

### **Reconnaissance**

Analyze the provided log files.

Look carefully at:

- What tools the attacker used- What endpoints the attacker tried to exploit- What endpoints were vulnerable

### What tools did the attacker use? (Order by the occurrence in the log)

Let’s skim through the logs from a high level. At the start, we see that the attacker used the nmap scripting engine![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-002-dd33c5d1.png)

If we scroll a bit, we can see that Hydra was used to brute-force the login page.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-003-3ddfeacf.png)

Later, we find that the attacker used SQLmap to discover SQL injection vulnerabilities.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-004-688804ed.png)

In the end, we can find a curl command. `Curl` (short for **Client URL**) is a **command-line tool** that sends HTTP requests and interacts. The attacker succeeded in the SQL injection attack as the curl command returned an HTTP 200 (success) code.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-005-ea84d0f3.png)

Then, we also see the use of feroxbuster. `feroxbuster` is a **directory and file brute-forcing tool** used to discover hidden paths (like `/admin`, `/backup`, `/ftp`, etc.) on web servers.

### What endpoint was vulnerable to a brute-force attack?

If we look at the previous Hydra attacks, we can get the full path of the vulnerable endpoint.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-006-71a9da53.png)

### What endpoint was vulnerable to SQL injection?

Similarly, by looking at the SQLmap logs, we can find the vulnerable endpoint.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-007-70722527.png)

### What parameter was used for the SQL injection?

Look at the parameter after the *search?*![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-008-ed1ad4d5.png)

### What endpoint did the attacker try to use to retrieve files? (Include the /)

The attacker retrieved two .bak files using the FTP endpoint.

### **Stolen data**

Analyze the provided log files.

Look carefully at:

- The attacker’s movement on the website- Response codes- Abnormal query strings

### What section of the website did the attacker use to scrape user email addresses?

Hint: Where can customers usually comment on a shopping website?

During the initial skimming of the logs, I found some product reviews. These must be where the attacker got the email addresses from.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-009-0713905d.png)

### Was their brute-force attack successful? If so, what is the timestamp of the successful login? (Yay/Nay, 11/Apr/2021:09:xx:xx +0000)

Let’s look at the Hydra logs. I found a single HTTP 200 return code indicating a successful login.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-010-784a3691.png)

### What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection?

We can find a query for email and password at the end of the SQL queries.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-011-a0508d11.png)

### What files did they try to download from the vulnerable endpoint? (endpoint from the previous task, question #5)

Previously, we saw that two .bak files were exported.

### What service and account name were used to retrieve files from the previous question? (service, username)

We already know that the service used is FTP. Let’s look at the vsftpd.log for the username.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-012-bd41144d.png)

Here, we can see that the attacker is logged in as anon — anonymous

### What service and username were used to gain shell access to the server? (service, username)

Looking at the auth.log, I saw many failed password attempts through brute forcing. From this, I deduced that the username is *www-data* and the service is ssh.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-013-6ebf05d3.png)

After scrolling a bit, I saw the successful login from the attacker.![image](/assets/images/posts/juicy-details-tryhackme-writeup/img-014-d3a21c21.png)

### Final Thoughts

This room is a fantastic entry into **log analysis**, **web attack methodology**, and **threat detection**.

### Thank you for reading my write-up. I hope you found it helpful.
