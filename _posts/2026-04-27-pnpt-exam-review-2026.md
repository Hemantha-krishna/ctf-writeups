---
title: "PNPT Exam Review 2026: Tips, Mistakes, and What Actually Matters"
date: 2026-04-27 00:00:00 +0000
categories: [Certifications, Penetration Testing]
tags: [pnpt, tcm-security, penetration-testing, certification, review, exam]
image:
  path: /assets/img/posts/pnpt-badge.png
  alt: TCM Security PNPT Certified Badge
---

![PNPT Certified Badge](/assets/images/posts/pnpt-badge.png){: width="200" .left}

After grinding through the courses, learning from a failed first attempt at the PJPT, and spending several intense days inside the exam environment, I can finally say I'm a certified **Practical Network Penetration Tester (PNPT)**. Here's my honest breakdown of the experience — the mistakes, the wins, and every tip that would have saved me time.

---

## The Mistake That Cost Me Two Hours

> **Turn off your personal VPN before connecting to the exam environment.**

I had mine running on Windows the entire time. I connected to the exam VPN from Kali Linux — and then couldn't ping a single machine. I restarted multiple times, went through various troubleshooting steps, and wasted two full hours before I finally spotted the culprit.

Two hours in a five-day exam is significant. Don't repeat this mistake.

---

## Set Up Your Kali Machine Before Exam Day

Install every tool you think you'll need and then **take a snapshot of your Kali VM**. The last thing you want during the exam is to be chasing down a broken tool or fighting a dependency issue.

- Install all required enumeration, exploitation, and pivoting tools in advance
- Take a clean snapshot of your Kali VM once everything is working
- Test that your exam VPN config connects cleanly with no personal VPN running
- Have OBS Studio set up and ready to record from the moment you start

---

## Which Courses Actually Matter

The TCM Security course library is excellent, but not everything is equally relevant for the PNPT specifically.

**Essential for the PNPT:**
- Practical Ethical Hacking (PEH)
- OSINT Fundamentals
- External Pentest Playbook

**Useful but not exam-critical:**
- Linux Privilege Escalation
- Windows Privilege Escalation

These priv esc courses are excellent and will sharpen your CTF skills, but they aren't the focus of the PNPT exam itself.

The **External Pentest Playbook** in particular is pure gold. I made the mistake of watching it passively the first time. During the exam, I rewatched relevant sections multiple times — and it consistently pointed me in the right direction. Revisiting the material mid-exam might feel like you're losing precious time, but you're not. It's the opposite.

> All the required concepts are in the courses. There is nothing you need to know outside of them. Every time I was stuck, the answer was in the material.

---

## Pivoting Is Non-Negotiable

Make sure you learn how to pivot before you sit the exam. The **two pivoting videos in the PEH course** are essential viewing. I'd also recommend spending time on the **Wreath network on TryHackMe** (at least 50% completion) to practice these skills in a realistic environment.

The internal segment of the exam is similar to the PJPT but a step up in difficulty, and your ability to move laterally through the network will directly determine how far you get.

---

## The External Phase Takes Time

I spent the better part of the first two days on the external portion alone. Don't rush it — thorough enumeration here sets the foundation for everything that follows. If you hit a wall, resist the urge to dive into rabbit holes.

**Mindset shift that consistently helped me:**

> "If I have X, what can I do with X? What can I access with X? Have I seen something similar in the course using X?"

This simple reframe consistently unlocked the next step whenever I was stuck.

Always remember: **enumerate, enumerate, enumerate.**

---

## Screenshot Everything — And Record Your Screen

Take a screenshot at every single step, even the ones that feel minor or obvious in the moment. During the PJPT, I missed one screenshot that ended up not mattering — but it was a stressful realization during report writing.

For the PNPT, I recorded the entire exam session using **OBS Studio** so I could grab any frame I'd missed. I didn't end up needing the footage, but having that safety net removed a lot of anxiety.

- Screenshot every step — assume nothing is too minor
- Record the full session with OBS Studio as a backup
- Keep all credentials in a dedicated `.txt` file, updated in real time
- Write down every command and its output as you go — don't rely on terminal history

---

## On Methodology and Finding Multiple Attack Paths

I compromised the domain before the five-day limit, and — just as Heath Adams recommends in the course — I went back and ran the penetration test again from scratch. Doing so revealed a **completely different attack path** I hadn't found the first time.

This matters both for completeness and for writing a more thorough report. The exam is fundamentally a test of your methodology, not just your technical knowledge. My approach to penetration testing measurably improved by going through this process.

---

## Notes, Report, and the Debrief

I finished my report in about two days, though it felt like I barely managed it. Use the [TCM Security Sample Pentest Report](https://github.com/hmaverickadams/TCM-Security-Sample-Pentest-Report) as a reference — it's well structured and covers everything the exam expects.

About a week after submitting the report, I received a scheduling link for the debrief. The debrief is a **15-minute walkthrough** of your attack chain and remediation recommendations. You can use a simple slide deck or walk through your report directly — either format works.

Be ready to explain not just *what* you did, but *why*, and what the client should do to fix it.

---

## General Tips Worth Repeating

- Take breaks — a rested brain solves problems a tired one can't see
- Don't anchor too long on any single issue — note it, set it aside, come back later
- Don't spend too much time on one thing; move on and revisit if you find nothing else
- Take detailed notes during the course with screenshots and full commands — you'll reference them during the exam
- Having a second attempt available genuinely reduces exam stress — use that mental relief to perform better on your first try
- Read other PNPT blogs before you sit the exam — the community's collective experience is valuable preparation

---

## Is the PNPT Worth It?

**Absolutely.**

This is one of the few certifications that genuinely makes you feel like you're conducting a real engagement from start to finish. You're not memorizing answers — you're hacking a real simulated company, writing a professional report, and defending it in a debrief.

The exam tests how solid your methodology is. Mine actually improved after taking it.

Thank you to Heath Adams and the entire TCM Security team for building something that actually matters.

---

## References

- [PNPT Certification — TCM Security](https://certifications.tcm-sec.com/pnpt/)
- [TCM Security Sample Pentest Report Template](https://github.com/hmaverickadams/TCM-Security-Sample-Pentest-Report)

---

*Feel free to reach out — always happy to connect with others on the same path.*

