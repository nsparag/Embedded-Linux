# LABS (Phase‑2) — Master Index & Navigation
## Embedded Linux (BeagleBone Black / TI AM335x)

This folder contains the **complete hands‑on lab sequence** for Embedded Linux. Labs are designed to progress from **user space → kernel modules → device tree → character driver → debugging**, with **serial console** as the always‑available safety net.

---

## Quick Start (How to Use This Lab Manual)

1. **Do labs in order (Lab‑1 → Lab‑8).**
2. Use **serial console** for every lab (SSH is optional).
3. If anything breaks, use the **rollback steps** in the lab (especially Lab‑6 DTB work).
4. Treat this as a repeatable engineering workflow:
   - Observe → change one thing → verify → capture evidence → proceed.

---

## Repository Layout (Recommended)

```text
Labs/
├── README.md                         <-- (this file)
├── Lab-01-Env-Setup.md
├── Lab-02-Cross-Compile-App.md
├── Lab-03-Explore-RootFS.md
├── Lab-04-Kernel-Module-Hello.md
├── Lab-05-Kernel-Module-Params.md
├── Lab-06-Device-Tree-Basics.md
├── Lab-07-Char-Driver-GPIO-LED.md
└── Lab-08-Debugging-Kernel-Drivers.md
````

> If you use a different naming scheme, update the links below accordingly.

***

## Lab Sequence at a Glance (Learning Progression)

```text
Lab‑1  Environment + Serial + Kernel Facts
   ↓
Lab‑2  Cross‑compile user app (Host → BBB)
   ↓
Lab‑3  RootFS exploration (BusyBox/GNU, init, /proc,/sys)
   ↓
Lab‑4  Kernel module lifecycle (insmod/rmmod/dmesg)
   ↓
Lab‑5  Module parameters (load‑time behavior control)
   ↓
Lab‑6  Device Tree workflow (DTB→DTS→edit→DTB→deploy→verify)
   ↓
Lab‑7  Char driver + DT binding + GPIO LED + user test
   ↓
