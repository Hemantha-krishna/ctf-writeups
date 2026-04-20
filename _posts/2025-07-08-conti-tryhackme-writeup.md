---
layout: post
title: 'Conti TryHackMe Writeup'
date: 2025-07-08 05:12:28 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "An Exchange server was compromised with ransomware. Use Splunk to investigate how the attackers compromised the server."
canonical_url: "https://medium.com/@hemanthakrishnach/conti-tryhackme-writeup-54227d34d0e9"
image: "/assets/images/posts/conti-tryhackme-writeup/img-000-60facfbb.jpg"
---

An Exchange server was compromised with ransomware. Use Splunk to investigate how the attackers compromised the server.

---

An Exchange server was compromised with ransomware. Use Splunk to investigate how the attackers compromised the server.![image](/assets/images/posts/conti-tryhackme-writeup/img-000-60facfbb.jpg)

Room: <https://tryhackme.com/room/contiransomwarehgh>![image](/assets/images/posts/conti-tryhackme-writeup/img-001-d5d0cbb1.png)

### Scenario :

Some employees from your company reported that they can’t log into Outlook. The Exchange system admin also reported that he can’t log in to the Exchange Admin Center. After initial triage, they discovered some weird readme files settled on the Exchange server.

Below is a copy of the ransomware note.

![image](/assets/images/posts/conti-tryhackme-writeup/img-002-a41cfce0.png)

Below are the error messages that the Exchange admin and employees see when they try to access anything related to Exchange or Outlook.

**Exchange Control Panel**:![image](/assets/images/posts/conti-tryhackme-writeup/img-003-49005d17.png)

**Outlook Web Access**:![image](/assets/images/posts/conti-tryhackme-writeup/img-004-d3a643c7.png)

**Task**: You are assigned to investigate this situation. Use Splunk to answer the questions below regarding the Conti ransomware.

### Can you identify the location of the ransomware?

First, let’s set the index to look at all the events.
```ini
index=*
```

Next, let’s look at the sysmon logs![image](/assets/images/posts/conti-tryhackme-writeup/img-005-6f046364.png)

And, lets target the .exe files
```ini
index=* sourcetype=”WinEventLog:Microsoft-Windows-Sysmon/Operational” “*.exe*”
```

We get 2,000+ hits. Let’s look at the CurrentDirectory Filter; there are 5 directories. Our answer is one among them![image](/assets/images/posts/conti-tryhackme-writeup/img-006-2dafb183.png)

Lets create a table for the CurrentDirectory and the images.
```diff
index=* sourcetype=”WinEventLog:Microsoft-Windows-Sysmon/Operational” “*.exe*”| table CurrentDirectory, Image| dedup CurrentDirectory
```
![image](/assets/images/posts/conti-tryhackme-writeup/img-007-c30884d1.png)

I found the cmd.exe in the Documents folder, which is suspicious. That’s our answer.

### What is the Sysmon event ID for the related file creation event?

This one’s straightforward: The Sysmon event ID for file creation events is 11

### Can you find the MD5 hash of the ransomware?

To find the hash, i addeId the image to the search
```ini
index=* sourcetype=”WinEventLog:Microsoft-Windows-Sysmon/Operational” Image=”C:\\Users\\Administrator\\Documents\\cmd.exe”
```

Then, I navigated to the first event with cmd.exe. In the details, I found the hash.![image](/assets/images/posts/conti-tryhackme-writeup/img-008-18167b76.png)

### What file was saved to multiple folder locations?

In Sysmon, Event ID 11 logs [FileCreate operations](https://www.google.com/search?client=firefox-b-d&cs=1&sca_esv=9c54605068baa6ec&q=FileCreate+operations&sa=X&ved=2ahUKEwiM_M_8saqOAxUNRWwGHensAPIQxccNegQIAhAC&mstk=AUtExfCQVANA1fSz3pbY3X_ztskZeExmYzRxWijefKsNZ3wT1cL9GMipFL76wrgLecSA-WEGUTfNTd0CbH16NpcgivPdrXLAZQthysH644RNPh5HbWSGqWr744vxaIqaAVCZnU-MRFh5z8k6ccnrnQXOUAwOB3dBmd12RrKKWCi4HBDZxDc&csui=3), which are triggered when a file is created or overwritten. So, let’s add it to the search. We get 104 hits. Let’s create a table for image and targetfilename to find the file.
```diff
index=* sourcetype=”WinEventLog:Microsoft-Windows-Sysmon/Operational” EventCode=11| table Image, TargetFilename
```

After scrolling a bit, I found cmd.exe saving a readme.txt file in multiple locations. This readme is the ransom note we saw earlier.![image](/assets/images/posts/conti-tryhackme-writeup/img-009-fca2b977.png)

### What was the command the attacker used to add a new user to the compromised system?

event ID 4720 is triggered when a new user account is created. When I searched, I found 2 events![image](/assets/images/posts/conti-tryhackme-writeup/img-010-cfd5492c.png)

If we take a look at the latest event, the first event, I found out that the username of the account is *securityninja.* Then I searched for it, and I got 10 events. I navigated to the first occurrence and found the command used to create the user.![image](/assets/images/posts/conti-tryhackme-writeup/img-011-273baaf9.png)

### The attacker migrated the process for better persistence. What is the migrated process image (executable), and what is the original process image (executable) when the attacker got on the system?

*Event ID 8:* The CreateRemoteThread event detects when a process creates a thread in another process. This is generally used for persistence operations — process migration.
```ini
index=* source=”WinEventLog:Microsoft-Windows-Sysmon/Operational” EventCode=8
```

We get 2 events. Now let’s create a table for SourceImage and TargetImage
```bash
index=* source=”WinEventLog:Microsoft-Windows-Sysmon/Operational” EventCode=8| table SourceImage TargetImage
```
![image](/assets/images/posts/conti-tryhackme-writeup/img-012-91bf62f1.png)

We can see that the attacker migrated from powershell.exe to unsecapp.exe and later to lsass.exe

### The attacker also retrieved the system hashes. What is the process image used for getting the system hashes?

Attackers commonly use **lsass.exe** to extract credentials, and we have seen earlier that the attacker migrated to lsass.exe

### What is the web shell the exploit deployed to the system?

Attackers use .aspx files to deploy webshells. So, let’s add it to the search. I found 1 event, and the name of the file is in the CommandLine
```ini
index=* source=”WinEventLog:Microsoft-Windows-Sysmon/Operational” aspx
```
![image](/assets/images/posts/conti-tryhackme-writeup/img-013-03b6fb62.png)

Another way to find this is by using the hint, “Try looking in the IIS logs for POST requests.”
```ini
index=* sourcetype=iis cs_method=POST
```

then, i looked at the *cs\_url\_stem* and found the aspx file![image](/assets/images/posts/conti-tryhackme-writeup/img-014-6327d164.png)

### What is the command line that executed this web shell?

From the previous found event, we can get the full command![image](/assets/images/posts/conti-tryhackme-writeup/img-015-b9840d3b.png)

### What three CVEs did this exploit leverage? Provide the answer in ascending order.

We have to search for this on our own, through external research. I found the answer after looking at:

<https://www.tenable.com/blog/contileaks-chats-reveal-over-30-vulnerabilities-used-by-conti-ransomware-affiliates>

<https://www.securin.io/articles/is-conti-ransomware-on-a-roll>

### Thank you for reading my write-up. I hope you found it useful.
