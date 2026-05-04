# MODULE‑1 
# Embedded Linux Architecture – BeagleBone Black View

---

## What Is Embedded Linux?

Embedded Linux is not a product you download and install like Ubuntu or Fedora.
Instead, it is:
* A collection of tightly coupled components
* Selected and built specifically for one hardware board
* Intended to run one product, not general workloads

For BeagleBone Black:
* Hardware = AM335x SoC
* Software must be prepared to match:
  * CPU architecture
  * Memory layout
  * Peripheral set

`Embedded Linux does not adapt to hardware. Hardware dictates how Linux is built.`

---

## Embedded Linux vs Desktop Linux

| Aspect                     | Desktop Linux                          | Embedded Linux                     |
|---------------------------|----------------------------------------|------------------------------------|
| Hardware Handling         | Hardware added/removed dynamically     | Hardware fully known before boot   |
| Driver Inclusion          | Many unused drivers present            | Only required drivers included     |
| Software Control          | User decides software installation     | End user never modifies OS         |



| Example Device     | Behavior Description                                  |
|--------------------|-------------------------------------------------------|
| Laptop             | User installs printer driver as needed                |
| Washing Machine    | OS never changes in the field after deployment        |


Embedded Linux: 

| Resulting Impact            | Reason                                           |
|-----------------------------|--------------------------------------------------|
| Smaller system size         | No unused drivers or packages                   |
| Faster boot time            | Minimal initialization and services             |
| Deterministic behavior      | Fixed hardware and controlled software stack    |

---

## Why Embedded Linux Exists

Industry requirements pushed Linux into embedded systems:
* Full POSIX APIs
* TCP/IP, USB, filesystem support
* Long‑term maintainability
* No vendor lock‑in

RTOS limitations:
* Limited filesystem
* Limited networking
* Vendor‑specific APIs

`Embedded Linux gives RTOS‑like control + OS‑level capability.`

---

## BeagleBone Black as Reference Platform


* TI AM335x SoC
* ARM Cortex‑A8 CPU
* DDR controller
* GPIO, UART, I2C, SPI
* On‑chip Boot ROM
* Industrial‑class architecture
* Representative embedded platform

Learning BBB prepares engineers for real products, not demos.

---

## Embedded Linux Layered Architecture



---

## Kernel Space vs User Space

| Aspect              | User Space                 | Kernel Space                     |
|---------------------|----------------------------|----------------------------------|
| Privilege Level     | Unprivileged               | Privileged (Kernel mode)         |
| Safety              | Safe (isolated execution)  | Unsafe if buggy                  |
| Hardware Access     | No direct hardware access  | Direct hardware access           |
| Fault Impact        | Process crash only         | One bug can crash whole system   |

Linux enforces strict separation:
* User programs cannot access registers
* Kernel alone controls memory maps
* Drivers exist because of this rule

BBB example:
* User app cannot toggle GPIO directly
* LED toggling must go via kernel

---

## Why Device Drivers Exist
* Applications cannot touch hardware
* Drivers mediate access
* Drivers hide hardware details

Drivers provide:
* Safety
* Portability
* Uniform access

Same user program:
* Runs on different boards
* Uses different drivers underneath

`This abstraction is why Linux scales across products.`

---

## Execution Flow Example (LED Control)

````
User Application
→ System Call
→ Kernel
→ LED Driver
→ GPIO Hardware
→ LED ON
````

---

## Root File System (RootFS)

- Provides the **userspace environment**
- Contains **programs, libraries, and configuration files**
- **Mounted by the kernel during system boot**

The Linux kernel **alone cannot run applications**. It only manages:
- CPU scheduling
- Memory
- Devices
- System calls

To execute any user-level program, the kernel **requires a Root File System (RootFS)**.

**What RootFS Supplies**: The RootFS provides all essential user‑space components, including:

- `/bin`, `/sbin` - Core system executables and administrative utilities

- `/lib`  - Shared libraries required by executables at runtime

- `/dev`  - Device nodes that act as interfaces to hardware

- `/proc`  - Virtual filesystem exposing process and kernel information

- `/sys`  - Interface to the kernel device model and drivers

These directories together enable applications to **start, execute, and interact with hardware**.

**RootFS Defaults**
- Default userspace layout
- Init system (systemd, busybox init, etc.)
- Startup scripts and configuration files
- Device nodes and permissions

This defines **how the system looks and behaves after boot**.

- **BeagleBone Black (BBB) Debian Image**  
  A full‑featured RootFS intended for development and general use

- **Embedded Products**  
  Typically use a **much smaller, custom RootFS**, containing:
  - Only required binaries
  - Only necessary libraries
  - No unused utilities or drivers

This customization helps reduce system size, boot time, and complexity.

---

## Board Support Package (BSP)

A **Board Support Package (BSP)** is the software layer that **binds Linux to a specific hardware platform**.

Linux, by itself, is **hardware‑agnostic**. It does not know:
- Which CPU is used
- What peripherals exist
- How memory is laid out
- How the board boots

All this knowledge is provided by the **BSP**.

**What a BSP Includes**

- Bootloader Configuration
  - Board‑specific initialization
  - DDR setup
  - Clock and pin multiplexing
  - Loading kernel and device tree into memory

  Examples:
  - U‑Boot board files
  - Environment variables
  - Boot scripts


- Kernel Configuration
  - CPU architecture support
  - Enabled drivers for on‑board peripherals
  - Power management and clock support
  - Removal of unused kernel features

The kernel is **configured and built specifically for the target board**.


- Device Tree
  - Description of hardware layout
  - Addresses, interrupts, GPIO mappings
  - Connection between devices and drivers

The device tree tells the kernel: > “This hardware exists, at this address, wired this way”

No device tree → no usable hardware.

**Porting Linux does NOT mean writing Linux.** Linux already exists.
Porting Linux means:
- Creating or adapting a **BSP**
- Teaching Linux about *your* hardware
- Assembling bootloader, kernel, device tree, and RootFS

In industry terms: > **Every embedded product ships with a BSP.**

Without a BSP:
- Linux cannot boot
- Hardware cannot be accessed
- Product cannot exist
