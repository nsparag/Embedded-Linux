# MODULE‑5
# Kernel Modules on BeagleBone Black
**(Safely Entering Kernel Space)**

---

## Purpose : This module introduces **kernel modules** as the **controlled, industry‑standard way** to extend and experiment with the Linux kernel **without rebuilding it**.

By the end of this module, you will:

- Understand **what kernel modules are and why they exist**  
- Understand **where kernel modules live and how they are loaded**  
- Be able to reason about **module compatibility and failures**  
- Clearly distinguish **kernel modules vs user‑space programs**  
- Be ready for hands‑on kernel module labs later

This module is the **gateway** to device drivers.

---

## TOPIC‑1: Why Kernel Modules Exist

- Kernel cannot be rebuilt frequently
- Hardware drivers may be optional
- Features need to be added dynamically
- Faster development and debugging

**A hard truth: > Rebuilding and rebooting the kernel for every change is impractical.**

Kernel modules solve this by allowing:

- Code to be **loaded into the kernel at runtime**
- Hardware support to be added **only when required**
- Faster development, testing, and debugging

On BBB:

- GPIO, I2C, SPI drivers may be built‑in or modular
- Custom hardware often needs **custom modules**

---

## TOPIC‑2: What Is a Kernel Module?

- Compiled C code
- Piece of code that runs in kernel space
- Loaded dynamically into running kernel
- Extends kernel functionality
- Not a user-space program
- Executed in **kernel space**
- Linked against kernel symbols

Key clarifications:

- It does NOT use `main()`
- It does NOT link with libc
- It CANNOT be executed like `./a.out`

Very important statement to say explicitly:

> A kernel module is **not a program** —  
> it becomes **part of the kernel**.

---

## TOPIC‑3: Kernel Module vs User Program

User Program:
- Runs in user space
- Uses libc
- Crash = process exits

Kernel Module:
- Runs in kernel space
- Uses kernel APIs
- Crash = system crash


The risk boundary:

| Aspect    | User Program | Kernel Module |
| --------- | ------------ | ------------- |
| Privilege | Low          | High          |
| Safety    | High         | Dangerous     |
| Access    | Restricted   | Full hardware |

On embedded systems:

- Kernel crashes often require **power cycle**
- There may be **no recovery shell**

That’s why we move **slowly and deliberately** here.

---

## TOPIC‑4: Where Kernel Modules Fit in the System

````
User Application
→ System Call
→ Kernel
→ Kernel Module
→ Hardware
````

Explain execution context:

- Module code executes in kernel context
- Can be invoked by:
    - System calls
    - Interrupts
    - Kernel subsystems

**Device drivers are implemented **as kernel modules**.**


---

## TOPIC‑5: Types of Kernel Modules

- Device drivers
- Filesystem drivers
- Network drivers
- Miscellaneous services

On BBB:

- GPIO, LED, I2C drivers
- Our custom drivers will be modules

---

## TOPIC‑6: Kernel Module Lifecycle

1.  Module is loaded into kernel memory
2.  Initialization function executes
3.  Module performs its role
4.  Cleanup function releases resources
5.  Module is unloaded

**Cleanup is **not optional** — it protects kernel stability.**

---

## TOPIC‑7: Module Entry and Exit Points

- Kernel calls module functions
- User never calls module functions directly

This inversion is new for many developers:

> In kernel programming, the kernel is the boss.

Later, this maps directly to `open`, `read`, `write` hooks in drivers.

---

## TOPIC‑8: Loading and Unloading Modules (Conceptual)

- insmod – insert module
- rmmod – remove module
- lsmod – list modules
- modinfo – module information


**what happens conceptually**

- `insmod`:
    - Copies module into kernel memory
    - Calls init function

- `rmmod`:
    - Calls exit function
    - Removes module safely

On embedded systems:

- Removing a module in use can crash the system

Hence:

> Module reference counts matter.

---

## TOPIC‑9: Kernel Logging from Modules

- printk() used for logging
- Messages go to kernel ring buffer
- Viewable via dmesg

The replacement for `printf()`:

- Kernel cannot use libc
- Logging uses `printk()`
- Output does NOT go to terminal directly

Instead:

- Stored in kernel ring buffer
- Retrieved using `dmesg`

Important debugging insight: printk is often the **only debugging tool** available.

---

## TOPIC‑10: Module Compatibility

Why modules fail to load:

- Kernel version mismatch
- Header mismatch
- ABI mismatch

Typical error:

    Invalid module format

On BBB:

- TI kernel is often customized
- Generic ARM headers do not work

---

## TOPIC‑11: Where Kernel Modules Live (BBB Context)


````
/lib/modules/<kernel-version>/
````

- Modules are stored per kernel version
- Directory structure matches kernel build

Implication: Updating kernel without rebuilding modules breaks the system.

This is an **industry‑critical lesson**.

---

## TOPIC‑12: Built‑In vs Loadable Modules

Embedded design choice:

- Built‑in drivers:
  - Required for boot (MMC, rootfs)
- Loadable modules:
  - Optional hardware
  - Loaded dynamically
  - Easier to debug

On BBB:

- LED, GPIO may be built‑in
- Custom hardware usually modular

---

## TOPIC‑13: Common Kernel Module Mistakes

- Forgetting cleanup
- Using libc functions
- Dereferencing invalid pointers
- Assuming safety



Stress consequences:

- No memory protection
- No segmentation safety net
- Errors are **fatal**

Kernel programming requires:

- Discipline
- Caution
- Minimal code

---

## TOPIC‑14: Relation to Device Drivers

- Most drivers are kernel modules
- Drivers use module mechanism
- Next step: device drivers

---



## Key Takeaways

- Kernel modules extend the kernel
- They run in privileged mode
- Bugs are dangerous but manageable
- Foundation for device drivers

Say either:

- **“Proceed with Module‑6 – Root File System & BusyBox”**
- **“Give me a one‑slide summary of Modules 1–5”**

We continue **step by step**, no jumps.
