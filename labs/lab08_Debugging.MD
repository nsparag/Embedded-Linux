# Lab‑8: Debugging Kernel & Drivers on BeagleBone Black (BBB)
## printk levels | dmesg discipline | /proc + /sys evidence | failure scenarios + fixes (field‑style)

---

## Why this lab exists (industry reality)

In production, you rarely get a nice reproducible bug with a debugger attached. What you usually get is:

- “LED doesn’t toggle anymore”
- “Device node missing”
- “Driver loads but hardware silent”
- “Works once, then hangs”
- “After reboot it’s fine” (most dangerous statement)

Kernel/driver debugging on embedded systems is about **collecting evidence**, **isolating the failing layer**, and **changing one thing at a time**.

This lab teaches a **repeatable debugging playbook** that you will reuse for:
- module load failures
- Device Tree mismatch
- GPIO conflicts
- `/dev` node issues
- user ↔ kernel interaction errors

We will use the driver from **Lab‑7** (`bbb_gpioled.ko`) as the primary specimen.

---

## Objective

By the end of this lab you will be able to:

1. Use kernel logging correctly:
   - `printk` levels
   - `dmesg` filtering & timestamping
2. Collect proof from:
   - `/proc` and `/sys`
3. Diagnose and fix common failure scenarios:
   - DT node not applied
   - probe not called
   - GPIO busy
   - `/dev` node missing
   - permissions problems
   - “Invalid module format”
4. Apply a **layer‑based debugging flow**:
   Boot → Kernel → DT → Driver → `/dev` → User app

---

## Prerequisites

- Labs 1–7 completed
- Serial console available (**mandatory**)
- `bbb_gpioled.ko` and `bbb_led_test` from Lab‑7 available on BBB
- `sudo` privileges

---

## Deliverables (save these logs)

