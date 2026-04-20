---
layout: post
title: 'Volt Typhoon TryHackMe Writeup'
date: 2025-07-12 17:19:18 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Investigate a suspected intrusion by the notorious APT group Volt Typhoon."
canonical_url: "https://medium.com/@hemanthakrishnach/volt-typhoon-61dc13664cad"
image: "/assets/images/posts/volt-typhoon-tryhackme-writeup/img-000-83c82a5c.png"
---

Investigate a suspected intrusion by the notorious APT group Volt Typhoon.

---

Investigate a suspected intrusion by the notorious APT group Volt Typhoon.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-000-83c82a5c.png)

Room: <https://tryhackme.com/room/volttyphoon>![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-001-5a95a285.png)

### **Scenario**

The SOC has detected suspicious activity indicative of an advanced persistent threat (APT) group known as Volt Typhoon, notorious for targeting high-value organizations. Assume the role of a security analyst and investigate the intrusion by retracing the attacker’s steps.

You have been provided with various log types from a two-week time frame during which the suspected attack occurred. Your ability to research the suspected APT and understand how they maneuver through targeted networks will prove to be just as important as your Splunk skills.

### Task 1: Initial Access

Given Info: “Volt Typhoon often gains initial access to target networks by exploiting vulnerabilities in enterprise software. In recent incidents, Volt Typhoon has been observed leveraging vulnerabilities in Zoho ManageEngine ADSelfService Plus, a popular self-service password management solution used by organizations.”

### Q1. Comb through the ADSelfService Plus logs to begin retracing the attacker’s steps. At what time (ISO 8601 format) was Dean’s password changed and their account taken over by the attacker?

Our investigation starts by searching for signs of account manipulation. We’ll focus on the compromised user, “Dean,” and look for password-related events.

A simple Splunk search gets us started:
```typescript
dean password
```

The logs quickly reveal a critical event. An attacker successfully reset Dean’s password, effectively hijacking the account.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-002-2d0a664c.png)

### Q2. Shortly after Dean’s account was compromised, the attacker created a new administrator account. What is the name of the new account that was created?

With control of an admin account, a common next step for an attacker is to create a new, dedicated account for persistence. This gives them a backdoor in case the originally compromised account is discovered and locked. We’ll search for account creation events around the time of the initial compromise.

I searched for *admin* and found two usernames in the username filter. One is the previous compromised *dean-admin* and the other account is *voltyp-admin.*
```typescript
admin
```
![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-003-4276664e.png)

### Task 2:Execution

Given Info: “Volt Typhoon is known to exploit Windows Management Instrumentation Command-line (WMIC) for a range of execution techniques. They leverage WMIC for tasks such as gathering information and dumping valuable databases, allowing them to infiltrate and exploit target networks. By using “living off the land” binaries (LOLBins), they blend in with legitimate system activity, making detection more challenging.”

### Q3. In an information gathering attempt, what command does the attacker run to find information about local drives on server01 & server02?

The attacker needs to map out their surroundings. A logical next step is to enumerate the available drives on key servers. We can search for commands targeting both servers simultaneously.

I searched for *server01 server02* and got 1 resultwhich had the command run by the attacker to find about the local drives.
```typescript
server01 server02
```

To not miss any events, I updated the search to
```markdown
*server01*server02*
```

I still got one event![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-004-b1edbf4d.png)

### Q4. The attacker uses ntdsutil to create a copy of the AD database. After moving the file to a web server, the attacker compresses the database. What password does the attacker set on the archive?

After identifying valuable data, the attacker’s next move is to package it for exfiltration. They use ntdsutil to dump the Active Directory database (ntds.dit), move it, and then compress it with a password.

I searched for 7z as it is one of the common compressed file’s extension. This got me two events. Then, I got the password from the command:
```sql
wmic /node:webserver-01 process call create “cmd.exe /c 7z a -v100m -p d5ag0nm@5t3r -t7z cisco-up.7z C:\inetpub\wwwroot\temp.dit”
```

