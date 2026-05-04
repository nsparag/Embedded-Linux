# Lab‑5: Kernel Module with Parameters (Load‑time Behavior Control)
## `module_param()` | `insmod name=value` | Inspect via `modinfo` and `/sys/module/.../parameters`

---

## Why this lab exists

Real embedded drivers rarely behave in only one fixed way. You often need **load‑time configuration** without recompiling:

- select a GPIO number
- enable/disable debug logs
- set default state (LED on/off)
- choose polling interval
- choose device instance index

Linux provides a clean, standard mechanism for this: **module parameters**.

This lab teaches you:
- how to declare parameters in kernel module
- how to pass parameters at load time
- how to validate what value the kernel accepted
- how to design safe parameter handling (avoid crashing the kernel)

---

## Objective

By the end of this lab you will be able to:

1. Add module parameters to a kernel module using `module_param()`.
2. Pass parameters using `insmod module.ko param=value`.
3. Validate values using:
   - `modinfo`
   - `dmesg`
   - `/sys/module/<module>/parameters/`
4. Handle invalid parameter values safely (defensive kernel programming).
5. Understand when to use load‑time parameters vs `sysfs` vs `ioctl`.

---

## Prerequisites

- Lab‑4 completed (you can build and load `hello.ko`).
- BBB access via serial console (SSH optional).
- You can build a module either:
  - **Track‑A:** build on BBB using `/lib/modules/$(uname -r)/build`
  - **Track‑B:** build on host with matching kernel build tree and copy `.ko` to BBB

> This lab follows the same build track you used for Lab‑4.

---

## Deliverables

In your module folder (`~/param_bbb/`):
- `param_mod.c`
- `Makefile`
- `param_mod.ko`
- Optional logs:
  - `lab5_dmesg_load.txt`
  - `lab5_params_sysfs.txt`

---

# Step‑by‑Step Procedure

---

## Step‑1: Create a new module workspace

On BBB (or host, depending on track):
```bash
mkdir -p ~/param_bbb
cd ~/param_bbb
````

***

## Step‑2: Create the parameterized module source (`param_mod.c`)

Create `param_mod.c`:

```bash
cat > param_mod.c <<'EOF'
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/moduleparam.h>

static int led_default = 0;          // 0=OFF, 1=ON (conceptual for now)
static int debug_enable = 0;         // 0=normal, 1=verbose
static char *owner = (char *)"FDP";  // string parameter

module_param(led_default, int, 0444);
MODULE_PARM_DESC(led_default, "Default LED state (0=OFF, 1=ON)");

module_param(debug_enable, int, 0644);
MODULE_PARM_DESC(debug_enable, "Enable debug logs (0=OFF, 1=ON)");

module_param(owner, charp, 0444);
MODULE_PARM_DESC(owner, "Owner tag printed in logs");

static int __init param_mod_init(void)
{
    // Defensive checks (never trust parameters blindly)
    if (led_default != 0 && led_default != 1) {
        printk(KERN_WARNING "param_mod: invalid led_default=%d, forcing 0\n", led_default);
        led_default = 0;
    }
    if (debug_enable != 0 && debug_enable != 1) {
        printk(KERN_WARNING "param_mod: invalid debug_enable=%d, forcing 0\n", debug_enable);
        debug_enable = 0;
    }

    printk(KERN_INFO "param_mod: init: owner=%s led_default=%d debug_enable=%d\n",
           owner, led_default, debug_enable);

    if (debug_enable) {
        printk(KERN_INFO "param_mod: debug: module loaded successfully\n");
        printk(KERN_INFO "param_mod: debug: use /sys/module/param_mod/parameters/* to inspect\n");
    }

    return 0;
}

static void __exit param_mod_exit(void)
{
    printk(KERN_INFO "param_mod: exit\n");
}

