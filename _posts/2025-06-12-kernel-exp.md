---
title: Guide to Kernel Exploitation
date: 2025-06-12
categories: ['guide']
tags: ['binary-exploitation']
author: "Nefeli"
---

Kernel exploitation is one of the many concepts within binary exploitation. Below I’ll introduce the basics of Operating Systems, and then we’ll dive into some methods of exploitation.

> My notes follow the book *A Guide to Kernel Exploitation - Attacking the Core* by Enrico Perla and Massimiliano Oldani. [Here](https://www.amazon.com/Guide-Kernel-Exploitation-Attacking-Core/dp/1597494860) is a link to purchase the textbook.
{: .prompt-info}

# Beginner Concepts 🤍

🎉 Goal: Learn what an operating system is, what kernel exploitation is, and why it’s so dangerous.

An **operating system (OS)** is just another layer of software. It controls all the hardware and gives applications the necessary environment to run on your computer. We refer to the core of an OS as the ***kernel***. The kernel directly interfaces with hardware. It manages I/O operations, schedules processes, etc.

A kernel holds two different types of runnable instructions:

- **Unprivileged instructions** can be run by the user (in user mode).
- **Privileged instructions** can ONLY be run by the OS (in kernel mode).

**Kernel exploitation** is all about elevating our privileges to take us from userland to kernel-land. This is so powerful because if an attacker is able to exploit this they can effectively execute code anyplace on our machine.

## User Privileges

A **userid** is a unique value identifying each user. Specific values are tied to users that have higher privilege (or higher permissions). These users are often called **admins** or **Administrators** in Windows. 

For security, users and user processes cannot use the kernel mode, because… imagine if a flappy bird game could rearrange memory in your system D:

If we can get **super user privileges**, we can bypass all security controls in place for a user or guest of that machine. We can now:

- Disable antiviruses and firewalls
- Read/Write/Execute practically *any* file
- Or install viruses and backdoors on the user’s machine.

An attacker can influence a system by writing both user-land and kernel-land exploits:

| User-land Exploits | Kernel-land Exploits |
| --- | --- |
| Application may crash and need to be restarted | Inconsistent state of machine, need to reboot |
| Attacker has control over environment and library subsystem | Attacker races with other applications that are scheduled |

> Many protections at the kernel level can easily be disabled and may not affect the kernel itself.
{: .prompt-tip}

## Virtual Memory and Concurrency

As I’m writing this I have Spotify, Youtube, even Discord open and running in the background… or so it seems. 

The operating system provides us the **illusion of multiple processes running concurrently,** even though this feat is not possible with just one CPU. So how is this achieved?

The operating system **provides virtualization** to the CPU, allowing many processes to share it by quickly switching between them. This is called **context switching**.

Before the OS switches to another process, it must *save the state of the current one*. Then, it loads the state of the next one. These states are saved and loaded from the **Process Control Block (PCB).** Below is some of the information (the *execution context*) saved in each of these blocks:

- Process ID, state, and priority
- CPU Registers
- Stack (function calls, local variables, return addresses, etc.)
- and more…

The component that decides which task to switch to at what time is called the **scheduler.** It makes it look like all processes are multitasking when, really, one process is running on the CPU at a time!

The scheduler must takes some metrics into account when how to schedule. Here are some important ones to keep in mind: 

- Fairness (every process gets a fair amount of CPU time)
- Throughput (how many processes are completed in a certain amount of time)
- Response Time (how long does it take for a response to happen when a request is made?)
- Turnaround Time (the time it takes from process start to finish, including delays)

However, with so many processes between switched between, there is a risk of **race conditions** occurring. Race conditions are when at least two processes are fighting over the same data block — meaning they access and alter shared data at the same time. This gives us unexpected results in our process and exposes it to a vulnerable state.
