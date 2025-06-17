---
title: Guide to Kernel Exploitation
date: 2025-06-12
categories: ['guide']
tags: ['binary-exploitation']
author: nefeli
---

Kernel exploitation is one of the many concepts within binary exploitation. Below I’ll introduce the basics of Operating Systems, and then we’ll dive into some methods of exploitation.

> My notes follow the book *A Guide to Kernel Exploitation - Attacking the Core* by Enrico Perla and Massimiliano Oldani. [Here](https://www.amazon.com/Guide-Kernel-Exploitation-Attacking-Core/dp/1597494860) is a link to purchase the textbook.
{: .prompt-info}

# Beginner Concepts ⭐

🎉 Goal: Learn what an operating system is, what kernel exploitation is, and why it’s so dangerous.

An **operating system (OS)** is a layer of software that controls all the hardware and gives applications the necessary environment to run on your computer. We refer to the core of an OS as the ***kernel***. The kernel directly interfaces with hardware. It manages I/O operations, schedules processes, etc.

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

## Concurrency and Scheduling

As I’m writing this I have Spotify, Youtube, even Discord open and running in the background… or so it seems. 

The operating system provides us the **illusion of multiple processes running concurrently,** even though this feat is not possible with just one CPU. So how is this achieved?

### Virtualization

The operating system **provides virtualization** to the CPU, allowing many processes to share it by quickly switching between them. This is called **context switching**.

Before the OS switches to another process, it must *save the state of the current one*. Then, it loads the state of the next one. These states are saved and loaded from the **Process Control Block (PCB).** Below is some of the information (the *execution context*) saved in each of these blocks:

- Process ID, state, and priority
- CPU Registers
- Stack (function calls, local variables, return addresses, etc.)
- and more…

![image](https://github.com/user-attachments/assets/c4fcd2f0-e070-4948-bd75-24aef56097ff)

### Scheduling

The component that decides which task to switch to at what time is called the **scheduler.** It makes it look like all processes are multitasking when, really, one process is running on the CPU at a time!

The scheduler must take some metrics into account when deciding how to schedule tasks. Here are some important ones to keep in mind: 

- Fairness (every process gets a fair amount of CPU time)
- Throughput (how many processes are completed in a certain amount of time)
- Response Time (how long does it take for a response to happen when a request is made?)
- Turnaround Time (the time it takes from process start to finish, including delays)

However, with so many processes being switched between, there is a risk of **race conditions** occurring. Race conditions are when at least two processes are fighting over the same data block — meaning they access and alter shared data at the same time. This gives us unexpected results in our process and exposes it to a vulnerable state.

### Real-World Exploit: Dirty COW 🐄

**Dirty COW (CVE-2016-5195)** is a kernel-level exploit which leveraged a race condition to gain write access to the Linux kernel’s memory mappings with copy-on-write, allowing an attacker to raise their privileges and disable security mechanisms. Here’s [a video](https://www.youtube.com/watch?v=kEsshExn7aE) if you want to learn more. 🐄

Okay now we all get it. Processes aren’t *really* running at the same time on one CPU, only one process per single-core CPU. Now that we have a good grasp on how processes share CPU time, let’s face another question…

## How do processes share memory?

**Every machine is constricted by a fixed amount of memory**, whether that’s 4GB (please upgrade if this is you) or 64GB. So how does your poor OS also manage memory between all the processes we need?

Well, the CPU can only access a certain amount of memory. This space is part of the **physical memory** — the actual RAM sticks you put on your motherboard. However, this is *not* how processes will end up using memory.

**Virtual memory** is an abstraction made by the OS to give the illusion to each process that it has its own memory space that is isolated from others. It maps virtual addresses to physical ones with the help of the **memory management unit (MMU).**

> ex. Two processes are using the virtual address of `0xCAFEBABE`, but in reality the OS is mapping that to two different locations — `0x404000` and `0x808000`.

What does virtual memory provide?

- Isolation (what’s happening in one process won’t affect the other, even if it seems like they’re sharing addresses virtually)
- Protection (processes don’t know the actual physical addresses, and it’s hard for an attacker to map those)
- Flexibility (shared libraries can be mapped into processes at the same virtual addresses with the same physical pages too)

### What is mapping again?

The OS also takes this responsibility. It maintains **page tables**, which are data structures that translate virtual addresses into physical addresses. As you can likely guess by the name, it keeps these addresses in a table of **pages**.

Every time a process wants to access an address in memory, the MMU resolves the virtual address into the correct physical one using this page table.

### Additional mitigations

On top of these basic security measures, modern OSes have implemented many more **kernel protections**. These defenses make kernel exploitation even more complex!

- KASLR: Kernel Address Space Layout Randomization, randomizes kernel address space to make them more unpredictable
- NX (No eXecute): Memory on the heap or stack is non-executable, preventing shellcode from being arbitrarily executed
- SMEP/SMAP (Supervisor Mode Execution/Access Prevention) Prevents the execution of user-land code by the kernel. SMAP prevents dereferencing user points in kernel mode, preventing additional attacks.
- PTI (Page Table Isolation):  Doesn’t map kernel memory when a user-land process is running, system calls switch to a separate page table with kernel memory

All of these mitigations provide a layer of protection, but attackers can still take advantage of other methods to access kernel memory. Let’s explore these more in depth!

## Kernel Protections

### KASLR (Kernel Address Space Layout Randomization)

- similar to ASLR in user-land (see my *Binary Exploitation Guide* for more on ASLR)
- randomizes the location of the kernel image base address
- randomizes where kernel components are loaded in memory

This makes it much more difficult for attackers to locate gadgets to jump to.

We can tell if KASLR is enabled on Linux (v3.14 and above) by checking if addresses of symbols differ between `/proc/kallsysms` and `System.map`.

> For further reading, check out [this article](https://dev.to/satorutakeuchi/a-brief-description-of-aslr-and-kaslr-2bbp) 🔖

### How is KASLR then bypassed?

1. Spotting a bug in a process that leaks memory and looking for kernel pointers
    1. These leaks can come from uninitialized memory, side channels (if timing-based), and even IOCTLs in some drivers
    2. Attackers can then calculate the kernel base address by subtracting a known offset
2. Brute-Forcing
    1. Easier within sandboxes, containers, or VMs with no panic mechanisms
    2. Effective when kernel space is small
3. Leaking kernel logs with `dmesg` 
    1. Can leak module addresses and stack traces with pointers
    2. Cannot be done if `dmesg_restrict` is set to 1 or `kptr_restrict` is set to 2
4. Load `/proc/kallsyms` or `System.map` to get `vmlinux` bases and get addresses
    1. More effective in older or misconfigured kernels

## Classifying Bugs

Now that we have a general idea of the difference between user-land and kernel-land, as well as virtual versus physical address spaces, we will delve into how to start exploiting kernels. To do this, we first need to analyze and a gain a great understanding of its underlying code.

The majority of exploits are born through something as simple as a bug in a piece of software. A **bug** is something wrong in a program that crashes it or produces weird behaviour/results that we aren’t seeking.

Now, just because there exists a bug in some software doesn’t necessarily mean that we can use it to compromise a system’s security. This way, bugs can be split into various **classes**, which each class being groups of bugs that share a set of similar characteristics.


> ℹ️ Some common bugs include NULL pointer deferencing, corrupted pointers, nonvalidated pointers, etc.
{: .prompt-info}

### Kernel Stack Vulnerabilities

Now that we’ve touched on some bug basics and know a little about kernel protections, let’s explore a common target in exploits: **the** kernel stack! ✨

The **kernel stack** is spaw every time a process requests some operation from the kernel (ex. the write() system call).

> This request process is also called **trapping** to the kernel
{: .prompt-info}

The kernel stack works just as you would a normal stack to in user-land, with some key differences.

| User Stack | Kernel Stack |
| --- | --- |
| Dynamic | 4KB or 8KB in size |
| Each process has its own | All kernel stacks share kernel address space |

…and if we’re not an attacker, this is not good news for us! 😢

The kernel stack’s limited space is what allows most exploits to slip through. Using **buffer overflows** (covered in my *Binary Exploitation* Guide), we can even overwrite memory that we shouldn’t have access to.

### Real-World Exploit: **CVE-2017-6074** 🧦

**CVE-2017-6074** is a Double Free vulnerability in the DCCP protocol within the Linux Kernel.

A function called `dccp_rcv_state_process` within the input handler file pertaining to the protocol didn’t handle `DCPP_PKT_REQUEST` structures correctly. 

Using a third party application users could make a `setsockopt`system call and escalate priviledges to root or even cause a DoS.

Check it out [here](https://nvd.nist.gov/vuln/detail/CVE-2017-6074)

## Kernel Heap Allocation

If the word *heap* sounds just as intimidating to you as it does to me, then we’re both in the same boat. I’ll try to simplify some of the hardest concepts below.

**Kernel heap** is like a regular heap — it stores dynamic objects within the heap. These objects can be anything from filesystem directory entries to internal kernel structures.

Because, at this point, none of us must be very fond of `malloc()` and `free()`, the kernel has it’s *own allocator*!

### Ok… how does this allocator work?

OSes may differ in implementations of this allocator, but overall the process remains the same:

1. Ask and get a page
2. Divide that page into chunks, aka **slabs** with a fixed size
3. Pages are grouped into **caches**
    1. They’re grouped by object type

The allocator will give the kernel system a pointer to a chunk upon request. It uses metadata or a linkedlist to keep track of freed objects.

But, the kernel still falls victim to overflow vulnerabilities and unsafe function usage, which makes it possible for a user to read the rest of the page or even past the page that the chunk lives in.