module_init(param_mod_init);
module_exit(param_mod_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("FDP-Participant");
MODULE_DESCRIPTION("Kernel module with parameters (load-time behavior control)");
MODULE_VERSION("1.0");
EOF
```

### Explanation (important)

*   `module_param(name, type, perm)` declares a module parameter.
*   `perm` controls visibility in `/sys/module/<mod>/parameters/`.
    *   `0444` = read‑only
    *   `0644` = readable + writable (root can write)
*   We add **defensive checks** so invalid values do not destabilize kernel code.

> We are not controlling real GPIO here yet — only printing behavior changes.  
> Hardware control comes in Lab‑7 (char driver), but the parameter mechanism stays the same.

***

## Step‑3: Create Makefile for Kbuild (`Makefile`)

```bash
cat > Makefile <<'EOF'
obj-m += param_mod.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
EOF
```

***

## Step‑4: Build the module

### Track‑A (build on BBB)

```bash
make
ls -l *.ko
```

### Track‑B (build on host with matching kernel build tree)

On HOST:

```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
export KDIR=/path/to/bbb-kernel-build
make -C "$KDIR" M="$PWD" modules
```

Copy to BBB:

```bash
scp param_mod.ko debian@<BBB_IP>:/home/debian/param_bbb/
```

On BBB:

```bash
cd ~/param_bbb
ls -l param_mod.ko
```

***

## Step‑5: Inspect module metadata before loading (`modinfo`)

On BBB:

```bash
modinfo ./param_mod.ko
```

Look specifically for the `parm:` lines.

### Expected outcome (representative)

You should see entries similar to:

```text
parm:           led_default:Default LED state (0=OFF, 1=ON)
parm:           debug_enable:Enable debug logs (0=OFF, 1=ON)
parm:           owner:Owner tag printed in logs
```

### Explanation

This is how you confirm the module **exposes** parameters correctly.

***

## Step‑6: Load the module WITHOUT parameters (default behavior)

Capture baseline logs:

```bash
dmesg -T | tail -n 20
```

Load:

```bash
sudo insmod param_mod.ko
```

Check:

```bash
dmesg -T | tail -n 30 | tee ~/lab5_dmesg_load.txt
```

### Expected output (representative)

```text
param_mod: init: owner=FDP led_default=0 debug_enable=0
```

***

## Step‑7: Unload the module

```bash
sudo rmmod param_mod
dmesg -T | tail -n 20
```

Expected:

```text
param_mod: exit
```

***

## Step‑8: Load the module WITH parameters (behavior change at load time)

```bash
sudo insmod param_mod.ko led_default=1 debug_enable=1 owner="COEP-FDP"
```

Check logs:

```bash
dmesg -T | tail -n 40
```

### Expected output (representative)

```text
param_mod: init: owner=COEP-FDP led_default=1 debug_enable=1
param_mod: debug: module loaded successfully
param_mod: debug: use /sys/module/param_mod/parameters/* to inspect
```

### Explanation

This is the key outcome:

*   Same module binary
*   Different behavior based on load‑time configuration

This is exactly how configurable drivers work in real systems.

***

## Step‑9: Verify parameters via sysfs (`/sys/module/.../parameters`)

List parameter files:

```bash
ls -l /sys/module/param_mod/parameters/
```

Read them:

```bash
cat /sys/module/param_mod/parameters/led_default
cat /sys/module/param_mod/parameters/debug_enable
cat /sys/module/param_mod/parameters/owner
```

Save output:

```bash
{
  echo "=== /sys/module/param_mod/parameters ==="
  ls -l /sys/module/param_mod/parameters/
  echo
  echo "led_default=$(cat /sys/module/param_mod/parameters/led_default)"
  echo "debug_enable=$(cat /sys/module/param_mod/parameters/debug_enable)"
  echo "owner=$(cat /sys/module/param_mod/parameters/owner)"
} | tee ~/lab5_params_sysfs.txt
```

### Expected outcome

Values should match what you passed at `insmod`.

### Explanation

This is your **ground truth** for module parameters at runtime.
In production, when someone says “I loaded it with debug=1”, you verify here.

***

## Step‑10 (Optional): Modify writable parameter at runtime (if permitted)

We gave `debug_enable` permission `0644`, so root can change it via sysfs.

Try:

```bash
cat /sys/module/param_mod/parameters/debug_enable
echo 0 | sudo tee /sys/module/param_mod/parameters/debug_enable
cat /sys/module/param_mod/parameters/debug_enable
```

### Expected outcome

`debug_enable` changes from `1` to `0`.

### Explanation

Some parameters can be modified at runtime through sysfs, but:

*   This is not always safe for all parameters
*   Many drivers accept only load‑time parameters

We will discuss the design tradeoff later.

***

# Common Mistakes & Fixes

## 1) `insmod: ERROR: could not insert module: Invalid module format`

**Cause**
Kernel/header mismatch (same as Lab‑4).

**Fix**
Confirm:

```bash
uname -r
modinfo param_mod.ko | grep vermagic
```

Then rebuild using correct kernel build tree.

***

## 2) Parameter not recognized / ignored

**Symptoms**

*   `insmod` fails with “unknown parameter”
*   or the value doesn’t change

**Cause**

*   Typo in parameter name
*   Parameter not declared using `module_param`
*   You’re loading the wrong `.ko`

**Fix**

*   Check `modinfo param_mod.ko | grep parm`
*   Ensure you used the exact parameter names shown there.

***

## 3) Cannot read `/sys/module/param_mod/parameters/...`

**Cause**

*   Module not loaded
*   Permission issue
*   Sysfs not mounted (rare)

**Fix**

```bash
lsmod | grep param_mod
mount | grep sysfs
```

***

## 4) String parameter quoting issues

**Cause**
Shell quoting mistakes.

**Fix**
Use quotes properly:

```bash
sudo insmod param_mod.ko owner="COEP-FDP"
```

or:

```bash
sudo insmod param_mod.ko owner='COEP-FDP'
```

***

# Optional Extension Tasks (Recommended)

## Extension‑A: Trigger defensive checks with invalid values

Unload then load:

```bash
sudo rmmod param_mod
sudo insmod param_mod.ko led_default=99 debug_enable=7 owner="TEST"
dmesg -T | tail -n 30
```

Expected logs include warnings:

```text
param_mod: invalid led_default=99, forcing 0
param_mod: invalid debug_enable=7, forcing 0
```

**What you learn**
Defensive coding prevents kernel instability.

***

## Extension‑B: Add a new parameter (poll interval) and rebuild

Add:

```c
static int poll_ms = 1000;
module_param(poll_ms, int, 0444);
MODULE_PARM_DESC(poll_ms, "Polling interval in milliseconds");
```

Validate via `modinfo` and `/sys/module/.../parameters/`.

***

## Extension‑C: Compare load‑time param vs sysfs vs ioctl (concept)

*   Module parameters: configuration at load time (sometimes runtime)
*   sysfs: runtime tuning exposed by driver
*   ioctl: structured runtime control for user apps

This becomes important when we design the char driver in Lab‑7.

***

# Expected Outcomes Checklist

*   [ ] Built `param_mod.ko`
*   [ ] Loaded with defaults and saw correct `dmesg` output
*   [ ] Loaded with parameters and saw behavior change
*   [ ] Verified parameter values via `/sys/module/param_mod/parameters/`
*   [ ] (Optional) Modified writable param at runtime via sysfs
