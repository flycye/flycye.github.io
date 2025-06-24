---
title: Beginner's Guide to Kernel Exploitation
date: 2025-06-12
categories: ['guide']
tags: ['binary-exploitation']
author: nefeli
---

## Introduction

Kernel exploitation is one of the many concepts within binary exploitation. Below I’ll introduce the basics of Operating Systems, and then we’ll dive into some methods of exploitation.

> My notes follow the book *A Guide to Kernel Exploitation - Attacking the Core* by Enrico Perla and Massimiliano Oldani. [Here](https://www.amazon.com/Guide-Kernel-Exploitation-Attacking-Core/dp/1597494860) is a link to purchase the textbook.
{: .prompt-info}

🎉 **Goal**: Learn what an operating system is, what kernel exploitation is, and why it’s so dangerous.

## OS & Kernel Basics

An **operating system (OS)** is a layer of software that controls all the hardware and gives applications the necessary environment to run on your computer. We refer to the core of an OS as the ***kernel***. The kernel directly interfaces with hardware. It manages I/O operations, schedules processes, etc.

A kernel holds two different types of runnable instructions:

- **Unprivileged instructions** can be run by the user (in user mode).
- **Privileged instructions** can ONLY be run by the OS (in kernel mode).

**Kernel exploitation** is all about elevating our privileges to take us from userland to kernel-land. This is so powerful because if an attacker is able to exploit this they can effectively execute code anyplace on our machine.

### User Privileges

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

![image](https://github.com/user-attachments/assets/48644867-991b-487e-8c51-8fd949f9ea85)


### Concurrency and Scheduling

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

## All About Virtual Memory

### Sharing Memory -- Virtually?

**Every machine is constricted by a fixed amount of memory**, whether that’s 4GB (please upgrade if this is you) or 64GB. So how does your poor OS also manage memory between all the processes we need?

Well, the CPU can only access a certain amount of memory. This space is part of the **physical memory** — the actual RAM sticks you put on your motherboard. However, this is *not* how processes will end up using memory.

**Virtual memory** is an abstraction made by the OS to give the illusion to each process that it has its own memory space that is isolated from others. It maps virtual addresses to physical ones with the help of the **memory management unit (MMU).**

> ex. Two processes are using the virtual address of `0xCAFEBABE`, but in reality the OS is mapping that to two different locations — `0x404000` and `0x808000`.

![image](https://github.com/user-attachments/assets/d39e55ab-1671-46bb-a253-1751d7355f23)

### Important Metrics
- Isolation (what’s happening in one process won’t affect the other, even if it seems like they’re sharing addresses virtually)
- Protection (processes don’t know the actual physical addresses, and it’s hard for an attacker to map those)
- Flexibility (shared libraries can be mapped into processes at the same virtual addresses with the same physical pages too)

The OS also takes this responsibility. It maintains **page tables**, which are data structures that translate virtual addresses into physical addresses. As you can likely guess by the name, it keeps these addresses in a table of **pages**.

Every time a process wants to access an address in memory, the MMU resolves the virtual address into the correct physical one using this page table.

### Modern Mitigations

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

### Methodology for Bypassing KASLR

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

### SMEP/SMAP (Supervisor Mode Execution/Prevention)

SMEP and SMAP are the two mechanisms that prevent user-generated code from running within the kernel. 

Supervisor Mode Execution Protection (SMEP)
- Disallows kernel from running in ring 0 (_refer to the diagram in the **User privileges** section_)
- Prevents `ret2usr` exploits from working. A **ret2usr** exploit is when the kernel runs pages from user-land, usually by redirecting a datapointer to code in the user-land.

We can check if we have SMEP enabled on our kernel with the `CPUID` instruction with leaf `0x7`, or by checking the flags of the cpuinfo command output (no root required).

```
cat /proc/cpuinfo | grep smep
```

> Hypervisorslike VBox, Hyper-V, and certain versions of VMWare do not offer SMEP support
> Read more on SMEP [here](https://wiki.osdev.org/Supervisor_Memory_Protection)
{: .prompt-info}

#### Bypassing SMEP

One easy way to bypass this kernel mechanism in older kernels is by **overwriting the CR4 register**. This register is one that contains multiple flags that enable/disable processor features, one of those including SMEP. 

> Read more [here](https://breaking-bits.gitbook.io/breaking-bits/exploit-development/linux-kernel-exploit-development/supervisor-mode-execution-protection-smep)
> 
> And/or, check out [this lovely lecture](https://repository.root-me.org/Exploitation%20-%20Syst%C3%A8me/Unix/EN%20-%20Practical%20SMEP%20bypass%20techniques%20on%20Linux%20-%20Vitaly%20Nikolenko.pdf)
{: .prompt-info}

### NX (no eXecute)

The NX mechanism marks certain parts of program memory as non-executable. This way, attackers cannot arbitrarily execute shellcode on the program heap or stack. 
- Think, even if we have an exploit crafted with an otherwise working buffer overflow, we won't be able to execute it

This mechanism is set into place by hardware like the Intel XD and AMD NX bit, and **applies to executing code in both user-land and kernel-land**.

#### Bypassing NX

Possibly one of the easiest mitigations to bypass, we can sneak by creating ROP chains. **ROP chains** are form of [Return Oriented Programming](https://ctf101.org/binary-exploitation/return-oriented-programming/) where various _gadgets_ are combined together.

These gadgets are functions that already exist within the kernel or various libraries. This way, an attacker can just **use existing code instead of injecting new code** to attack and execute requests in kernel-land.

A very common form of a gadget is a `ret` gadget. Often, it will look like this:

```
pop rdi; ret
```

When using this gadget, we have control over the `rdi` argument -- which we will set to the function we want to jump to in that case. Once `ret` is executed, the instruction pointer register (`rip`) will be populated with that information, thus **jumping to our specified location!** Here is a diagram to show the mapping between parts of the program stack and their matching ROP gadgets:

![image](https://github.com/user-attachments/assets/45475da1-ab59-4f1d-a1d0-43b9c60d0951)

> If you want more resources to understand ROP gadgets, check out my _Guide to Binary Exploitation_ or [this guide here](https://ir0nstone.gitbook.io/notes/binexp/stack/return-oriented-programming/gadgets)

### PTI (Page Table Isolation)

Page Table Isolation separates page tables from kernel-land and user-land. Before the [Meltdown vulnerability](https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability), pagetables would reside in the same memory no matter their privilege access.
- Doesn't map kernel memory into address space when user code is running, only during requests
- Might increase the overhead of syscalls (more context-switching needed)
- Makes it far harder to bypass PTI

#### Bypassing PTI

Using corrupted or broken syscall implementations within an outdated device driver is the easiest way to bypass PTI. Check out more [here](https://github.com/pr0cf5/kernel-exploit-practice/blob/master/bypass-smap/README.md).

## Classifying Bugs

Now that we have a general idea of the difference between user-land and kernel-land, as well as virtual versus physical address spaces, we will delve into how to start exploiting kernels. To do this, we first need to analyze and a gain a great understanding of its underlying code.

The majority of exploits are born through something as simple as a bug in a piece of software. A **bug** is something wrong in a program that crashes it or produces weird behaviour/results that we aren’t seeking.

Now, just because there exists a bug in some software doesn’t necessarily mean that we can use it to compromise a system’s security. This way, bugs can be split into various **classes**, which each class being groups of bugs that share a set of similar characteristics.

ex.
### Integer Overflows

This type of overflow occurs when a value that falls outside of the range is stored into an integer variable (can be signed or unsigned). This leads to _undefined behaviour_ in C++.

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

#### Real-World Exploit: **CVE-2017-6074** 🧦

**CVE-2017-6074** is a Double Free vulnerability in the DCCP protocol within the Linux Kernel.

A function called `dccp_rcv_state_process` within the input handler file pertaining to the protocol didn’t handle `DCPP_PKT_REQUEST` structures correctly. 

Using a third party application users could make a `setsockopt`system call and escalate priviledges to root or even cause a DoS.

Check it out [here](https://nvd.nist.gov/vuln/detail/CVE-2017-6074)

### Kernel Heap Allocation

If the word *heap* sounds just as intimidating to you as it does to me, then we’re both in the same boat. I’ll try to simplify some of the hardest concepts below.

**Kernel heap** is like a regular heap — it stores dynamic objects within the heap. These objects can be anything from filesystem directory entries to internal kernel structures.

Because, at this point, none of us must be very fond of `malloc()` and `free()`, the kernel has it’s *own allocator*!

#### A Special Allocator

OSes may differ in implementations of this allocator, but overall the process remains the same:

1. Ask and get a page
2. Divide that page into chunks, aka **slabs** with a fixed size
3. Pages are grouped into **caches**
    1. They’re grouped by object type

The allocator will give the kernel system a pointer to a chunk upon request. It uses metadata or a linkedlist to keep track of freed objects.

But, the kernel still falls victim to overflow vulnerabilities and unsafe function usage, which makes it possible for a user to read the rest of the page or even past the page that the chunk lives in.

## Race Conditions

### I tried telling a race condition joke...
but you might have heard the punchline first

One last topic to touch up on before we finally have the basics of kernel exploitation in our toolkit is race conditions. A **race condition** happens when two or more threads try to access a shared resource concurrently, altering it in different orders and producing unexpected results.

ex.
![image](https://github.com/user-attachments/assets/589b88b6-97a8-4fb2-baf8-b813667ba87d)

Many solutions have been presented for this solution -- locks, semaphores, etc. However, these have downsides such as performance overhead, complex implementation, and falling into situations like deadlock. [More on deadlocks](https://web.mit.edu/java_v1.0.2/www/tutorial/java/threads/deadlock.html)

> Race conditions are only one type of logic bug. **Logic bugs** are bugs that do not cause a program to crash, but instead produce unexpected behaviour or results.
> Read about [filesystem logic bugs](https://y3a.github.io/2023/07/06/filesystem-logic-bugs/), [reference counter overflows](https://ruffell.nz/programming/writeups/2020/09/02/debugging-a-zero-page-reference-counter-overflow-on-4-15-kernel.html), and even [automating vulnerable environment generation for linux kernel vulnerabilities](https://arxiv.org/html/2404.11107v3).
{: .prompt-info}

## Closing

Although these are just my notes on the textbook, I hope you now have a better understanding of kernel exploitation and what it entails. Next, consider practicing or watching some case studies. [Here](https://github.com/xairy/linux-kernel-exploitation) is a good resource for finding links to futher your studying.

