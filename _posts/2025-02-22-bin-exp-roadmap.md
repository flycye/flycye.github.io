---
title: Binary Exploitation Roadmap
date: 2025-02-22
categories: ['roadmap']
tags: ['binary-exploitation']
author: "Nefeli"
---

An in-depth roadmap for binary exploitation with explanations of common exploitations methods and more links.

# Pre-requisites/requirements

- Basic understanding of programming in C and/or Python
- A Linux environment with `gcc` and `gdb` installed (I recommend [this](https://infosecwriteups.com/pwndbg-gef-peda-one-for-all-and-all-for-one-714d71bf36b8/) link)
- **(Optional):** Familiarity with assembly language (i.e. x86 or x64)

# Beginner Concepts

Start out with exploring these resources that cover an introduction to buffers, buffer overflows, and binary security, as well as some general assembly review.

- **CTF Handbook - Introduction**
    - [https://ctf101.org/binary-exploitation/what-are-buffers/](https://ctf101.org/binary-exploitation/what-are-buffers/)
    - [https://ctf101.org/binary-exploitation/buffer-overflow/](https://ctf101.org/binary-exploitation/buffer-overflow/)
    - [https://ctf101.org/binary-exploitation/what-is-binary-security/](https://ctf101.org/binary-exploitation/what-is-binary-security/)
- [https://pwn.college/](https://pwn.college/)
    - [https://pwn.college/intro-to-cybersecurity/](https://pwn.college/intro-to-cybersecurity/)
- **Nightmare**
    - [https://guyinatuxedo.github.io/00-intro/index.html](https://guyinatuxedo.github.io/00-intro/index.html)
    - [https://guyinatuxedo.github.io/04-bof_variable/csaw18_boi/index.html](https://guyinatuxedo.github.io/04-bof_variable/csaw18_boi/index.html)

After checking these links out and trying a few challenges for yourself, you should have a good all-round understanding of what buffer overflows are and their vulnerabilities. You should be able to complete basic CTF challenges like **`baby-pwn` ’**s.

# Intermediate Concepts

Now we’ll start learning about some traditional binary exploitation methods that are commonly used in CTFs. You don’t have to read about every method, but having a general idea of how to conduct ROP chaining and finding/leaking offsets and addresses as well as gadgets is important.

- **CTF Handbook**
    - [https://ctf101.org/binary-exploitation/return-oriented-programming/](https://ctf101.org/binary-exploitation/return-oriented-programming/)
    - [https://ctf101.org/binary-exploitation/what-is-a-format-string-vulnerability/](https://ctf101.org/binary-exploitation/what-is-a-format-string-vulnerability/)
- **pwn.college**
    - [https://pwn.college/system-security](https://pwn.college/system-security)
    - [https://pwn.college/program-security/](https://pwn.college/program-security/)
- **Nightmare**
    - [https://guyinatuxedo.github.io/rop.html](https://guyinatuxedo.github.io/rop.html)
    - [https://guyinatuxedo.github.io/19-shellcoding_pt1/csaw18_shellpointcode/index.html](https://guyinatuxedo.github.io/19-shellcoding_pt1/csaw18_shellpointcode/index.html)
    
    By the end, you should be able to identify most vulnerabilities. Try out tools like `ROPgadget` and use `pwntools` to simplify your scripts. You should be able to do most CTF challenges.
    

# Advanced Concepts

You will learn how to exploit programs with more binary security measures in place, and start dealing with heap exploitation.

- CTF Handbook
    - [https://ctf101.org/binary-exploitation/what-is-the-heap/](https://ctf101.org/binary-exploitation/what-is-the-heap/)
    - [https://ctf101.org/binary-exploitation/heap-exploitation/](https://ctf101.org/binary-exploitation/heap-exploitation/)
- pwn.college
    - [https://pwn.college/software-exploitation/](https://pwn.college/software-exploitation/)
- Nightmare
    - [https://guyinatuxedo.github.io/25-heap/index.html](https://guyinatuxedo.github.io/25-heap/index.html)
    - [https://guyinatuxedo.github.io/16-srop/index.html](https://guyinatuxedo.github.io/16-srop/index.html)

Try solving more complex CTF challenges and practice writing vulnerable programs. You can also learn about **kernel exploitation** here:

- [https://github.com/xairy/linux-kernel-exploitation](https://github.com/xairy/linux-kernel-exploitation)
