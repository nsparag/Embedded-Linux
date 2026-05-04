# 🧩 MODULE‑5  
# Kernel Modules on BeagleBone Black  
**Safely Entering Kernel Space**

---

## 🎯 Module Objective

This module introduces **kernel modules** — the **industry‑standard, controlled way** to extend and experiment with the Linux kernel **without rebuilding it**.

This is the **first module where we intentionally place our code inside the kernel**.

By the end of this module, you will:

✅ Understand **what kernel modules are and why they exist**  
✅ Know **where modules live and how they are loaded**  
✅ Be able to reason about **module compatibility and failures**  
✅ Clearly distinguish **kernel modules vs user‑space programs**  
✅ Be mentally ready for **hands‑on kernel module labs**

> ⚠️ This module is the **gateway to device drivers**.  
> Everything after this assumes you understand it clearly.

---

## 🧠 TOPIC‑1: Why Kernel Modules Exist

Hard embedded reality:

> ❌ Rebuilding and rebooting the kernel for every change is impractical.

Reasons:

- Kernel rebuilds take time
- Reboots disrupt debugging
- Some drivers are optional
- Hardware support may change across variants

Kernel modules solve this by allowing:

✅ Code to be loaded **at runtime**  
✅ Hardware support added **only when needed**  
✅ Faster development and debugging  

On BeagleBone Black:
- Some drivers are built‑in
- Many peripherals are **modular**
- Custom hardware almost always needs **custom modules**

---

## 🧩 TOPIC‑2: What Is a Kernel Module?

A kernel module is:

✅ Compiled C code  
✅ Dynamically loaded into a running kernel  
✅ Executed in **kernel space**  
✅ Linked against kernel symbols  
✅ Part of the kernel while loaded  

It is **NOT**:

❌ A user‑space program  
❌ Linked with libc  
❌ Started using `./program`  

Very important statement:

> 🔥 A kernel module is **not a program**  
> 🔥 It becomes **part of the kernel itself**

---

## 🔐 TOPIC‑3: Kernel Module vs User Program

| Aspect | User Program | Kernel Module |
|----|----|----|
| Execution Space | User space | Kernel space |
| Privilege | Low | Extremely high |
| Libraries | libc | Kernel APIs |
| Crash Impact | Process exits | System crash |

Embedded implication:

- Kernel crash → reset required
- Often no shell
- Sometimes no logs

> ⚠️ **Kernel programming is powerful — and unforgiving.**  
> We proceed **slowly and deliberately**.

---

## 🧬 TOPIC‑4: Where Kernel Modules Fit Architecturally

```

User Application
↓
System Call
↓
Kernel
↓
Kernel Module
↓
Hardware

```

Execution context:

- Module code always runs in kernel context
- Triggered by:
  - System calls
  - Interrupts
  - Kernel subsystems

Key statement to internalise:

> 🧠 **Most device drivers are kernel modules.**

---

## 🧱 TOPIC‑5: Types of Kernel Modules

Kernel modules can implement:

- Device drivers
- File systems
- Network protocols
- Miscellaneous kernel services

On BBB specifically:
- GPIO drivers
- LED drivers
- I2C / SPI drivers
- Custom peripheral drivers

---

## 🔁 TOPIC‑6: Kernel Module Lifecycle

Every kernel module follows a lifecycle:

1. Module is loaded into kernel memory
2. Initialisation function executes
3. Module performs its role
4. Cleanup function releases resources
5. Module is unloaded safely

> ⚠️ **Cleanup is NOT optional**  
> It protects kernel stability and prevents leaks

---

## 🚪 TOPIC‑7: Module Entry and Exit (Inversion of Control)

Important inversion for new developers:

> ✅ You do NOT call kernel code  
> ✅ The kernel calls YOUR code

- Kernel invokes module init function
- Kernel invokes cleanup on unload
- User never calls module functions directly

This maps directly later to:
- `open()`
- `read()`
- `write()`
hooks in drivers

---

## 🔧 TOPIC‑8: Loading and Unloading Modules (Conceptual View)

Common tools:

- `insmod` — insert module
- `rmmod` — remove module
- `lsmod` — list loaded modules
- `modinfo` — module metadata

What happens internally:

### `insmod`
- Module copied into kernel memory
- Symbol linkage happens
- Init function executes

### `rmmod`
- Exit function executes
- Resources freed
- Module removed

⚠️ Removing a module in use can crash the system.

> 🧠 **Reference counts exist for safety — respect them.**

---

## 🪵 TOPIC‑9: Kernel Logging from Modules

In kernel space:

- ❌ `printf()` does not exist
- ✅ `printk()` is used

Key points:

- Messages go into kernel ring buffer
- Viewed using `dmesg`
- Output does not go directly to terminal

Embedded debugging reality:

> 🔧 `printk()` is often your **only debugging tool**

Use it wisely.

---

## ⚙️ TOPIC‑10: Kernel Module Compatibility

Common reasons modules fail to load:

- Kernel version mismatch
- Header mismatch
- ABI mismatch

Typical error:
```

Invalid module format

```

BBB-specific reality:
- Vendor kernels are customised
- Generic ARM headers often fail

> 🧠 **Module must match the *running* kernel — exactly.**

---

## 📂 TOPIC‑11: Where Kernel Modules Live (BBB Context)

```
/lib/modules/<kernel-version>/
```

Important implications:

- Modules stored per kernel version
- Directory mirrors kernel build
- Kernel upgrade ≠ module compatible

> ⚠️ Updating kernel without rebuilding modules breaks the system

This is an **industry‑critical lesson**.

---

## ⚖️ TOPIC‑12: Built‑In vs Loadable Modules

Embedded design decision:

### Built‑In Drivers
- Required for boot
- Root filesystem access
- Essential peripherals

### Loadable Modules
- Optional hardware
- Dynamically added
- Easier to debug

BBB example:
- MMC → built‑in
- Custom actuator → modular

---

## 🚨 TOPIC‑13: Common Kernel Module Mistakes

Classic beginner errors:

- Forgetting cleanup
- Using libc functions
- Invalid pointer access
- Assuming safety checks exist

Consequences:

❌ No memory protection  
❌ No segmentation safety net  
❌ Errors are system‑fatal  

Kernel programming requires:
✅ Discipline  
✅ Minimalism  
✅ Extreme caution  

---

## 🔗 TOPIC‑14: Relation to Device Drivers

- Most drivers are kernel modules
- Drivers rely on module infrastructure
- Modules are the foundation for:
  - Character drivers
  - GPIO drivers
  - Bus drivers

> 🧠 **Device drivers are specialised kernel modules.**

---

## ✅ End‑of‑Module Mental Summary

- Kernel modules extend the kernel safely
- They run in privileged mode
- Bugs are dangerous but manageable with discipline
- Modules are the foundation of device drivers

---
*   RootFS deep‑dive
*   BusyBox
*   Device drivers
*   GPIO character drivers

Say **“Proceed with Module‑6”** whenever ready.