Create a folder:
```bash
mkdir -p ~/lab8
````

You will produce:

*   `~/lab8/dmesg_baseline.txt`
*   `~/lab8/dmesg_after_insmod.txt`
*   `~/lab8/proc_sys_evidence.txt`
*   `~/lab8/failure_case_notes.txt` (your observations)

***

# Part‑0: The Debugging Mindset (do this every time)

## The Golden Rule

> **Never “fix” before you can reproduce and measure.**

## The Layer Checklist (print this in your head)

1.  Board booted? (serial shows login)
2.  Kernel ok? (`dmesg` has no panic)
3.  DT node exists? (`/proc/device-tree/...`)
4.  Driver loaded? (`lsmod`, `dmesg`)
5.  Device node exists? (`/dev/...` or `mknod`)
6.  App call succeeded? (`open/write` return values)
7.  Hardware wiring ok? (physical check)

***

# Part‑1: Logging Discipline (`printk` + `dmesg`)

## Step‑1: Capture baseline kernel log (before touching anything)

```bash
dmesg -T | tail -n 200 | tee ~/lab8/dmesg_baseline.txt
```

### Expected outcome

*   You have a baseline snapshot to compare after each action.

### Explanation

In real debugging, “what changed” matters. Baseline logs provide context.

***

## Step‑2: Know your kernel log level (why some messages don’t show)

Check current printk console level:

```bash
cat /proc/sys/kernel/printk
```

Typical output format:

```text
<console_loglevel> <default_message_loglevel> <minimum_console_loglevel> <default_console_loglevel>
```

### Temporary increase verbosity (optional)

To show more logs on console:

```bash
echo 8 | sudo tee /proc/sys/kernel/printk
```

> Keep this temporary. Over‑logging can hide important signals.

***

## Step‑3: Understand printk levels (practical subset)

You’ll most commonly use:

*   `KERN_ERR`     → serious failures (probe failed, resource missing)
*   `KERN_WARNING` → suspicious but survivable
*   `KERN_INFO`    → normal progress messages
*   `KERN_DEBUG`   → very verbose (use sparingly)

**Why this matters:** In the field, you want logs that help isolate quickly, not floods.

***

# Part‑2: Evidence Collection from `/sys` and `/proc`

## Step‑4: Collect a “system evidence bundle”

Run this once (and reuse on every bug report):

```bash
{
  echo "=== DATE ==="; date; echo
  echo "=== UNAME ==="; uname -a; echo
  echo "=== CMDLINE ==="; cat /proc/cmdline; echo
  echo "=== DT MODEL ==="; tr -d '\0' </proc/device-tree/model 2>/dev/null; echo; echo
  echo "=== MODULES (hello/bbb) ==="; lsmod | grep -E "bbb_gpioled|hello|param_mod" || true; echo
  echo "=== DEVICES (/dev) ==="; ls -l /dev/bbb_led 2>/dev/null || echo "/dev/bbb_led missing"; echo
  echo "=== PROC DEVICES (bbb_led major?) ==="; grep -w bbb_led /proc/devices || echo "bbb_led not registered"; echo
  echo "=== SYSFS MODULE PARAMS (if loaded) ==="
  [ -d /sys/module/bbb_gpioled ] && ls -l /sys/module/bbb_gpioled || echo "bbb_gpioled sysfs node missing"
  echo
  echo "=== DT NODE (fdp_gpioled) ==="
  ls /proc/device-tree/fdp_gpioled 2>/dev/null && echo "DT node present" || echo "DT node NOT present"
} | tee ~/lab8/proc_sys_evidence.txt
```

### Explanation

This “bundle” becomes your standard diagnostic artifact. It captures:

*   kernel identity
*   boot cmdline
*   DT presence
*   driver state
*   device node existence

***

# Part‑3: Debugging the Lab‑7 Driver (Happy Path First)

## Step‑5: Verify DT node exists (this decides 50% of driver bugs)

```bash
ls /proc/device-tree/fdp_gpioled 2>/dev/null && echo "DT OK" || echo "DT NOT APPLIED"
```

If present, verify label:

```bash
tr -d '\0' </proc/device-tree/fdp_gpioled/label 2>/dev/null; echo
```

***

## Step‑6: Load driver and read dmesg immediately (tight loop)

```bash
cd ~/lab7/driver
sudo insmod bbb_gpioled.ko
dmesg -T | tail -n 80 | tee ~/lab8/dmesg_after_insmod.txt
```

### Expected (representative)

You should see:

*   “probed OK… /dev/bbb\_led major=…”

***

## Step‑7: Confirm `/dev` node existence

```bash
ls -l /dev/bbb_led || echo "/dev/bbb_led missing"
```

If missing, read major number:

```bash
grep -w bbb_led /proc/devices || echo "bbb_led not in /proc/devices"
```

If major is, say, `240`, create node:

```bash
sudo mknod /dev/bbb_led c 240 0
sudo chmod 666 /dev/bbb_led
ls -l /dev/bbb_led
```

***

## Step‑8: Run user test and correlate logs

```bash
cd ~/lab7/user
./bbb_led_test
dmesg -T | tail -n 80 | grep -i bbb_led || true
```

### Explanation

You are correlating:

*   user action (write “1/0”)
*   kernel evidence (`dmesg` “set=1/0”)
*   physical LED response

This triangulation is the core embedded debug technique.

***

# Part‑4: Failure Scenarios (Practice Real Bugs + Fixes)

> For each scenario:
>
> 1.  observe symptom
> 2.  collect evidence
> 3.  locate failing layer
> 4.  apply minimal fix
> 5.  retest

Create a notes file:

```bash
nano ~/lab8/failure_case_notes.txt
```

***

## Failure‑1: “DT node not present” (Driver never probes)

### Symptom

*   `insmod` succeeds, but no “probed OK”
*   or driver cannot find GPIO property
*   `/proc/device-tree/fdp_gpioled` missing

### Evidence

```bash
ls /proc/device-tree/fdp_gpioled 2>/dev/null || echo "DT node missing"
dmesg -T | tail -n 100 | grep -iE "bbb|of|dt|gpio" || true
```

### Root cause (most common)

*   You edited/replaced a DTB that is **not the one used by bootloader**
*   DTB not deployed correctly

### Fix

*   Re‑identify DTB in use (Lab‑6 process)
*   Re‑deploy the correct DTB
*   Reboot
*   Confirm node exists under `/proc/device-tree`

**Minimal immediate rollback**

```bash
export DTB=/boot/dtbs/$(uname -r)/am335x-boneblack.dtb   # adjust if different
sudo cp -av ~/lab7/dt/bbb.dtb.backup "$DTB"
sync
sudo reboot
```

***

## Failure‑2: “GPIO busy / resource conflict”

### Symptom

*   Probe fails, often with messages indicating resource unavailable
*   LED doesn’t toggle, driver logs show failure

### Evidence

```bash
dmesg -T | tail -n 120 | grep -iE "gpio|busy|resource|failed" || true
```

### Root cause

*   Another driver or subsystem already claimed that GPIO line
    (very common if you pick a pin used by HDMI, eMMC, etc.)

### Fix options

1.  **Choose a different header GPIO** (recommended)
    *   Update DT node `led-gpios = <&gpioX Y flags>;`
    *   Update pinmux accordingly
2.  Advanced (not recommended in first FDP run):
    *   disable conflicting device in DT overlays

***

## Failure‑3: `/dev/bbb_led` missing (udev not present)

### Symptom

*   Driver probes OK
*   But `/dev/bbb_led` does not exist

### Evidence

```bash
dmesg -T | tail -n 80 | grep -i "probed OK" || true
grep -w bbb_led /proc/devices || true
```

### Root cause

*   Minimal RootFS without udev/mdev auto node creation

### Fix

Manually create node:

```bash
MAJOR=$(grep -w bbb_led /proc/devices | awk '{print $1}')
echo "MAJOR=$MAJOR"
sudo mknod /dev/bbb_led c "$MAJOR" 0
sudo chmod 666 /dev/bbb_led
```

***

## Failure‑4: “Permission denied” accessing `/dev/bbb_led`

### Symptom

*   `open("/dev/bbb_led")` fails in user program

### Evidence

```bash
ls -l /dev/bbb_led
id
```

### Root cause

*   node created with restrictive permissions

### Fix

For lab purposes:

```bash
sudo chmod 666 /dev/bbb_led
```

**Production note**
In real products, you’d use:

*   udev rules
*   group permissions
*   capabilities
    Not `666`.

***

## Failure‑5: `insmod: Invalid module format`

### Symptom

*   Module fails to insert

### Evidence

```bash
uname -r
modinfo ~/lab7/driver/bbb_gpioled.ko | grep vermagic
```

### Root cause

*   module built against different kernel headers/config than the running kernel

### Fix

*   rebuild module using the exact kernel build tree matching `uname -r`
*   never mix vendor kernels with generic headers

***

## Failure‑6: “Driver loads, /dev exists, but LED doesn’t light”

### Symptom

*   Logs show `set=1`
*   No physical LED response

### Evidence

1.  Verify user action reaches kernel:

```bash
dmesg -T | tail -n 80 | grep -i "bbb_led: set" || echo "No set logs => user not reaching driver"
```

2.  Confirm DT node content (GPIO mapping sanity):

```bash
# If 'led-gpios' exists, it may not be easily printable; but DT presence matters:
ls /proc/device-tree/fdp_gpioled
```

### Root causes (most common)

*   wiring wrong (LED reversed / no resistor / wrong pin)
*   wrong GPIO mapping for that header pin
*   pinmux not applied / wrong pinctrl style for your DT

### Fix (minimal)

*   Recheck wiring: P9\_12 → resistor → LED → GND
*   Try a different known GPIO pin and update DT accordingly
*   If your image provides `config-pin`:

```bash
config-pin -q P9_12
```

***

# Part‑5: “Field Debug Script” (Optional but highly useful)

When debugging many boards, you want a standard script.
Create:

```bash
cat > ~/lab8/collect_debug_bundle.sh <<'EOF'
#!/bin/sh
OUT=~/lab8/bundle_$(date +%Y%m%d_%H%M%S).txt
{
  echo "=== DATE ==="; date; echo
  echo "=== UNAME ==="; uname -a; echo
  echo "=== CMDLINE ==="; cat /proc/cmdline; echo
  echo "=== MODEL ==="; tr -d '\0' </proc/device-tree/model 2>/dev/null; echo; echo
  echo "=== DMESG (last 200) ==="; dmesg -T | tail -n 200; echo
  echo "=== LSMOD ==="; lsmod; echo
  echo "=== /proc/devices (grep bbb_led) ==="; grep -w bbb_led /proc/devices || true; echo
  echo "=== /dev node ==="; ls -l /dev/bbb_led 2>/dev/null || echo "missing"; echo
  echo "=== DT node ==="; ls /proc/device-tree/fdp_gpioled 2>/dev/null && echo "present" || echo "missing"
} > "$OUT"
echo "Saved: $OUT"
EOF

