# MODULE‑7
# Device Tree on BeagleBone Black  
## *Describing Hardware to the Linux Kernel*

---

## Purpose: This module explains **why the Linux kernel needs a Device Tree** and **how embedded hardware is described without hardcoding it into the kernel**.

By the end of this module, you will be able to:

- Understand **what Device Tree is and why it exists**
- Understand **why embedded Linux cannot auto‑detect hardware**
- Read and reason about **DTS / DTB at a conceptual level**
- Understand **how drivers bind to hardware using Device Tree**
- Understand **pin multiplexing on AM335x via Device Tree**
- Be mentally prepared to **safely modify Device Tree** in later labs

This module is the **missing link** between:

- Kernel  
- Hardware  
- Drivers  

---

## TOPIC‑1: The Core Problem Device Tree Solves

- Kernel must know hardware layout
- Embedded systems have fixed hardware
- No hardware auto‑detection
- Hardcoding is not scalable

the fundamental mismatch:

> The Linux kernel binary is **generic**  
> Embedded hardware is **specific**

On PCs:
- BIOS/UEFI enumerates devices
- ACPI provides hardware information
- Kernel discovers hardware dynamically

On BeagleBone Black:
- No BIOS
- No ACPI
- No dynamic discovery

Therefore:

> The kernel must be **explicitly told** what hardware exists

That description mechanism is **Device Tree**.

---

## TOPIC‑2: What Is Device Tree?


Device Tree is:
- A **data structure**
- Not executable code
- Not a driver
- Not kernel logic
- Passed to the kernel at boot
- Hardware description only

It describes:
- CPU
- Memory
- Peripherals
- GPIOs
- Interrupts
- Pin configuration

Key statement to emphasize:

> Device Tree says **WHAT hardware exists**  
> Drivers define **HOW to use it**

---

## TOPIC‑3: Why Hardware Info Is NOT in Kernel Code

In embedded systems:
- One kernel supports thousands of boards
- Hardware varies across products:
  - Memory size
  - GPIO usage
  - Peripherals enabled

If hardware were hardcoded into the kernel:
- Kernel source would explode
- Every hardware change would require recompiling kernel code

Device Tree solves this by:
- Moving **board‑specific data outside kernel source**

---

## TOPIC‑4: Device Tree in the Boot Flow

    Bootloader
    → loads Kernel
    → loads Device Tree Blob (DTB)
    → passes DTB to Kernel

Tie this back to Module‑2 boot flow.

U‑Boot:
- Loads kernel into RAM
- Loads DTB into RAM
- Passes DTB address to the kernel

The kernel then:
- Parses the Device Tree
- Instantiates devices
- Matches drivers to hardware

If DTB is missing or wrong:
- Kernel may boot
- Hardware will not function

---

## TOPIC‑5: DTS, DTSI, DTB — Key Terms

    DTS  – Device Tree Source
    DTSI – Device Tree Include
    DTB  – Device Tree Blob (binary)

Clarify terminology carefully:

- **DTS**
  - Human‑readable text file
  - Written by developers
  - Describes board‑specific hardware

- **DTSI**
  - Included files
  - Shared SoC or common board description
  - Avoid duplication

- **DTB**
  - Binary compiled form
  - Loaded by bootloader
  - Consumed by kernel

---

## TOPIC‑6: Logical Structure of Device Tree

- Tree structure
- Nodes represent hardware blocks
- Properties describe characteristics

Device Tree is hierarchical:

- Root node
- Child nodes for hardware components

Conceptually:

    SoC
     ├── CPU
     ├── Memory
     ├── GPIO
     ├── I2C
     └── SPI

This hierarchy mirrors the **physical SoC layout**.

---

## TOPIC‑7: Nodes and Properties (Conceptual)


Node:
- Hardware component

Properties:
- Address
- Interrupts
- GPIOs
- Clocks
- Compatible string

Mental model:

- **Node** → “this hardware exists”
- **Properties** → “this is how it is wired”

Examples of property meaning:
- Which GPIO pin
- Which interrupt line
- Which clock source

The kernel **never guesses**.  
It trusts only what Device Tree declares.

---

## TOPIC‑8: The `compatible` Property (CRITICAL)

- Identifies hardware type
- Used for driver binding
- Key to driver probing

This is one of the **most important Device Tree concepts**.

The `compatible` string:
- Identifies the hardware
- Allows the kernel to select the correct driver

Binding flow:
1. Device Tree declares `compatible`
2. Kernel scans registered drivers
3. Matching driver is probed

Key rule:

> No compatible match  
> → driver never loads

This explains many real‑world  
“driver is not working” issues.

---

## TOPIC‑9: Device Tree and Drivers — Relationship



Device Tree:
- Describes hardware

Driver:
- Implements hardware control

Strict separation of responsibility:

- Device Tree:
  - No logic
  - No code
  - No behavior

- Driver:
  - Implements behavior
  - Uses kernel APIs
  - Accesses registers

Device Tree enables or disables hardware **without changing driver code**.

---

## TOPIC‑10: AM335x Pin Multiplexing Problem

- Pins have multiple functions
- Only one function active at a time
- Controlled via pinmux

On AM335x:
- One physical pin can function as:
  - GPIO
  - UART
  - SPI
  - I2C

This is **not automatic**. Pin function must be selected explicitly. > Wrong pinmux = hardware appears dead

---

## TOPIC‑11: Pinmux via Device Tree (BBB Context)

On BeagleBone Black:
- Pin multiplexing is defined in Device Tree
- Applied early by U‑Boot or kernel
- Drivers assume pinmux is already correct

This explains cases where:
- GPIO driver loads correctly
- But pin shows no activity


---

## TOPIC‑12: Why Device Tree Is NOT Optional

- Kernel depends on it
- Drivers depend on it
- Hardware depends on it

Without Device Tree:
- Kernel has no hardware knowledge
- Drivers cannot bind
- Peripherals remain unused

On modern ARM platforms:

> No Device Tree  
> → No usable Linux system

---

## TOPIC‑13: Common Device Tree Mistakes

Typical real‑world failures:
- Driver not loading → compatible string mismatch
- Hardware silent → incorrect pinmux
- No visible change → old DTB still in use

Key lesson:

> Editing DTS without deploying DTB  
> changes **nothing**

---

## MODULE‑7 — Key Takeaways

- Kernel cannot auto‑detect hardware
- Device Tree describes hardware
- Drivers bind via compatible strings
- Pinmux is critical on AM335x
- DT errors are common but logical
