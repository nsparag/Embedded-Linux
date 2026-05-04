# Lab‑6: Device Tree Basics on BeagleBone Black (BBB)
## Locate DTS/DTSI | Pinmux Basics | Modify a Simple Node | Rebuild DTB | Verify from Linux

---

## Why this lab exists

On ARM embedded boards (including **TI AM335x / BBB**), Linux typically **cannot auto‑discover** hardware like a PC. Instead, the kernel relies on **Device Tree (DT)** to know:

- what peripherals exist (GPIO/I2C/SPI/UART)
- which pins are used and how they’re muxed (pinmux)
- which drivers should bind (via `compatible`)

This lab teaches the **minimum correct workflow** used in real products:

> Identify the running DTB → safely modify a simple node → rebuild DTB → deploy → verify in Linux → recover if something goes wrong.

We will keep the modification **safe and reversible** (no risky pinmux/peripheral enable changes yet).

---

## Objective

By the end of this lab you will be able to:

1. Identify which **DTB** your BBB is currently using.
2. Locate DT artifacts on the running system (`/boot` / `/proc/device-tree`).
3. Decompile DTB → edit a simple node → compile DTB back.
4. Deploy the new DTB **safely** (with backup + rollback method).
5. Verify the change from Linux using `/proc/device-tree` and sysfs.

---

## Prerequisites

- BBB booted into Linux (Debian or minimal is OK)
- Serial console available (**mandatory** for recovery)
- `sudo` access on BBB
- Tools on BBB:
  - `dtc` (Device Tree Compiler)
  - basic editor (`nano` or `vi`)

