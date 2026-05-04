# Lab‑7: Character Device Driver on BBB (GPIO‑based) + `/dev` node + User‑Space Test Program
## Register char device → claim GPIO via Device Tree → control LED from user space

---

## Why this lab exists

This lab is the **capstone “driver engineering” loop**

- **Device Tree** tells the kernel *what hardware / pin / GPIO is used*
- **Kernel driver** claims the GPIO and exposes a **stable user interface** (`/dev/bbb_led`)
- **User application** uses normal POSIX I/O (`open/write/read`) to control hardware

In real products, the same pattern is used for:
- relays, buzzers, motor‑enable pins
- custom ASIC/FPGA control signals
- GPIO‑based status and safety lines

---

## Objective

By the end of this lab you will:

1. Wire a simple external LED to a BBB header pin (safe hardware).
2. Add a **Device Tree node** that declares a GPIO for our driver.
3. Build and load a **platform driver kernel module** that:
   - binds via `compatible = "fdp,gpioled"`
   - gets GPIO via gpiod (`devm_gpiod_get`)
   - registers a character device `/dev/bbb_led`
4. Run a user‑space test program that toggles the LED.
5. Debug the common failure cases: DT not applied, GPIO busy, `/dev` node missing, permissions, module mismatch.

---

## Prerequisites

- Lab‑1 to Lab‑6 completed
- BBB boots reliably with serial console available (**mandatory**)
- You can build kernel modules using one of these:
  - **Track‑A:** build on BBB (requires `/lib/modules/$(uname -r)/build`)
  - **Track‑B:** build on Host with matching kernel build tree and copy `.ko` to BBB
- `dtc` available on BBB (or you rebuild DTB using your host flow from Lab‑6)

---

## Hardware Setup (simple and safe)

### Parts
- 1 × LED (any color)
- 1 × resistor (330Ω recommended; 220Ω–1kΩ acceptable)
- 2 × jumper wires

### Wiring (recommended)
Use **P9_12 (GPIO1_28)** as output.

Connect:
- **P9_12 → resistor → LED anode (+)**
- **LED cathode (–) → GND (P9_1 or P9_2)**

This wiring means:
- GPIO HIGH → LED ON (**active high**)

> If you wire LED to 3.3V instead (active‑low), you must change DT polarity; keep it active‑high for this lab.

---

## Deliverables (what you will create)

On BBB (or host then copy):
- `~/lab7/driver/bbb_gpioled.c`
- `~/lab7/driver/Makefile`
- `~/lab7/driver/bbb_gpioled.ko`
- `~/lab7/user/bbb_led_test.c` (user program)
- Optional logs:
  - `~/lab7/lab7_dmesg.txt`

---

# Part‑A: Device Tree — Declare the GPIO for our driver

> **Goal:** Add a device node that our driver can bind to and read its GPIO from.

## Step‑A1: Create lab workspace
On BBB:
```bash
mkdir -p ~/lab7/dt ~/lab7/driver ~/lab7/user
````

## Step‑A2: Identify DTB in use and make a backup (same discipline as Lab‑6)

Set your DTB path (adjust to your system):

```bash
export DTB=/boot/dtbs/$(uname -r)/am335x-boneblack.dtb
ls -l "$DTB"
cp -av "$DTB" ~/lab7/dt/bbb.dtb.backup
```

If your DTB is elsewhere, find it:

```bash
sudo find /boot -maxdepth 4 -name "*.dtb" | head -n 50
```

Then set `DTB` accordingly.

## Step‑A3: Decompile DTB → DTS

```bash
dtc -I dtb -O dts -o ~/lab7/dt/bbb_lab7.dts "$DTB"
```

## Step‑A4: Modify DTS — add pinmux + device node

Edit:

```bash
nano ~/lab7/dt/bbb_lab7.dts
```

### A4.1 Add pinmux for P9\_12 as GPIO (AM335x)

Find the `am33xx_pinmux` node (commonly `&am33xx_pinmux { ... }`).
Inside it, add:

```dts
    fdp_led_pins: pinmux_fdp_led_pins {
        pinctrl-single,pins = <
            0x078 0x07
        >;
    };
