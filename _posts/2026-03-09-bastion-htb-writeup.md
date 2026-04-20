---
layout: post
title: 'Bastion - HTB Writeup'
date: 2026-03-09 02:38:14 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
description: "Room Link: https://app.hackthebox.com/machines/Bastion"
canonical_url: "https://medium.com/@hemanthakrishnach/bastion-htb-writeup-f11e3b0cf8c5"
image: "/assets/images/posts/bastion-htb-writeup/img-000-ff16b44b.png"
---

Room Link: https://app.hackthebox.com/machines/Bastion

---

### Bastion — HTB Writeup

![image](/assets/images/posts/bastion-htb-writeup/img-000-ff16b44b.png)

Room Link: <https://app.hackthebox.com/machines/Bastion>

### Enumeration

### Nmap

Starting with a full TCP port scan:
```bash
nmap -T4 -p- -A <MACHINE_IP>
```
![image](/assets/images/posts/bastion-htb-writeup/img-001-e05964b7.png)

Key findings:

- **Port 22** — OpenSSH for Windows 7.9- **Port 135/139/445** — SMB (Windows Server 2016 Standard)- **Port 5985 / 47001** — WinRM (Microsoft HTTPAPI)- Multiple high RPC ports

### SMB Enumeration

Listing shares anonymously:
```cpp
smbclient -L //<MACHINE_IP>
```
![image](/assets/images/posts/bastion-htb-writeup/img-002-76ec2c9b.png)

Four shares are visible: `ADMIN$`, `C$`, `IPC$`, and **Backups**. The `Backups` share is the interesting one — let's see if anonymous access is permitted:
```cpp
smbclient //<MACHINE_IP>/Backups -U anonymous
```

It works without a password. Inside we find a `note.txt` and a `WindowsImageBackup` directory.![image](/assets/images/posts/bastion-htb-writeup/img-003-06ba2122.png)

The note warns sysadmins not to transfer the entire backup locally due to VPN speed — conveniently, we don’t have to.![image](/assets/images/posts/bastion-htb-writeup/img-004-7e8eb9ec.png)

### Mounting the VHD Without Downloading It

The `WindowsImageBackup` folder contains a Windows backup for a user called **L4mpje-PC**, including two `.vhd` (Virtual Hard Disk) files.![image](/assets/images/posts/bastion-htb-writeup/img-005-bdcf3c66.png)

Rather than pulling gigabytes over the network, we mount the share directly and use `guestmount` to access the VHDs in place.

[**Mounting VHD file on Kali Linux through remote share**\
*Recently I came across an instance where I found a .vhd image through a SMB share. Virtual Hard Disk (VHD) files are…*medium.com](https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25 "https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25")

#### Step 1 — Mount the SMB share:

```bash
sudo mkdir /mnt/bastionsudo mount -t cifs //<MACHINE_IP>/Backups /mnt/bastion -o ‘rw,username=anonymous’
```
![image](/assets/images/posts/bastion-htb-writeup/img-006-390ca7f2.png)![image](/assets/images/posts/bastion-htb-writeup/img-007-b979e94b.png)

#### Step 2 — Mount each VHD:

```bash
sudo guestmount -a “/mnt/bastion/WindowsImageBackup/L4mpje-PC/Backup 2019–02–22 124351/9b9cfbc3–369e-11e9-a17c-806e6f6e6963.vhd” -m /dev/sda1 — ro /mnt/vhd
```

```bash
sudo guestmount -a “/mnt/bastion/WindowsImageBackup/L4mpje-PC/Backup 2019–02–22 124351/9b9cfbc4–369e-11e9-a17c-806e6f6e6963.vhd” -m /dev/sda1 — ro /mnt/vhd2
```
![image](/assets/images/posts/bastion-htb-writeup/img-008-048ea6db.png)![image](/assets/images/posts/bastion-htb-writeup/img-009-253ebe47.png)![image](/assets/images/posts/bastion-htb-writeup/img-010-86043db4.png)![image](/assets/images/posts/bastion-htb-writeup/img-011-eb40ff21.png)

