# 🧩 MODULE‑7  
# Device Tree on BeagleBone Black  
*Describing Hardware to the Linux Kernel*

---

## 🎯 Module Objective

This module explains **why the Linux kernel needs a Device Tree** and **how embedded hardware is described without hardcoding it into the kernel**.

By the end of this module, you will be able to:

✅ Understand **what Device Tree is and why it exists**  
✅ Understand **why embedded Linux cannot auto‑detect hardware**  
✅ Read and reason about **DTS / DTSI / DTB at a conceptual level**  
✅ Understand **how drivers bind to hardware using Device Tree**  
✅ Understand **pin multiplexing on AM335x via Device Tree**  
✅ Be mentally prepared to **modify Device Tree safely in labs**

> 🧠 This module is the **missing link** between  
> **Kernel ↔ Hardware ↔ Drivers**

---

## 🧠 TOPIC‑1: The Core Problem Device Tree Solves

Fundamental embedded mismatch:

> ✅ Linux kernel binary is **generic**  
> ✅ Embedded hardware is **specific**

The kernel **cannot assume hardware**.

On desktop PCs:
- BIOS / UEFI enumerates hardware
- ACPI describes devices
- Kernel discovers hardware dynamically

On BeagleBone Black:
- ❌ No BIOS
- ❌ No ACPI
- ❌ No auto‑discovery

Therefore:

> ⚠️ The kernel must be **explicitly told** what hardware exists.

That description mechanism is the **Device Tree**.

---

## 🧩 TOPIC‑2: What Is Device Tree?

Device Tree is:

✅ A **data structure**  
✅ Hardware description only  
✅ Passed to the kernel during boot  
✅ Independent of kernel logic  

Device Tree is **NOT**:
- Executable code
- A driver
- Kernel behaviour

It describes:
- CPU
- Memory
- Peripherals
- GPIOs
- Interrupts
- Pin configuration

Key principle to engrave:

> 🧠 Device Tree says **WHAT hardware exists**  
> 🧠 Drivers define **HOW to use it**

---

## 🧱 TOPIC‑3: Why Hardware Is NOT Hardcoded in the Kernel

Embedded reality:

- One kernel supports thousands of boards
- Boards vary in:
  - Memory
  - GPIO usage
  - Peripherals
  - Pin connections

If hardware was hardcoded:
- Kernel source would explode
- Every board change = kernel rebuild

Device Tree solves this by:

✅ Moving board‑specific data **outside kernel source**  
✅ Keeping the kernel reusable and generic  

---

## 🔁 TOPIC‑4: Device Tree in the Boot Flow

```

U‑Boot
↓
loads Kernel
↓
loads DTB
↓
passes DTB to Kernel

```

Tie‑in to Module‑2:

- U‑Boot loads kernel into RAM
- U‑Boot loads Device Tree Blob (DTB)
- Kernel receives DTB address

The kernel then:
✅ Parses Device Tree  
✅ Creates device instances  
✅ Matches drivers to hardware  

> ⚠️ Wrong or missing DTB:  
> Kernel may boot, but hardware will not work.

---

## 📄 TOPIC‑5: DTS, DTSI, DTB — Key Terminology

```

DTS   – Device Tree Source
DTSI  – Device Tree Include
DTB   – Device Tree Blob

```

### DTS
- Human‑readable text file
- Board‑specific
- Written by developers

### DTSI
- Included/helper files
- Common SoC‑level description
- Avoid duplication

### DTB
- Binary compiled form
- Loaded by bootloader
- Consumed by kernel only

Important lifecycle:

> 🧠 Edit DTS → Compile → Deploy DTB  
> Editing DTS **alone changes nothing**

---

## 🌳 TOPIC‑6: Logical Structure of Device Tree

Device Tree follows a **tree hierarchy**:

```

SoC
├── CPU
├── Memory
├── GPIO
├── I2C
└── SPI

```

Conceptual mapping:

- Tree mirrors **physical SoC layout**
- Nodes represent hardware blocks
- Relationships are explicit

---

## 🔑 TOPIC‑7: Nodes and Properties

### Node
- Represents a hardware component

### Properties
- Describe how it is wired:
  - Address
  - Interrupts
  - GPIOs
  - Clocks
  - Pin configuration

Mental model:

> ✅ Node → “This hardware exists”  
> ✅ Properties → “This is how it is connected”

The kernel **never guesses**.  
It only trusts what Device Tree declares.

---

## 🔗 TOPIC‑8: The `compatible` Property (CRITICAL)

The most important Device Tree property:

`compatible`

Purpose:
- Identifies hardware type
- Enables driver binding

Driver probing flow:
1. Device Tree declares `compatible`
2. Kernel scans registered drivers
3. Matching driver is probed

Hard rule:

> ❌ No compatible match  
> ✅ No driver load

This explains many real‑world:
> “Driver compiled fine but doesn’t work” problems

---

## 🤝 TOPIC‑9: Device Tree vs Drivers — Clear Separation

### Device Tree:
- Describes hardware
- No logic
- No behaviour
- No code execution

### Driver:
- Implements behaviour
- Controls registers
- Uses kernel APIs

Key advantage:

> 🧠 Change hardware description  
> without touching driver code

---

## 🧲 TOPIC‑10: AM335x Pin Multiplexing Problem

AM335x pins are **multi‑functional**:

One pin can be:
- GPIO
- UART
- SPI
- I2C

⚠️ Only **one function** can be active at a time.

Pinmux realities:
- Not automatic
- Must be selected explicitly
- Wrong pinmux = dead hardware

---

## 📍 TOPIC‑11: Pin Multiplexing via Device Tree (BBB Context)

On BeagleBone Black:
- Pinmux is defined in Device Tree
- Applied early by U‑Boot or kernel
- Drivers assume pinmux is correct

Common confusion:
- Driver loads successfully
- Pin shows no activity

Root cause:
> ❗ Pinmux mismatch in Device Tree

---

## 🚫 TOPIC‑12: Why Device Tree Is NOT Optional

On modern ARM systems:

- Kernel depends on DT
- Drivers depend on DT
- Hardware depends on DT

Without Device Tree:
- Kernel is blind
- Drivers cannot bind
- Hardware remains unused

> ⚠️ No Device Tree  
> → No usable Embedded Linux system

---

## 🚨 TOPIC‑13: Common Device Tree Mistakes (Industry Reality)

Typical real‑world errors:

- Driver not loading → wrong `compatible`
- Hardware silent → incorrect pinmux
- “No change after editing DTS” → DTB not updated
- Old DTB still in boot partition

Key lesson:

> 🧠 Device Tree issues are logical,  
> not mysterious.

---

## ✅ End‑of‑Module Mental Model

- Kernel is hardware‑agnostic
- Device Tree provides hardware *truth*
- Drivers bind via `compatible`
- Pinmux is critical on AM335x
- Wrong DT → silent failure

---

✅ You now understand **how hardware is described to the Linux kernel**.  
This completes the **Kernel–Hardware–Driver relationship**.

---

Say **“Proceed with Module‑8”** when ready.