We can get the password from after the -p parameter.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-005-c857afa7.png)

Another way to solve this is by searching for *ntdsutil* leads us to the creation of *temp.dit.* Pivoting our search to this filename or the 7z compression utility reveals the command used to create the archive.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-006-10463c92.png)![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-007-5790d42c.png)

### Persistence

Given Info: “Our target APT frequently employs web shells as a persistence mechanism to maintain a foothold. They disguise these web shells as legitimate files, enabling remote control over the server and allowing them to execute commands undetected.”

### Q5. To establish persistence on the compromised server, the attacker created a web shell using base64 encoded text. In which directory was the web shell placed?

Volt Typhoon uses Base64 encoding to obfuscate their web shell payload. We can hunt for PowerShell or certutil commands that decode Base64 strings. A search for certutil -decode yields a direct hit. The command line shows exactly where the decoded file (the web shell) was saved.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-008-a13b4824.png)

### Defense Evasion

Given Info: “Volt Typhoon utilizes advanced defense evasion techniques to significantly reduce the risk of detection. These methods encompass regular file purging, eliminating logs, and conducting thorough reconnaissance of their operational environment.”

### Q6. In an attempt to begin covering their tracks, the attackers remove evidence of the compromise. They first start by wiping RDP records. What PowerShell cmdlet does the attacker use to remove the “Most Recently Used” record?

To erase evidence of their remote connections, the attacker targets the registry where RDP history is stored. In PowerShell, the cmdlet used to **remove a registry entry** is: *Remove-ItemProperty*.

![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-009-08ece47e.png)

Command:
```bash
Remove-ItemProperty -Path $registryPath -Name MRU0 -ErrorAction SilentlyContinue
```

**Explanation:**

- `Remove-ItemProperty`: Deletes a property from a registry key.- `-Path $registryPath`: Specifies the path to the registry key (stored in the variable `$registryPath`).- `-Name MRU0`: Specifies the property (registry value) to remove — in this case, `"MRU0"`.- `-ErrorAction SilentlyContinue`: Suppresses error messages if the property does not exist or an error occurs.

### Q7. The APT continues to cover their tracks by renaming and changing the extension of the previously created archive. What is the file name (with extension) created by the attackers?

.7z file containing an AD database is highly suspicious. To hide it in plain sight, the attacker renames it to something that looks benign, like an image file. Earlier, we got 2 events when we searched for the archive file.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-010-ad7fc72c.png)

The first command:
```r
wmic /node:webserver-01 process call create “cmd.exe /c ren \\webserver-01\c$\inetpub\wwwroot\cisco-up.7z cl64.gif”
```

It **remotely renames a file** on another computer (`webserver-01`) by using `WMIC` (Windows Management Instrumentation Command-line). The file`cisco-up.7z` is renamed to `cl64.gif`.

### Q8. Under what regedit path does the attacker check for evidence of a virtualized environment?

Attackers often check if they are operating within a virtual machine or sandbox, as this could indicate an analysis environment. They do this by querying specific registry keys. A search for Get-ItemProperty combined with terms like virtual points us to the exact path they checked.

**Top-Level Registry Hives:**![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-011-8aa02763.png)

When I searched for HKEY\_LOCAL\_MACHINE, I got 3 hits. One of them is the attacker checking for virtual environment. We can get the path from the command.
```vbnet
Get-ItemProperty -Path “HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control” | Select Object -Property *Virtual*
```

![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-012-77911e8d.png)

### Credential Access

Given Info: “Volt Typhoon often combs through target networks to uncover and extract credentials from a range of programs. Additionally, they are known to access hashed credentials directly from system memory.”

### Q9. Using reg query, Volt Typhoon hunts for opportunities to find useful credentials. What three pieces of software do they investigate? Answer Format: Alphabetical order separated by a comma and space.

Saved credentials in common remote access and SSH tools are a goldmine. A reg query search shows the attacker systematically hunting for credentials associated with popular software. We can find the name of the software from the command line. Softwares investigated: OpenSSH, putty, realvnc

