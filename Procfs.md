
# Linux Procfs Driver

# 1. Introduction

Procfs stands for:

```text
Process File System
````

It is a virtual filesystem in Linux that provides information about:

* Kernel
* Processes
* Drivers
* Memory
* CPU
* Runtime system information

Procfs is mounted at:

```bash
/proc
```

Example:

```bash id="4r0w6x"
/proc/cpuinfo
/proc/meminfo
/proc/interrupts
```

Linux kernel drivers can create custom entries inside `/proc` to:

* Exchange data with user space
* Debug drivers
* Monitor kernel state
* Configure runtime parameters

---

# 2. What is a Procfs Driver?

A Procfs Driver is a Linux kernel module that creates entries inside:

```bash
/proc
```

These entries behave like files.

User applications can:

* Read from procfs
* Write to procfs

using standard Linux commands:

```bash id="m3v8kp"
cat
echo
```

---

# 3. Why Do We Use Procfs?

Procfs is mainly used for:

| Usage         | Purpose                   |
| ------------- | ------------------------- |
| Debugging     | View driver state         |
| Monitoring    | Runtime statistics        |
| Configuration | Dynamic parameter updates |
| Communication | Kernel ↔ User interaction |
| Diagnostics   | Troubleshooting           |

---

# 4. Real-Time Examples

| Procfs Entry     | Purpose              |
| ---------------- | -------------------- |
| /proc/cpuinfo    | CPU information      |
| /proc/meminfo    | Memory usage         |
| /proc/interrupts | Interrupt statistics |
| /proc/modules    | Loaded modules       |
| /proc/version    | Kernel version       |

Driver-specific examples:

* GPIO status
* Sensor values
* Driver statistics
* Debug counters
* Runtime configuration

---

# 5. Procfs Architecture

```text id="m8r5pw"
+------------------------------+
| User Space Application       |
|------------------------------|
| cat                          |
| echo                         |
+--------------+---------------+
               |
               v
+------------------------------+
| /proc Virtual Filesystem     |
+--------------+---------------+
               |
               v
+------------------------------+
| Procfs Driver                |
|------------------------------|
| proc_read()                  |
| proc_write()                 |
+--------------+---------------+
               |
               v
+------------------------------+
| Kernel Data                  |
+------------------------------+
```

---

# 6. Procfs vs Sysfs

| Feature                     | Procfs              | Sysfs                  |
| --------------------------- | ------------------- | ---------------------- |
| Purpose                     | Process/kernel info | Device model           |
| Location                    | /proc               | /sys                   |
| Data Type                   | Text/debug          | Structured device info |
| Recommended for New Drivers | Limited             | Yes                    |

---

# 7. Important Procfs APIs

Important APIs:

```c id="z0m4xq"
proc_create()
remove_proc_entry()
copy_to_user()
copy_from_user()
```

---

# 8. Important Header Files

```c id="t7r5mw"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/proc_fs.h>
#include <linux/uaccess.h>
```

---

# 9. Procfs Driver Flow

## Step 1 – Module Loaded

Driver creates `/proc` entry.

---

## Step 2 – User Reads Proc File

Kernel calls:

```c id="6v2n1x"
proc_read()
```

---

## Step 3 – User Writes Proc File

Kernel calls:

```c id="w5q8pd"
proc_write()
```

---

## Step 4 – Module Removed

Driver removes proc entry.

---

# 10. Full Procfs Driver Example

## procfs_driver.c

```c id="p3m7wt"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/proc_fs.h>
#include <linux/uaccess.h>

#define PROC_NAME "myprocfs"
#define BUFFER_SIZE 100

static char proc_buffer[BUFFER_SIZE];

static ssize_t proc_read(struct file *file,
                         char __user *user_buf,
                         size_t count,
                         loff_t *ppos)
{
    int len;

    len = snprintf(proc_buffer,
                   BUFFER_SIZE,
                   "Hello from Procfs Driver\n");

    return simple_read_from_buffer(user_buf,
                                   count,
                                   ppos,
                                   proc_buffer,
                                   len);
}

static ssize_t proc_write(struct file *file,
                          const char __user *user_buf,
                          size_t count,
                          loff_t *ppos)
{
    if (count > BUFFER_SIZE)
        count = BUFFER_SIZE;

    if (copy_from_user(proc_buffer, user_buf, count))
        return -EFAULT;

    printk(KERN_INFO "Procfs Received: %s\n", proc_buffer);

    return count;
}

static const struct proc_ops proc_fops = {
    .proc_read  = proc_read,
    .proc_write = proc_write,
};

static int __init procfs_driver_init(void)
{
    proc_create(PROC_NAME,
                0666,
                NULL,
                &proc_fops);

    printk(KERN_INFO "Procfs Driver Loaded\n");

    return 0;
}

static void __exit procfs_driver_exit(void)
{
    remove_proc_entry(PROC_NAME, NULL);

    printk(KERN_INFO "Procfs Driver Removed\n");
}

