
# Linux IOCTL Driver

# 1. Introduction

IOCTL stands for:

```text
Input Output Control
````

It is a mechanism in Linux device drivers used for:

* Device-specific operations
* Configuration commands
* Runtime control
* Special communication between user space and kernel space

Unlike normal:

```text id="k5w8mq"
read()
write()
```

operations, IOCTL allows custom commands to be sent to drivers.

---

# 2. What is an IOCTL Driver?

An IOCTL Driver is a Linux character driver that implements:

```c id="v8m2qy"
unlocked_ioctl()
```

to handle custom control commands from user applications.

IOCTL is commonly used when:

* read/write are insufficient
* Driver needs multiple commands
* Device configuration is required
* Special operations must be performed

---

# 3. Why Do We Use IOCTL?

Normal file operations only support:

| Operation | Purpose      |
| --------- | ------------ |
| read()    | Receive data |
| write()   | Send data    |

But drivers often require:

* Enable/disable device
* Set baud rate
* Configure hardware
* Reset device
* Read status registers

IOCTL solves this problem.

---

# 4. Real-Time Examples

| Device             | IOCTL Usage             |
| ------------------ | ----------------------- |
| UART Driver        | Baud rate configuration |
| Camera Driver      | Resolution setup        |
| Network Driver     | Interface control       |
| GPIO Driver        | Pin configuration       |
| RTC Driver         | Set alarm               |
| Audio Driver       | Volume control          |
| Touchscreen Driver | Calibration             |

---

# 5. IOCTL Architecture

```text id="g4m9wy"
+------------------------------+
| User Space Application       |
|------------------------------|
| ioctl(fd, cmd, arg)          |
+--------------+---------------+
               |
               v
+------------------------------+
| VFS Layer                    |
+--------------+---------------+
               |
               v
+------------------------------+
| IOCTL Driver                 |
|------------------------------|
| unlocked_ioctl()             |
+--------------+---------------+
               |
               v
+------------------------------+
| Hardware Device              |
+------------------------------+
```

---

# 6. IOCTL Communication Flow

```text id="y7r2vx"
User Application
        ↓
ioctl(fd, cmd, arg)
        ↓
Kernel VFS
        ↓
Driver unlocked_ioctl()
        ↓
Hardware Operation
```

---

# 7. Important IOCTL Concepts

## Command Number

Each ioctl operation has unique command ID.

---

## Magic Number

Identifies driver category.

Example:

```c id="m1w6qp"
#define MY_MAGIC 'A'
```

---

## Direction

Defines data flow:

| Macro | Purpose          |
| ----- | ---------------- |
| _IO   | No data transfer |
| _IOR  | Read from driver |
| _IOW  | Write to driver  |
| _IOWR | Read + Write     |

---

# 8. Important Header Files

```c id="p5m9xr"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/ioctl.h>
```

---

# 9. IOCTL Command Macros

Example:

```c id="x4r8mw"
#define WR_VALUE _IOW('A', 'a', int32_t *)
#define RD_VALUE _IOR('A', 'b', int32_t *)
```

Explanation:

| Field     | Meaning        |
| --------- | -------------- |
| 'A'       | Magic number   |
| 'a'       | Command number |
| int32_t * | Data type      |

---

# 10. IOCTL Driver Flow

## Step 1 – Driver Loaded

Character device registered.

---

## Step 2 – User Opens Device

Example:

```bash id="j3m7wy"
/dev/my_ioctl
```

---

## Step 3 – ioctl() Called

Kernel routes request to:

```c id="f6w2mx"
unlocked_ioctl()
```

---

## Step 4 – Driver Processes Command

Performs device-specific action.

---

# 11. Full IOCTL Driver Example

## ioctl_driver.c

```c id="z7m1qx"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/uaccess.h>
#include <linux/ioctl.h>

#define DEVICE_NAME "my_ioctl"

#define WR_VALUE _IOW('A', 'a', int32_t *)
#define RD_VALUE _IOR('A', 'b', int32_t *)

static int32_t value = 0;
static int major;
static struct class *dev_class;
static struct cdev my_cdev;

static long my_ioctl(struct file *file,
                     unsigned int cmd,
                     unsigned long arg)
{
    switch (cmd) {

    case WR_VALUE:

        if (copy_from_user(&value,
                           (int32_t *)arg,
                           sizeof(value)))
            return -EFAULT;

        printk(KERN_INFO "Value Written = %d\n", value);

        break;

    case RD_VALUE:

        if (copy_to_user((int32_t *)arg,
                         &value,
                         sizeof(value)))
            return -EFAULT;

        printk(KERN_INFO "Value Read = %d\n", value);

        break;

    default:
        printk(KERN_INFO "Invalid Command\n");
        return -EINVAL;
    }

    return 0;
}

static struct file_operations fops = {
    .owner          = THIS_MODULE,
    .unlocked_ioctl = my_ioctl,
};

static int __init ioctl_driver_init(void)
{
    dev_t dev;

    alloc_chrdev_region(&dev, 0, 1, DEVICE_NAME);

    major = MAJOR(dev);

    cdev_init(&my_cdev, &fops);

    cdev_add(&my_cdev, dev, 1);

    dev_class = class_create("my_class");

    device_create(dev_class,
                  NULL,
                  dev,
                  NULL,
                  DEVICE_NAME);

    printk(KERN_INFO "IOCTL Driver Loaded\n");

    return 0;
}

static void __exit ioctl_driver_exit(void)
{
    dev_t dev = MKDEV(major, 0);

    device_destroy(dev_class, dev);

    class_destroy(dev_class);

    cdev_del(&my_cdev);

    unregister_chrdev_region(dev, 1);

    printk(KERN_INFO "IOCTL Driver Removed\n");
}

