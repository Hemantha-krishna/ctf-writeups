---
layout: post
title: "Hacking AI Is the New Pentesting: My Honest PAPA Exam Review"
date: 2026-11-15 12:00:00 -0600
categories: [Certifications, Penetration Testing]
tags: [papa, tcm-security, ai-security, llm, prompt-injection, penetration-testing, certification, review, exam]
description: "My review of the TCM Security PAPA (Practical AI Pentest Associate) certification: the courses, the lab security level mistake I made, exam-day experience, report writing, and tips to pass."
image:
  path: https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/%24%7Bmdname%7D/ca9572abff1ab0932a931228f5e0206e13be597ecfbf47ab9c10282fb67acbcd.png
  alt: TCM Security PAPA Certified Badge
---

I recently passed the **Practical AI Pentest Associate (PAPA)** exam from TCM Security. It is my fifth TCM certification, but my first one focused on hacking AI.

![aa3d4edb00831a46762783f2ddfcb827779f62ab0ddf27711162bf076bed834f](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/%24%7Bmdname%7D/aa3d4edb00831a46762783f2ddfcb827779f62ab0ddf27711162bf076bed834f.png)  


If you have taken a TCM exam before, a lot of this will feel familiar. Still, a few things caught me off guard. Here is what I learned, what I would do differently, and how I would prepare if I were starting again today.

---

## What Is the PAPA Exam?

PAPA is an associate-level, hands-on exam that tests whether you can pentest an **agentic AI application**. You get two full days to attack the target and two more days to write a professional report. To pass, you have to exploit the application and submit a detailed, well-written report.

![ca9572abff1ab0932a931228f5e0206e13be597ecfbf47ab9c10282fb67acbcd](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/%24%7Bmdname%7D/ca9572abff1ab0932a931228f5e0206e13be597ecfbf47ab9c10282fb67acbcd.png)  


Like every TCM exam, it is **not a CTF**. There are zero flags to capture and no multiple-choice questions. It is built to feel like a real client engagement.

---

## Which Courses Actually Matter

The voucher includes two courses: **AI 100: Fundamentals** and **AI Hacking 101**.

![c1cc7f8738a245b962a32da8aed993a6544a608ddac8111e7f32116146ee2e2c](https://raw.githubusercontent.com/Hemantha-krishna/ctf-images/main/posts/%24%7Bmdname%7D/c1cc7f8738a245b962a32da8aed993a6544a608ddac8111e7f32116146ee2e2c.png)  


I skipped AI 100 because I already knew the basics of AI, such as neural networks, transformer architecture and running local LLMs. If you are new to how LLMs work, don't skip it.

**AI Hacking 101** was very good, and it is the course that matters most for the exam. It covers:

- Prompt injection and jailbreaking
- Sensitive information disclosure and data exfiltration (for example, through RAG)
- Improper output handling (harmful content, misinformation)
- System prompt leakage
- Excessive agency and logic or behavioural manipulation
- RAG and vector database attacks (poisoning, retrieval manipulation)
- Reconnaissance and model fingerprinting

> Same as all TCM exams, everything is from the course. Take good notes. I went back to the course during the exam to clarify a few things, and my notes saved me a lot of time.

---

## The Mistake I Wish I'd Avoided

> **Practise the lab at a high security level, not just level 1.**

When you set up the lab's AI at the start of the course, you choose a security level from **1 to 5**. I set it to 1 and forgot about it. At level 1, almost every attack technique works on the first try, so practice feels easy.

Level 5 is a different story. It has more guardrails and actively blocks malicious prompts and inputs. That is what forces you to get creative with your payloads.

My advice: once a technique works at level 1, turn the dial up and try it again at levels 3, 4 and 5. Learning to get around the guardrails is where the real skill is.

---

## The Exam: There Is No Finish Line

The hardest part was not knowing when I was done.

In TCM's Active Directory exams like the [PJPT]({% post_url 2026-02-10-pjpt-exam-strategy-how-to-approach-it-like-a-real-pentester %}) and [PNPT]({% post_url 2026-04-27-pnpt-exam-review-2026 %}), there is a clear end point: you get root or compromise the domain controller, and you stop. PAPA has nothing like that. You are not told how many vulnerabilities there are to find.

So I kept attacking until the very end of the 48 hours. I followed the course methodology exactly and tried every attack it teaches, one after another, against the target.

I also learned more during the exam than during the course. The exam didn't just test what I knew; it completed and added to what the course taught me.

---

## Hardware, Speed and One Annoying Bug

I ran the lab locally on my laptop:

| Component | Spec |
| --- | --- |
| CPU | AMD Ryzen 7 7435HS, 8 cores |
| RAM | 16 GB DDR5 |
| GPU | NVIDIA RTX 4070, 8 GB |

On this setup, the AI chatbot was usable but on the slow side. It wasn't painfully laggy, but expect to wait a little for each reply, and plan your time with that in mind.

There was also a small bug: **the chatbot would not accept my input when I pressed Enter, but it worked when I clicked the Send button.** I only knew this because someone had mentioned it in the TCM Discord.

> If anything seems broken or isn't behaving as intended, head to the Discord first. It can save you a lot of confusion.

---

## Screenshot Everything and Record Your Screen

The report is a big part of the exam, so collect your evidence as you go. I screen-recorded the whole session and took screenshots of everything:

- Every prompt you send and every response you get
- Every successful exploit, step by step
- Anything unusual the application does, even if it seems minor at the time

When it was time to write, I had all the proof I needed and didn't have to recreate anything.

For the report itself, I used the same template I've used for all my TCM certs: the [TCM Security Sample Pentest Report](https://github.com/hmaverickadams/TCM-Security-Sample-Pentest-Report). It gives you a clean, professional structure, so you can focus on the findings instead of the formatting.

---

## Extra Practice (Optional)

You don't need any outside resources to pass. But if you want more AI hacking practice, these sites are worth a look:

- [Hack The Agent](https://hacktheagent.com/)
- [Lakera Agent Breaker](https://play.lakera.ai/agent-breaker)
- [HackAPrompt](https://www.hackaprompt.com/)
- [Prompt Airlines](https://promptairlines.com/)

One warning: I found these harder and much more CTF-style than the exam. If your goal is to pass PAPA, stick to the course and its lab first, and use these only as extra practice.

---

## Quick Tips for Passing PAPA

- Practise the lab at security level 5, not just level 1
- Take detailed notes during AI Hacking 101 - everything on the exam comes from the course
- Follow the course methodology step by step and try every attack
- Don't stop early - there is no "you're done" moment, so use all 48 hours
- Screen-record and screenshot everything for the report
- Use the TCM sample report template to save time on formatting
- Join the TCM Discord - it is the fastest way to solve lab quirks like the Enter key bug
- Running locally? Expect slower chatbot replies and plan your time for it

---

## Is the PAPA Worth It?

**Yes.**

PAPA is a practical, well-built introduction to AI penetration testing. It doesn't hand you a flag or tell you when you're finished, which is exactly why it feels like real work. If you are a pentester moving into AI, a developer building AI apps, or just curious about how these systems break, it is well worth the time.

---

## References

- [PAPA Certification - TCM Security](https://certifications.tcm-sec.com/papa/)
- [TCM Security Sample Pentest Report Template](https://github.com/hmaverickadams/TCM-Security-Sample-Pentest-Report)

---

*Feel free to reach out - always happy to connect with others on the same path.*
