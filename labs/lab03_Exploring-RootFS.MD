# Lab‑3: Exploring Root File System (RootFS) on BeagleBone Black  
*BusyBox vs GNU | Init Process | Startup Behaviour*

---

## Why this lab exists

Up to now:

- You have seen **how the board boots**
- You have seen **how the kernel works**
- You have **cross‑compiled and run applications**

Now comes a critical realization:

> **A running kernel is still useless without RootFS.**

Most embedded Linux problems in the field are **RootFS problems**, not kernel problems:
- “Command not found”
- “Binary runs on host but not on target”
- “System boots but nothing works”
- “Why does this image feel different from Debian?”

This lab builds **deep intuition** about:
- What RootFS actually contains
- How Embedded RootFS differs from desktop Linux
- BusyBox vs GNU tools
- How user space starts (`init`)
- Why embedded systems deliberately remove things

---

## Objective

By the end of this lab, you will be able to:

1. Identify and explain **RootFS directory structure** on BBB
2. Verify **which tools come from BusyBox vs GNU**
3. Understand **how user space starts after kernel**
4. Identify the **init system** used on your BBB image
5. Understand **why embedded RootFS is minimal**
6. Predict and diagnose common RootFS‑related failures

---

## Prerequisites

- BBB booted into **any Linux image** (Debian / vendor / minimal)
- Serial console OR SSH access
- Lab‑1 and Lab‑2 completed successfully

> This lab is **image‑agnostic**.  
> It works with Debian, BusyBox‑based, or Yocto‑based images.

---

## Deliverables (save for later labs)

On BBB, create:
- `lab3_rootfs_layout.txt`
- `lab3_busybox_check.txt`
- `lab3_init_info.txt`

---

## Step‑by‑Step Procedure

---

## Step‑1: Confirm RootFS is mounted as `/`