module_init(procfs_driver_init);
module_exit(procfs_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple Procfs Driver");
```

---

# 11. Makefile

```Makefile id="8n5v4r"
obj-m += procfs_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 12. Compile Driver

```bash id="n4r8pt"
make
```

Output:

```bash id="x6m1wz"
procfs_driver.ko
```

---

# 13. Insert Driver

```bash id="v2m7xy"
sudo insmod procfs_driver.ko
```

---

# 14. Verify Proc Entry

```bash id="g0w5mj"
ls /proc/myprocfs
```

---

# 15. Read Proc File

```bash id="p7v3mk"
cat /proc/myprocfs
```

Expected Output:

```text id="j6r0vn"
Hello from Procfs Driver
```

---

# 16. Write to Proc File

```bash id="4m8z7x"
echo "Hello Kernel" > /proc/myprocfs
```

---

# 17. Check Kernel Logs

```bash id="u1q6vb"
dmesg | tail
```

Expected:

```text id="m5t2yr"
Procfs Received: Hello Kernel
```

---

# 18. Important Procfs APIs Explained

## proc_create()

Creates procfs entry.

Example:

```c id="8v1y5r"
proc_create(name,
            permissions,
            parent,
            ops);
```

---

## remove_proc_entry()

Removes procfs entry.

Example:

```c id="c9q7wt"
remove_proc_entry(name, NULL);
```

---

## simple_read_from_buffer()

Safely copies kernel data to user space.

---

# 19. proc_ops Structure

Modern kernels use:

```c id="3t5n9x"
struct proc_ops
```

Older kernels used:

```c id="h8m0wp"
struct file_operations
```

---

# 20. Procfs Permissions

Example:

```text id="z3k7mr"
0666
```

Meaning:

| Value | Permission   |
| ----- | ------------ |
| 6     | Read + Write |
| 4     | Read Only    |
| 2     | Write Only   |

---

# 21. Advantages of Procfs

| Advantage             | Description            |
| --------------------- | ---------------------- |
| Simple Interface      | Easy user interaction  |
| Debugging Friendly    | Useful for diagnostics |
| Runtime Configuration | Dynamic updates        |
| Lightweight           | Minimal overhead       |
| Standard Linux Access | cat/echo support       |

---

# 22. Disadvantages of Procfs

| Disadvantage         | Description                      |
| -------------------- | -------------------------------- |
| Not Structured       | Free-form text                   |
| Limited Device Model | Sysfs preferred                  |
| Security Risks       | Improper permissions dangerous   |
| Deprecated Usage     | Sysfs recommended for many cases |

---

# 23. Procfs vs Sysfs vs Debugfs

| Feature                 | Procfs              | Sysfs       | Debugfs           |
| ----------------------- | ------------------- | ----------- | ----------------- |
| Purpose                 | Process/kernel info | Device info | Debugging         |
| Stable ABI              | Limited             | Yes         | No                |
| Recommended for Drivers | Partial             | Yes         | Debug only        |
| Location                | /proc               | /sys        | /sys/kernel/debug |

---

# 24. Common Interview Questions

## Q1. What is Procfs?

A virtual filesystem used to expose kernel and process information.

---

## Q2. Why Use Procfs?

For:

* Debugging
* Runtime configuration
* Monitoring

---

## Q3. What is proc_create()?

Creates a procfs file entry.

---

## Q4. Difference Between Procfs and Sysfs?

Procfs is process/kernel oriented.

Sysfs is device-oriented.

---

## Q5. Why is Sysfs Preferred for New Drivers?

Sysfs provides structured device representation.

---

# 25. Common Errors

## Error: Proc Entry Not Created

Cause:

* proc_create() failed

Fix:

* Verify permissions
* Check return values

---

## Error: Kernel Crash During Read

Cause:

* Invalid user buffer handling

Fix:

* Use simple_read_from_buffer()

---

## Error: Permission Denied

Cause:

* Incorrect file permissions

Fix:

* Verify proc_create() mode

---

# 26. Procfs Debugging Techniques

## Check Proc Entry

```bash id="9m1x4p"
ls /proc/
```

---

## Read Proc File

```bash id="5v7r0n"
cat /proc/myprocfs
```

---

## Kernel Logs

```bash id="k8n2mq"
dmesg | tail
```

---

# 27. Advanced Procfs Topics

After learning basic procfs drivers, move to:

* seq_file interface
* proc directories
* dynamic proc entries
* driver statistics
* debug frameworks
* sysfs migration
* debugfs interfaces

---

# 28. seq_file Interface

Used for large proc outputs.

Example:

```c id="x3t8mw"
seq_read()
seq_printf()
single_open()
```

Commonly used for:

* `/proc/meminfo`
* `/proc/interrupts`

---

# 29. Best Practices

## Use Sysfs for Device Configuration

Use procfs mainly for:

* Debugging
* Monitoring

---

## Validate User Input

Always check:

```c id="f5n7vp"
copy_from_user()
```

return values.

---

## Avoid Large Buffers

Procfs should remain lightweight.

---

## Use seq_file for Large Outputs

Efficient and scalable.

---

# 30. Real Hardware Platforms

Procfs is widely used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---

# 31. Real Linux Procfs Examples

| File             | Purpose           |
| ---------------- | ----------------- |
| /proc/cpuinfo    | CPU details       |
| /proc/meminfo    | Memory statistics |
| /proc/modules    | Loaded modules    |
| /proc/interrupts | IRQ statistics    |
| /proc/version    | Kernel version    |

---
