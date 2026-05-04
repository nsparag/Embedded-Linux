# MODULE‑6  
# Root File System & BusyBox  
*User Space in Embedded Linux (BeagleBone Black View)*

---

## Purpose: This module explains **what actually runs after the Linux kernel boots**.

By the end of this module, you will clearly understand:
- Why the kernel **alone is useless**
- What the **Root File System (RootFS)** really is
- How Linux transitions from **kernel space → user space**
- What `/init` does and why it is critical
- Why embedded systems prefer **BusyBox**
- How RootFS fits into real embedded products (BBB context)

This module **completes the boot story** started in Module‑2.

---

## TOPIC‑1: Kernel Alone Is NOT a Complete System

- Kernel boots hardware
- Kernel cannot run applications
- User space is mandatory
- RootFS provides user space

A critical misconception must be eliminated early:

> A booted kernel is **not** a running system.

The Linux kernel:
- Initializes CPU and memory
- Controls devices
- Has no shell
- Has no commands
- Cannot interact with users or applications

If the kernel boots but RootFS is missing:
- No login
- No commands
- No usable system

On BeagleBone Black: > Kernel + no RootFS = dead‑end system

---

## TOPIC‑2: What Is Root File System (RootFS)?

The Root File System is:
- A directory hierarchy
- Mounted by the kernel as `/`
- The **foundation of user space**

RootFS provides:
- Executable programs
- Shared libraries
- Configuration files
- Interfaces to the kernel

Mental rule to internalize:

> RootFS is not “storage”  
> RootFS is the **environment**

---

## TOPIC‑3: RootFS in the Boot Flow

````
Boot ROM → SPL → U‑Boot → Kernel → RootFS → init
````

This is the **exact system transition**:

1. Kernel finishes hardware initialization
2. Kernel mounts the RootFS
3. Kernel executes `/sbin/init`
4. User space starts running

If RootFS cannot be mounted:
- Kernel panics  
  or  
- Kernel halts execution

This explains many real‑world embedded boot failures.

---

## TOPIC‑4: Standard RootFS Directory Structure


````
/bin    – User commands
/sbin   – System commands
/lib    – Shared libraries
/etc    – Configuration files
/dev    – Device nodes
/proc   – Kernel runtime info
/sys    – Kernel objects
````

Directories you will use constantly:

- `/bin`, `/sbin`  Commands and system utilities

- `/lib`    Shared libraries for executables

- `/etc`    Configuration controlling system behavior

- `/dev`    Kernel‑created device interfaces

- `/proc`, `/sys`  Live views into kernel internals

Important boundary:

> `/proc` and `/sys` are **not real files on disk**  
> They are **live kernel views**

---

## TOPIC‑5: `/dev`, `/proc`, `/sys` — Why They Matter


- `/dev`   – User ↔ driver interface
- `/proc`  – Process and kernel information
- `/sys`   – Device and driver representation

These directories are essential for embedded engineering:

- `/dev`    Primary interface where applications access drivers

- `/proc`    Inspect processes, memory, kernel state

- `/sys`    Structured representation of:
  - Devices
  - Drivers
  - Kernel parameters

Preview example:

> BeagleBone Black onboard LEDs are controlled via `/sys`

---

## TOPIC‑6: What Is `init`?

- First user‑space process
- Started by the kernel
- PID = 1
- Parent of all processes

After mounting RootFS, the kernel executes:

```
/sbin/init
```

Key facts:
- `init` is **not optional**
- Always runs as PID 1
- If `init` crashes, the system becomes unusable

Responsibilities of `init`:
- Start system services
- Bring up login or application
- Handle shutdown and reboot

---

## TOPIC‑7: init Systems

- sysvinit
- systemd
- BusyBox init

Keep this strictly conceptual.

- **sysvinit**
  - Script‑based
  - Simple and predictable

- **systemd**
  - Unit‑file based
  - Used by Debian BBB images

- **BusyBox init**
  - Extremely small
  - Common in production embedded systems

Key principle:
- Init system choice affects **startup behavior**,  
- not kernel behavior.

---

## TOPIC‑8: Debian RootFS on BeagleBone Black

- Debian‑based RootFS
- GNU utilities
- systemd init
- Development‑friendly

BeagleBone Black typically ships with Debian because:
- Beginner friendly
- Rich command set
- Familiar PC‑like environment
- Excellent for experimentation

But: Debian is usually **not shipped in real products**

Products prefer:
- Smaller RootFS
- Fewer services
- Faster boot
- Minimal attack surface

---

## TOPIC‑9: What Is BusyBox?

- Single executable
- Provides multiple Linux commands
- Optimized for size
- Embedded‑friendly

BusyBox is a **multi‑call binary**:
- One executable
- Behaves like many commands:
  - `ls`, `cp`, `mount`, `ifconfig`, etc.

Why BusyBox exists:
- Embedded systems have limited flash and RAM

BusyBox trade‑off:
- Fewer features  
- For dramatically smaller size

---

## TOPIC‑10: BusyBox vs GNU Utilities

GNU utilities:
- Full features
- Large binaries
- Desktop‑oriented

BusyBox:
- Minimal features
- Very small footprint
- Embedded‑oriented

Design mindset shift:

> Sufficient functionality  
> is more important than  
> feature completeness

This mindset distinguishes embedded Linux from desktop Linux.

---

## TOPIC‑11: RootFS and Embedded Constraints

- Limited storage
- Limited RAM
- Fast boot requirement
- Controlled software set

Every file and service has a cost:
- Storage consumption
- RAM usage
- Boot‑time delay

Embedded RootFS design focuses on:
- Removing unused utilities
- Disabling unnecessary daemons
- Keeping runtime predictable

---

## TOPIC‑12: RootFS Is Mostly Hardware‑Independent

- RootFS mostly hardware‑agnostic
- Kernel & Device Tree handle hardware
- Same RootFS can run on many boards

Clear separation of responsibility:
- Kernel + drivers + device tree → board specific
- RootFS → mostly generic

This enables:
- Faster development
- BSP reuse across products

Important constraint:

> Kernel and RootFS **must be compatible**

---

## TOPIC‑13: Common RootFS Failures (Real Issues)

Typical failure reasoning:
- Kernel panic during boot → wrong root device
- Immediate reboot or halt → missing `init`
- Application fails silently → missing libraries
- Command not found → BusyBox vs GNU difference

These failures are common, predictable, and **diagnosable**.

---

## Key Takeaways

- Kernel alone is not enough
- RootFS enables user space
- init is the first process
- Embedded RootFS is minimal
- BusyBox is common in products
