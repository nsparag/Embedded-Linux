# MODULE‑1 
# Embedded Linux Architecture – BeagleBone Black View

----------------

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

## BeagleBone Black as Reference Platform


* TI AM335x SoC
* ARM Cortex‑A8 CPU
* DDR controller
* GPIO, UART, I2C, SPI
* On‑chip Boot ROM
* Industrial‑class architecture
* Representative embedded platform

Learning BBB prepares engineers for real products, not demos.

## Embedded Linux Layered Architecture




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
