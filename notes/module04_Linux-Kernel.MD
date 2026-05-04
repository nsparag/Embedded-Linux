# MODULE‑4
# Linux Kernel Fundamentals  
**(Embedded Engineer’s View – Not Academic)**

---

Purpose: This module answers one question rigorously: **What does the Linux kernel ACTUALLY do in an embedded system like BeagleBone Black?**

By the end of this module, you will be able to:
- Understand the **role of the kernel** in embedded Linux
- Reason about **user space vs kernel space failures**
- Understand **why system calls exist**
- See how **drivers fit inside the kernel**
- Stop treating kernel issues as “magic”

This module **intentionally avoids** deep kernel internals and academic theory.

---

## TOPIC‑1: What Is the Linux Kernel?

- Core of the operating system
- Runs in privileged mode
- Manages hardware and system resources
- Provides services to user space

The Linux kernel is often misunderstood.

The kernel is **not**:
- A collection of drivers only  
- A background service  
- An optional part of Linux  

The kernel **is**:
- Always resident in memory  
- The **first software** after the bootloader  
- The **authority** for CPU, memory, and devices  

On BeagleBone Black:
- Until the kernel runs, **hardware is unusable**
- After the kernel boots, **the system becomes alive**

---

## TOPIC‑2: Kernel vs Operating System (Very Important Clarification)



Kernel:
- Resource manager
- Hardware access
- Always running

Operating System:
- Kernel + user space
- Libraries + tools + apps

Linux kernel alone does **not** make a usable system.

An operating system is:
- Kernel  
- Root filesystem  
- Libraries  
- Utilities  
- Applications  

On BBB:
- Kernel may boot perfectly
- If RootFS is missing, system is **still unusable**

This distinction is critical for understanding:
- Kernel panics
- Boot failures
- “Kernel boots but system doesn’t start” issues

---

## TOPIC‑3: Kernel Responsibilities

- CPU scheduling
- Memory management
- Device drivers
- Filesystems
- System calls

What the kernel manages in practice:

- **Scheduling** - Decides which process or thread runs on the CPU

- **Memory management** - Isolates processes and prevents corruption

- **Drivers** - The **only** path to hardware access

- **Filesystems** - Abstract storage into files and directories

- **System calls** - Controlled gateways into kernel functionality

---

## TOPIC‑4: Kernel Space vs User Space

User Space:
- Applications
- Libraries
- Restricted

Kernel Space:
- Kernel
- Drivers
- Privileged

Execution domains are strictly enforced.

Consequences:
- User space crash → only the application exits
- Kernel space crash → **entire system hangs or reboots**

`Any bug in kernel space is a **system bug**, not an app bug.`

---

## TOPIC‑5: Why the Kernel Is Protected

- Stability
- Security
- Fault isolation

Failure impact comparison:

- Application segmentation fault → restart application
- Kernel segmentation fault → reboot the board

Embedded reality:
- No keyboard
- No display
- No recovery UI

Therefore: > Kernel protection is not a luxury — it is **survival**.

---

## TOPIC‑6: System Calls – The Only Legal Entry

- Controlled entry to kernel
- Requested by user space
- Validated by kernel

Rule to engrave in memory: > User space NEVER calls kernel functions directly.

Actual flow:
- Application calls a library function
- Library issues a system call instruction
- CPU switches to privileged mode
- Kernel validates and executes request

This mechanism is **hardware‑enforced**, not a software choice.

---

## TOPIC‑7: System Call Flow

User App
→ Library
→ System Call
→ Kernel
→ Driver / Kernel Code

System calls define the **only legal path** from user space into the kernel.

Consider a concrete example:

```c
fd = open("/dev/mydevice", O_RDWR);
````

What actually happens under the hood:

- `open()` executes inside **libc** (user space)
- libc issues a **system call instruction**
- CPU switches from user mode to **privileged mode**
- Kernel validates parameters and permissions
- VFS layer resolves the file
- Request is dispatched to the **device driver**
- Driver executes hardware‑related logic inside kernel space

This flow explains why:

- Applications cannot access hardware directly
- All driver interaction happens **through system calls**

---

## TOPIC‑8: Kernel and Processes (Embedded View)

- Kernel creates processes
- Kernel schedules processes
- Kernel enforces isolation

All process lifecycle events are **kernel‑controlled**.

- `fork()` → Kernel creates a new process context

- `exec()` → Kernel replaces process image with a new program

- Context switch → Kernel saves and restores CPU state

Embedded perspective:

- Fewer running processes
- Predictable workloads
- Stability and determinism preferred over throughput

In embedded systems, uncontrolled process behavior is a **design flaw**, not a performance feature.

---

## TOPIC‑9: Kernel and Threads

- Threads scheduled by kernel
- No special "thread engine"
- Kernel sees scheduling entities

From the kernel’s point of view: > A thread is simply a **schedulable entity**.

Important clarifications:

- POSIX threads (pthreads) are **user‑space abstractions**
- The kernel does not understand “threads vs processes” conceptually
- The scheduler operates on **tasks**

This understanding prevents confusion later when dealing with:

- Blocking calls
- Sleep and wake‑up behavior
- Driver I/O paths

---

## TOPIC‑10: Kernel and Memory

- Virtual memory
- Per‑process address space
- Kernel memory protected

Key ideas only:

- Each process has its **own virtual address space**
- Processes cannot see or corrupt each other’s memory
- Kernel memory is shared but **protected from user space**

Critical embedded implication:

> Bad memory access in a **driver**  
> \= immediate kernel crash  
> \= system reboot or hang

This is why memory handling in kernel code requires extreme discipline.

---

## TOPIC‑11: Kernel and Device Drivers (Critical Bridge)

- Drivers live inside kernel
- Drivers manage hardware
- Kernel invokes drivers

Understand the control hierarchy clearly:

```

User Application
|
Kernel
|
Driver
|
Hardware

```

This hierarchy explains **who talks to whom** and **who controls what**.

Key principle to state explicitly:

> Drivers are NOT standalone programs.  
> They are **part of the kernel itself**.

Because of this:

- Drivers must **match the running kernel version**
- Drivers **cannot use libc** or standard user‑space utilities
- A bug in a driver affects the **entire system**
- Debugging drivers is inherently harder than debugging applications

---

## TOPIC‑12: /dev, /proc, /sys — Kernel Interfaces

- `/dev`   → device access
- `/proc`  → process/kernel info
- `/sys`   → kernel objects

These directories are how user space **observes and interacts with the kernel**.

Brief roles (details deferred to later modules):

- `/dev` - Entry point for applications to communicate with **device drivers**

- `/proc` - Window into kernel internals such as:
  - Process information
  - Kernel parameters
  - Runtime status

- `/sys` Structured representation of:
  - Kernel objects
  - Devices
  - Drivers
  - Attributes

Critical preview:

> These are NOT real files on disk.  
> They are **kernel‑generated views**.

Treating them like normal files leads to misunderstandings.

---

## TOPIC‑13: What the Kernel Does NOT Do

- No UI
- No application logic
- No policy decisions

It is equally important to understand **what the kernel intentionally avoids**.

Explicit boundary:

- Kernel provides **mechanisms**
- User space defines **policy**

Example:

- Kernel provides scheduling capability
- Application decides:
  - How many threads to create
  - What workload to run

This separation keeps the kernel:
- Generic
- Stable
- Reusable across products

Violating this boundary leads to poor design and fragile systems.

---

## TOPIC‑14: Common Embedded Kernel Misconceptions

- "Kernel is only for experts" – false
- "Driver bugs are rare" – false
- "If it boots, kernel is fine" – false

Reality:

- A large percentage of embedded issues are **kernel‑space problems**
- Engineers must be able to:
  - Reason about kernel behavior
  - Identify kernel vs user‑space faults
- You do **not** need to modify kernel source
  to **understand** how the kernel works

Kernel knowledge is not an advanced luxury — it is a **baseline requirement** for embedded engineers.