```

**Explanation**

*   `0x078` corresponds to the control module offset for the P9\_12 pad in many BBB DTs.
*   `0x07` selects GPIO mode with pull settings (typical safe value for GPIO output).
*   Your exact offset can differ across DT versions; if compile fails or pin doesn’t work, see “Common Mistakes” below.

> If your DTS does not use `pinctrl-single`, the pinctrl style may differ. In that case, keep the driver portion working first, and use the optional “pin verification” section to confirm mux.

### A4.2 Add a simple device node that our driver binds to

Find the root node `/ { ... }` and add:

```dts
    fdp_gpioled {
        compatible = "fdp,gpioled";
        pinctrl-names = "default";
        pinctrl-0 = <&fdp_led_pins>;

        /* GPIO1_28 => P9_12 */
        led-gpios = <&gpio1 28 0>;
        label = "fdp-led-p9_12";
    };
```

**Explanation**

*   `compatible = "fdp,gpioled"` is the match key for our driver.
*   `led-gpios = <&gpio1 28 0>` declares GPIO controller + line.
    *   `0` means active‑high in many DT conventions. (If your DT uses flags like `GPIO_ACTIVE_HIGH`, that’s fine too.)
*   We attach `pinctrl` so pinmux is set before driver probe.

> If your DT uses `gpios` instead of `led-gpios`, we’ll handle it in driver by looking up `led` line name. We will keep `led-gpios` here for clarity.

Save and exit.

## Step‑A5: Recompile DTS → DTB

```bash
dtc -I dts -O dtb -o ~/lab7/dt/bbb_lab7.dtb ~/lab7/dt/bbb_lab7.dts
ls -l ~/lab7/dt/bbb_lab7.dtb
```

If `dtc` fails with syntax errors, fix braces/semicolons and retry.

## Step‑A6: Deploy DTB safely and reboot

```bash
sudo cp -av "$DTB" "${DTB}.prelab7"
sudo cp -av ~/lab7/dt/bbb_lab7.dtb "$DTB"
sync
sudo reboot
```

## Step‑A7: Verify DT node exists after reboot

After login:

```bash
ls /proc/device-tree/fdp_gpioled 2>/dev/null && echo "DT node present" || echo "DT node NOT present"
```

Also verify label:

```bash
tr -d '\0' </proc/device-tree/fdp_gpioled/label; echo
```

Expected:

```text
fdp-led-p9_12
```

***

# Part‑B: Kernel Driver — Platform Driver + Char Device `/dev/bbb_led`

> **Goal:** Create a kernel module that:
>
> *   probes our DT node
> *   claims GPIO
> *   creates `/dev/bbb_led`
> *   supports `read()` (state) and `write()` (set 0/1)

## Step‑B1: Create driver source `bbb_gpioled.c`

On BBB (or host if using Track‑B):

```bash
cat > ~/lab7/driver/bbb_gpioled.c <<'EOF'
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/uaccess.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/gpio/consumer.h>

#define DEVNAME "bbb_led"

static dev_t devno;
static struct cdev bbb_cdev;
static struct class *bbb_class;

struct bbb_led_dev {
    struct device *dev;
    struct gpio_desc *led_gpiod;
    int state;
};

static struct bbb_led_dev g_led;

static ssize_t bbb_led_read(struct file *f, char __user *buf, size_t len, loff_t *off)
{
    char kbuf[4];
    int n;

    if (*off > 0)
        return 0;

    g_led.state = gpiod_get_value(g_led.led_gpiod) ? 1 : 0;
    n = scnprintf(kbuf, sizeof(kbuf), "%d\n", g_led.state);

    if (len < n)
        return -EINVAL;

    if (copy_to_user(buf, kbuf, n))
        return -EFAULT;

    *off += n;
    return n;
}

