# Platform Drivers (Complete Guide)

# Table of Contents

1. Introduction
2. What is a Platform Driver?
3. Why Platform Drivers are Needed
4. Platform Device vs Platform Driver
5. Platform Bus Architecture
6. How Platform Drivers Work
7. Device Tree and Platform Drivers
8. Platform Driver Life Cycle
9. Probe and Remove Functions
10. Resources in Platform Drivers
11. Interrupt Handling
12. DMA Support
13. Memory-Mapped I/O (MMIO)
14. Power Management
15. Linux Platform Driver Framework
16. Platform Driver Example (UART)
17. Advantages
18. Disadvantages
19. Common Mistakes
20. Applications
21. Interview Questions
22. Summary

---

# 1. Introduction

In embedded Linux systems, many peripherals are **built directly into the SoC (System on Chip)**.

Examples:

* UART
* SPI Controller
* I2C Controller
* GPIO Controller
* Watchdog Timer
* RTC
* ADC
* PWM
* DMA Controller

These devices are not connected through buses like:

* PCI
* USB

Instead, they are integrated into the SoC.

Linux manages these devices using:

```text id="platform1"
Platform Devices + Platform Drivers
```

---

# 2. What is a Platform Driver?

A Platform Driver is:

```text id="platform2"
A Linux kernel driver used to control devices that are integrated directly into the SoC and discovered through Device Tree or board files.
```

Examples:

* UART Driver
* SPI Controller Driver
* I2C Controller Driver
* GPIO Driver
* RTC Driver

---

# 3. Why Platform Drivers are Needed

Because embedded devices:

✔ Do not support enumeration

✔ Are permanently present

✔ Have fixed memory addresses

✔ Have fixed interrupt numbers

✔ Are SoC specific

---

Example:

UART0:

```text id="platform3"
Base Address : 0xFF000000

IRQ          : 32

Clock        : uart_clk
```

OS must know:

* Address
* IRQ
* DMA channels
* Clock source

Platform driver obtains these details.

---

# 4. Platform Device vs Platform Driver

---

## Platform Device

Represents:

```text id="platform4"
Hardware Description
```

Contains:

✔ Memory address

✔ Interrupt number

✔ DMA info

✔ Clock info

---

Example:

```dts id="platform5"
uart0 {

    compatible = "vendor,myuart";

    reg = <0xff000000 0x1000>;

    interrupts = <32>;

};
```

---

## Platform Driver

Represents:

```text id="platform6"
Driver Code
```

Contains:

✔ probe()

✔ remove()

✔ suspend()

✔ resume()

---

Relationship:

```text id="platform7"
Platform Device

       ↓ Match

Platform Driver

       ↓

Driver Controls Hardware
```

---

# 5. Platform Bus Architecture

Linux internally creates:

```text id="platform8"
Platform Bus
```

Architecture:

```text id="platform9"
+----------------------+

| User Applications    |

+----------------------+

          |

          v

+----------------------+

| System Calls         |

+----------------------+

          |

          v

+----------------------+

| Platform Driver      |

+----------------------+

          |

          v

+----------------------+

| Platform Bus         |

+----------------------+

          |

          v

+----------------------+

| Platform Device      |

+----------------------+

          |

          v

+----------------------+

| SoC Peripheral       |

+----------------------+
```

---

# 6. How Platform Drivers Work

Boot Process:

```text id="platform10"
Kernel Boot

↓

Device Tree Parsed

↓

Platform Device Created

↓

Platform Driver Registered

↓

Matching Happens

↓

probe()

↓

Hardware Initialized
```

---

# 7. Device Tree and Platform Drivers

Device Tree describes hardware.

Example:

```dts id="platform11"
uart0 {

compatible = "vendor,myuart";

reg = <0x10000000 0x1000>;

interrupts = <45>;

clock-frequency = <48000000>;

status = "okay";

};
```

---

Fields:

### compatible

```text id="platform12"
Matches driver
```

---

### reg

```text id="platform13"
Base Address

Size
```

---

### interrupts

```text id="platform14"
IRQ Number
```

---

### status

```text id="platform15"
okay

disabled
```

---

Driver:

```c id="platform16"
static const struct of_device_id ids[] = {

{ .compatible = "vendor,myuart" },

};
```

Kernel matches automatically.

---

# 8. Platform Driver Life Cycle

```text id="platform17"
Module Load

↓

Register Platform Driver

↓

Kernel Finds Match

↓

probe()

↓

Initialize Hardware

↓

Driver Active

↓

remove()

↓

Unregister Driver

↓

Module Unload
```

---

# 9. Probe and Remove Functions

---

## probe()

Called when:

```text id="platform18"
Device Found
```

Responsibilities:

✔ Map registers

✔ Request IRQ

✔ Setup DMA

✔ Enable clocks

✔ Initialize hardware

---

Example:

```c id="platform19"
static int my_probe(

struct platform_device *pdev)

{

    return 0;

}
```

---

## remove()

Called when:

```text id="platform20"
Driver Removed
```

Responsibilities:

✔ Free IRQ

✔ Disable clocks

✔ Free DMA

✔ Unmap memory

---

Example:

```c id="platform21"
static int my_remove(

struct platform_device *pdev)

{

    return 0;

}
```

---

# 10. Resources in Platform Drivers

Resources include:

---

### Memory

```text id="platform22"
Registers

Memory Mapped IO
```

---

### Interrupts

```text id="platform23"
IRQ Numbers
```

---

### DMA Channels

```text id="platform24"
TX DMA

RX DMA
```

---

### Clocks

```text id="platform25"
Peripheral Clock
```

---

### Reset Controls

```text id="platform26"
Hardware Reset
```

---

Access APIs:

```c id="platform27"
platform_get_resource()

platform_get_irq()

devm_ioremap_resource()
```

---

# 11. Interrupt Handling

Example:

UART RX Interrupt:

```text id="platform28"
Character Received

↓

UART Raises IRQ

↓

ISR Executes

↓

Read RX Register

↓

Store Data

↓

Wake Task
```

ISR:

✔ clear interrupt

✔ short execution

✔ avoid sleeping

---

IRQ Registration:

```c id="platform29"
request_irq()
```

---

# 12. DMA Support

Many platform devices support DMA.

Example:

```text id="platform30"
SPI Controller

↓

DMA Engine

↓

RAM
```

Without DMA:

```text id="platform31"
SPI

↓

CPU Copies Data

↓

RAM
```

Slow.

---

With DMA:

```text id="platform32"
SPI

↓

DMA

↓

RAM
```

Fast.

---

APIs:

```c id="platform33"
dma_alloc_coherent()

dma_map_single()
```

---

# 13. Memory-Mapped I/O (MMIO)

Most platform devices expose:

```text id="platform34"
Registers

at

Fixed Memory Addresses
```

Example:

```text id="platform35"
UART Base

0xFF000000
```

Registers:

```text id="platform36"
TX Register

RX Register

Status Register

Control Register
```

---

Mapping:

```c id="platform37"
ioremap()
```

Access:

```c id="platform38"
readl()

writel()
```

---

# 14. Power Management

Platform drivers support:

---

## suspend()

When:

```text id="platform39"
System Sleep
```

Tasks:

✔ save registers

✔ disable clocks

✔ stop DMA

---

## resume()

When:

```text id="platform40"
Wake Up
```

Tasks:

✔ restore registers

✔ enable clocks

✔ restart DMA

---

# 15. Linux Platform Driver Framework

Important structures:

---

### platform_driver

```c id="platform41"
struct platform_driver
```

Contains:

```c id="platform42"
probe

remove

suspend

resume

driver
```

---

### platform_device

```c id="platform43"
struct platform_device
```

Represents:

✔ hardware

✔ resources

✔ device tree info

---

Registration:

```c id="platform44"
platform_driver_register()
```

---

Unregister:

```c id="platform45"
platform_driver_unregister()
```

---

# 16. Platform Driver Example (UART)

```text id="platform46"
Device Tree

↓

Platform Device

↓

UART Platform Driver

↓

probe()

↓

ioremap()

↓

request_irq()

↓

Enable Clock

↓

Initialize UART

↓

Ready
```

---

User:

```c id="platform47"
printf("Hello");
```

Flow:

```text id="platform48"
Application

↓

write()

↓

UART Driver

↓

TX Register

↓

UART Hardware

↓

Serial Cable
```

---

# 17. Advantages

✔ Perfect for SoC peripherals

✔ Device Tree support

✔ Automatic matching

✔ Resource management

✔ Power management

✔ DMA support

✔ Interrupt support

---

# 18. Disadvantages

❌ SoC dependent

❌ Complex Device Tree

❌ Difficult debugging

❌ Clock dependencies

❌ DMA configuration complexity

---

# 19. Common Mistakes

❌ Missing compatible string

❌ Forgetting iounmap()

❌ Wrong IRQ number

❌ Missing clock enable

❌ DMA cache issues

❌ Sleeping in ISR

❌ Memory leaks

---

# 20. Applications

Platform drivers are used in:

* UART Controllers
* SPI Controllers
* I2C Controllers
* GPIO Controllers
* RTC
* Watchdog Timers
* DMA Controllers
* PWM Controllers
* ADC
* CAN Controllers
* Ethernet MAC
* USB Controllers
* SD/MMC Controllers

---

# 21. Interview Questions

### Q1. What is a Platform Driver?

A Linux driver for devices integrated directly into the SoC.

---

### Q2. Platform Driver vs PCI Driver?

Platform:

* Device Tree based
* Fixed hardware

PCI:

* Bus enumeration
* Dynamic discovery

---

### Q3. What is probe()?

Function called when matching device is found.

---

### Q4. What is remove()?

Function called when driver is unloaded.

---

### Q5. Why Device Tree is used?

To describe hardware independently from driver code.

---

### Q6. What is MMIO?

Memory Mapped I/O.

Registers accessed using memory addresses.

---

# 22. Summary

Platform Drivers are:

```text id="platform49"
Linux kernel drivers used to control SoC-integrated devices that are discovered through Device Tree or board files.
```

Main responsibilities:

✔ Device initialization

✔ Register mapping

✔ Interrupt handling

✔ DMA configuration

✔ Clock management

✔ Power management

✔ Device Tree matching

They are fundamental to:

**Embedded Linux, ARM SoCs, UART, SPI, I2C, GPIO, Ethernet MAC, USB Controllers, SD/MMC Controllers, Automotive ECUs, IoT Devices, and Industrial Systems.**
