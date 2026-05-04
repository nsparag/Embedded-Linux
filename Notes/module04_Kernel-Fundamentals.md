# 🧠 MODULE‑4  
# Linux Kernel Fundamentals  
**Embedded Engineer’s View – Not Academic**

---

## 🎯 Module Objective

This module answers one question rigorously:

> **What does the Linux kernel ACTUALLY do in an embedded system like BeagleBone Black?**

By the end of this module, you will be able to:

✅ Understand the **true role of the kernel**  
✅ Reason about **user‑space vs kernel‑space failures**  
✅ Understand **why system calls exist**  
✅ See where **drivers fit inside the kernel**  
✅ Stop treating kernel behaviour as “magic”

⚠️ This module **intentionally avoids academic kernel internals**.  
The goal is **engineering clarity**, not theory.

---

## 🧠 TOPIC‑1: Why the Kernel Exists (Embedded Reality)

Before the kernel runs:

- Hardware is unusable
- Memory is unmapped
- No processes exist
- No protection exists

After the kernel runs:

✅ CPU is managed  
✅ Memory is controlled  
✅ Devices are accessible  
✅ Applications can exist  

> 🧠 **The kernel is what turns dead silicon into a system.**

On BeagleBone Black:
- Kernel boot = system comes alive
- No kernel = no embedded system

---

## 🧩 TOPIC‑2: What Is the Linux Kernel?

The Linux kernel is:

✅ Core of the OS  
✅ Always resident in memory  
✅ Runs in privileged mode  
✅ Authority over CPU, memory, and devices  

The kernel is **NOT**:
- Just a collection of drivers
- A background service
- Optional software

> 🔑 **Nothing runs without kernel permission.**

---

## 🏗️ TOPIC‑3: Kernel vs Operating System (Critical Distinction)

### Kernel
- Resource manager
- Hardware access
- Always running

### Operating System
- Kernel  
- Root filesystem  
- Libraries  
- Utilities  
- Applications  

Important embedded reality:

> ✅ Kernel may boot perfectly  
> ❌ System is still unusable without RootFS  

This distinction explains:
- Boot failures
- Kernel panic vs userspace issues
- “Kernel boots but nothing happens” problems

---

## ⚙️ TOPIC‑4: Kernel Responsibilities

What the kernel actually manages:

- **CPU Scheduling**
  - Decides which task runs

- **Memory Management**
  - Isolates processes
  - Prevents corruption

- **Device Drivers**
  - The only legal path to hardware

- **Filesystems**
  - Abstract storage into files

- **System Calls**
  - Controlled gateway into kernel

> 🧠 The kernel does not *do everything* — it **controls everything**.

---

## 🔐 TOPIC‑5: User Space vs Kernel Space

### User Space
- Applications
- Libraries
- Restricted privileges

### Kernel Space
- Kernel
- Drivers
- Full hardware control

Crash consequences:

| Failure Location | Impact |
|----|----|
| User Space | Only app dies |
| Kernel Space | Entire system dies |

> ⚠️ **Any kernel bug is a system bug.**

---

## 🛡️ TOPIC‑6: Why the Kernel Is Protected

Why strict separation exists:

✅ Stability  
✅ Security  
✅ Fault isolation  

Embedded reality:
- No keyboard
- No monitor
- No recovery UI

> 🔒 Kernel protection is not a luxury — it is **survival**.

---

## 🚪 TOPIC‑7: System Calls – The Only Legal Gateway

Golden rule to engrave in memory:

> ❌ User space NEVER calls kernel functions directly  
> ✅ System calls are the **only legal entry**

How it works:

1. App calls a library function
2. Library issues a system call instruction
3. CPU switches to privileged mode
4. Kernel validates request
5. Kernel executes or rejects

This is enforced by **hardware**, not convention.

---

## 🔁 TOPIC‑8: System Call Flow (Concrete View)

```

User Application
↓
Library (glibc)
↓
System Call
↓
Kernel
↓
Driver / Kernel Code

````

Example:
```c
fd = open("/dev/mydevice", O_RDWR);
````

What really happens:

*   `open()` runs in user space
*   System call triggers kernel entry
*   Kernel validates permissions
*   VFS resolves file
*   Driver handles hardware interaction

> 🧠 This explains why applications can never touch hardware directly.

***

## 🧬 TOPIC‑9: Kernel, Processes & Threads

Kernel controls:

*   Process creation
*   Scheduling
*   Context switching

Key points:

*   `fork()` → kernel creates new process
*   `exec()` → kernel replaces program image
*   Threads are just **schedulable entities**

Important clarification:

> 🧠 Kernel does NOT think in “process vs thread” terms  
> It schedules **tasks**

This avoids confusion later during:

*   Blocking calls
*   Driver waits
*   Sleep/wakeup behaviour

***

## 🧠 TOPIC‑10: Kernel and Memory (Embedded View)

Key realities:

*   Each process has its **own virtual address space**
*   Processes cannot corrupt each other
*   Kernel memory is protected

Critical implication:

> ❗ Bad memory access in a **driver**  
> ➜ Kernel crash  
> ➜ Board hang or reboot

This is why kernel memory handling requires **extreme discipline**.

***

## 🔌 TOPIC‑11: Kernel and Device Drivers (Critical Bridge)

Drivers:

✅ Live inside kernel  
✅ Access hardware  
✅ Are NOT standalone programs

Control hierarchy:

    User Application
          ↓
        Kernel
          ↓
        Driver
          ↓
       Hardware

Important truths:

*   Driver bugs affect entire system
*   Drivers must match running kernel
*   Drivers do NOT use libc
*   Debugging drivers is inherently hard

> 🧠 Understanding drivers is impossible without understanding the kernel first.

***

## 📂 TOPIC‑12: Kernel Interfaces – /dev, /proc, /sys

How user space interacts with kernel:

*   `/dev`  → Device driver access
*   `/proc` → Process & kernel info
*   `/sys`  → Kernel objects and devices

⚠️ Critical reminder:

> These are **not real disk files**  
> They are **kernel‑generated views**

Treating them like normal files leads to bugs and confusion.

***

## 🚫 TOPIC‑13: What the Kernel Does NOT Do

The kernel intentionally avoids:

❌ UI  
❌ Application logic  
❌ Policy decisions

Design philosophy:

*   Kernel provides **mechanisms**
*   User space defines **policy**

Example:

*   Kernel enables scheduling
*   Application decides how to use it

This separation keeps Linux:
✅ Stable  
✅ Generic  
✅ Reusable

***

## 🧯 TOPIC‑14: Common Embedded Kernel Misconceptions

❌ “Kernel is only for experts”  
❌ “Driver bugs are rare”  
❌ “If it boots, kernel is fine”

Reality:

*   Many embedded bugs are kernel‑space issues
*   Engineers must distinguish:
    *   Kernel faults
    *   User‑space faults

> 🧠 Kernel knowledge is not optional — it is **baseline embedded knowledge**.

***

## ✅ End‑of‑Module Mental Model

*   Kernel controls **everything critical**
*   Userspace is **protected**
*   System calls are **mandatory gateways**
*   Drivers are **kernel code**
*   Kernel bugs are **system‑level failures**

***
Say **“Proceed with Module‑5”** when ready.
```
