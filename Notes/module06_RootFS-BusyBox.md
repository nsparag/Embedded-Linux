# 🧩 MODULE‑6  
# Root File System & BusyBox  
*User Space in Embedded Linux (BeagleBone Black View)*

---

## 🎯 Module Objective

This module explains **what actually runs after the Linux kernel boots**.

It closes the final gap between *booting the kernel* and *having a usable system*.

By the end of this module, you will clearly understand:

✅ Why the kernel **alone is useless**  
✅ What the **Root File System (RootFS)** really is  
✅ How Linux transitions from **kernel space → user space**  
✅ What `/init` does and why it is **non‑negotiable**  
✅ Why embedded systems prefer **BusyBox**  
✅ How RootFS fits into real embedded products (BBB context)

> 🧠 This module **completes the boot story** started in Module‑2.

---

## 🧠 TOPIC‑1: Kernel Alone Is NOT a Complete System

A critical misconception must be removed early:

> ❌ A booted kernel is **NOT** a running system.

What the kernel can do:
- Initialize CPU and memory
- Control devices
- Enforce protection

What the kernel **cannot** do:
- Provide a shell
- Run applications by itself
- Interact with users

Without RootFS:
- No login
- No commands
- No applications
- No usable system

On BeagleBone Black:

> ⚠️ **Kernel + no RootFS = dead‑end system**

---

## 🧩 TOPIC‑2: What Is the Root File System (RootFS)?

The Root File System is:

✅ A directory hierarchy  
✅ Mounted by the kernel as `/`  
✅ The **foundation of user space**

RootFS provides:
- Executable programs
- Shared libraries
- Configuration files
- Interfaces to interact with the kernel

Mental rule to engrave:

> 🧠 RootFS is not *storage*  
> 🧠 RootFS is the **runtime environment**

---

## 🔁 TOPIC‑3: RootFS in the Boot Flow

```

Boot ROM → SPL → U‑Boot → Kernel → RootFS → init

```

The transition to a real system happens here:

1. Kernel finishes hardware initialisation
2. Kernel mounts the RootFS
3. Kernel executes `/sbin/init`
4. User space begins execution

Failure scenarios:

- RootFS not found → kernel panic  
- RootFS mounted, no `init` → system halts  

These explain **many real embedded boot failures**.

---

## 📂 TOPIC‑4: Standard RootFS Directory Structure

```

/bin     – User commands
/sbin    – System utilities
/lib     – Shared libraries
/etc     – Configuration files
/dev     – Device nodes
/proc    – Kernel runtime information
/sys     – Kernel objects and devices

```

Directories you will constantly work with:

- `/bin`, `/sbin` → Commands and utilities
- `/lib` → Runtime libraries
- `/etc` → System behaviour control
- `/dev` → Interface to drivers
- `/proc`, `/sys` → Kernel views

Important boundary:

> ⚠️ `/proc` and `/sys` are **NOT real disk files**  
> ✅ They are **live kernel‑generated views**

---

## 🔌 TOPIC‑5: `/dev`, `/proc`, `/sys` — Why They Matter

These directories bridge user space and kernel space:

- `/dev`  → Application ↔ driver interface  
- `/proc` → Process and kernel state  
- `/sys`  → Structured device and driver model  

Embedded engineers rely on these heavily.

Preview example:

> 🧠 BeagleBone Black onboard LEDs are controlled via `/sys`

---

## 🚀 TOPIC‑6: What Is `init`?

After mounting RootFS, the kernel executes:

```

/sbin/init

```

Critical facts:

- `init` is the **first user‑space process**
- PID = **1**
- Parent of all processes
- Failure of `init` = unusable system

Responsibilities of `init`:
- Start system services
- Launch login or application
- Control shutdown and reboot

> ❗ PID‑1 is special — if it dies, everything dies.

---

## ⚙️ TOPIC‑7: Init Systems (Conceptual Overview)

Common init systems:

- **sysvinit**
  - Script‑based
  - Simple and predictable

- **systemd**
  - Unit‑based
  - Used by Debian BBB images

- **BusyBox init**
  - Extremely small
  - Preferred in production embedded systems

Key principle:

> 🧠 Init system affects **startup behaviour**,  
> not kernel behaviour.

---

## 🐧 TOPIC‑8: Debian RootFS on BeagleBone Black

BeagleBone Black typically ships with:

- Debian‑based RootFS
- GNU utilities
- systemd init

Why Debian is used:
✅ Developer friendly  
✅ Rich command set  
✅ Familiar PC‑like environment  

But industry reality:

> ⚠️ Debian is **rarely shipped in final products**

---

## 📦 TOPIC‑9: What Is BusyBox?

BusyBox is:

✅ A single executable  
✅ Provides many Linux commands  
✅ Extremely small  
✅ Embedded‑friendly  

It is a **multi‑call binary**:
- One executable
- Behaves as `ls`, `cp`, `mount`, `ifconfig`, etc.

Why BusyBox exists:

> 🧠 Embedded systems have tight flash and RAM budgets

---

## ⚖️ TOPIC‑10: BusyBox vs GNU Utilities

| Aspect | GNU Utilities | BusyBox |
|----|----|----|
| Features | Very rich | Minimal |
| Binary size | Large | Very small |
| Target | Desktop | Embedded |

Embedded mindset shift:

> ✅ “Sufficient functionality”  
> ❌ not “maximum features”

This mindset fundamentally differentiates embedded Linux from desktop Linux.

---

## 📉 TOPIC‑11: RootFS and Embedded Constraints

Embedded systems impose:

- Limited storage
- Limited RAM
- Fast boot requirements
- Controlled software stacks

Every extra file costs:
- Flash space
- RAM
- Boot time

Real embedded RootFS design focuses on:
✅ Removing unused utilities  
✅ Avoiding unnecessary daemons  
✅ Predictable runtime behaviour  

---

## 🧩 TOPIC‑12: RootFS Is Mostly Hardware‑Independent

Important architectural separation:

- Kernel + drivers + device tree → **board‑specific**
- RootFS → **mostly generic**

This allows:
- RootFS reuse
- Faster development
- BSP scalability

Critical constraint:

> ⚠️ Kernel and RootFS **must be compatible**

---

## 🚨 TOPIC‑13: Common RootFS Failures (Real‑World)

Common and predictable issues:

- Kernel panic at boot → wrong root device
- Immediate halt → missing `init`
- Command not found → BusyBox vs GNU mismatch
- App crash → missing libraries

These failures are:
✅ Common  
✅ Diagnosable  
✅ Fixable  

---

## ✅ End‑of‑Module Mental Model

- Kernel boots hardware
- RootFS enables user space
- `init` starts the system
- BusyBox optimises for embedded constraints
- Embedded Linux is **intentionally minimal**

---

✅ You now understand **what makes a Linux system usable after the kernel boots**.  
This completes the foundational Embedded Linux architecture.

---
