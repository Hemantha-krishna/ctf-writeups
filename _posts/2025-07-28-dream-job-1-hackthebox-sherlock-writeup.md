---
layout: post
title: 'Dream Job-1 HackTheBox Sherlock Writeup'
date: 2025-07-28 17:28:41 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
  - sherlock
description: "Sherlock Link: https://app.hackthebox.com/sherlocks/Dream%20Job-1/play"
canonical_url: "https://medium.com/@hemanthakrishnach/dream-job-1-hackthebox-sherlock-325799c244a3"
image: "/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-000-bc3855c5.png"
---

Sherlock Link: https://app.hackthebox.com/sherlocks/Dream%20Job-1/play

---

### Dream Job-1 HackTheBox Sherlock Writeup

Sherlock Link: <https://app.hackthebox.com/sherlocks/Dream%20Job-1/play>![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-000-bc3855c5.png)

### Sherlock Scenario

You are a junior threat intelligence analyst at a Cybersecurity firm. You have been tasked with investigating a Cyber espionage campaign known as Operation Dream Job. The goal is to gather crucial information about this operation.

### Who conducted Operation Dream Job?

[We can find this easily on the MITRE ATT&CK® page of the Operation Dream Job.](https://markettrendingcenter.com/lk_job_oppor.docx)![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-001-208f8dbe.png)

Answer: **Lazarus Group**

### When was this operation first observed?

On the same page, look at the First Seen.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-002-c95608cc.png)

Answer: **September 2019**

### There are 2 campaigns associated with Operation Dream Job. One is `Operation North Star`, what is the other?

We can find this under Associated Campaign Descriptions.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-003-bc8053b4.png)

Answer: **Operation Interception**

### During Operation Dream Job, there were the two system binaries used for proxy execution. One was `Regsvr32`, what was the other?

Press Ctrl+F and search for *proxy.* Then we can find the two system binaries used for proxy execution.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-004-49ca6722.png)

Answer: **Rundll32**

### What lateral movement technique did the adversary use?

Go to the ATT&CK navigate layer, then look at the lateral movement column.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-005-e29beaec.png)

Answer: **Internal Spearphishing**

### What is the technique ID for the previous answer?

![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-006-72175871.png)

Answer: **T1534**

### What Remote Access Trojan did the Lazarus Group use in Operation Dream Job?

Scroll down to Software, and then find the Remote Access Trojan.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-007-814c1b61.png)

Answer: **DRATzarus**

### What technique did the malware use for execution?

Let’s go to the ATT&CK navigate layer of the DRATzarus software and check under the execution column.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-008-7fee2219.png)

Answer: **Native API**

### What technique did the malware use to avoid detection in a sandbox?

Now, let’s look at the defense evasion techniques column. There are a few; we need to look for the virtualization part.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-009-291c8136.png)

Answer: **Time Based Evasion**

### To answer the remaining questions, utilize VirusTotal and refer to the IOCs.txt file. What is the name associated with the first hash provided in the IOC file?

This is the result after searching the first hash in VirusTotal.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-010-60c66c59.png)

Answer: **IEXPLORE.exe**

### When was the file associated with the second hash in the IOC first created?

Search the second hash and under the detail tab, look under History.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-011-9418a71a.png)

Answer: **2020–05–12 19:26:17**

### What is the name of the parent execution file associated with the second hash in the IOC?

Navigate to relations and look under the Execution Parents.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-012-c28eab4a.png)

Answer: **BAE\_HPC\_SE.iso**

### Examine the third hash provided. What is the file name likely used in the campaign that aligns with the adversary’s known tactics?

Under the details->names, the file *Salary\_Lockheed\_Martin\_job\_opportunities\_confidential.doc* looked convincing, as the victims of this campaign are job seekers.![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-013-fac95490.png)

Answer: **Salary\_Lockheed\_Martin\_job\_opportunities\_confidential.doc**

### Which URL was contacted on 2022–08–03 by the file associated with the third hash in the IOC file?

In the Relations tab, look at the data 2022–08–03![image](/assets/images/posts/dream-job-1-hackthebox-sherlock-writeup/img-014-6b606be8.png)

Answer: [**https://markettrendingcenter.com/lk\_job\_oppor.docx**](https://markettrendingcenter.com/lk_job_oppor.docx)

### Conclusion

This exercise provided a hands-on walk-through of threat hunting and cyber espionage analysis using real-world frameworks like **MITRE ATT&CK** and platforms like **VirusTotal**. It highlights the importance of IOC correlation, malware behavior analysis, and understanding adversary tactics to build threat intelligence.

### Thank you for reading my write-up. I hope you found it helpful.
