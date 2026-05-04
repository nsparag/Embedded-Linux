# Lab‑2: Cross‑Compiling an Application for BeagleBone Black (BBB)
*Host vs Target | ARM Toolchain | Validate with `file`, `readelf`, `ldd`*

---

## Why this lab exists
In embedded Linux, **build happens on a powerful host** and **execution happens on the target**. If you mix these up, you’ll waste hours on errors like:

- `Exec format error` (wrong architecture)
- `No such file or directory` even though the file exists (missing dynamic loader)
- runtime crashes due to library/ABI mismatch

This lab trains the **repeatable workflow** you will use for:
- user apps (now)
- kernel modules (later)
- tool utilities used in debugging

---

## Objective
By the end of this lab you will be able to:

1. Install and verify an ARM cross‑toolchain on the host.
2. Cross‑compile a simple C application for BBB.
3. Prove the **host binary** and **target binary** are different using `file` and `readelf`.
4. Transfer the ARM binary to BBB (SCP/USB) and run it.
5. Validate runtime dependencies using `ldd` (or equivalent) on the target.

---

## Prerequisites

### Hardware / Access
- BBB is booted into **any Linux** (Debian, minimal, Yocto — doesn’t matter)
- Serial console working (from Lab‑1)
- Optional (recommended): BBB reachable over network (Ethernet/Wi‑Fi) for `scp`

### Host machine (Ubuntu recommended)
Install required packages:
```bash
sudo apt update
sudo apt install -y gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf \
  binutils-arm-linux-gnueabihf make cmake pkg-config
````

> Toolchain note:
>
> *   BBB is **32‑bit ARMv7 (armv7l)** in most common setups.
> *   The correct cross compiler typically is: `arm-linux-gnueabihf-gcc`

***

## Deliverables (save these)

On host, create a working folder:

```bash
mkdir -p ~/bbb-labs/lab2
cd ~/bbb-labs/lab2
```

You will produce:

*   `hello.c`
*   `Makefile`
*   `hello_host` (x86\_64 binary)
*   `hello_bbb` (ARM binary)
*   Optional logs: `lab2_checks.txt`

***

## Step‑by‑Step Procedure

***

## Step‑1: Confirm host and target architectures (do not skip)

### 1A) On HOST (your Ubuntu PC)

```bash
uname -m
```

**Expected output (typical):**

```text
x86_64
```

### 1B) On BBB (serial console)

```bash
uname -m
```

**Expected output (typical):**

```text
armv7l
```

**Explanation**
You are building on **x86\_64** but running on **armv7l**.  
So the output binary formats must differ.

***

## Step‑2: Verify cross‑compiler is installed correctly (host)

```bash
which arm-linux-gnueabihf-gcc
arm-linux-gnueabihf-gcc --version
```

**Expected**

*   `which` prints a valid path (e.g., `/usr/bin/arm-linux-gnueabihf-gcc`)
*   `--version` prints GCC version details

**Common mistake**

*   Using `aarch64-linux-gnu-gcc` (64‑bit ARM) → will not run on armv7l BBB.

***

## Step‑3: Write a small application (with an “embedded‑friendly” identity)

Create `hello.c`:

```c
cat > hello.c <<'EOF'
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    printf("Hello from BBB cross-compiled app!\n");
    printf("PID=%d\n", getpid());
    printf("If you see this on BBB, host->target pipeline is correct.\n");
    return 0;
}
EOF
```

**Why this code**

*   It uses only standard libc + a simple syscall (`getpid()`) to prove the app is running correctly on target.

***

## Step‑4: Build TWO binaries (host-native and BBB-target)

### 4A) Build host-native binary (for comparison)

```bash
gcc -O2 -o hello_host hello.c
```

### 4B) Build BBB ARM binary (cross compile)

```bash
arm-linux-gnueabihf-gcc -O2 -o hello_bbb hello.c
```

***

## Step‑5: Prove binaries are different using `file` (host)

```bash
file hello_host hello_bbb
```

**Expected output (representative)**

*   For `hello_host`: `ELF 64-bit LSB ... x86-64 ...`
*   For `hello_bbb`: `ELF 32-bit LSB ... ARM ... EABI5 ...`

**Explanation**
This is your first “truth check”.  
Never copy a binary to BBB until `file` proves it is ARM.

***

## Step‑6: Verify ELF interpreter (dynamic loader) with `readelf` (host)

This step helps diagnose the famous “No such file or directory” issue.

```bash
readelf -l hello_bbb | grep -i 'interpreter' || true
```

**Expected (typical on glibc systems)**
You will see something like:

```text
[Requesting program interpreter: /lib/ld-linux-armhf.so.3]
```

**Explanation**
If the interpreter path does not exist on BBB, running the binary fails with:

*   `No such file or directory`
    even though the binary is present and executable.

***

## Step‑7: Transfer the ARM binary to BBB

### Option A (Recommended): Use SCP (requires network)

1.  Find BBB IP (on BBB):

```bash
ip a
```

2.  From HOST:

```bash
scp ./hello_bbb debian@<BBB_IP>:/home/debian/
```

> Replace `debian` with your BBB username.

### Option B: Use USB drive / SD card copy (no network required)

1.  Copy `hello_bbb` to a USB drive on host
2.  Plug into BBB, mount, and copy to home directory

Example on BBB:

```bash
lsblk
sudo mount /dev/sda1 /mnt
cp /mnt/hello_bbb ~/
sudo umount /mnt
```

***

## Step‑8: Run on BBB and validate behavior

On BBB:

```bash
chmod +x ~/hello_bbb
./hello_bbb
```

**Expected output**

```text
Hello from BBB cross-compiled app!
PID=1234
If you see this on BBB, host->target pipeline is correct.
```

**Explanation**
At this moment, you have validated:

*   cross toolchain works
*   binary transfer works
*   BBB can execute your compiled output

***

## Step‑9: Validate runtime dependencies on BBB (`ldd`)

On BBB:

```bash
ldd ./hello_bbb
```

**Expected**
You should see libc and loader dependencies, e.g.:

*   `libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (...)`
*   `/lib/ld-linux-armhf.so.3 (...)`

**Explanation**
This confirms:

*   binary is dynamically linked (normal)
*   required libraries exist on target

***

# Common Mistakes & Fixes (Real BBB failures)

## 1) `./hello_bbb: Exec format error`

**Cause**

*   You copied an x86\_64 binary to BBB.
    **Fix**
*   Run `file hello_bbb` on host.
*   Ensure it says `ARM` not `x86-64`.

***

## 2) `./hello_bbb: No such file or directory` (but the file exists)

**Cause**

*   Missing program interpreter (dynamic loader) on BBB.
    **How to confirm**
    On host:

```bash
readelf -l hello_bbb | grep -i interpreter
```

If interpreter is `/lib/ld-linux-armhf.so.3`, verify on BBB:

```bash
ls -l /lib/ld-linux-armhf.so.3
```

**Fix options**

*   Use a BBB image that provides glibc armhf loader (Debian usually does)
*   Or build a **static binary** (next section)

***

## 3) `Permission denied`

**Cause**

*   Execute bit missing or mounted noexec filesystem.
    **Fix**

```bash
chmod +x hello_bbb
mount | grep -i noexec
```

***

## 4) Runs on BBB but crashes immediately

**Cause**

*   ABI/lib mismatch (rare for this simple program, common for real apps).
    **Fix**
*   Run `ldd` to verify libraries.
*   Ensure you used `gnueabihf` for hard-float on armv7l.

***

# Optional Extension Tasks (Highly Recommended)

## Extension‑A: Build a static binary and compare tradeoffs

Static binaries reduce dependency problems but increase size.

On host:

```bash
arm-linux-gnueabihf-gcc -O2 -static -o hello_bbb_static hello.c
file hello_bbb_static
```

Copy to BBB and run:

```bash
chmod +x ~/hello_bbb_static
./hello_bbb_static
```

Compare file sizes on host:

```bash
ls -lh hello_bbb hello_bbb_static
```

**What you learn**

*   Static is bigger but portable across many rootfs setups.
*   Dynamic is smaller but depends on target loader/libs.

***

## Extension‑B: Add a pthread

Create `hello_thread.c`:

```c
cat > hello_thread.c <<'EOF'
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

