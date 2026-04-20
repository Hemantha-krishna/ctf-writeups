---
layout: post
title: 'W1seGuy - TryHackMe'
date: 2025-07-04 08:58:21 +0000
categories: [cybersecurity]
tags:
  - tryhackme
description: "A w1se guy 0nce said, the answer is usually as plain as day."
canonical_url: "https://medium.com/@hemanthakrishnach/w1seguy-tryhackme-24783da0e5bc"
image: "/assets/images/posts/w1seguy-tryhackme/img-000-dbb30c60.png"
---

A w1se guy 0nce said, the answer is usually as plain as day.

---

### W1seGuy TryHackMe Writeup

A w1se guy 0nce said, the answer is usually as plain as day.

Room: <https://tryhackme.com/room/w1seguy>![image](/assets/images/posts/w1seguy-tryhackme/img-000-dbb30c60.png)![image](/assets/images/posts/w1seguy-tryhackme/img-001-ef8d8858.png)

### Task 1: Source Code

![image](/assets/images/posts/w1seguy-tryhackme/img-002-fde904ba.png)

### 📜 Provided Source Code

```python
import randomimport socketserver import socket, osimport stringflag = open('flag.txt','r').read().strip()def send_message(server, message):    enc = message.encode()    server.send(enc)def setup(server, key):    flag = 'THM{thisisafakeflag}'     xored = ""    for i in range(len(flag)):        xored += chr(ord(flag[i]) ^ ord(key[i % len(key)]))    hex_encoded = xored.encode().hex()    return hex_encodeddef start(server):    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))    key = str(res)    hex_encoded = setup(server, key)    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")    send_message(server, "What is the encryption key? ")    key_answer = server.recv(4096).decode().strip()        try:        if key_answer == key:            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")        else:            send_message(server, "Close but no cigar\n")    except:        send_message(server, "Something went wrong. Please try again. :)\n")    finally:        server.close()class RequestHandler(socketserver.BaseRequestHandler):    def handle(self):        start(self.request)if __name__ == '__main__':    socketserver.ThreadingTCPServer.allow_reuse_address = True    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)    server.serve_forever()
```

### 🧠 Script Explanation

1. A random 5-character key is generated.- A hardcoded fake flag `THM{thisisafakeflag}` is XOR-encrypted with the key.- The hex-encoded result is sent to the client.- The client is prompted to input the key.- If the input matches, the real flag (from `flag.txt`) is revealed

### 🔓 Attack Strategy

Since the plaintext is known, we can use the XOR relationship:

Key = Ciphertext ⊕ Plaintext

We’ll use [CyberChef](https://gchq.github.io/CyberChef/) to reverse-engineer the key.

### Task 2: Get those flags!

![image](/assets/images/posts/w1seguy-tryhackme/img-003-150c8521.png)

### 🧩 Key Recovery Steps

1. Use CyberChef:

- First, **“From Hex,”** the provided XOR-encoded string.- Then, **XOR with the known text** `THM{` to recover the start of the key.- Example: `rb9b`

![image](/assets/images/posts/w1seguy-tryhackme/img-004-a05fd867.jpg)

2. For the final character, XOR the last byte with `}` to get the fifth key character.![image](/assets/images/posts/w1seguy-tryhackme/img-005-d767b424.jpg)

### 🏁 Flag Retrieval

- Once the 5-character key is reconstructed:- Use CyberChef to decrypt the XORed fake flag.

![image](/assets/images/posts/w1seguy-tryhackme/img-006-0667984d.jpg)

- Submit the correct key in the prompt to receive **Flag 2**.

![image](/assets/images/posts/w1seguy-tryhackme/img-007-2aca362f.jpg)

### Thank you for reading my write-up. I hope you found it useful.