### Install dtc if missing (Debian)
```bash
sudo apt update
sudo apt install -y device-tree-compiler
````

> If `apt` is not available (minimal image), use the host build path later or install `dtc` via your image’s package method.

***

## Safety Rules (READ BEFORE YOU START)

1.  **Backup first. Always.**
2.  Only modify a **safe node** in this first DT lab (LED trigger change).
3.  Use **serial console** for recovery.
4.  If the board fails to boot, you must be able to revert the DTB.

***

## Deliverables (files you will create)

On BBB:

*   `~/lab6/dtb_active_path.txt`
*   `~/lab6/bbb.dtb.backup`
*   `~/lab6/bbb.dts` (decompiled source)
*   `~/lab6/bbb_modified.dts`
*   `~/lab6/bbb_modified.dtb`

***

# Step‑by‑Step Procedure

***

## Step‑1: Confirm you are on BBB and capture kernel + model info

```bash
uname -r
uname -m
tr -d '\0' </proc/device-tree/model; echo
```

### Expected outcome

*   `uname -m` → `armv7l` (typical BBB)
*   model string prints something like “BeagleBone Black …”

### Explanation

This confirms:

*   device tree is present (`/proc/device-tree`)
*   kernel is running normally

***

## Step‑2: Identify the active DTB (the one the kernel actually used)

### 2A) Check kernel command line (often contains dtb hints)

```bash
cat /proc/cmdline
```

### 2B) Common DTB locations to inspect

On Debian-style BBB images, DTBs are usually under `/boot/dtbs/<kernel-version>/` or `/boot/`.
Try:

```bash
ls -l /boot
ls -l /boot/dtbs 2>/dev/null || true
ls -l /boot/dtbs/$(uname -r) 2>/dev/null || true
```

### 2C) Find DTB files

```bash
sudo find /boot -maxdepth 4 -type f -name "*.dtb" | head -n 30
```

### 2D) (Optional but useful) Identify FDT file configured by U‑Boot

Check for common U‑Boot environment references:

```bash
sudo fw_printenv 2>/dev/null | grep -E "fdtfile|fdt_file|dtb" || echo "fw_printenv not available or env not accessible"
```

> If `fw_printenv` is unavailable, don’t worry. We will still proceed using DTB we can locate in `/boot`.

### Record what you believe is the active DTB path

Create a working folder:

```bash
mkdir -p ~/lab6
```

Save a note:

```bash
echo "Candidate DTB paths found under /boot (inspect and choose one):" | tee ~/lab6/dtb_active_path.txt
sudo find /boot -maxdepth 4 -type f -name "*.dtb" | tee -a ~/lab6/dtb_active_path.txt
```

***

## Step‑3: Choose a DTB to modify (safe choice)

Pick the DTB that matches BBB (often contains `am335x` + `boneblack` in name).

Example (names vary by image):

*   `am335x-boneblack.dtb`
*   `am335x-boneblack-wireless.dtb`
*   vendor-specific names

Let’s store the chosen DTB path in a variable (replace with your actual file):

```bash
export DTB=/boot/dtbs/$(uname -r)/am335x-boneblack.dtb
echo "Using DTB: $DTB"
ls -l "$DTB"
```

### Common mistake

If that exact file doesn’t exist, use the path you discovered in Step‑2.

***

## Step‑4: Backup the DTB (non-negotiable)

```bash
cp -av "$DTB" ~/lab6/bbb.dtb.backup
ls -l ~/lab6/bbb.dtb.backup
```

### Explanation

If anything goes wrong, you can restore the original DTB quickly.

***

## Step‑5: Decompile DTB → DTS (readable source)

```bash
dtc -I dtb -O dts -o ~/lab6/bbb.dts "$DTB"
ls -l ~/lab6/bbb.dts
```

### Expected outcome

*   `bbb.dts` created successfully

### Explanation

*   `.dtb` is the binary blob the kernel uses
*   `.dts` is a source representation you can read/edit
*   Decompiling is common in field debugging when you don’t have original source tree

***

## Step‑6: Make a SAFE modification (LED trigger change)

### Why LED node is a safe target

*   It does not affect boot-critical peripherals (MMC, UART, DDR)
*   If misconfigured, worst case is **LED behavior changes**, not boot failure

### 6A) Find the LED nodes in the DTS

```bash
grep -n "leds" -n ~/lab6/bbb.dts | head -n 20
grep -n "beaglebone:green:usr" ~/lab6/bbb.dts | head -n 20
```

You should see a section similar to (structure may vary):

*   `leds { ... }`
*   nodes for `usr0/usr1/usr2/usr3`

### 6B) Copy original DTS to modified file

```bash
cp -av ~/lab6/bbb.dts ~/lab6/bbb_modified.dts
```

### 6C) Edit `bbb_modified.dts`

Use `nano` or `vi`:

```bash
nano ~/lab6/bbb_modified.dts
```

#### What to change

Inside the `leds` node, locate the node for `beaglebone:green:usr0` (or `usr0`) and change/add:

*   Set default trigger to `heartbeat` (or `mmc0` etc.)
*   Optionally set default state

Example snippet you want conceptually (do not copy blindly; align with your DTS node names):

```dts
linux,default-trigger = "heartbeat";
default-state = "on";
```

**Recommended safe change (do only this):**

*   Change `linux,default-trigger` of `usr0` to `"heartbeat"`

Save and exit.

### Explanation

*   The LED subsystem reads DT properties to decide default triggers/states.
*   You are practicing the DT change + rebuild + deploy loop without risking boot.

***

## Step‑7: Recompile modified DTS → DTB

```bash
dtc -I dts -O dtb -o ~/lab6/bbb_modified.dtb ~/lab6/bbb_modified.dts
ls -l ~/lab6/bbb_modified.dtb
file ~/lab6/bbb_modified.dtb
```

### Expected outcome

*   `bbb_modified.dtb` exists
*   `file` shows “Device Tree Blob”

### Common mistakes & fixes

*   **dtc errors like “syntax error”**:
    *   You likely broke DTS braces `{}` or semicolons `;`
    *   Fix by reopening file and ensuring:
        *   every property ends with `;`
        *   braces are balanced

***

## Step‑8: Deploy the modified DTB (safe deployment method)

### Strategy: Replace DTB with backup available

You will copy the modified DTB over the active DTB file.

**Important**: Keep the backup you created in `~/lab6/`.

```bash
sudo cp -av "$DTB" "${DTB}.prelab6"
sudo cp -av ~/lab6/bbb_modified.dtb "$DTB"
sync
```

### Explanation

*   `${DTB}.prelab6` gives you an extra backup in `/boot`
*   `sync` flushes filesystem buffers to avoid corruption

***

## Step‑9: Reboot and verify change

```bash
sudo reboot
```

Watch boot on **serial console**.

After login, verify LED behavior:

*   `usr0` should now show heartbeat blinking (if your DT change applied)

### Verify using sysfs (Linux view)

```bash
ls /sys/class/leds/
cat /sys/class/leds/beaglebone\:green\:usr0/trigger
```

### Expected outcome

The `trigger` file shows available triggers with the selected one in brackets:

```text
none rc-feedback [heartbeat] mmc0 mmc1 timer ...
```

### Also verify via `/proc/device-tree` (DT view)

Find your LED node path (varies). A simple approach:

```bash
find /proc/device-tree -name "*usr0*" 2>/dev/null | head -n 20
```

Then inspect a property:

```bash
# Example: if you find a node path, try reading the trigger property (null-terminated)
NODE=$(find /proc/device-tree -name "*usr0*" 2>/dev/null | head -n 1)
echo "NODE=$NODE"
tr -d '\0' <"$NODE/linux,default-trigger" ; echo
```

> If the node path differs, adjust accordingly. The goal is to prove DT data is visible through `/proc/device-tree`.

***

# Expected Outcomes Checklist

*   [ ] Identified DTB files on BBB under `/boot`
*   [ ] Backed up the active DTB
*   [ ] Decompiled DTB → DTS using `dtc`
*   [ ] Modified a safe node (LED trigger)
*   [ ] Recompiled DTS → DTB successfully
*   [ ] Deployed DTB and rebooted
*   [ ] Verified LED trigger changed via `/sys/class/leds/.../trigger`

***

# Recovery / Rollback (If BBB fails to boot or behaves oddly)

## Recovery‑1: Restore DTB from serial console (if you still reach a shell)

If system boots but hardware is weird, restore:

```bash
sudo cp -av ~/lab6/bbb.dtb.backup "$DTB"
sync
sudo reboot
```

## Recovery‑2: Restore using `/boot` backup

```bash
sudo cp -av "${DTB}.prelab6" "$DTB"
sync
sudo reboot
```

## Recovery‑3: If system does not boot to login

You must use **U‑Boot** and boot from a known good SD card image, then restore `/boot` DTB from there.

> This is why serial console is mandatory.

***

# Common Mistakes & Fixes

## 1) “I edited DTS but nothing changed after reboot”

**Cause**

*   You rebuilt DTB but did not deploy it to the correct DTB path used by bootloader.

**Fix**

*   Re-check which DTB file is actually loaded.
*   Confirm your `$DTB` path matches the boot configuration.
*   If `fw_printenv` is available, check `fdtfile`.

***

## 2) `dtc` compilation fails

**Cause**

*   DTS syntax error (missing semicolon, broken braces).

**Fix**

*   Undo last edit.
*   Make small edits only.
*   Use:
    ```bash
    dtc -I dts -O dtb -o /tmp/test.dtb ~/lab6/bbb_modified.dts
    ```
    until it compiles.

***

## 3) Board boots but some devices disappear

**Cause**

*   You accidentally modified non-LED nodes or pinmux references.

**Fix**

*   Roll back immediately using backups (Recovery‑1).

***

## Optional Extension Tasks (Do ONLY if Lab core works)

### Extension‑A: Change `usr0` trigger to `mmc0` and observe behavior

Edit DTS and set:

```dts
linux,default-trigger = "mmc0";
```

Rebuild, deploy, reboot, and observe LED activity during SD/eMMC access.

### Extension‑B: Add a custom “training” node and verify via `/proc/device-tree`

Add under root `/ { ... }`:

```dts
fdp_training {
    compatible = "fdp,training-node";
    message = "hello-device-tree";
};
```

Rebuild & deploy DTB, reboot, then verify:

```bash
ls /proc/device-tree/fdp_training 2>/dev/null
tr -d '\0' </proc/device-tree/fdp_training/message; echo
```

**What you learn**

*   Device Tree is just structured hardware/config data
*   Kernel exposes it via `/proc/device-tree`

***

# Why this lab matters for upcoming work

This lab enables:

*   Lab‑7 (Character Driver + GPIO): DT tells driver which GPIO/pins to use
*   Lab‑8 (Debugging): “driver not working” often means DT mismatch or wrong DTB deployed

You now have a working DT workflow:

> locate → modify → compile → deploy → verify → rollback

***

## Ready for the next lab

When you’re ready, say:

**“Proceed with Lab‑7: Character Driver on BBB (GPIO‑based) with /dev node + user‑space test program”**

```
```