![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-013-b7c8df73.png)![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-014-1e9454df.png)

### Q10. What is the full decoded command the attacker uses to download and run mimikatz?

Mimikatz is a powerful tool for dumping credentials from memory. It’s generally obfuscated. We can find it by looking for encoded PowerShell commands (-e or -enc). After finding the Base64 string, we can decode it using a tool like CyberChef.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-015-be493c86.png)

Decoding the command using Cyberchef — <https://gchq.github.io/CyberChef/>![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-016-506a867b.png)

### Discovery

Given Info: “Volt Typhoon uses enumeration techniques to gather additional information about network architecture, logging mechanisms, successful logins, and software configurations, enhancing their understanding of the target environment for strategic purposes.”

### Lateral Movement

Given Info: “The APT has been observed moving previously created web shells to different servers as part of their lateral movement strategy. This technique facilitates their ability to traverse through networks and maintain access across multiple systems.”

### Q11. The attacker uses wevtutil, a log retrieval tool, to enumerate Windows logs. What event IDs does the attacker search for? Answer Format: Increasing order separated by a space.

To understand login patterns, the attacker queries the Windows Event Logs for specific event IDs related to successful logons. A search for *wevtutil* reveals which IDs they were interested in.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-017-dc86a6ec.png)

### Q12. Moving laterally to server-02, the attacker copies over the original web shell. What is the name of the new web shell that was created?

To expand their access, the attacker copies the web shell to another server. A search for file copy operations targeting server-02 around the time of the lateral movement shows the new filename they used.

Searching for *server-02* and got 500+ results, to narrow it down, I added *copy* to the search. I got exactly one result.
```go
server-02 copy
```
![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-018-37c959ec.png)

This command copies a file from a local path to a remote server’s web directory over the network.

### Collection

Given Info: “During the collection phase, Volt Typhoon extracts various types of data, such as local web browser information and valuable assets discovered within the target environment.”

### Q13. The attacker is able to locate some valuable financial information during the collection phase. What three files does Volt Typhoon make copies of using PowerShell? **Answer Format: Increasing order separated by a space.**

During the collection phase, the attacker found and staged sensitive financial documents.

I Searched for *\*finance\** and got 4 events. We can get the file name and its path from the CommandLine
```markdown
*finance*
```

![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-019-533df176.png)

### C2

Given Info: “Volt Typhoon utilizes publicly available tools as well as compromised devices to establish discreet command and control (C2) channels.”

### Cleanup

Given Info: “To cover their tracks, the APT has been observed deleting event logs and selectively removing other traces and artifacts of their malicious activities.”

### The attacker uses netsh to create a proxy for C2 communications. What connect address and port does the attacker use when setting up the proxy? Answer Format: IP Port

To exfiltrate data and maintain C2, the attacker sets up a network proxy using the netsh utility. The logs show the portproxy command, which specifies the IP and port they are connecting back to.
```typescript
netsh
```

![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-020-b44bd2a0.png)

### To conceal their activities, what are the four types of event logs the attacker clears on the compromised system?

The final, and perhaps loudest, act is to erase the evidence. We already saw that wevtutil was used to access the windows event log. Later, The attacker uses *wevtutil* with the *cl (clear-log)* argument to wipe the very logs we’ve been analyzing. The command explicitly lists the log types they cleared.![image](/assets/images/posts/volt-typhoon-tryhackme-writeup/img-021-1bab6f2e.png)
```sql
wevtutil cl Application Security Setup System
```

### Conclusion

By meticulously following the breadcrumbs in Splunk, we’ve successfully reconstructed the entire attack chain of a Volt Typhoon intrusion. From the initial password reset to the final log clearing, each step highlights the group’s reliance on stealth, built-in tools, and careful evasion. This investigation underscores the importance of comprehensive logging and the power of a SIEM like Splunk to unmask even the most sophisticated threats.

### Thank you for reading my write-up. I hope you found it helpful.
