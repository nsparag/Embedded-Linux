# MODULE‑2
# BeagleBone Black Boot Flow
**ROM → SPL → U‑Boot → Kernel → RootFS**

---
Purpose: This module explains how the BeagleBone Black comes alive from power‑on.

By the end of this module, you will be able to:
✅ Read BBB boot logs confidently
✅ Understand why multiple boot stages exist
✅ Know where things fail when a board does not boot
✅ Connect boot stages to kernel, device tree, and rootfs

---

## Topic‑1: What Does “Booting” Mean in Embedded Linux?

Booting in embedded systems is not magical and not automatic.
At power‑on:
- CPU has no RAM
- No stack
- No filesystem
- No kernel

Execution begins from a fixed address hardwired in silicon. Everything that follows must be explicitly staged. This is why multiple boot stages exist.

## Topic‑2: Why Is Boot Split into Multiple Stages?

A single program cannot:
- Initialize DDR
- Load kernel
- Parse filesystems

So boot is split into incremental responsibility stages:
- Minimal silicon code (ROM)
- Small RAM initializer (SPL)
- Full bootloader (U‑Boot)
- Kernel
- Root filesystem

This pattern is common across all modern SoCs.

## Topic‑3: High-Level BBB Boot Flow



---

## Topic‑4: Boot ROM (AM335x)

- Code resides **inside the silicon**
- **First code** executed after reset
- **Cannot be modified**
- Responsible for **locating and loading SPL**


The **Boot ROM** is a small piece of code:
- Permanently embedded inside the **AM335x SoC**
- Written and signed by **Texas Instruments**
- Execution starts at the **reset vector** immediately after power‑on or reset

The Boot ROM has **only one critical job**: > **Find and load the next boot image**

**Boot Source Search Order**
The Boot ROM checks boot media in a **predefined sequence**:

1. SD card  
2. eMMC  
3. UART (used for recovery/programming)  
4. USB (optional, board‑dependent)

The first valid boot image found is loaded.

⚠️ If the Boot ROM **fails to find SPL**, the system **cannot boot**. > **If Boot ROM fails, the board is effectively dead.**

---

## Topic‑5: SPL (Secondary Program Loader) — `MLO`

- **First external program** loaded by Boot ROM
- **Very small** in size
- Initializes **DDR RAM**
- Loads **full U‑Boot**


**Why SPL Exists**
The Boot ROM is extremely limited:
- It **cannot initialize DDR**
- DDR configuration is **board‑specific**

Therefore, a second stage is required.


**SPL Responsibilities**
SPL performs essential early hardware setup:

- Pin multiplexing
- Clock initialization
- DDR RAM initialization
- Loading full **U‑Boot** into DDR

Only after DDR is available can larger programs run.


**SPL on BeagleBone Black**
- File name: **`MLO`**
- Must be placed at **specific offsets** on SD/eMMC
- Loaded **directly by Boot ROM**

📌 **Most common BBB boot failure = broken or missing SPL**

If SPL fails, nothing else can execute.

---

## Topic‑6: U‑Boot (Full Bootloader)

- Runs **from DDR RAM**
- Initializes board peripherals
- Loads **kernel, DTB, initramfs**
- Passes **boot parameters** to the kernel

**U‑Boot** is a **mini operating environment** with rich capabilities:
- MMC/SD access
- Ethernet networking
- USB support
- Filesystem parsing (FAT, ext*)
- Environment variables
- Scripting

This is the stage where **user control begins**.

U‑Boot decides **what to boot and how**:
- Select boot source
- Load Linux kernel image
- Load Device Tree Blob (DTB)
- Pass kernel command line arguments

🧠 **Serial console becomes available here**, enabling debugging and control.

---

## Topic‑7: U‑Boot Environment

- Stored boot configuration
- Determines boot source
- Selects kernel and DTB
- Editable via **U‑Boot shell**

The **U‑Boot environment** controls critical boot behavior:

- SD vs eMMC selection
- Kernel image name (`zImage`, `Image`)
- Device tree choice
- Kernel command line (`root=`, `console=`, etc.)

---

## Topic‑8: Linux Kernel Stage

- Loaded into RAM by U‑Boot
- Decompressed and executed
- Initializes kernel subsystems
- Mounts RootFS

Once control is handed over:

- Kernel decompresses in RAM
- ARM kernel entry point is executed
- Internal subsystems initialize:
  - Scheduler
  - Memory management
  - Device drivers

The kernel then:
- Parses the **Device Tree**
- Mounts the **Root File System**

`From this point onward, **Linux controls the hardware**. U‑Boot has no further role.`

---

## Topic‑9: Device Tree Role in Boot

- Hardware description passed to kernel
- No hardcoded board information
- Essential for driver probing

Unlike PCs, embedded systems **cannot auto‑detect hardware**.

Instead:
- U‑Boot passes a **Device Tree Blob (DTB)** to the kernel
- DTB describes:
  - CPU
  - Memory
  - GPIOs
  - Peripherals and buses

If the Device Tree is incorrect:
- Drivers do not load
- Kernel boots, but hardware **does not function**


---

## Topic‑10: Root File System (RootFS)

- Mounted by the kernel
- Provides user space
- Starts the init process

After kernel initialization:

- Root File System is mounted
- `/sbin/init` (or equivalent) is executed
- System services and daemons start

At this stage:
- Boot messages stop scrolling
- Login prompt or application appears
- System is considered **fully booted**

> **Kernel without RootFS = useless**

Both are required for a functional system.

---

## Topic‑11: Complete Boot Responsibility Chain
`Boot ROM → SPL → U‑Boot → Kernel → RootFS`

Each stage loads the next
Each stage depends on the previous

---

📌 Key Takeaways
- Boot is multi-stage
- Each stage has clear responsibility
- BBB boot failures are diagnosable
- Kernel needs DT and RootFS