On BBB:
```bash
mount | head -n 20
````

### Expected outcome

You should see a line similar to:

```text
/dev/mmcblkXpY on / type ext4 (rw,relatime,...)
```

or sometimes:

```text
rootfs on / type rootfs (rw)
```

### Explanation

*   `/` is the **Root File System**
*   Everything in user space exists **under `/`**
*   Kernel mounts `/` before starting `init`

***

## Step‑2: Explore standard RootFS directory structure

Run:

```bash
ls -l /
```

Save output:

```bash
ls -l / | tee ~/lab3_rootfs_layout.txt
```

### Expected directories (most images)

    bin   sbin   lib   etc   dev   proc   sys
    usr   tmp    var   home  boot

### Explanation (very important)

Each directory exists for a **specific reason**:

| Directory | Purpose                       |
| --------- | ----------------------------- |
| `/bin`    | Essential user commands       |
| `/sbin`   | System/admin commands         |
| `/lib`    | Shared libraries              |
| `/etc`    | System configuration          |
| `/dev`    | Device nodes (kernel-created) |
| `/proc`   | Kernel runtime info           |
| `/sys`    | Kernel objects & devices      |
| `/usr`    | Non-essential programs & libs |

> Embedded systems may **compress**, **merge**, or **remove** many of these.

***

## Step‑3: Identify BusyBox presence

Run:

```bash
which busybox || echo "busybox not found"
busybox 2>/dev/null | head -n 5
```

Save:

```bash
busybox | head -n 20 | tee ~/lab3_busybox_check.txt
```

### Expected outcomes

**Case‑1: BusyBox present**
You’ll see:

```text
BusyBox v1.xx.x (YYYY-MM-DD) multi-call binary
```

**Case‑2: BusyBox not present**

*   System uses full GNU utilities (common on Debian)

### Explanation

BusyBox is:

*   A **single binary**
*   Implements many commands via symlinks
*   Designed for embedded systems with limited memory/flash

BusyBox strongly influences:

*   Available command options
*   Script compatibility
*   Debugging behaviour

***

## Step‑4: Check which command is BusyBox vs GNU

### 4.1 Check `ls`

```bash
ls --version
```

**If GNU:**

```text
ls (GNU coreutils) X.Y
```

**If BusyBox:**

```text
ls: invalid option -- -
```

or minimal help output.

### 4.2 Check command source

```bash
which ls
ls -l $(which ls)
```

### 4.3 Check symlink pattern

```bash
ls -l /bin | head -n 10
```

### Explanation

BusyBox systems often have:

```text
/bin/ls -> busybox
/bin/cp -> busybox
/bin/sh -> busybox
```

GNU systems have:

*   Separate binaries for each command
*   Richer options
*   Larger storage footprint

***

## Step‑5: `/bin` vs `/usr/bin` (embedded difference)

Run:

```bash
ls -l /bin
ls -l /usr/bin | head -n 10
```

### Explanation

In embedded systems:

*   `/bin` contains **minimal, critical commands**
*   `/usr/bin` may be:
    *   absent
    *   very small
    *   mounted later

This separation avoids boot failures if `/usr` is unavailable early.

***

## Step‑6: Shared libraries are part of RootFS

Check libc:

```bash
ls -l /lib
ls -l /lib/arm-linux-gnueabihf 2>/dev/null || true
```

Check dynamic loader:

```bash
ls -l /lib/ld-linux* 2>/dev/null
```

### Explanation

*   Every dynamically linked binary depends on:
    *   libc
    *   loader (`ld-linux-*.so`)
*   Missing loader → `"No such file or directory"` error

This directly explains failures seen in **Lab‑2**.

***

## Step‑7: Verify virtual filesystems `/proc` and `/sys`

```bash
mount | grep -E 'proc|sysfs'
```

Then:

```bash
ls /proc | head
ls /sys | head
```

### Explanation

*   `/proc` and `/sys` are **not real filesystems**
*   They are kernel‑generated views
*   Used heavily for:
    *   debugging
    *   configuration
    *   hardware control

Without these mounted → system is effectively broken.

***

## Step‑8: Identify the init process (PID 1)

Run:

```bash
ps -p 1 -o pid,comm,args
```

Save:

```bash
ps -p 1 -o pid,comm,args | tee ~/lab3_init_info.txt
```

### Expected outcomes

| Init system | Output example         |
| ----------- | ---------------------- |
| systemd     | `/lib/systemd/systemd` |
| sysvinit    | `/sbin/init`           |
| BusyBox     | `init`                 |

### Explanation

*   PID 1 is **the first user‑space process**
*   Started directly by kernel
*   Responsible for starting everything else

If PID 1 fails → system halts or reboots.

***

## Step‑9: Inspect startup behaviour (high level)

### For systemd (Debian images)

```bash
systemctl list-units --type=service | head -n 10
```

### For sysvinit / BusyBox

```bash
ls /etc/init.d 2>/dev/null
```

### Explanation

We do **not** edit startup scripts here.
Purpose is to:

*   Understand what starts automatically
*   Identify resource usage
*   Appreciate why embedded systems disable many services

***

## Step‑10: Observe disk usage (embedded constraint)

```bash
df -h /
```

Check how much space RootFS uses.

### Explanation

Embedded systems are designed to:

*   Minimize storage
*   Reduce attack surface
*   Reduce boot time

BusyBox exists **only because of this constraint**.

***

## Common RootFS‑Related Failures (Real Life)

| Symptom                         | RootFS Cause      |
| ------------------------------- | ----------------- |
| Kernel panic: cannot mount root | Wrong root device |
| Command not found               | BusyBox vs GNU    |
| App fails to run                | Missing library   |
| Binary exists but won’t run     | Missing loader    |
| Boards boots, no services       | init failure      |

You can now **reason** about these, not guess.

***

## Expected Outcomes Checklist

By the end of this lab, you should be able to say:

*   [ ] I understand RootFS layout (`/bin`, `/lib`, `/etc`, `/dev`, `/proc`, `/sys`)
*   [ ] I can tell whether BusyBox is used
*   [ ] I understand why embedded Linux often removes utilities
*   [ ] I can identify the init system via PID 1
*   [ ] I understand how RootFS impacts application execution

***

## Optional Extension Tasks

### Extension‑A: Compare BusyBox and GNU `ls`

```bash
busybox ls --help | head -n 10
ls --help | head -n 10
```

### Extension‑B: Break‑fix thought experiment

Ask:

*   What happens if `/lib/ld-linux*.so` is deleted?
*   What happens if `/sbin/init` is missing?

(Do NOT try on live board — think logically.)
