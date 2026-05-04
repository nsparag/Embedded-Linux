# MODULE‑9  
# Character Device Driver on BeagleBone Black  
## *From User Space to Hardware Control*

---

Purpose: This module teaches **how hardware is controlled by custom kernel code** using a **character device driver**, the **most common driver type in embedded Linux**.

By the end of this module, you will be able to:

- Understand **what a character device driver is**
- Understand **how drivers expose `/dev` nodes**
- Understand **major / minor numbers conceptually**
- Understand **file operations: open / read / write / ioctl**
- Understand **how GPIO is controlled from a driver**
- Connect:
  - Device Tree
  - Kernel module
  - User‑space application

This is where **all earlier modules converge**.

---

## TOPIC‑1: Why Character Device Drivers?

- Simple, flexible driver model
- File‑like access from user space
- Ideal for GPIO, sensors, LEDs
- Dominant type in embedded Linux

Industry truth:

> Most embedded peripherals are best modeled as **character devices**.

Reasons:

- Data is sequential
- No block buffering required
- Matches file‑based access semantics

Typical examples:

- LED control
- Button input
- Sensor readouts
- Custom registers

On BeagleBone Black:

> GPIO‑based devices are **naturally modeled as character devices**

---

## TOPIC‑2: What Is a Character Device?

- Kernel driver exposed as a file
- Accessed via /dev
- Uses open / read / write / ioctl
- No filesystem involved

A character device:

- Is **not** a real file on disk
- Is a **kernel object**
- Appears as a file under `/dev`

From user space:

```c
fd = open("/dev/mydevice", O_RDWR);
````

From kernel space:

- The driver’s `open()` function is invoked

Key mental model:

> `/dev` is a **portal into the kernel**

---

## TOPIC‑3: Character Driver Position in the System


    User Application
    → /dev/mydevice
    → VFS
    → Character Driver
    → Hardware (GPIO)

End‑to‑end flow:

1.  Application opens `/dev/mydevice`
2.  VFS routes the request
3.  Character driver executes kernel code
4.  Driver accesses hardware
5.  Result flows back to user space

This flow directly ties together:

- Module‑5: Kernel modules
- Module‑7: Device Tree
- Module‑8: User ↔ Kernel interaction

---

## TOPIC‑4: What a Character Driver Really Is

A character driver is:

- A **kernel module**
- With added responsibilities:
    - Register device numbers
    - Implement callbacks
    - Handle user requests

Important statement to emphasize:

> A character driver is not magic  
> It is **disciplined C code inside the kernel**

---

## TOPIC‑5: Major and Minor Numbers (Conceptual Only)


- Major number identifies driver
- Minor number identifies device instance
- Used by kernel for routing

Conceptual analogy:

- **Major number**  
    → Which driver should handle this device?

- **Minor number**  
    → Which instance of that device?

Example idea:

- `/dev/led0`, `/dev/led1`
- Same driver, different GPIOs

On BeagleBone Black:

> Often one GPIO device maps to one minor number

No APIs yet — **concept only**.

---

## TOPIC‑6: Device Registration Flow

    Driver loads
    → Registers character device
    → Creates /dev node
    → Ready for user access

Step‑by‑step:

1.  Driver module loads
2.  Character device is registered
3.  Kernel assigns major / minor numbers
4.  `/dev` entry appears
5.  User space can open the device

Critical rule:

> No `/dev` node  
> → No user‑space access

---

## TOPIC‑7: `file_operations` Structure (Core Concept)



- Defines driver interface
- Maps system calls to functions
- Entry point for user space

This is the **heart of a character driver**.

Mental mapping:

- User calls `open()`
- Kernel calls driver’s `open()`

The mapping is defined by the:

    file_operations

This structure connects:

- System calls
- To driver functions

Every character driver in Linux uses this mechanism.

---

## TOPIC‑8: Core File Operations (Role‑Based)

    open()    → access begins
    read()    → data to user space
    write()   → data from user space
    ioctl()   → custom control
    release() → close / cleanup

Practical meaning:

- `open()`  Allocate resources, validate access

- `read()`  Copy data kernel → user

- `write()`  Accept commands user → kernel

- `ioctl()`  Custom operations beyond read/write

- `release()`  Cleanup and resource release

Key reminder:

> The kernel decides **when** these functions run  
> not the driver

---

## TOPIC‑9: GPIO Control from a Character Driver (BBB Context)


- Driver configures GPIO
- Driver controls pin state
- User space triggers behavior

BBB LED control flow:

1.  Device Tree enables GPIO
2.  Driver requests and configures the GPIO
3.  Driver exposes `/dev/bbb_led`
4.  User writes `1` or `0`
5.  Driver toggles GPIO pin

This demonstrates **complete end‑to‑end control**.

---

## TOPIC‑10: Role of Device Tree in Character Drivers

- Device Tree describes hardware
- Driver reads hardware info
- Driver does not hardcode addresses

Clear separation:

- Device Tree says:
    - Which GPIO
    - Which pin
    - Which hardware block

- Driver says:
    - How to control it

Without Device Tree:

> Driver has no idea **what** hardware to manage

This is exactly how **real embedded products** work.

---

## TOPIC‑11: User‑Space Interaction Model

- User writes to /dev node
- Driver interprets input
- Hardware changes state

User‑space example:

```c
write(fd, "1", 1);
```

Driver behavior:

- Parse user data
- Update GPIO state

Result:

- LED turns ON

Key takeaway:

> Applications stay unprivileged  
> Kernel code handles all hardware access

---

## TOPIC‑12: Error Handling and Validation (Critical)


- Never trust user input
- Validate all data
- Protect kernel stability

Kernel mindset:

- User space can be buggy or malicious
- Driver must assume bad input

Driver responsibilities:

- Validate sizes
- Check values
- Protect memory access

Embedded reality:

> One unchecked pointer  
> can crash or brick a device

---

## TOPIC‑13: Common Character Driver Mistakes

- Forgetting cleanup
- Not validating user data
- Blocking forever
- Incorrect GPIO handling

Typical outcomes:

- Missing cleanup → kernel memory leaks
- Invalid pointer usage → kernel panic
- Infinite blocking → system hang
- Wrong GPIO usage → hardware malfunction or damage

Drivers must be:

> Minimal  
> Defensive  
> Predictable

---

## Key Takeaways

- Character drivers expose hardware via /dev
- file_operations defines the interface
- User space never touches hardware directly
- GPIO control is a real driver problem
- Drivers must be defensive and minimal