static ssize_t bbb_led_write(struct file *f, const char __user *buf, size_t len, loff_t *off)
{
    char kbuf[16];
    int val;

    if (len == 0)
        return 0;

    if (len >= sizeof(kbuf))
        return -EINVAL;

    if (copy_from_user(kbuf, buf, len))
        return -EFAULT;

    kbuf[len] = '\0';

    /* Accept "0" or "1" (optionally with newline) */
    if (kbuf[0] == '0')
        val = 0;
    else if (kbuf[0] == '1')
        val = 1;
    else
        return -EINVAL;

    gpiod_set_value(g_led.led_gpiod, val);
    g_led.state = val;

    dev_info(g_led.dev, "bbb_led: set=%d\n", val);
    return len;
}

static int bbb_led_open(struct inode *i, struct file *f)
{
    return 0;
}

static int bbb_led_release(struct inode *i, struct file *f)
{
    return 0;
}

static const struct file_operations bbb_led_fops = {
    .owner   = THIS_MODULE,
    .open    = bbb_led_open,
    .read    = bbb_led_read,
    .write   = bbb_led_write,
    .release = bbb_led_release,
};

static int bbb_led_probe(struct platform_device *pdev)
{
    int ret;

    g_led.dev = &pdev->dev;

    /* Get GPIO from DT: property name "led" by convention in gpiod_get */
    g_led.led_gpiod = devm_gpiod_get(&pdev->dev, "led", GPIOD_OUT_LOW);
    if (IS_ERR(g_led.led_gpiod)) {
        dev_err(&pdev->dev, "Failed to get LED GPIO from DT (led-gpios)\n");
        return PTR_ERR(g_led.led_gpiod);
    }

    /* Character device setup */
    ret = alloc_chrdev_region(&devno, 0, 1, DEVNAME);
    if (ret) {
        dev_err(&pdev->dev, "alloc_chrdev_region failed\n");
        return ret;
    }

    cdev_init(&bbb_cdev, &bbb_led_fops);
    ret = cdev_add(&bbb_cdev, devno, 1);
    if (ret) {
        unregister_chrdev_region(devno, 1);
        dev_err(&pdev->dev, "cdev_add failed\n");
        return ret;
    }

    bbb_class = class_create(THIS_MODULE, "bbb");
    if (IS_ERR(bbb_class)) {
        cdev_del(&bbb_cdev);
        unregister_chrdev_region(devno, 1);
        dev_err(&pdev->dev, "class_create failed\n");
        return PTR_ERR(bbb_class);
    }

    /* device_create triggers udev to create /dev/bbb_led (if udev is present) */
    device_create(bbb_class, NULL, devno, NULL, DEVNAME);

    dev_info(&pdev->dev, "bbb_led: probed OK. /dev/%s major=%d minor=%d\n",
             DEVNAME, MAJOR(devno), MINOR(devno));

    return 0;
}

static int bbb_led_remove(struct platform_device *pdev)
{
    device_destroy(bbb_class, devno);
    class_destroy(bbb_class);
    cdev_del(&bbb_cdev);
    unregister_chrdev_region(devno, 1);

    dev_info(&pdev->dev, "bbb_led: removed\n");
    return 0;
}

static const struct of_device_id bbb_led_of_match[] = {
    { .compatible = "fdp,gpioled" },
    { }
};
MODULE_DEVICE_TABLE(of, bbb_led_of_match);

static struct platform_driver bbb_led_driver = {
    .probe  = bbb_led_probe,
    .remove = bbb_led_remove,
    .driver = {
        .name           = "bbb_gpioled",
        .of_match_table = bbb_led_of_match,
    },
};

