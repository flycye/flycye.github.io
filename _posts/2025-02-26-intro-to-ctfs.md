---
title: Intro to CTFs Guide
date: 2025-02-26
categories: ['guide']
tags: ['ctf', 'beginner']
author: nefeli
---

## Finding CTFs

One of my favorite websites for viewing and signing up for upcoming CTFs is

[https://ctftime.org/](https://ctftime.org/)

It shows you:

- event difficulty
- format and categories included
- duration (from a couple of hours to a couple of days)
- rank weighting

Where the higher the difficulty and weighting, the more competitive the CTF is

You don't have to make an account on there (I will update this post if we decide to do that), but you can easily view upcoming CTFs here:

[https://ctftime.org/event/list/upcoming](https://ctftime.org/event/list/upcoming)/]

## Basic Tools for CTFs

As per usual I highly recommend using **Kali Linux** as it comes preinstalled with many tools you may need

Because these tools can be overwhelming, I would really encourage signing up for **picoCTF**, they put all their past CTF challenges on there and you can easily sort by category and track your progress (the link will be posted at the bottom).

### **Pwn / Binary Exploitation**

- pwntools - a python library you should be using in your scripts
- ROPgadget - finds ROP gadgets for you to build chains
- checksec - checks protections on binaries
- gdb + gef/peda/pwndbg - debuggers for ease of exploitation

### **Rev**

- ghidra - basic reverse engineering tool & disassembler by the NSA
- radare2 - confusing but very useful alternative
- binaryninja - more user friendly than Ghidra, has an in-built debugger

> most of the tools for Pwn / Binary Exploitation also apply to Rev, they overlap in many areas
> 

## **Web**

- Burp Suite (web security testing tool)
- SQLmap (automated SQL injection tool)
- ffuf (web fuzzing, discovers hidden files and parameters)
- Gobuster (directory brute-forcing)
- cURL (fetch webpages, interact with APIs, view responses, etc.)

### **Crypto**

- Cyberchef (swiss army knife for encoding/decoding, if something looks like its encoded just drop it in there)
- Hashcat (password cracking tool)
- John the Ripper (password hash cracking)
- xortool (cracking XOR ciphers)
- sage (for more math-heavy crypto challenges)

### **Forensics**

- binwalk (extract hidden files from other files)
- foremost (file carving)
- exiftool (file metadata)
- stegsolve (steganography analysis, meaning hidden messages)
- volatility (memory forensics)
- wireshark (packet capture and network traffic analysis)

> kali comes with most of these tools, including others like
> 
> 
> ```
> pdfid
> 
> ```
> 

### **Misc (useful for any challenge)**

- strings (extract any readable text from a file/binary)
- xxd / hexdump (hex editor)
- tcpdump (can use instead of wireshark)

> man <any_tool
> 
> 
> ```
> --help
> 
> ```
> 

### **Important Links**

Extremely useful repo for most methods and tools you could possibly need in a CTF, also **check out his YouTube channel**, especially his solves for past picoCTF challenges:

- [https://github.com/JohnHammond/ctf-katana/](https://github.com/JohnHammond/ctf-katana/)
- [https://www.youtube.com/@_JohnHammond/playlists/](https://www.youtube.com/@_JohnHammond/playlists/)
- [https://play.picoctf.org/](https://play.picoctf.org/)
