# Lab‑4: Kernel Module – “Hello BBB”
*Build a Loadable Kernel Module (LKM) | `insmod`/`rmmod`/`lsmod` | `dmesg` Analysis*

---

## Why this lab exists
This is your **first controlled entry into kernel space**.

A kernel module is the safest way to:
- run your own code **inside the kernel**
- test kernel logging (`printk`)
- understand the **load → init → unload → exit** lifecycle

In industry, module skills are essential because:
- most drivers are built as LKMs during development
- you troubleshoot by loading/unloading and inspecting logs
- “Invalid module format” and “vermagic mismatch” are everyday problems

This lab is intentionally simple: **no hardware access yet**.  
We are only proving: **“I can build and run kernel code correctly on BBB.”**

---

## Objective
By the end of this lab you will be able to:
1. Verify the module build prerequisites for the running kernel.
2. Build a kernel module (`.ko`) using the kernel build system (Kbuild).
3. Load the module using `insmod` and validate it using `lsmod` and `dmesg`.
4. Unload the module using `rmmod` and verify cleanup.
5. Diagnose the most common module build/load failures.

---

## Prerequisites
- BBB is booted (any Linux image is okay).
- Serial console available (SSH optional).
- From Lab‑1 you know:
  - `uname -r` (running kernel release)
  - Whether `/lib/modules/$(uname -r)/build` exists

> **Important:** Module compilation requires the **exact kernel build headers/tree** matching the running kernel.  
> If headers are missing on BBB, you can still complete this lab by building on the **host** using the correct kernel source (Track‑B).

---

## Deliverables
You will create:
- `hello_bbb/hello.c`
- `hello_bbb/Makefile`
- `hello.ko`
- Optional: `lab4_dmesg_before.txt`, `lab4_dmesg_after.txt`

---

## Choose Your Build Track (Do this first)
There are two valid industry workflows:

### Track‑A (Preferred if available): Build on BBB (native)
Use this if:
- `/lib/modules/$(uname -r)/build` exists on BBB

### Track‑B (Fallback / also common in industry): Build on HOST (cross) and copy `.ko` to BBB
Use this if:
- BBB does not have matching headers
- or you want the standard embedded workflow

This lab provides both. Pick one and follow it fully.

---

# Step‑by‑Step Procedure

---

## Step‑1: Capture kernel version and module build status (BBB)
On BBB:
```bash
uname -r
uname -m
````

Expected:

- `uname -m` should be `armv7l` on BBB (typical)

Now check module build directory:

```bash
ls -l /lib/modules/$(uname -r)/
ls -l /lib/modules/$(uname -r)/build 2>/dev/null || echo "NO build link (headers missing)"
```

### Expected outcomes

- If `build` exists → you can do **Track‑A**
- If missing → do **Track‑B**

### Explanation

Kernel modules must match:

- kernel release (`uname -r`)
- architecture (`armv7l`)
- build configuration (“vermagic”)

***

## Step‑2: Create a clean working directory

Do this either on BBB (Track‑A) or host (Track‑B).

```bash
mkdir -p ~/hello_bbb
cd ~/hello_bbb
```

***

## Step‑3: Write the module source (`hello.c`)

Create `hello.c`:

```bash
cat > hello.c <<'EOF'
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int __init hello_init(void)
{
    printk(KERN_INFO "hello_bbb: init: Hello from BBB kernel module!\n");
    return 0;
}

