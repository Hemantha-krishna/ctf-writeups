---
layout: post
title: 'Cracking the Dev Box: From NFS Misconfiguration to Sudo Zip Escalation'
date: 2026-01-23 20:04:41 +0000
categories: [cybersecurity]
tags:
  - cybersecurity
description: "This article is a walkthrough for the “Dev” machine, a capture-the-flag (CTF) challenge featured in TCM Security’s Practical Ethical…"
canonical_url: "https://medium.com/@hemanthakrishnach/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation-6eec4fa20cca"
image: "/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-000-b83318e8.png"
---

This article is a walkthrough for the “Dev” machine, a capture-the-flag (CTF) challenge featured in TCM Security’s Practical Ethical…

---

This article is a walkthrough for the “Dev” machine, a capture-the-flag (CTF) challenge featured in **TCM Security’s Practical Ethical Hacking (PEH)** course. In this write-up, I will break down the exploitation of this box, which demonstrates a classic chain of vulnerabilities: exposed network shares, weak password protection, web application vulnerabilities (LFI), and a misconfigured sudoer binary.

### Phase 1: Reconnaissance

I started with a comprehensive Nmap scan against the target IP to identify open ports and services.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-000-b83318e8.png)![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-001-bf03502e.png)

The scan revealed several interesting ports:

- **Port 22:** OpenSSH 7.9p1.- **Port 80 & 8080:** Apache httpd 2.4.38.- **Port 111 & 2049:** NFS (Network File System) and rpcbind.

Visiting port 80 revealed a “Bolt Installation error” , while port 8080 hosted a BoltWire site labeled “dev”.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-002-d979533e.png)

### Phase 2: Enumerating NFS

Given the presence of port 2049, I checked for exposed file shares using `showmount`. The server was exporting `/srv/nfs`, which was accessible to our network.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-003-83daa04a.png)

I mounted the share to our local machine to inspect the contents:
```bash
mkdir /tmp/nfs_sharesudo mount -t nfs 192.168.57.7:/srv/nfs /tmp/nfs_share
```
![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-004-8f0754b5.png)

Inside the share, I found a file named `save.zip`. Attempting to unzip it revealed it was password-protected.

### Phase 3: Cracking the Archive

To access the zip file, I used `zip2john` (implied by the hash format) to extract the hash and saved it to `save.txt`. I then ran **John the Ripper** using the `rockyou.txt` wordlist:
```bash
john save.txt — wordlist=/usr/share/wordlists/rockyou.txt
```
![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-005-bf09a715.png)

John successfully cracked the password: `java101`

With the password, I extracted the archive, which contained an SSH private key (`id_rsa`) and a `todo.txt` file. The text file contained a hint: "Keep coding in Java because it's awesome," and suggested the user might be named `jp` or similar.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-006-2327a953.png)

### Phase 4: Web Enumeration and LFI

While I had an SSH key, I didn’t have the correct username or the passphrase for the key yet. I tried the cracked password of the save.zip file but it didn’t work.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-007-149314f4.png)

I turned my attention to the web server on port 8080. I ran a directory scan using `ffuf` and confirmed the existence of the `/dev/` directory running BoltWire.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-008-447b3d99.png)![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-009-83fc4725.png)![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-010-2f87169f.png)

I discovered that this version of BoltWire (6.03) is vulnerable to **Local File Inclusion (LFI)**. By manipulating the search action in the URL, I could traverse directories and read system files.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-011-c500c0f8.png)

The Payload:
```bash
http://<MACHINE_IP>:8080/dev/index.php?p=action.search&action=../../../../../../../etc/passwd
```

The server returned the contents of `/etc/passwd`, allowing me to identify a human user on the system: `jeanpaul`![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-012-7202886d.png)

### Phase 5: Gaining a Foothold

I previously found a database dump or config info mentioning a password “I love java”.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-013-8ae355bc.png)

Using the username `jeanpaul` found via LFI and the extracted `id_rsa` key, I attempted to login via SSH
```bash
ssh -i id_rsa jeanpaul@MACHINE_IP
```
![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-014-9a2a698e.png)

The key required a passphrase. Based on the previous enumeration hints, I tried `I_love_java`, which was accepted. I successfully logged in as `jeanpaul`

### Phase 6: Privilege Escalation

Once inside, I checked the user’s sudo privileges:
```typescript
sudo -l
```

The output showed that `jeanpaul` could run `/usr/bin/zip` as root without a password. I checked [GTFOBins](https://gtfobins.org/gtfobins/zip/) and found that `zip` can be used to spawn a shell if it runs with sudo privileges.![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-015-960f23f8.png)

The Exploit:
```bash
sudo zip $TF /etc/hosts -T -TT ‘sh #’
```

These commands successfully dropped me into a root shell. I navigated to the root directory and captured the flag:
```csharp
Congratz on rooting this box!
```
![image](/assets/images/posts/cracking-the-dev-box-from-nfs-misconfiguration-to-sudo-zip-escalation/img-016-ea559ee0.png)

### Thanks for reading! I hope you found it useful.
