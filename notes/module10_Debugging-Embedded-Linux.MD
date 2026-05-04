# MODULE‑10  
# Debugging Embedded Linux on BeagleBone Black  
## *An Embedded Engineer’s Mindset*

---

## Purpose of This Module

This module teaches **how to debug Embedded Linux systems systematically**, focusing on **real failures that occur on BeagleBone Black–class devices**.

By the end of this module, you will be able to:

- Develop a **debugging mindset** (not guesswork)
- Identify **which layer is failing**  
  (bootloader / kernel / rootfs / driver / user app)
- Use **kernel‑provided debug mechanisms** correctly
- Recognize common **embedded Linux failure patterns**
- Debug **kernel modules and drivers safely**
- Prepare students for **field‑level debugging scenarios**

This module ties together **ALL previous modules**.

---

## TOPIC‑1: Debugging in Embedded Linux Is Different

- No GUI debugger
- Limited visibility
- Failures can be fatal
- Debugging must be systematic

Reset expectations clearly:

> Debugging embedded Linux is **NOT** like debugging desktop applications.

Key differences:

- No GUI debuggers
- Often no gdb (or very limited use)
- No display, keyboard, or mouse
- Failures may:
  - Hang the board
  - Require power cycling
  - Leave no obvious traces

Therefore:

> Embedded debugging relies on  
> **observation, reasoning, and elimination**

---

## TOPIC‑2: The First Rule of Embedded Debugging

- Never assume the problem
- Always identify the failing layer
- Debug intentionally, not randomly

Most embedded failures come from **wrong assumptions**.

Bad assumption:

> “The driver is broken.”

Correct reasoning sequence:

1. Did the board power up?
2. Did bootloader execute?
3. Did kernel boot?
4. Did RootFS mount?
5. Did driver load?
6. Did user app interact correctly?

Key rule:

> If you don’t know **where it failed**,  
> you cannot know **how to fix it**.

---

## TOPIC‑3: Embedded Linux Debugging Layers

- Bootloader
- Kernel
- Root File System
- Device Driver
- User Space Application

Every failure belongs to **one primary layer** (sometimes spills into the next).

Example reasoning:

- LED not toggling could be:
  - User app logic error
  - Driver bug
  - Missing DT entry
  - Wrong pinmux

Random fixes waste time.

> Identify the failing layer **before touching code**.

---

## TOPIC‑4: Serial Console – The Lifeline

- Primary debugging interface
- Available from early boot
- Mandatory for embedded work

Serial console is **non‑negotiable**.

Why serial matters:

- Works before Ethernet
- Works before USB
- Works before RootFS
- Shows:
  - Boot ROM output
  - SPL logs
  - U‑Boot messages
  - Kernel boot logs

On BBB:

> No serial console  
> = debugging blindfolded

---

## TOPIC‑5: Boot‑Stage Debugging Using Output

    No output at all       → Power / ROM
    SPL output only        → DDR / MLO
    U‑Boot prompt seen     → Kernel issue
    Kernel panic           → RootFS / DT

This mapping is from **real field experience**:

- Nothing printed  
  → Power or Boot ROM issue

- SPL messages only  
  → DDR init or MLO problem

- U‑Boot prompt reached  
  → Kernel or DT issue

- Kernel panic  
  → RootFS, DT, or init problem

- Login prompt reached  
  → User space / application issue

This lets you localize failure **without changing code**.

---

## TOPIC‑6: `printk` – The Kernel Debug Tool

- Kernel logging mechanism
- Executed from kernel space
- Output stored in ring buffer

In kernel space:

- `printf()` does NOT exist
- `printk()` is the primary debug tool

`printk()`:
- Logs messages from kernel and drivers
- Writes to kernel ring buffer
- Works in:
  - Kernel init code
  - Modules
  - Drivers

Rule to internalize:

> For kernel debugging,  
> `printk()` is your flashlight.

---

## TOPIC‑7: printk Log Levels (Conceptual)

- ERROR
- WARNING
- INFO
- DEBUG

