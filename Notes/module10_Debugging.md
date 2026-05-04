# 🧩 MODULE‑10  
# Debugging Embedded Linux on BeagleBone Black  
*An Embedded Engineer’s Mindset*

---

## 🎯 Module Objective

This module teaches **how to debug Embedded Linux systems systematically**,  
focusing on **real failures that occur on BeagleBone Black–class devices**.

This is **not about tools alone** — it is about *thinking correctly*.

By the end of this module, you will be able to:

✅ Develop a **debugging mindset** (not guesswork)  
✅ Identify **which layer is failing**  
   *(bootloader / kernel / rootfs / driver / application)*  
✅ Use **kernel‑provided debugging mechanisms** effectively  
✅ Recognise **common embedded Linux failure patterns**  
✅ Debug **kernel modules and drivers safely**  
✅ Handle **field‑level and product‑level issues**

> 🧠 This module ties together **everything learned so far**.

---

## 🧠 TOPIC‑1: Embedded Linux Debugging Is Different

Reset expectations clearly:

> ❌ Embedded debugging is **NOT** like desktop debugging.

Key differences:

- No GUI debugger
- Often no `gdb` (or extremely limited)
- No keyboard, mouse, or monitor
- Failures can:
  - Hang the board
  - Require power cycling
  - Leave no obvious trace

Embedded debugging relies on:

> ✅ Observation  
> ✅ Reasoning  
> ✅ Elimination  

Not guessing.

---

## 🧩 TOPIC‑2: The First Rule of Embedded Debugging

Golden rule:

> 🔑 **Never assume the cause. Identify the failing layer first.**

Most failures persist because engineers jump to conclusions.

Bad assumption:
> “My driver is broken.”

Correct reasoning sequence:
1. Did the board power up?
2. Did the bootloader execute?
3. Did the kernel boot?
4. Did RootFS mount?
5. Did the driver load?
6. Did the application behave correctly?

> ⚠️ If you don’t know **where it failed**,  
> you cannot know **how to fix it**.

---

## 🧱 TOPIC‑3: Embedded Linux Debugging Layers

Every failure belongs to a **primary layer**:

- Power / Hardware
- Bootloader
- Kernel
- Root File System
- Device Driver
- User‑Space Application

Example:
> LED not toggling does NOT mean “GPIO driver bug”.

It could be:
- Wrong pinmux
- Missing DT entry
- Driver not probing
- Application writing wrong data

> 🧠 Debug the **layer**, not the symptom.

---

## 🔌 TOPIC‑4: Serial Console — The Lifeline

Non‑negotiable rule:

> ❌ No serial console  
> ✅ No effective embedded debugging

Why serial console matters:

- Works before Ethernet
- Works before USB
- Works before RootFS
- Shows:
  - SPL output
  - U‑Boot messages
  - Kernel boot logs
  - Panic messages

On BeagleBone Black:

> 🔥 Serial console is your **single most important debug tool**.

---

## 🔍 TOPIC‑5: Boot‑Stage Debugging via Symptoms

Real‑world failure mapping:

```

No output at all       → Power / ROM
SPL output only       → DDR / MLO
U‑Boot prompt seen    → Kernel / DT issue
Kernel panic          → RootFS / DT / init
Login prompt seen     → User‑space problem

```

This lets you localise failures **without touching code**.

---

## 🪵 TOPIC‑6: `printk` — The Kernel Flashlight

In kernel space:

- ❌ `printf()` does not exist
- ✅ `printk()` is the primary debug tool

`printk()`:
- Logs messages from kernel and drivers
- Writes to kernel ring buffer
- Works in:
  - Kernel init code
  - Modules
  - Device drivers

> 🧠 For kernel debugging,  
> `printk()` is your **flashlight in the dark**.

---

## 🧩 TOPIC‑7: printk Log Levels (Conceptual)

Conceptual levels:
- ERROR
- WARNING
- INFO
- DEBUG

