# Lab‑1: BeagleBone Black Development Environment Setup
*Serial + Kernel + Headers + Toolchain Sanity*

## Why this lab exists (read this first)
Before we touch cross‑compilation, kernel modules, or drivers, we must **trust the environment**. Most “kernel module failed” or “driver not loading” issues later are not code problems — they are **mismatched kernel/headers**, **wrong console setup**, or **wrong assumptions about the board image**.

This lab establishes:
- A **reliable console** (serial) you can always fall back to (even if networking breaks)
- Exact **kernel version + architecture**
- Whether **kernel headers** are present and match the running kernel
- A baseline check of **toolchains** (target gcc presence and host toolchain readiness)

---

## Objective
By the end of this lab you will be able to:
1. Connect and use **BBB serial console** at 115200 baud.
2. Identify the running **kernel version**, **architecture**, and distro.
3. Verify whether **kernel module build prerequisites** exist on the BBB (headers / build symlink).
4. Capture a **boot log** and key system facts for future labs.

---

## Prerequisites

### Hardware
- BeagleBone Black (BBB)
- micro‑USB cable (data cable, not “charge‑only”)
- Host machine (Ubuntu recommended)
- Optional: Ethernet cable / Wi‑Fi dongle (SSH is optional; serial is mandatory)

### Host software (Ubuntu)
Install these once:
```bash
sudo apt update
sudo apt install -y picocom minicom screen usbutils
````

> You can use **any one** terminal program: `picocom`, `minicom`, or `screen`. I’ll show `picocom` primarily.

***

## What you will produce (deliverables)

Create a folder on host and save outputs:

- `lab1_host_usb_serial.txt` (USB serial detection)
- `lab1_bbb_facts.txt` (kernel, distro, arch, modules path)
- `lab1_bootlog.txt` (boot messages)
- Optional: `lab1_dmesg.txt`

***

NOTE:
This lab does NOT assume any specific Linux image
(Debian, Yocto, or minimal).

The objective is to IDENTIFY and UNDERSTAND
what is currently running on the BeagleBone Black,
not to change it.


## Step‑by‑Step Procedure

***

### Step‑0: Identify the serial device on the host (USB enumeration)

1.  Connect BBB to host using the **mini‑USB** port (near Ethernet).
2.  On the host, run:

```bash
dmesg --follow
```

Now plug/unplug the USB cable and observe new lines.

In another terminal, run:

```bash
dmesg | grep -E "ttyUSB|ttyACM" | tail -n 20
```

#### Expected outcome

You should see something like:

- `/dev/ttyUSB0` **or**
- `/dev/ttyACM0`

Example (typical):

```text
cdc_acm 1-2:1.0: ttyACM0: USB ACM device
```

#### Explanation

BBB exposes a USB serial gadget / UART bridge depending on host drivers and cable/port. You just need the actual device node name.

#### Common mistakes & fixes

- **No new dmesg lines**:
    - Cable may be charge‑only → replace with a **data cable**
    - Try a different USB port
- **Permission denied opening tty**\*:
    - Add your user to `dialout`:
        ```bash
        sudo usermod -aG dialout $USER
        ```
        Log out/in.

***

### Step‑1: Open the serial console (BBB lifeline)

Use 115200 baud (BBB default UART console speed).

#### Using picocom (recommended)

```bash
sudo picocom -b 115200 /dev/ttyACM0
```

(Replace `/dev/ttyACM0` with the device you found.)

#### Expected outcome

- If BBB is already powered, you may see a login prompt.
- If you reset the BBB, you will see boot output.

To reset (optional):

- Tap the **reset** button on BBB (near Ethernet) and watch output.

#### How to exit picocom

Press:

- `Ctrl + A` then `Ctrl + X`

#### Explanation

Serial console works even when:

- Network is down
- RootFS is broken
- Kernel panics
    It is the only way to debug early boot stages later.

#### Common mistakes & fixes

- **Garbled output**: baud rate wrong → ensure `115200`
- **No output**:
    - BBB not powered (some setups need external power; try 5V barrel jack if needed)
    - Ensure you used the correct device node (`ttyACM0` vs `ttyUSB0`)

***

### Step‑2: Login and capture baseline system facts

Once you get a shell prompt, login.
(Default BBB Debian often uses something like `debian` user; but your image may differ. Use the credentials provided by your image provider.)

Now run the following commands **on BBB**.

#### 2.1 Kernel version and architecture

```bash
uname -a
uname -r
uname -m
```

##### Expected outcome

- `uname -m` should show: `armv7l` (BBB is 32‑bit ARMv7)
- `uname -r` shows the exact running kernel release string

##### Explanation

Kernel modules **must match** `uname -r`. We will use this exact string later for headers and module builds.

***

#### 2.2 Distro and image identity

```bash
cat /etc/os-release
```

##### Expected outcome

You should see Debian or a minimal distro identifier.

##### Explanation

This tells you what package manager is available (`apt` on Debian) and what init system you likely have (systemd is common on Debian images).

***

#### 2.3 CPU and board model (BBB confirmation)

```bash
cat /proc/cpuinfo | head -n 30
cat /proc/device-tree/model 2>/dev/null; echo
```

##### Expected outcome

- CPU shows ARMv7 / AM335x family information
- Device tree model prints something like BeagleBone Black model string

##### Explanation

We’ll rely on DT later. This step confirms DT is present and mounted.

***

### Step‑3: Verify kernel module infrastructure (the #1 future failure source)

This is the most important part for upcoming kernel module labs.

#### 3.1 Check module directory exists

```bash
ls -l /lib/modules/$(uname -r)/
```

##### Expected outcome

You should see at least:

- `modules.dep`, `modules.alias`, etc.
    Possibly directories like:
- `kernel/`
- `build -> ...` (symlink)
- `source -> ...` (symlink)

##### Explanation

- `/lib/modules/<version>/` is where the kernel expects modules for that kernel.
- `build` symlink usually points to kernel headers used for compiling external modules.

***

#### 3.2 Check if headers/build link exists

```bash
ls -l /lib/modules/$(uname -r)/build
```

##### Expected outcome (best case)

A symlink or directory exists.

##### If it is missing (common on minimal images)

On Debian, try:

```bash
sudo apt update
sudo apt install -y linux-headers-$(uname -r)
```

If that fails (very common because BBB kernels are sometimes vendor/custom and headers aren’t packaged), do this instead:

```bash
echo "Headers package for this exact kernel may not exist in apt."
echo "We will build modules using kernel source/headers on the HOST later."
```

##### Explanation (important)

For embedded boards, the running kernel is often:

- vendor‑patched
- not packaged in standard distro repositories

So, module building typically happens using:

- the exact kernel source tree used for that image (often on host)
- cross‑compilation (Module‑3 concept)

But even if we build on host later, this step tells us whether the target can build modules locally (nice to have, not mandatory).

***

### Step‑4: Verify target compiler presence (informational check)

Check whether BBB has a native compiler:

```bash
gcc --version
make --version
```

#### Expected outcome

- On Debian full images, `gcc` may be present or installable.
- On minimal images, `gcc` may be missing.

If missing and you want it for quick experiments:

```bash
sudo apt update
sudo apt install -y build-essential
```

#### Explanation

For **real embedded workflows**, we prefer host cross‑compilation, but a native compiler is useful for:

- quick user‑space tests
- small utilities
- sanity checks

***

### Step‑5: Capture boot log (for later debugging labs)

This is a habit: always capture boot logs.

#### 5.1 Capture kernel ring buffer

```bash
dmesg -T | tee ~/lab1_dmesg.txt
```

#### 5.2 Capture key facts into one file

```bash
{
  echo "=== DATE ==="
  date
  echo
  echo "=== UNAME ==="
  uname -a
  echo
  echo "=== OS RELEASE ==="
  cat /etc/os-release
  echo
  echo "=== CPUINFO (first 30 lines) ==="
  cat /proc/cpuinfo | head -n 30
  echo
  echo "=== DEVICE TREE MODEL ==="
  cat /proc/device-tree/model 2>/dev/null; echo
  echo
  echo "=== MODULES DIR ==="
  ls -l /lib/modules/$(uname -r)/ 2>&1
  echo
  echo "=== BUILD LINK ==="
  ls -l /lib/modules/$(uname -r)/build 2>&1
} | tee ~/lab1_bbb_facts.txt
```

#### Expected outcome

Two files appear in your home directory:

- `~/lab1_dmesg.txt`
- `~/lab1_bbb_facts.txt`

#### Explanation

When later labs fail, this file becomes your “truth snapshot”:

- exact kernel version
- whether headers were present
- what image you used

***

## Expected Outcomes Checklist (End of Lab)

By the end, you must be able to say:

- [ ] I can open BBB serial console at **115200** and log in.
- [ ] I know my BBB kernel release: `uname -r = __________`
- [ ] I confirmed architecture: `uname -m = armv7l`
- [ ] I checked `/lib/modules/<version>/` exists.
- [ ] I verified whether `/lib/modules/<version>/build` exists or not.
- [ ] I saved `lab1_bbb_facts.txt` and `lab1_dmesg.txt`.

***

## Common Mistakes & Fixes (Quick Reference)

### 1) “No serial device appears on host”

**Fix**

- Use a data USB cable
- Try different port
- Run:
    ```bash
    lsusb
    dmesg | tail -n 50
    ```

### 2) “Permission denied opening /dev/ttyACM0”

**Fix**

```bash
sudo usermod -aG dialout $USER
```

Log out/in (or reboot host).

### 3) “Headers package not found”

**Reality**

- This is common on BBB vendor kernels.
    **Fix**
- Accept that module builds will be done using kernel source on host in later labs.

### 4) “Garbled output on serial”

**Fix**

- Ensure `115200`
- Ensure correct port `/dev/ttyACM0` vs `/dev/ttyUSB0`

***

## Optional Extension Tasks (For deeper understanding)

### A) Determine boot source (eMMC vs SD)

```bash
lsblk
```

Look for:

- `mmcblk0` / `mmcblk1` roles (varies by boot method).

### B) Identify loaded device tree

```bash
tr -d '\0' < /proc/device-tree/model; echo
```

### C) Check LED sysfs availability (preview for later)

```bash
ls /sys/class/leds/
```

You should see:

- `beaglebone:green:usr0` … `usr3` (typical)

***

```