At a conceptual level:

- Log levels control:
  - Importance
  - Visibility
- Critical errors must remain visible
- Debug logs can be filtered

Embedded best practice:

> Do NOT flood the kernel log  
> Log only what aids diagnosis

---

## TOPIC‑8: `dmesg` – Viewing Kernel Messages

- Displays kernel ring buffer
- Includes boot and runtime logs
- Essential debugging tool

`dmesg` reveals:

- Kernel boot messages
- Driver probe output
- Errors and warnings
- Panic information

Typical use cases:

- Driver not loading
- GPIO not behaving
- Module insertion failing

Key truth:

> `dmesg` usually tells the truth  
> engineers often just don’t read it carefully

---

## TOPIC‑9: Debugging Kernel Modules

- Verify module load/unload
- Check dmesg output
- Verify kernel compatibility

Systematic checklist when a module “does nothing”:

1. Did `insmod` succeed?
2. Is module visible in `lsmod`?
3. Did module init function run?
4. Any errors or warnings in `dmesg`?
5. Is cleanup happening on unload?

Skipping steps guarantees confusion.

---

## TOPIC‑10: Debugging Device Drivers


- Validate Device Tree first
- Confirm driver probe
- Check /dev or /sys entries

Correct debugging order:

1. Is the DT node present?
2. Does `compatible` string match?
3. Did probe function execute?
4. Was `/dev` or `/sys` entry created?
5. Does user action reach driver?

Critical rule from industry:

> Most “driver bugs”  
> are actually **Device Tree or pinmux bugs**

---

## TOPIC‑11: `/proc` and `/sys` as Debug Tools

- Runtime kernel visibility
- No code changes required
- Safe inspection interface

Use kernel views before writing code:

- `/proc`
  - Process state
  - Memory info
  - Interrupt counts

- `/sys`
  - Driver state
  - Device attributes
  - Hardware binding

Embedded advantage:

> You can debug without recompiling anything.

---

## TOPIC‑12: Debugging User ↔ Kernel Interaction

- Verify system call success
- Check return values
- Correlate with driver logs

Common user‑space mistakes:

- Wrong permissions on `/dev`
- Incorrect write size
- Wrong ioctl arguments
- Ignoring return values

Debugging discipline:

- Check return codes
- Match user call to driver log
- Correlate with `printk` + `dmesg`

---

## TOPIC‑13: Common Embedded Linux Failure Patterns

- “It boots but nothing works”
- “Driver loads but device doesn’t work”
- “Works once, then hangs”

Recognizable real‑world patterns:

- Boots but no hardware  
  → DT or pinmux issue

- Driver loads, no effect  
  → Wrong GPIO or configuration

- Random hangs  
  → Blocking or race condition

- Works once  
  → Resource leak or missing cleanup

Pattern recognition accelerates debugging dramatically.

---

## TOPIC‑14: Defensive Debugging Mindset

- Minimal code changes
- One change at a time
- Continuous verification

Embedded debugging discipline:

- Change one thing
- Test immediately
- Observe logs
- Revert if needed

Never:

- Change driver + DT + app together
- Guess blindly
- Ignore logs

Discipline beats brilliance here.

---

## TOPIC‑15: When to Reboot (And When Not To)

- Reboot clears symptoms
- Reboot hides root causes
- Use reboot intentionally

Rebooting is sometimes required, often abused.

Good reboot:
- After kernel panic
- After module crash

Bad reboot:
- After minor anomaly
- Without reading logs

Rule to follow:

> Observe → Analyze → Then reboot

---

## TOPIC‑16: Debugging Hierarchy Summary

    Power → Bootloader → Kernel → RootFS → Driver → Application

Always debug **in this order**.

No amount of application debugging will:
- Fix a kernel bug
- Fix a Device Tree error

Layer correctness is non‑negotiable.

---

## MODULE‑10 — Key Takeaways

- Embedded debugging is systematic
- Serial console is mandatory
- printk and dmesg are primary tools
- Device Tree mistakes are common
- Layer‑based reasoning is essential
