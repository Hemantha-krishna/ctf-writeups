---
layout: post
title: 'Retracted'
date: 2025-07-05 10:38:28 +0000
categories: [cybersecurity]
tags:
  - cybersecurity
description: "Investigate the case of the missing ransomware."
canonical_url: "https://medium.com/@hemanthakrishnach/retracted-418d2b52a7e6"
image: "/assets/images/posts/retracted/img-000-6402bad1.png"
---

Investigate the case of the missing ransomware.

---

Investigate the case of the missing ransomware.

Room: <https://tryhackme.com/room/retracted>![image](/assets/images/posts/retracted/img-000-6402bad1.png)![image](/assets/images/posts/retracted/img-001-6ed7c51e.png)

### Scenario

> “Thanks for coming. I know you’re busy with your new job, but I didn’t know who else to call…”

Sophie’s system was hit by what appeared to be ransomware: files were locked, the wallpaper changed, and a ransom note was left behind. But by the time she returned, everything was back to normal, except for a strange message and an alert about a Bitcoin transaction.

As analysts, our job is to dive into the endpoint, examine the digital trail, and make sense of this bizarre turn of events.

The machine looks like this:![image](/assets/images/posts/retracted/img-002-db1b193c.jpg)![image](/assets/images/posts/retracted/img-003-c811e0b8.jpg)

### What is the full path of the text file containing the “message”?

Check the **Desktop** for any unusual files. You’ll find a file named `SOPHIE.txt`. Right-click → **Properties** → Note the full path:
```makefile
C:\Users\Sophie\Desktop\SOPHIE.txt
```

### What program was used to create the text file?

We can find this using **Sysmon logs**.

1. Open **Event Viewer**.- Navigate to: `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`- Use **Find** → Search for `SOPHIE.txt`.- The corresponding event shows the parent process as notepad.exe

![image](/assets/images/posts/retracted/img-004-9d5c32a9.jpg)

### What is the time of execution of the process that created the text file? Timezone UTC (Format YYYY-MM-DD hh:mm:ss)

From the same event where we found `notepad.exe`, note the exact UTC timestamp:![image](/assets/images/posts/retracted/img-005-1f799ff2.jpg)![image](/assets/images/posts/retracted/img-006-62814b08.jpg)

### What is the filename of this “installer”? (Including the file extension)

Since, it is downloaded form the google, Open the browser, go to **Downloads**. You’ll find the suspicious file:![image](/assets/images/posts/retracted/img-007-e901a28e.jpg)

### What is the download location of this installer?

Click the folder icon next to the download in the browser. It leads to:![image](/assets/images/posts/retracted/img-008-a2e5ad9b.jpg)

### The installer encrypts files and then adds a file extension to the end of the file name. What is this file extension?

Inspect the Sysmon logs for file modification events related to `antivirus.exe`. You’ll notice several files ending with .dmp![image](/assets/images/posts/retracted/img-009-5e6a1712.jpg)

### The installer reached out to an IP. What is this IP?

To investigate network connections:

1. In Event Viewer, filter Sysmon logs to **Event ID 3** (network connections).- Use **Find** → Search for `antivirus.exe`.

You’ll find it connected to:![image](/assets/images/posts/retracted/img-010-50cc6b9a.jpg)![image](/assets/images/posts/retracted/img-011-bed76683.jpg)

### The threat actor logged in via RDP right after the “installer” was downloaded. What is the source IP?

Filter Event ID 3 for RDP activity. Look for the RDP session **after the installer runs**.

You’ll find a connection from:![image](/assets/images/posts/retracted/img-012-ec37c7b6.jpg)

### This other person downloaded a file and ran it. When was this file run? Timezone UTC (Format YYYY-MM-DD hh:mm:ss)

Back in the **Downloads** folder, there are two files: `antivirus.exe` and `decryptor.exe`.

Search the logs for `decryptor.exe`. Note the process creation time:![image](/assets/images/posts/retracted/img-013-782e480a.jpg)![image](/assets/images/posts/retracted/img-014-806847d8.jpg)

### 🔄 Arrange the events in the correct order:

1. Sophie downloaded the malware and ran it.- The downloaded malware encrypted the files and showed a ransomware note.- After seeing the ransomware note, Sophie ran out and reached out to you for help.- While Sophie was away, an intruder logged into Sophie’s machine via RDP and started looking around.- The intruder realized he infected a charity organization. He then downloaded a decryptor and decrypted all the files.- After all the files are restored, the intruder left the desktop telling Sophie to check her Bitcoin.- Sophie and I arrive on the scene to investigate. At this point, the intruder was gone.

![image](/assets/images/posts/retracted/img-015-54fc7ec6.jpg)

### Conclusion

Sometimes, even ransomware authors have a conscience. It seems the attacker realized they’d hit a charity, reversed the damage, and made a donation. Unusual? Definitely. But it wraps this cyber mystery with an oddly humane twist.

### Thank you for reading my write-up. I hope you found it useful