static void* worker(void* arg)
{
    (void)arg;
    for (int i=0; i<3; i++) {
        printf("[worker] iteration %d\n", i);
        sleep(1);
    }
    return NULL;
}

int main(void)
{
    pthread_t t;
    printf("Main starting thread...\n");
    pthread_create(&t, NULL, worker, NULL);
    pthread_join(t, NULL);
    printf("Thread finished.\n");
    return 0;
}
EOF
```

Cross compile:

```bash
arm-linux-gnueabihf-gcc -O2 -o hello_thread_bbb hello_thread.c -pthread
file hello_thread_bbb
```

Run on BBB and observe output order.

***

## Extension‑C: Create a simple Makefile (repeatable build)

Create `Makefile`:

```bash
cat > Makefile <<'EOF'
CC_HOST=gcc
CC_BBB=arm-linux-gnueabihf-gcc
CFLAGS=-O2 -Wall

all: host bbb

host:
	$(CC_HOST) $(CFLAGS) -o hello_host hello.c

bbb:
	$(CC_BBB) $(CFLAGS) -o hello_bbb hello.c

clean:
	rm -f hello_host hello_bbb
EOF
```

Build:

```bash
make clean
make all
file hello_host hello_bbb
```

***

# Expected Outcomes Checklist (End of Lab)

*   [ ] Verified host arch is x86\_64 and BBB arch is armv7l
*   [ ] Verified cross compiler exists: `arm-linux-gnueabihf-gcc`
*   [ ] Built `hello_host` and `hello_bbb`
*   [ ] Confirmed `hello_bbb` is ARM using `file`
*   [ ] Confirmed interpreter path using `readelf`
*   [ ] Transferred `hello_bbb` to BBB
*   [ ] Executed successfully on BBB
*   [ ] Checked dependencies using `ldd` on BBB
