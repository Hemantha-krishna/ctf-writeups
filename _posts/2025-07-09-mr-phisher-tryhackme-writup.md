---
layout: post
title: 'Mr. Phisher TryHackMe Writup'
date: 2025-07-09 09:37:48 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "I received a suspicious email with a very weird looking attachment. It keeps on asking me to “enable macros”. What are those?"
canonical_url: "https://medium.com/@hemanthakrishnach/mr-phisher-tryhackme-writup-08dd4b8aaddf"
image: "/assets/images/posts/mr-phisher-tryhackme-writup/img-000-b53ea3aa.png"
---

I received a suspicious email with a very weird looking attachment. It keeps on asking me to “enable macros”. What are those?

---

I received a suspicious email with a very weird looking attachment. It keeps on asking me to “enable macros”. What are those?![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-000-b53ea3aa.png)

Room: <https://tryhackme.com/room/mrphisher>![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-001-27012ed0.png)

After launching the virtual machine, I navigated to the working directory.Here, I found two files:

- `MrPhisher.docm`- `mr-phisher.zip,` which contains the same `.docm` file.

![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-002-ec94d2ad.png)

When opening `MrPhisher.docm`, LibreOffice Writer prompted a security warning about **macros.**![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-003-75d46c03.png)

Click OK and continue.![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-004-4f4aa054.png)

Let’s investigate the macros. Navigate to Tools->Macros->Edit Macros.![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-005-3d32cf22.png)

I found the script in MrPhisher.docm->Project->Modules->NewMacros->Format. We can also find the macros using the *oledump* tool.![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-006-d9a08b89.png)

Here’s the full code:
```vbnet
Rem Attribute VBA_ModuleType=VBAModuleOption VBASupport 1Sub Format()Dim a()Dim b As Stringa = Array(102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88)For i = 0 To UBound(a)b = b & Chr(a(i) Xor i)NextEnd Sub
```

### What’s Going On Here?

This macro uses **XOR obfuscation** to hide a message inside an array of integers. Here’s what happens:

1. An array `a` is filled with encoded values.- A loop XORs each value with its index.- The result is converted to a character (`Chr(...)`) and appended to a string `b`.

This is a simple but effective method to hide malicious or sensitive strings from static analysis.

### Decrypting the Payload

To reveal what the macro was hiding, I translated it into a Python script for easy decoding:![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-007-6d4f06ed.png)
```python
a = [102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88]b=””for i in range(len(a)): b += chr(a[i]^i)print(b)
```

### Explanation

- The list `a` contains obfuscated character codes.- Each value is XORed with its index (`a[i] ^ i`) to get the original character code.- The result is decoded into a character using `chr(...)` and appended to the string `b`.- The final output is our flag

![image](/assets/images/posts/mr-phisher-tryhackme-writup/img-008-8b95c452.png)

### Final Thoughts

This was a fun and educational challenge demonstrating the **importance of caution around macro-enabled documents**. Phishing attacks commonly use `.docm` files with obfuscated scripts just like this, except in real incidents, the payloads are far more dangerous.

### Thank you for reading my write-up. I hope you found it helpful.