The first VHD is the boot partition. The second (`vhd2`) is the system drive — it contains the full Windows directory structure including `Program Files`, `Users`, and `Windows`.

### Extracting Credentials from the Registry

With the system drive mounted, we have access to the Windows registry hives. These files live at:
```bash
/mnt/vhd2/Windows/System32/config/
```
![image](/assets/images/posts/bastion-htb-writeup/img-012-39ccf017.png)

Copy the three key files — SAM, SECURITY, and SYSTEM — to our local machine and run Impacket’s `secretsdump`:
```sql
secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL
```

The output reveals NTLM hashes for three accounts:

- `Administrator`- `Guest`- `L4mpje`

![image](/assets/images/posts/bastion-htb-writeup/img-013-f6d47bc5.png)

We crack the hashes with hashcat against rockyou:![image](/assets/images/posts/bastion-htb-writeup/img-014-a145ed2c.png)
```bash
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```
![image](/assets/images/posts/bastion-htb-writeup/img-015-37b607f7.png)

The L4mpje hash cracks quickly to: `bureaulampje`

### Initial Access — SSH as L4mpje

With valid credentials, we SSH in directly:
```kotlin
ssh L4mpje@<MACHINE_IP>
```
![image](/assets/images/posts/bastion-htb-writeup/img-016-ed37b14d.png)

We land in a Windows command prompt. The user flag is sitting on the Desktop:
```bash
C:\Users\L4mpje\Desktop> type user.txt
```
![image](/assets/images/posts/bastion-htb-writeup/img-017-598cef6f.png)

### Privilege Escalation — mRemoteNG Password Extraction

Checking installed software in `Program Files (x86)`, we notice **mRemoteNG version 1.76.11** — a remote connection manager known to store credentials in an encrypted (but breakable) format.![image](/assets/images/posts/bastion-htb-writeup/img-018-13aead9d.png)

Searching for CVE-2023–30367 turns up a password dumper: [S1lkys/CVE-2023–30367-mRemoteNG-password-dumper](https://github.com/S1lkys/CVE-2023-30367-mRemoteNG-password-dumper). We can’t clone it directly on the target, but we know where mRemoteNG stores its config:
```makefile
C:\Users\L4mpje\AppData\Roaming\mRemoteNG\confCons.xml
```
![image](/assets/images/posts/bastion-htb-writeup/img-019-9de1df83.png)

Reading `confCons.xml` reveals an XML node for an **Administrator** connection with an AES-GCM encrypted password field.![image](/assets/images/posts/bastion-htb-writeup/img-020-35a7784e.png)![image](/assets/images/posts/bastion-htb-writeup/img-021-eb93cf98.png)

We copy the password string and use the [decryption tool](https://github.com/S1lkys/CVE-2023-30367-mRemoteNG-password-dumper) on our Kali machine:
```bash
python mremoteng_decrypt.py -s “aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==”
```

The password is revealed in plaintext.![image](/assets/images/posts/bastion-htb-writeup/img-022-45a71d4c.png)

### Root Access

With the Administrator password in hand:
```kotlin
ssh administrator@<MACHINE_IP>
```
![image](/assets/images/posts/bastion-htb-writeup/img-023-ca23fce5.png)

We’re in as Administrator. The root flag is on the Desktop:
```bash
C:\Users\Administrator\Desktop> type root.txt
```
![image](/assets/images/posts/bastion-htb-writeup/img-024-d9615161.png)

### Key Takeaways

- **Anonymous SMB shares are dangerous**, especially when they expose backup data. Backup files should never be accessible without authentication.- **Windows image backups contain registry hives** which hold password hashes. With offline access to these files, credential extraction is trivial.- **mRemoteNG stores passwords using weak encryption** with a known static default key, making stored credentials recoverable without any brute force.- The entire privilege escalation path involved zero CVE exploitation on the live system — purely credential recovery from misconfigured storage.

### Thanks for reading!