static void __exit hello_exit(void)
{
    printk(KERN_INFO "hello_bbb: exit: Goodbye from BBB kernel module!\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("FDP-Participant");
MODULE_DESCRIPTION("Hello BBB - first kernel module");
MODULE_VERSION("1.0");
EOF
```

### Explanation (important)

- `module_init()` and `module_exit()` declare entry/exit points.
- `printk()` is kernel logging (no `printf` in kernel).
- `MODULE_LICENSE("GPL")` avoids “tainted kernel” warnings for many setups and is a best practice.

***

## Step‑4: Write the Kbuild Makefile (`Makefile`)

Create `Makefile`:

```bash
cat > Makefile <<'EOF'
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
EOF
```

### Explanation

This is the standard external module build pattern:

- `obj-m += hello.o` tells Kbuild to build `hello.ko`
- `-C .../build` points to kernel build tree/headers
- `M=$(PWD)` tells Kbuild: “module sources are here”

***

# Track‑A: Build the module on BBB (native)

## Step‑5A: Build

On BBB inside `~/hello_bbb`:

```bash
make
```

### Expected output (representative)

You should see lines referencing:

- `CC [M]  /home/.../hello.o`
- `LD [M]  /home/.../hello.ko`

### Verify output file

```bash
ls -l *.ko
file hello.ko
```

Expected:

- `hello.ko` exists
- `file hello.ko` indicates it’s an ELF object for ARM (not x86)

***

# Track‑B: Build on HOST (cross compile) and copy `.ko` to BBB

> Use Track‑B if BBB headers are missing or you prefer standard embedded workflow.

## What you need for Track‑B

You need a kernel build tree matching BBB kernel:

- Typically: the exact vendor kernel source or headers used to build that BBB image

### Minimal rule

> **The kernel build directory used to compile the module must match the kernel running on BBB.**

If you already have the kernel source tree for your BBB image, set:

- `KDIR=/path/to/bbb-kernel-build`

## Step‑5B: Build on host using cross compiler

On HOST:

```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
export KDIR=/path/to/bbb-kernel-build
```

Build:

```bash
make -C "$KDIR" M="$PWD" modules
```

Verify:

```bash
ls -l hello.ko
file hello.ko
```

## Step‑6B: Copy module to BBB

```bash
scp hello.ko debian@<BBB_IP>:/home/debian/
```

(or use USB copy method if no network)

On BBB:

```bash
mkdir -p ~/hello_bbb
mv ~/hello.ko ~/hello_bbb/
cd ~/hello_bbb
```

***

# Common Steps (After you have `hello.ko` on BBB)

## Step‑6: Capture baseline kernel log

On BBB:

```bash
dmesg -T | tail -n 20 | tee ~/lab4_dmesg_before.txt
```

***

## Step‑7: Insert module (`insmod`)

On BBB in `~/hello_bbb`:

```bash
sudo insmod hello.ko
```

### Expected outcome

No output on success.

### Verify module is loaded

```bash
lsmod | grep hello || echo "hello not found in lsmod"
```

Expected:

- One line showing module name `hello` and size.

***

## Step‑8: Check kernel messages (`dmesg`)

```bash
dmesg -T | tail -n 30 | tee ~/lab4_dmesg_after.txt
```

### Expected log line (similar)

```text
hello_bbb: init: Hello from BBB kernel module!
```

### Explanation

`printk()` goes to the kernel ring buffer. `dmesg` reads it.
This is your primary kernel debugging tool for the entire FDP.

***

## Step‑9: Remove module (`rmmod`)

```bash
sudo rmmod hello
```

Verify it’s gone:

```bash
lsmod | grep hello || echo "hello module unloaded"
```

Check `dmesg` for exit message:

```bash
dmesg -T | tail -n 30
```

Expected:

```text
hello_bbb: exit: Goodbye from BBB kernel module!
```

***

## Step‑10: Clean build artifacts (optional)

In your module directory:

```bash
make clean
```

***

# Expected Outcomes Checklist

At the end of this lab, you should be able to confirm:

- [ ] `hello.ko` built successfully
- [ ] `sudo insmod hello.ko` works without errors
- [ ] `lsmod` shows the module loaded
- [ ] `dmesg` shows your init message
- [ ] `sudo rmmod hello` unloads it successfully
- [ ] `dmesg` shows your exit message

***

# Common Mistakes & Fixes (Real World)

## 1) `insmod: ERROR: could not insert module hello.ko: Invalid module format`

### Cause (most common)

- Module built against headers that **do not match** the running kernel (`uname -r`).
- “vermagic” mismatch.

### How to confirm

Run:

```bash
modinfo hello.ko | grep vermagic
uname -r
```

If these differ → mismatch.

### Fix

- Build module using the correct kernel build tree matching BBB kernel.
- If using Track‑B, ensure `KDIR` is the correct kernel build directory for that exact kernel.

***

## 2) `make: *** /lib/modules/<ver>/build: No such file or directory`

### Cause

Kernel headers/build tree is missing on BBB.

### Fix

- Use **Track‑B** (build on host with correct kernel source).
- Or attempt install headers (works only if packaged):

```bash
sudo apt update
sudo apt install -y linux-headers-$(uname -r)
```

***

## 3) Module loads but no `printk` output appears

### Causes

- You didn’t check the right part of `dmesg`
- Kernel log level filtering

### Fix

```bash
dmesg -T | tail -n 200 | grep hello_bbb
```

***

## 4) `rmmod: ERROR: Module hello is in use`

### Cause

Module reference count not zero (more common in real drivers).

### Fix

- Ensure no process is holding it open (not relevant for this simple lab).
- In later labs, you’ll use `lsof` (if available) or check open handles.

***

## 5) `Operation not permitted`

### Cause

You didn’t use sudo or kernel lockdown policy (rare on BBB).

### Fix

```bash
sudo insmod hello.ko
```

***

# Optional Extension Tasks (Recommended for deeper understanding)

## Extension‑A: Use `modinfo`

```bash
modinfo hello.ko
```

Observe:

- description, author, license, version

## Extension‑B: Use `pr_info()` macros (modern style)

Replace `printk(KERN_INFO ...)` with:

```c
pr_info("hello_bbb: ...\n");
```

Rebuild and retest.  
(Keep `#include <linux/printk.h>` if needed depending on kernel.)

## Extension‑C: Show kernel taint status

After loading:

```bash
cat /proc/sys/kernel/tainted
```

Discuss what “taint” means in production debugging contexts.