module_platform_driver(bbb_led_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("FDP-Participant");
MODULE_DESCRIPTION("BBB GPIO LED char driver (/dev/bbb_led) bound via Device Tree");
MODULE_VERSION("1.0");
EOF
```

### What to teach while presenting this code (the “why”)

*   **Platform driver + `of_match_table`** is the clean way to bind DT nodes to drivers.
*   `devm_gpiod_get(..., "led", ...)` looks for `led-gpios` in DT (prefix `led`) and claims it.
*   We create a **character device** and map syscalls to our functions via `file_operations`.
*   User space writes `'0'` or `'1'` to set GPIO value.

***

## Step‑B2: Create Makefile for the module

```bash
cat > ~/lab7/driver/Makefile <<'EOF'
obj-m += bbb_gpioled.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
EOF
```

***

## Step‑B3: Build the module

### Track‑A (build on BBB)

```bash
cd ~/lab7/driver
make
ls -l *.ko
```

### Track‑B (build on host with matching kernel build tree)

On host:

```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
export KDIR=/path/to/bbb-kernel-build
make -C "$KDIR" M="$PWD" modules
```

Copy to BBB:

```bash
scp bbb_gpioled.ko debian@<BBB_IP>:/home/debian/lab7/driver/
```

***

## Step‑B4: Insert module and verify probe

On BBB:

```bash
cd ~/lab7/driver
sudo insmod bbb_gpioled.ko
dmesg -T | tail -n 50 | tee ~/lab7/lab7_dmesg.txt
```

### Expected output (representative)

You should see something like:

```text
bbb_led: probed OK. /dev/bbb_led major=XXX minor=0
```

If you see:

```text
Failed to get LED GPIO from DT (led-gpios)
```

Then DT node or property name is wrong → see “Common Mistakes”.

***

## Step‑B5: Confirm `/dev/bbb_led` exists

```bash
ls -l /dev/bbb_led || echo "/dev/bbb_led not present (udev may be missing)"
```

### If `/dev/bbb_led` is missing (common on minimal images)

Use the major number printed in dmesg (or read from sysfs):

```bash
cat /proc/devices | grep bbb_led || true
```

If dmesg showed `major=240` for example, create node:

```bash
sudo mknod /dev/bbb_led c 240 0
sudo chmod 666 /dev/bbb_led
ls -l /dev/bbb_led
```

> On Debian with udev, `/dev/bbb_led` should appear automatically.

***

# Part‑C: User‑Space Test Program

## Step‑C1: Create user test program `bbb_led_test.c`

On host (recommended; cross compile) or on BBB if gcc is available:

```bash
cat > ~/lab7/user/bbb_led_test.c <<'EOF'
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

static void write_val(int fd, const char *v)
{
    if (write(fd, v, strlen(v)) < 0) {
        perror("write");
    }
}

int main(int argc, char **argv)
{
    int fd;
    char buf[16] = {0};

    fd = open("/dev/bbb_led", O_RDWR);
    if (fd < 0) {
        perror("open(/dev/bbb_led)");
        return 1;
    }

    /* Read initial state */
    lseek(fd, 0, SEEK_SET);
    if (read(fd, buf, sizeof(buf)-1) > 0)
        printf("Initial state: %s", buf);

    /* Simple toggle sequence */
    printf("Turning ON...\n");
    write_val(fd, "1\n");
    sleep(1);

    printf("Turning OFF...\n");
    write_val(fd, "0\n");
    sleep(1);

    printf("Blinking 5 times...\n");
    for (int i = 0; i < 5; i++) {
        write_val(fd, "1\n");
        usleep(200000);
        write_val(fd, "0\n");
        usleep(200000);
    }

    /* Read final state */
    memset(buf, 0, sizeof(buf));
    lseek(fd, 0, SEEK_SET);
    if (read(fd, buf, sizeof(buf)-1) > 0)
        printf("Final state: %s", buf);

    close(fd);
    return 0;
}
EOF
```

## Step‑C2: Compile the test program

### Option 1 (Recommended): Cross compile on host

```bash
arm-linux-gnueabihf-gcc -O2 -Wall -o bbb_led_test bbb_led_test.c
file bbb_led_test
```

Copy to BBB:

```bash
scp bbb_led_test debian@<BBB_IP>:/home/debian/lab7/user/
```

### Option 2: Compile on BBB (if gcc exists)

On BBB:

```bash
cd ~/lab7/user
gcc -O2 -Wall -o bbb_led_test bbb_led_test.c
```

***

## Step‑C3: Run the test program and observe LED

On BBB:

```bash
cd ~/lab7/user
chmod +x bbb_led_test
./bbb_led_test
```

### Expected outcome

*   External LED connected to P9\_12 should:
    *   turn ON
    *   turn OFF
    *   blink 5 times
*   Output should print initial and final state like:

```text
Initial state: 0
Turning ON...
Turning OFF...
Blinking 5 times...
Final state: 0
```

Also check kernel logs:

```bash
dmesg -T | tail -n 50 | grep bbb_led
```

Expected (representative):

```text
bbb_led: set=1
bbb_led: set=0
...
```

***

## Step‑C4: Unload module and confirm cleanup

On BBB:

```bash
sudo rmmod bbb_gpioled
lsmod | grep bbb_gpioled || echo "module unloaded"
```

***

# Expected Outcomes Checklist

*   [ ] Wired LED + resistor on P9\_12 to GND
*   [ ] Added DT node `fdp_gpioled` and rebooted
*   [ ] Verified DT node exists under `/proc/device-tree/fdp_gpioled`
*   [ ] Built `bbb_gpioled.ko` and loaded it successfully
*   [ ] Saw “probed OK” message in `dmesg` with major/minor
*   [ ] `/dev/bbb_led` exists (or created via `mknod`)
*   [ ] User program toggles the LED reliably
*   [ ] Driver unloads cleanly

***

# Common Mistakes & Fixes (real BBB failure modes)

## 1) Driver loads but says “Failed to get LED GPIO from DT”

**Cause**

*   DT property name mismatch.
*   `devm_gpiod_get(..., "led", ...)` expects `led-gpios` in DT.

**Fix**
Ensure your DT node contains:

```dts
led-gpios = <&gpio1 28 0>;
```

Rebuild DTB, deploy, reboot.

***

## 2) No change after DT edit (DT not actually applied)

**Cause**

*   You edited/replaced a DTB file that is not the one used by bootloader.

**Fix**

*   Re‑confirm DTB path used in boot (`/boot` layout, U‑Boot env if available).
*   Validate DT node exists in `/proc/device-tree/` after reboot.
    *   If node isn’t there, DTB wasn’t applied.

***

## 3) GPIO busy / resource conflict

**Symptoms**

*   Probe fails with GPIO busy or cannot request line.

**Cause**

*   That GPIO is already claimed by another driver (very common).

**Fix**

*   Choose a different header GPIO line.
*   Or disable conflicting node (advanced; not recommended in first driver lab).

***

## 4) `/dev/bbb_led` not present

**Cause**

*   udev not present on minimal images (no auto device node creation).

**Fix**

*   Create manually using major from dmesg:

```bash
sudo mknod /dev/bbb_led c <major> 0
sudo chmod 666 /dev/bbb_led
```

***

## 5) `insmod: Invalid module format`

**Cause**

*   Built against wrong kernel headers (vermagic mismatch).

**Fix**

```bash
uname -r
modinfo bbb_gpioled.ko | grep vermagic
```

Rebuild using the exact kernel build tree matching `uname -r`.

***

## 6) LED never lights up (but driver works)

**Causes**

*   Wiring mistake (most common)
*   Wrong GPIO mapping for chosen pin
*   Pinmux not applied / wrong pinctrl style

**Fix**

*   Recheck wiring: P9\_12 → resistor → LED → GND
*   Confirm the pin is correct GPIO for your DT:
    *   Try another known GPIO pin if needed.
*   Verify pinmux (optional tools):
    *   If `config-pin` exists on your image:
        ```bash
        config-pin -q P9_12
        ```
    *   Or use `gpioinfo` if `gpiod` tools installed:
        ```bash
        gpioinfo | head -n 50
        ```

***

# Optional Extension Tasks (high value)

## Extension‑A: Add `ioctl` for blinking rate (driver API design)

*   Add an ioctl command `SET_BLINK_MS`
*   Driver stores blink period
*   User app sends ioctl and driver uses a kernel timer/workqueue (advanced)

## Extension‑B: Support multiple LEDs (minor numbers)

*   Add `led1-gpios`, `led2-gpios` in DT
*   Create `/dev/bbb_led0`, `/dev/bbb_led1`
*   Use minor number in `open()` to select GPIO

## Extension‑C: Add `poll()` support for button input (prelude to interrupts)

*   Add a second GPIO as input (button)
*   Use wait queues + poll/select
*   This connects strongly to Phase‑1 (blocking I/O, threads)