Best practice:
- Log critical failures clearly
- Avoid flooding logs
- Debug logs must help **diagnosis**, not noise

---

## 📜 TOPIC‑8: `dmesg` — The Truth Source

`dmesg` displays:
- Kernel boot logs
- Driver probe output
- Errors and warnings
- Panic traces

When something “does not work”:

> 🔑 **Read `dmesg` first. Carefully.**

Most of the time:
> The kernel already told you what’s wrong.

---

## 🧱 TOPIC‑9: Debugging Kernel Modules

When a module “does nothing”, check:

1. Did `insmod` succeed?
2. Is the module visible in `lsmod`?
3. Did the init function run?
4. Any error in `dmesg`?
5. Does cleanup run on unload?

Skipping steps guarantees confusion.

---

## 🔌 TOPIC‑10: Debugging Device Drivers (Correct Order)

Correct diagnostic order:

1. Is the DT node present?
2. Does `compatible` match?
3. Did the probe function execute?
4. Was `/dev` or `/sys` entry created?
5. Does user‑space call reach driver?

Industry truth:

> ⚠️ Most “driver bugs”  
> are actually **Device Tree / pinmux bugs**.

---

## 🪟 TOPIC‑11: `/proc` and `/sys` as Debug Tools

These are **zero‑risk debugging tools**:

- `/proc`
  - Process state
  - Memory usage
  - Interrupt counts

- `/sys`
  - Device status
  - Driver binding
  - Hardware parameters

> ✅ Observe first  
> ❌ Modify later

---

## 🔄 TOPIC‑12: Debugging User ↔ Kernel Interaction

Common mistakes in user space:

- Ignoring return values
- Wrong write sizes
- Permission issues
- Wrong `ioctl` arguments

Proper discipline:
- Check return codes
- Correlate with `printk`
- Cross‑verify with `dmesg`

---

## 🚨 TOPIC‑13: Common Embedded Linux Failure Patterns

Recognisable real‑world patterns:

- Boot OK, hardware dead  
  → DT or pinmux issue

- Driver loads, no effect  
  → Wrong configuration or GPIO

- Works once, then hangs  
  → Resource leak or missing cleanup

- Random hangs  
  → Blocking or race condition

Pattern recognition cuts debug time by **orders of magnitude**.

---

## 🛡️ TOPIC‑14: Defensive Debugging Discipline

Embedded debugging discipline:

✅ Make one change at a time  
✅ Test immediately  
✅ Read logs carefully  
✅ Revert if needed  

Never:
❌ Change driver + DT + app together  
❌ Guess blindly  
❌ Ignore warning logs  

> 🧠 Discipline beats brilliance in embedded debugging.

---

## 🔄 TOPIC‑15: When to Reboot (And When Not To)

Truth:

- Reboot clears symptoms
- Reboot often hides root causes

Good reboot:
- After kernel panic
- After module crash

Bad reboot:
- Immediately after something “feels wrong”
- Without checking logs

Rule:

> ✅ Observe → Analyse → Then reboot

---

## 🧱 TOPIC‑16: Debugging Hierarchy (Non‑Negotiable)

```

Power
→ Bootloader
→ Kernel
→ RootFS
→ Driver
→ Application

```

> ❌ Application debugging cannot fix kernel bugs  
> ❌ Driver fixes cannot fix DT errors  

Always debug **top‑down by layer**.

---

## ✅ End‑of‑Module Mental Model

- Embedded debugging is **systematic**
- Serial console is essential
- `printk` + `dmesg` are your primary tools
- Device Tree mistakes are common
- Layer‑based reasoning saves time and boards

---

## 🎓 Course‑Level Closure

✅ You now know how Embedded Linux boots  
✅ You understand kernel, drivers, DT, RootFS  
✅ You can write and debug character drivers  
✅ You can reason about real field failures  

> 🧠 You are no longer *learning* Embedded Linux —  
> you are **thinking like an embedded Linux engineer**.
---
