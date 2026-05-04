# 🚀 MODULE‑2  
# BeagleBone Black Boot Flow  
**ROM → SPL → U‑Boot → Kernel → RootFS**

---

## 🎯 Module Objective

This module explains **how the BeagleBone Black comes alive** from power‑on.

By the end of this module, you will be able to:

✅ Read BBB boot logs with confidence  
✅ Understand **why booting is multi‑stage**  
✅ Identify **where and why boot fails**  
✅ Connect boot stages to **kernel, device tree, and RootFS**

---

## 🧠 Topic‑1: What Does “Booting” Mean in Embedded Linux?

Booting in embedded systems is **not automatic** and **not magical**.

At power‑on, the processor has:

❌ No RAM initialised  
❌ No stack  
❌ No filesystem  
❌ No kernel  

The CPU simply begins executing **the very first instruction stored in silicon**.

> 🧠 **Everything that runs after power‑on must be explicitly loaded.**

This limitation is the **root reason** for multiple boot stages.

---

## 🧩 Topic‑2: Why Is Boot Split Into Multiple Stages?

A **single program cannot**:

- Configure DDR memory
- Load large binaries
- Parse filesystems
- Support multiple boot sources

Therefore, boot is broken into **incremental responsibility stages**:

| Stage | Purpose |
|----|----|
| ROM | Find next boot image |
| SPL | Bring DDR RAM online |
| U‑Boot | Load kernel & DT |
| Kernel | Initialise OS |
| RootFS | Start userspace |

> 🧠 This staged pattern is used by **all modern SoCs**, not just BBB.

---

## 🧭 Topic‑3: High‑Level BeagleBone Black Boot Flow

```

Power On
↓
Boot ROM (AM335x)
↓
SPL (MLO)
↓
U‑Boot
↓
Linux Kernel
↓
Root File System
↓
User Application / Login

```

Each stage:
- Does **only what it is capable of**
- Hands control to the **next stage**

---

## 🧬 Topic‑4: Boot ROM (AM335x)

### What It Is

- Code embedded **inside silicon**
- First code executed after reset
- Cannot be modified
- Written and signed by **Texas Instruments**

### What It Does

> 🎯 **Single responsibility:**  
> **Locate and load the SPL**

---

### Boot Source Search Order (BBB)

1. SD Card  
2. eMMC  
3. UART (recovery)  
4. USB (board‑dependent)

The **first valid SPL image found** is executed.

⚠️ **If Boot ROM cannot find SPL:**  
➡️ The CPU has **nothing else to execute**

> 🧠 At this point, the board shows **no life signs** (no serial output).

---

## 🪜 Topic‑5: SPL (Secondary Program Loader) — `MLO`

### Why SPL Exists

Boot ROM:
- ❌ Cannot initialise DDR
- ❌ Is hardware‑generic

DDR configuration is **board‑specific**, so a second stage is required.

---

### SPL Responsibilities

SPL performs **early hardware bring‑up**:

✅ Clock configuration  
✅ Pin multiplexing  
✅ DDR RAM initialisation  
✅ Loading full U‑Boot into DDR  

> 🧠 Once DDR is available, **larger programs can run**.

---

### SPL on BeagleBone Black

- Filename: **`MLO`**
- Loaded directly by Boot ROM
- Must be placed at **exact offsets**
- Runs **before any console output**

📌 **Most common BBB boot failure**:  
➡️ Missing or corrupted **SPL**

If SPL fails:
- No U‑Boot
- No kernel
- No logs

---

## 🧰 Topic‑6: U‑Boot (Full Bootloader)

### What Changes at U‑Boot Stage?

✅ DDR is available  
✅ Serial console becomes active  
✅ User interaction begins  

---

### What U‑Boot Does

U‑Boot acts like a **mini operating system**:

- Reads SD / eMMC
- Supports USB & Ethernet
- Understands filesystems
- Executes scripts
- Stores environment variables

---

### U‑Boot Responsibilities

- Select boot source
- Load:
  - Linux kernel
  - Device Tree Blob (DTB)
  - Optional initramfs
- Pass kernel command‑line arguments

> 🧠 **This is the primary debugging stage for engineers.**

---

## ⚙️ Topic‑7: U‑Boot Environment

The **U‑Boot environment** controls boot behaviour:

- SD vs eMMC boot
- Kernel image name
- Device tree selection
- Kernel arguments:
  - `root=`
  - `console=`
  - `rootwait`

Environment variables are:
✅ Editable  
✅ Scriptable  
✅ Persistent  

---

## 🐧 Topic‑8: Linux Kernel Stage

When U‑Boot hands control to the kernel:

- Kernel decompresses in RAM
- ARM kernel entry point executes
- Core subsystems initialise:
  - Scheduler
  - Memory
  - Drivers

The kernel then:
✅ Parses Device Tree  
✅ Mounts RootFS  

> ⚠️ From this point onward, **U‑Boot has no role**.

---

## 🌳 Topic‑9: Role of Device Tree

Embedded systems **cannot auto‑detect hardware**.

Instead:
- Hardware description is passed via **DTB**
- DTB tells kernel:
  - What hardware exists
  - Where it is mapped
  - Which driver should bind

⚠️ **Wrong Device Tree = dead peripherals**

Kernel may boot, but:
- No UART
- No GPIO
- No Ethernet

---

## 📂 Topic‑10: Root File System (RootFS)

Once the kernel finishes initialisation:

- RootFS is mounted
- `/sbin/init` is executed
- Userspace services begin

This is when:
✅ Login prompt appears  
✅ Application starts  
✅ System feels “alive”

> ❗ **Kernel without RootFS = useless system**

---

## 🔗 Topic‑11: Complete Boot Responsibility Chain

```

Boot ROM
↓
SPL
↓
U‑Boot
↓
Kernel
↓
RootFS

```

Each stage:
- Loads the next
- Depends on the previous

---

## ✅ Key Takeaways

- Embedded boot is **multi‑stage by necessity**
- Each stage has **clear, limited responsibility**
- BBB boot failures are **predictable and diagnosable**
- Device Tree and RootFS are **mandatory companions** to the kernel
---
