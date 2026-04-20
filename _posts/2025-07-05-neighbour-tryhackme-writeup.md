---
layout: post
title: 'Neighbour TryHackMe Writeup'
date: 2025-07-05 10:23:56 +0000
categories: [cybersecurity]
tags:
  - tryhackme
  - writeup
description: "Check out our new cloud service, Authentication Anywhere. Can you find other user’s secrets?"
canonical_url: "https://medium.com/@hemanthakrishnach/neighbour-tryhackme-writeup-36abc0bb8816"
image: "/assets/images/posts/neighbour-tryhackme-writeup/img-000-2ca5521e.png"
---

Check out our new cloud service, Authentication Anywhere. Can you find other user’s secrets?

---

Check out our new cloud service, Authentication Anywhere. Can you find other user’s secrets?

Room: <https://tryhackme.com/room/neighbour>![image](/assets/images/posts/neighbour-tryhackme-writeup/img-000-2ca5521e.png)![image](/assets/images/posts/neighbour-tryhackme-writeup/img-001-6d2a97af.png)

### Scenario

Check out our new cloud service, Authentication Anywhere — log in from anywhere you would like! Users can enter their username and password, for a totally secure login process! You definitely wouldn’t be able to find any secrets that other people have in their profile, right?

### Initial Access

Navigate to the **Target Machine IP** from the **AttackBox**.![image](/assets/images/posts/neighbour-tryhackme-writeup/img-002-fd6002e7.jpg)

On the homepage, view the page source.![image](/assets/images/posts/neighbour-tryhackme-writeup/img-003-ceb945ab.jpg)

There’s an interesting HTML comment:
```vbnet
<! — use guest:guest credentials until registration is fixed. “admin” user account is off limits!!!!! →
```

This suggests we can log in using the credentials:

Username: guest Password: guest

### Exploring the Application

![image](/assets/images/posts/neighbour-tryhackme-writeup/img-004-be714ed6.jpg)

After logging in as the guest user, examine the page source again.![image](/assets/images/posts/neighbour-tryhackme-writeup/img-005-287c3dc1.jpg)

Another comment is present: <! — admin account could be vulnerable, need to update →

This implies the **admin** account might have a security flaw.

### Profile Access via URL Manipulation

The current page URL is:
```bash
http://MACHINE_IP/profile.php?user=guest
```

We can try changing the `user` parameter from `guest` to `admin`:
```bash
http://MACHINE_IP/profile.php?user=admin
```
![image](/assets/images/posts/neighbour-tryhackme-writeup/img-006-750d403f.jpg)

### 🎉 Flag found!

The flag was successfully displayed in the browser.

### Conclusion

This room demonstrates the risks associated with insecure comments in HTML source code and insufficient access controls. By reviewing the page source, we discovered hardcoded credentials and hints about a potentially vulnerable admin account. Simple URL manipulation allowed unauthorized access to the admin profile, indicating a lack of proper authorization checks on server-side resources. This highlights the importance of:

- Avoiding sensitive information in client-side code (e.g., HTML comments),- Implementing robust access controls,- And thoroughly testing user input and session validation mechanisms.

Overall, this room is a great example of how seemingly minor oversights can lead to critical vulnerabilities.

### Thank you for reading my write-up. I hope you found it useful
