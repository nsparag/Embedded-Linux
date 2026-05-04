# 🧩 MODULE‑8  
# User Space ↔ Kernel Space Interaction  
*How Applications Control Hardware in Embedded Linux*

---

## 🎯 Module Objective

This module explains **how user‑space applications legally and safely interact with the Linux kernel** to control hardware.

By the end of this module, you will be able to:

✅ Understand **how user programs cross into kernel space**  
✅ Clearly explain **system calls and kernel mediation**  
✅ Understand **/dev, /sys, /proc as kernel interfaces**  
✅ Distinguish **sysfs vs device‑file based access**  
✅ Understand **where character drivers fit**  
✅ Be fully prepared for **Module‑9: Character Device Driver**

> 🧠 This module is the **bridge** between  
> **Applications → Drivers**

---

## 🧠 TOPIC‑1: The Fundamental Rule (NON‑NEGOTIABLE)

State this rule clearly and permanently:

> ❌ User‑space programs **CANNOT touch hardware**  
> ❌ Not registers  
> ❌ Not GPIO  
> ❌ Not interrupts  

All hardware access is:
✅ Controlled  
✅ Validated  
✅ Mediated by the kernel  

This rule exists to ensure:
- System stability
- Security
- Fault isolation

Breaking this rule is **physically impossible**, not just “forbidden”.

---

## 🧩 TOPIC‑2: Why Interaction Is Needed at All

Embedded products exist to **do something physical**:

- Toggle LEDs
- Read sensors
- Communicate over UART, SPI, I2C
- Control actuators

But applications:
- Run in unprivileged mode
- Have no hardware access
- Cannot touch registers

Therefore:

> 🧠 The kernel provides **defined doors**  
> 🧠 Applications may use **only those doors**

---

## 🔗 TOPIC‑3: The Big Picture Interaction Flow

```

User Application
↓
System Call
↓
Kernel
↓
Driver
↓
Hardware

````

This is the **only valid execution chain**:

1. Application requests an operation
2. Kernel validates the request
3. Kernel invokes the correct driver
4. Driver accesses hardware
5. Result flows back to application

> ⚠️ No user‑space request ever bypasses the kernel.

This exact flow is what you will **implement concretely in Module‑9**.

---

## 🚪 TOPIC‑4: System Calls – The Legal Gateway

System calls are:
✅ Controlled entry points  
✅ Initiated by user space  
✅ Enforced by CPU privilege levels  

Why they exist:
- CPUs support privilege separation
- User space runs unprivileged
- Kernel runs privileged

When an application calls:

```c
open(), read(), write(), ioctl()
````

What actually happens:

1.  CPU switches to kernel mode
2.  Kernel executes the requested service
3.  Control returns safely to user space

> 🔒 This transition is **hardware‑enforced**, not optional.

***

## 🧠 TOPIC‑5: POSIX APIs vs System Calls

Destroy a common misconception:

    POSIX API  ≠  System Call
    POSIX API  →  MAY invoke a system call

Examples:

*   `printf()` → no direct system call
*   `open()` → system call
*   `pthread_create()` → multiple system calls internally

Clarification:

*   **POSIX APIs**
    *   Defined by standards
    *   Implemented by libraries (glibc)

*   **System Calls**
    *   Defined by kernel
    *   Executed in kernel space

***

## 🔌 TOPIC‑6: Major User ↔ Kernel Interfaces

Linux exposes **four major interfaces**:

*   ✅ **System calls** – Direct service requests
*   ✅ **/dev** – File‑based device access
*   ✅ **/sys** – Attribute‑based control/configuration
*   ✅ **/proc** – Kernel visibility and diagnostics

Embedded engineers must know **when to use which**.

***

## 📂 TOPIC‑7: `/dev` – Device File Interface

`/dev` provides a **file abstraction for hardware**.

Characteristics:

*   Backed by device drivers
*   Supports `open`, `read`, `write`, `ioctl`

Examples:

    /dev/ttyS0
    /dev/i2c-1
    /dev/gpiochip0

From user space:

*   Devices look like files
*   Kernel routes operations to driver callbacks

> 🧠 This is the **foundation of character device drivers**.

***

## ⚙️ TOPIC‑8: `/sys` – sysfs Interface

`/sys` is:
✅ Attribute‑based  
✅ Kernel‑generated at runtime  
✅ Structured representation of kernel objects

Used to:

*   Enable/disable hardware
*   Configure parameters
*   Expose device attributes

On BeagleBone Black:

*   LEDs
*   GPIOs
*   PWM

are often controlled via `/sys`.

Important rule:

> ⚠️ `/sys` is for **control and configuration**  
> ❌ Not for bulk data transfer

***

## 🪟 TOPIC‑9: `/proc` – Kernel Insight Interface

`/proc` exposes:

*   Process information
*   Memory statistics
*   Kernel state
*   Interrupt counts

Examples:

    /proc/cpuinfo
    /proc/meminfo
    /proc/interrupts

Embedded relevance:

*   Lightweight diagnostics
*   Remote debugging
*   Field issue analysis

***

## ⚖️ TOPIC‑10: `/dev` vs `/sys` — Engineering Discipline

Clear distinction:

### `/dev`

*   Data path
*   Stream‑based access
*   Driver‑centric

### `/sys`

*   Control path
*   Configuration
*   Attribute‑centric

Design guidance:

*   Use `/dev` for:
    *   Continuous data transfer

*   Use `/sys` for:
    *   Enable/disable
    *   Parameter settings

Misuse leads to:

*   Poor design
*   Performance issues
*   Maintenance headaches

***

## ⏳ TOPIC‑11: Blocking vs Non‑Blocking Interaction

High‑level concepts:

*   **Blocking I/O**
    *   Simpler design
    *   Can stall application

*   **Non‑blocking I/O**
    *   More complex
    *   Better responsiveness

Embedded reality:

> ⚠️ Poor blocking design can freeze  
> the entire product logic.

This concept reappears in **driver design**.

***

## 🔧 TOPIC‑12: `ioctl` — Controlled Escape Hatch

Sometimes `read()` and `write()` are insufficient.

`ioctl` provides:

*   Structured commands
*   Driver‑specific control
*   Custom operations

Typical use cases:

*   Mode configuration
*   Parameter selection
*   Status queries

Important caution:

> 🧠 `ioctl` is powerful  
> ❗ Must be used **sparingly and carefully**

***

## 🛡️ TOPIC‑13: Security and Stability Rationale

Why strict mediation exists:

*   User code is untrusted
*   Kernel code is trusted
*   Interfaces must be narrow and validated

Embedded reality:

*   No keyboard
*   No UI
*   No user recovery

Therefore:

> 🔒 Strict kernel mediation is **non‑negotiable**

***

## ✅ End‑of‑Module Mental Model

*   User space cannot touch hardware
*   Kernel mediates all interaction
*   System calls are the gateway
*   `/dev`, `/sys`, `/proc` expose kernel functionality
*   Drivers make hardware usable

***

## 🚀 Readiness Check

✅ You now understand **how applications legally reach hardware**.  
✅ You understand **why drivers are necessary**.  
✅ You are ready to **implement a character device driver**.

***