chmod +x ~/lab8/collect_debug_bundle.sh
~/lab8/collect_debug_bundle.sh
```

***

# Expected Outcomes Checklist

By the end of this lab you should be able to:

*   [ ] Capture baseline `dmesg` and compare after actions
*   [ ] Interpret missing logs vs filtered logs using `/proc/sys/kernel/printk`
*   [ ] Collect evidence from `/proc` and `/sys` to locate failing layer
*   [ ] Diagnose and fix:
    *   DT node missing
    *   GPIO busy
    *   `/dev` node missing
    *   permission denied
    *   invalid module format
*   [ ] Correlate user actions with kernel logs and physical behavior

***

# Common Mistakes (Debugging anti‑patterns)

1.  **Changing DT + driver + user app at the same time**
    *   You lose causality.
2.  **Rebooting before saving logs**
    *   You destroy the evidence.
3.  **Assuming “driver bug”**
    *   Most failures are DT/pinmux/wiring.
4.  **Not using serial console**
    *   You’re blind to early failures.

***

## Completion Note (what you now have)

After Lab‑8, you have the complete embedded Linux development cycle:

*   inspect board environment (Lab‑1)
*   cross‑compile user apps (Lab‑2)
*   understand RootFS/runtime (Lab‑3)
*   enter kernel via modules (Lab‑4/5)
*   modify DT safely (Lab‑6)
*   write a real char driver bound via DT (Lab‑7)
*   debug like an embedded engineer (Lab‑8)
