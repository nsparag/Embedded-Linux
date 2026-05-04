# Embedded-Linux


Builds **directly on Linux & POSIX fundamentals** and transitions into **system‑level embedded Linux engineering**, using the **BeagleBone Black (BBB)** as the reference target.

The material is written from an **industry practitioner’s perspective**, focusing on:
- how embedded Linux **actually behaves on hardware**
- how engineers **build, extend, and debug** real systems
- not academic kernel internals or theory‑only explanations

---

## Target Audience

This is designed for:
- Strong background in:
  - C programming
  - Linux basics
  - POSIX APIs (processes, threads, IPC)
- Little or **no prior kernel / driver experience**

No prior Yocto or BSP expertise is assumed.

---

## Hardware & Platform

- **Target board:** BeagleBone Black  
- **SoC:** TI AM335x (ARM Cortex‑A8)  
- **Architecture:** ARMv7 (32‑bit)  
- **Console:** UART (serial) – **mandatory**  
- **OS images:** Debian / minimal / vendor images (labs are image‑tolerant unless explicitly stated)

---

## Repository Structure

```text
.
├── README.md                <-- (this file)
│
├── Notes/                 <-- Phase‑2 THEORY (Modules 1–10)
│   ├── Module-01-Architecture/
│   ├── Module-02-Boot-Flow/
│   ├── Module-03-Cross-Compilation/
│   ├── Module-04-Kernel-Fundamentals/
│   ├── Module-05-Kernel-Modules/
│   ├── Module-06-RootFS-BusyBox/
│   ├── Module-07-Device-Tree/
│   ├── Module-08-User-Kernel-Interaction/
│   ├── Module-09-Character-Driver/
│   └── Module-10-Debugging/
│
└── Labs/                    <-- Phase‑2 HANDS‑ON (Lab‑1 to Lab‑8)
    ├── README.md            <-- Master Lab Index & Navigation
    ├── Lab-01-Env-Setup.md
    ├── Lab-02-Cross-Compile-App.md
    ├── Lab-03-Explore-RootFS.md
    ├── Lab-04-Kernel-Module-Hello.md
    ├── Lab-05-Kernel-Module-Params.md
    ├── Lab-06-Device-Tree-Basics.md
    ├── Lab-07-Char-Driver-GPIO-LED.md
    └── Lab-08-Debugging-Kernel-Drivers.md
````

***

## Phase‑2 Learning Flow (Big Picture)

    What is Embedded Linux?
            ↓
    BBB Architecture & Boot Flow
            ↓
    Cross‑Compilation (Host → Target)
            ↓
    Kernel Fundamentals (practical view)
            ↓
    Kernel Modules
            ↓
    RootFS & BusyBox
            ↓
    Device Tree (hardware description)
            ↓
    User ↔ Kernel interaction
            ↓
    Character Device Driver (GPIO LED)
            ↓
    Debugging Embedded Linux systems

Modules and labs are **strictly aligned** to this flow.

***

## Modules (Theory Content)

Consists of **10 instructor‑led modules**.  

| Module | Title                                     |
| ------ | ----------------------------------------- |
| 1      | Embedded Linux Architecture – BBB View    |
| 2      | BeagleBone Black Boot Flow                |
| 3      | Cross‑Compilation for BBB                 |
| 4      | Linux Kernel Fundamentals (Embedded View) |
| 5      | Kernel Modules on BBB                     |
| 6      | Root File System & BusyBox                |
| 7      | Device Tree on BBB                        |
| 8      | User Space ↔ Kernel Space Interaction     |
| 9      | Character Device Driver (BBB‑based)       |
| 10     | Debugging Embedded Linux on BBB           |


***

## Labs (Hands‑On)

Includes **8 extensive, step‑by‑step labs**, consolidated in the `Labs/` folder.

Lab design principles:

*   Serial‑console‑first (always recoverable)
*   Image‑agnostic where possible
*   Industry‑realistic workflows
*   Explicit *why*, *what*, *how*, *expected output*, *failure modes*

| Lab   | Title                                      |
| ----- | ------------------------------------------ |
| Lab‑1 | BBB Development Environment Setup          |
| Lab‑2 | Cross‑Compiling an Application for BBB     |
| Lab‑3 | Exploring RootFS (BusyBox vs GNU, init)    |
| Lab‑4 | Kernel Module – Hello BBB                  |
| Lab‑5 | Kernel Module with Parameters              |
| Lab‑6 | Device Tree Basics on BBB                  |
| Lab‑7 | Character Driver – GPIO LED (`/dev` based) |
| Lab‑8 | Debugging Kernel & Drivers on BBB          |

***

## Minimum Setup Checklist

### Host (Instructor / Participant)

*   Ubuntu Linux recommended
*   ARM cross toolchain (`arm-linux-gnueabihf-gcc`)
*   Serial terminal (`picocom` / `minicom`)
*   `dtc`, `make`, `binutils`

### Target (BBB)

*   Any bootable Linux image
*   Serial console access
*   `sudo` access

***

## How to Use This Repository

1.  Teach **Modules 1–10 in order**
2.  Run **Labs 1–8 sequentially**
3.  Use labs as:
    *   in‑session demos
    *   guided hands‑on sessions
    *   post‑FDP practice material

### For Self‑Study

*   Read module README files top‑down
*   Execute labs carefully, **do not skip safety notes**
*   Always keep serial console connected

***

## Final Outcome

After completing participants will be able to:

*   Explain embedded Linux architecture correctly
*   Understand BBB boot and BSP concepts
*   Cross‑compile confidently
*   Load and debug kernel modules
*   Modify and deploy Device Tree changes
*   Write and test a real character device driver
*   Debug embedded Linux systems methodically

***

## Maintainer / Authoring Style

Content is authored in **clean Markdown**, suitable for:

*   GitHub repositories
*   classroom distribution
*   slide creation
*   long‑term reuse
