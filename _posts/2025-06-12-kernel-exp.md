Kernel exploitation is one of the many concepts within binary exploitation. Below I’ll introduce the basics of Operating Systems, and then we’ll dive into some methods of exploitation.

- My notes follow the book *A Guide to Kernel Exploitation - Attacking the Core* by Enrico Perla and Massimiliano Oldani. [Here](https://www.amazon.com/Guide-Kernel-Exploitation-Attacking-Core/dp/1597494860) is a link to purchase the textbook.
{: .prompt-info}

# Beginner Concepts 🤍

🎉 Goal: Learn what an operating system is, what kernel exploitation is, and why it’s so dangerous.

An **operating system (OS)** is just another layer of software. It controls all the hardware and gives applications the necessary environment to run on your computer. We refer to the core of an OS as the ***kernel***. The kernel directly interfaces with hardware. It manages I/O operations, schedules processes, etc.

A kernel holds two different types of runnable instructions:

- **Unprivileged instructions** can be run by the user (in user mode).
- **Privileged instructions** can ONLY be run by the OS (in kernel mode).

**Kernel exploitation** is all about elevating our privileges to take us from userland to kernel-land. This is so powerful because if an attacker is able to exploit this they can effectively execute code anyplace on our machine.

## More on User Privileges

A **userid** is a unique value identifying each user. Specific values are tied to users that have higher privilege (or higher permissions). These users are often called **admins** or **Administrators** in Windows. 

For security, users and user processes cannot use the kernel mode, because… imagine if a flappy bird game could rearrange memory in your system D:

If we can get **super user privileges**, we can bypass all security controls in place for a user or guest of that machine. We can now:

- Disable antiviruses and firewalls
- Read/Write/Execute practically *any* file
- Or install viruses and backdoors on the user’s machine.