Lab‑8  Debugging playbook (printk/dmesg/sys/proc + failure scenarios)
```

***

## Prerequisites (Global)

### Hardware

*   BeagleBone Black (BBB), TI AM335x
*   micro‑USB **data** cable
*   Serial console access (USB‑serial)
*   Optional: Ethernet/Wi‑Fi for `scp` / SSH

### Host (Ubuntu recommended)

*   Terminal program: `picocom` / `minicom` / `screen`
*   ARM cross toolchain: `arm-linux-gnueabihf-gcc`
*   Basic build tools: `make`, `binutils`, etc.

### Target (BBB)

*   Any bootable Linux image (Debian, minimal, vendor, Yocto — **labs are image‑tolerant**, except where noted)
*   `sudo` access recommended
*   `dtc` recommended for DT lab (install if available)

***

## Labs Index (Links + Prereqs + Outcomes)

### Lab‑1 — BBB Development Environment Setup

**File:** Lab-01-Env-Setup.md

**Prerequisites**

*   BBB can boot **some** Linux (from eMMC or SD)
*   Host has serial terminal program installed

**Outcomes**

*   Serial console working at **115200**
*   Captured baseline facts: `uname -r`, `uname -m`, distro, DT model
*   Verified `/lib/modules/$(uname -r)/` presence and header/build status
*   Saved boot/dmesg snapshots for later labs

***

### Lab‑2 — Cross‑Compiling an Application for BBB

**File:** Lab-02-Cross-Compile-App.md

**Prerequisites**

*   Lab‑1 completed
*   Host has ARM cross toolchain installed
*   Transfer method available: `scp` (preferred) or USB copy

**Outcomes**

*   Built **host** and **target** binaries and proved difference via `file`
*   Validated ELF interpreter via `readelf`
*   Executed cross‑compiled program on BBB
*   Checked runtime dependencies via `ldd` on target
*   Understood common errors: `Exec format error`, missing loader

***

### Lab‑3 — Exploring RootFS on BBB

**File:** Lab-03-Explore-RootFS.md

**Prerequisites**

*   Lab‑1 completed (board access)
*   Basic shell access (serial/SSH)

**Outcomes**

*   Understood RootFS directory roles: `/bin /sbin /lib /etc /dev /proc /sys`
*   Determined BusyBox vs GNU utilities presence
*   Identified PID‑1 (`init`) and init system style (systemd/sysv/busybox init)
*   Connected RootFS contents to runtime failures (missing libs/loader/commands)

***

### Lab‑4 — Kernel Module: Hello BBB

**File:** Lab-04-Kernel-Module-Hello.md

**Prerequisites**

*   Lab‑1 completed
*   One build path available:
    *   Target headers present **OR**
    *   Host build using matching kernel build tree + copy `.ko`

**Outcomes**

*   Built a loadable kernel module (`.ko`)
*   Loaded/unloaded module using `insmod`/`rmmod`
*   Verified module presence with `lsmod`
*   Validated kernel logs using `printk` + `dmesg`
*   Learned to diagnose `Invalid module format` (vermagic mismatch)

***

### Lab‑5 — Kernel Module with Parameters (Load‑time Behavior Control)

**File:** Lab-05-Kernel-Module-Params.md

**Prerequisites**

*   Lab‑4 completed (module build workflow works)

**Outcomes**

*   Declared parameters using `module_param()`
*   Passed parameters at load time (`insmod name=value`)
*   Validated parameters via `modinfo` and `/sys/module/.../parameters/`
*   Practiced defensive parameter validation to prevent kernel instability

***

### Lab‑6 — Device Tree Basics on BBB

**File:** Lab-06-Device-Tree-Basics.md

**Prerequisites**

*   Lab‑1 completed (serial console mandatory)
*   `dtc` available on BBB (or available on host)
*   Ability to modify `/boot` and reboot (with rollback plan)

**Outcomes**

*   Identified candidate active DTB(s) under `/boot`
*   Decompiled DTB → DTS, edited a **safe** node, recompiled DTS → DTB
*   Deployed DTB safely with backup and rollback
*   Verified DT effect via `/sys` and `/proc/device-tree`
*   Built disciplined DT workflow: locate → edit → build → deploy → verify → rollback

***

### Lab‑7 — Character Driver on BBB (GPIO‑based) + `/dev` node + User‑space Test

**File:** Lab-07-Char-Driver-GPIO-LED.md

**Prerequisites**

*   Lab‑2 (cross compile) recommended for user program
*   Lab‑4 (module build) required
*   Lab‑6 (DT workflow) required
*   Optional hardware: LED + resistor + jumper wires (recommended)

**Outcomes**

*   Created DT node for driver binding via `compatible`
*   Built a platform driver module that claims GPIO via gpiod
*   Exposed `/dev/bbb_led` through character device registration
*   Ran user program to control LED using POSIX file operations
*   Understood `/dev` node creation (udev vs manual `mknod`)

***

### Lab‑8 — Debugging Kernel & Drivers on BBB

**File:** Lab-08-Debugging-Kernel-Drivers.md

**Prerequisites**

*   Lab‑7 completed (driver specimen available)
*   Serial console mandatory

**Outcomes**

*   Built a repeatable debugging workflow:
    *   capture baseline `dmesg`
    *   tighten action→log correlation loop
    *   collect `/proc` and `/sys` evidence bundle
*   Practiced realistic failure scenarios and fixes:
    *   DT node not applied
    *   probe not called
    *   GPIO busy
    *   `/dev` missing
    *   permission denied
    *   invalid module format
*   Learned field‑grade discipline: observe → isolate layer → minimal change → verify

***

## Suggested Run Order for a Live FDP Session

*   Day‑1: Lab‑1, Lab‑2, Lab‑3
*   Day‑2: Lab‑4, Lab‑5
*   Day‑3: Lab‑6 (with strong rollback discipline)
*   Day‑4: Lab‑7, Lab‑8 (debug playbook + failure drills)

***

## Troubleshooting Quick Notes (One‑Page)

### If serial console shows nothing

*   Wrong cable (charge‑only) or wrong device node (`ttyACM0` vs `ttyUSB0`)
*   Fix: recheck `dmesg | grep tty` on host, use `115200`

### If module insert fails: `Invalid module format`

*   Kernel/headers mismatch
*   Fix: compare `uname -r` vs `modinfo *.ko | grep vermagic`

### If DT changes don’t take effect

*   You edited a DTB not used by bootloader
*   Fix: verify DT node existence under `/proc/device-tree` after reboot

### If `/dev/bbb_led` is missing

*   No udev/mdev
*   Fix: create node using major number from `/proc/devices` or dmesg
If you want, I can also generate a **top-level repository `README.md`** that links **Modules/** and **Labs/** together (single entry point for your GitHub repo).
```