module_init(ioctl_driver_init);
module_exit(ioctl_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple IOCTL Driver");
```

---

# 12. User Space Application Example

## test_ioctl.c

```c id="v3m8wy"
#include <stdio.h>
#include <fcntl.h>
#include <sys/ioctl.h>

#define WR_VALUE _IOW('A', 'a', int32_t *)
#define RD_VALUE _IOR('A', 'b', int32_t *)

int main()
{
    int fd;
    int value = 100;

    fd = open("/dev/my_ioctl", O_RDWR);

    ioctl(fd, WR_VALUE, &value);

    value = 0;

    ioctl(fd, RD_VALUE, &value);

    printf("Value = %d\n", value);

    close(fd);

    return 0;
}
```

---

# 13. Makefile

```Makefile id="k8w5mx"
obj-m += ioctl_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 14. Compile Driver

```bash id="x1m7vz"
make
```

Output:

```bash id="n4w2qy"
ioctl_driver.ko
```

---

# 15. Insert Driver

```bash id="r9m6wx"
sudo insmod ioctl_driver.ko
```

---

# 16. Verify Device File

```bash id="t5w1my"
ls /dev/my_ioctl
```

---

# 17. Compile User Application

```bash id="v6m3qx"
gcc test_ioctl.c -o test
```

---

# 18. Run Application

```bash id="z8r4wy"
./test
```

Expected Output:

```text id="f2m9vx"
Value = 100
```

---

# 19. Check Kernel Logs

```bash id="j7w5mx"
dmesg | tail
```

Expected:

```text id="y3m8qw"
Value Written = 100
Value Read = 100
```

---

# 20. Important IOCTL APIs

## unlocked_ioctl()

Handles ioctl commands.

Example:

```c id="m5w2vx"
.unlocked_ioctl = my_ioctl
```

---

## copy_from_user()

Copies data from user space to kernel space.

---

## copy_to_user()

Copies data from kernel space to user space.

---

# 21. IOCTL Command Types

## _IO

No data transfer.

Example:

```c id="w9m4qy"
#define RESET _IO('A', 1)
```

---

## _IOR

Driver → User.

---

## _IOW

User → Driver.

---

## _IOWR

Bidirectional transfer.

---

# 22. Advantages of IOCTL

| Advantage             | Description              |
| --------------------- | ------------------------ |
| Flexible              | Supports custom commands |
| Runtime Configuration | Device control           |
| Efficient             | Direct communication     |
| Standard Linux API    | Widely supported         |
| Hardware Control      | Advanced operations      |

---

# 23. Disadvantages of IOCTL

| Disadvantage       | Description           |
| ------------------ | --------------------- |
| Complex Design     | Multiple commands     |
| Security Risks     | Invalid user pointers |
| Hard Debugging     | Custom interfaces     |
| No Standardization | Driver-specific       |

---

# 24. IOCTL vs Read/Write

| Feature     | IOCTL                 | Read/Write     |
| ----------- | --------------------- | -------------- |
| Purpose     | Control/configuration | Data transfer  |
| Complexity  | High                  | Simple         |
| Flexibility | Very high             | Limited        |
| Usage       | Special commands      | Streaming data |

---

# 25. Common Interview Questions

## Q1. What is IOCTL?

A Linux mechanism for device-specific control operations.

---

## Q2. Why Use IOCTL?

To perform operations that cannot be handled using read/write.

---

## Q3. What is unlocked_ioctl()?

Kernel callback function that processes ioctl commands.

---

## Q4. Difference Between _IOR and _IOW?

| Macro | Direction     |
| ----- | ------------- |
| _IOR  | Driver → User |
| _IOW  | User → Driver |

---

## Q5. Why Use copy_to_user()?

Kernel cannot directly access user memory safely.

---

# 26. Common Errors

## Error: Invalid Command

Cause:

* Wrong ioctl command number

Fix:

* Verify macros

---

## Error: Segmentation Fault

Cause:

* Invalid user pointer

Fix:

* Validate pointers

---

## Error: Permission Denied

Cause:

* Device permissions issue

Fix:

```bash id="g6m8vx"
chmod 666 /dev/my_ioctl
```

---

# 27. IOCTL Debugging Techniques

## Kernel Logs

```bash id="r2w7my"
dmesg | tail
```

---

## Trace System Calls

```bash id="v4m9qx"
strace ./test
```

---

## Verify Device File

```bash id="t8w5mz"
ls -l /dev/my_ioctl
```

---

# 28. Advanced IOCTL Topics

After learning basic ioctl drivers, move to:

* Compat ioctl
* 32-bit/64-bit compatibility
* ioctl command validation
* Complex structures
* DMA communication
* Async notifications

---

# 29. Security Considerations

## Never Trust User Input

Always validate:

```c id="m7r3wy"
copy_from_user()
```

data.

---

## Check Buffer Sizes

Avoid:

* Buffer overflow
* Kernel corruption

---

## Validate Commands

Always handle:

```c id="x5m1vq"
default:
```

case.

---

# 30. Best Practices

## Use IOCTL Only When Necessary

Prefer:

* sysfs
* procfs
* read/write

for simple interfaces.

---

## Keep Commands Simple

Avoid overly complex interfaces.

---

## Document Command Numbers

Maintain stable API.

---

## Validate User Data

Prevent kernel crashes.

---

# 31. Real Hardware Platforms

IOCTL drivers are widely used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---

# 32. Real Linux IOCTL Examples

| Device         | Usage                  |
| -------------- | ---------------------- |
| TTY Driver     | Terminal configuration |
| Network Driver | Interface settings     |
| Camera Driver  | Resolution control     |
| Audio Driver   | Mixer settings         |
| RTC Driver     | Alarm setup            |

---

