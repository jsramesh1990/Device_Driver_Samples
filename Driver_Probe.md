# Driver Probe (Linux Device Drivers)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Driver Probe?](#what-is-driver-probe)
3. [Why Do We Need Probe()?](#why-do-we-need-probe)
4. [When is Probe Called?](#when-is-probe-called)
5. [How Probe Works Internally](#how-probe-works-internally)
6. [Probe Flow Diagram](#probe-flow-diagram)
7. [Probe Function Signature](#probe-function-signature)
8. [What Happens Inside Probe()?](#what-happens-inside-probe)
9. [Platform Driver Probe](#platform-driver-probe)
10. [I2C Driver Probe](#i2c-driver-probe)
11. [SPI Driver Probe](#spi-driver-probe)
12. [PCI Driver Probe](#pci-driver-probe)
13. [Device Tree and Probe](#device-tree-and-probe)
14. [Deferred Probe](#deferred-probe)
15. [Probe vs Remove](#probe-vs-remove)
16. [Memory Representation](#memory-representation)
17. [Real Embedded Example](#real-embedded-example)
18. [Advantages](#advantages)
19. [Disadvantages](#disadvantages)
20. [Common Mistakes](#common-mistakes)
21. [Interview Questions](#interview-questions)
22. [Quick Revision](#quick-revision)
23. [Conclusion](#conclusion)

---

# Introduction

In Linux Device Drivers, the **probe()** function is one of the most important callbacks.

It is called when:

- A device is detected.
- The kernel finds a matching driver.
- Device initialization is required.

Almost every Linux driver contains:

- `probe()`
- `remove()`

Examples:

- Platform Drivers
- I2C Drivers
- SPI Drivers
- PCI Drivers
- USB Drivers

---

# What is Driver Probe?

## Definition

`probe()` is a callback function executed by the Linux kernel when a device matches a driver.

---

## Interview Answer

Probe is the driver initialization function called automatically by the kernel when a device and driver match successfully.

---

# Why Do We Need Probe?

Without probe:

```text
Device Found

↓

No Initialization

↓

Hardware Not Ready

↓

Driver Cannot Work
```

---

With probe:

```text
Device Found

↓

Driver Matched

↓

probe()

↓

Initialize Hardware

↓

Allocate Resources

↓

Register Interrupt

↓

Device Ready
```

---

# When is Probe Called?

Probe is called:

### 1. Driver Loaded First

```text
Driver Loaded

↓

Device Appears

↓

Match Found

↓

probe()
```

---

### 2. Device Loaded First

```text
Device Registered

↓

Driver Loaded

↓

Match Found

↓

probe()
```

---

### 3. During Boot

```text
Boot

↓

Parse Device Tree

↓

Register Devices

↓

Register Drivers

↓

Match

↓

probe()
```

---

# How Probe Works Internally

Kernel flow:

```text
Device Registered

↓

device_register()

↓

Driver Registered

↓

driver_register()

↓

bus_match()

↓

driver_probe_device()

↓

really_probe()

↓

probe()

↓

Device Ready
```

---

# Probe Flow Diagram

```text
+-------------------+
| Device Registered |
+-------------------+

          |

          v

+-------------------+
| Driver Registered |
+-------------------+

          |

          v

+-------------------+
| Bus Match         |
+-------------------+

          |

          v

+-------------------+
| really_probe()    |
+-------------------+

          |

          v

+-------------------+
| Driver probe()    |
+-------------------+

          |

          v

+-------------------+
| Device Ready      |
+-------------------+
```

---

# Probe Function Signature

## Platform Driver

```c
int probe(struct platform_device *pdev);
```

---

## I2C Driver

```c
int probe(struct i2c_client *client);
```

---

## SPI Driver

```c
int probe(struct spi_device *spi);
```

---

## PCI Driver

```c
int probe(struct pci_dev *pdev,
          const struct pci_device_id *id);
```

---

# What Happens Inside Probe()?

Usually probe performs:

1. Allocate driver data

2. Map device memory

3. Request IRQ

4. Configure hardware

5. Register character device

6. Create sysfs entries

7. Initialize DMA

8. Enable interrupts

---

Example:

```text
probe()

↓

Allocate Memory

↓

ioremap()

↓

request_irq()

↓

Initialize Hardware

↓

Register Device

↓

Ready
```

---

# Platform Driver Probe

Example:

```c
static int led_probe(struct platform_device *pdev)
{
    printk("LED Driver Probed\n");

    return 0;
}
```

---

Driver Structure

```c
static struct platform_driver led_driver =
{
    .probe  = led_probe,

    .remove = led_remove,

    .driver =
    {
        .name = "led_driver",
    },
};
```

---

# I2C Driver Probe

```c
static int temp_probe(struct i2c_client *client)
{
    printk("Temperature Sensor Found\n");

    return 0;
}
```

---

I2C Driver

```c
static struct i2c_driver temp_driver =
{
    .probe = temp_probe,

    .driver =
    {
        .name = "temp_sensor",
    },
};
```

---

# SPI Driver Probe

```c
static int spi_probe(struct spi_device *spi)
{
    printk("SPI Device Initialized\n");

    return 0;
}
```

---

# PCI Driver Probe

```c
static int pci_probe(struct pci_dev *pdev,
                     const struct pci_device_id *id)
{
    printk("PCI Device Found\n");

    return 0;
}
```

---

# Device Tree and Probe

Device Tree:

```dts
led@0
{
    compatible = "mycompany,led";
};
```

---

Driver:

```c
static const struct of_device_id led_ids[] =
{
    {
        .compatible = "mycompany,led"
    },

    { }
};
```

---

Flow:

```text
Device Tree

↓

compatible

↓

of_match_table

↓

Driver Match

↓

probe()
```

---

# Deferred Probe

Sometimes resources are unavailable.

Example:

```text
probe()

↓

GPIO Driver Missing

↓

Return

-EPROBE_DEFER

↓

Kernel Retries Later

↓

probe()

↓

Success
```

---

## Example

```c
return -EPROBE_DEFER;
```

---

# Probe vs Remove

| Probe | Remove |
|------|-------|
| Initialize device | Cleanup device |
| Allocate memory | Free memory |
| Request IRQ | Free IRQ |
| Register device | Unregister device |
| Enable hardware | Disable hardware |

---

# Memory Representation

```text
RAM

-------------------------------------------------

struct platform_driver

|

+--- probe

|

+--- remove

|

+--- driver

-------------------------------------------------


struct platform_device

|

+--- dev

|

+--- id

|

+--- resources

-------------------------------------------------
```

---

# Real Embedded Example

GPIO Driver:

```c
static int gpio_probe(struct platform_device *pdev)
{
    gpio_request(10,"LED");

    gpio_direction_output(10,1);

    return 0;
}
```

---

UART Driver:

```c
static int uart_probe(struct platform_device *pdev)
{
    request_irq();

    uart_init();

    return 0;
}
```

---

I2C Temperature Sensor:

```c
static int temp_probe(struct i2c_client *client)
{
    sensor_init();

    return 0;
}
```

---

# Advantages

✔ Automatic initialization

✔ Supports hotplug

✔ Device Tree integration

✔ Bus independent

✔ Standard framework

✔ Easy driver management

---

# Disadvantages

✘ Complex initialization

✘ Probe ordering issues

✘ Deferred probe complexity

✘ Debugging can be difficult

---

# Common Mistakes

❌ Forgetting error handling

```c
ptr = kmalloc(...);

/* NULL not checked */
```

---

❌ Not freeing resources

```c
request_irq();

/* remove() missing free_irq() */
```

---

❌ Ignoring deferred probe

```c
return -EPROBE_DEFER;
```

---

❌ Incorrect Device Tree compatible string

Driver never probes.

---

# Interview Questions

### What is probe()?

A callback executed when a device matches a driver.

---

### Who calls probe()?

The Linux Kernel.

---

### When is probe() called?

After successful device-driver matching.

---

### What is deferred probe?

When resources are unavailable, the driver returns:

```c
-EPROBE_DEFER
```

and the kernel retries later.

---

### Difference between probe() and remove()?

Probe initializes device.

Remove cleans up allocated resources.

---

# Quick Revision

```text
Device Appears

↓

Driver Registered

↓

Bus Match

↓

really_probe()

↓

probe()

↓

Allocate Resources

↓

Initialize Hardware

↓

Device Ready
```

---

# Conclusion

`probe()` is the heart of Linux Device Drivers.

It is responsible for:

- Device initialization
- Resource allocation
- Interrupt registration
- Hardware configuration
- Sysfs integration

A strong understanding of probe is essential for:

- Platform Drivers
- I2C Drivers
- SPI Drivers
- PCI Drivers
- Embedded Linux
- Linux Kernel Development
