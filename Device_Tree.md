# Device Tree in Embedded Linux

## Overview

A **Device Tree (DT)** is a data structure used by the Linux kernel to describe hardware components of an embedded system.

It allows:
- hardware description separation from kernel code
- generic kernels for multiple boards
- easier hardware support

Device Tree is heavily used in:
- ARM
- ARM64
- RISC-V
- PowerPC embedded systems

---

# 1. Why Device Tree Exists

Before Device Tree:
- kernel source contained board-specific code
- every board required custom kernel modifications

Problems:
- difficult maintenance
- duplicated code
- poor scalability

Device Tree solved this by:
- moving hardware description outside kernel source

---

# 2. High-Level Boot Flow with Device Tree

```text
Power ON
    ↓
ROM Code
    ↓
SPL
    ↓
U-Boot
    ↓
Load Kernel
    ↓
Load DTB
    ↓
Kernel Parses Device Tree
    ↓
Driver Initialization
``` id="boot1"

---

# 3. What Device Tree Describes

Device Tree describes:
- CPU
- memory
- UART
- GPIO
- I2C
- SPI
- interrupts
- clocks
- buses
- peripherals

---

# 4. Device Tree Architecture

```text
Device Tree Source (.dts)
          ↓
Device Tree Compiler (dtc)
          ↓
Device Tree Blob (.dtb)
          ↓
Loaded by Bootloader
          ↓
Parsed by Linux Kernel
``` id="flow1"

---

# 5. Device Tree Components

| Component | Purpose |
|-----------|---------|
| .dts | Device Tree Source |
| .dtsi | Shared include files |
| .dtb | Binary Device Tree Blob |
| dtc | Device Tree Compiler |

---

# 6. Device Tree Source (.dts)

Human-readable text file.

Example:

```dts
/dts-v1/;

/ {
    model = "MyBoard";
};
``` id="dts1"

---

# 7. Device Tree Blob (.dtb)

Compiled binary version loaded by bootloader.

Kernel consumes:
```text
DTB
```

during boot.

---

# 8. Device Tree Compiler (dtc)

Compiles:
```text
.dts → .dtb
```

---

## Example

```bash
dtc -I dts -O dtb board.dts -o board.dtb
``` id="dtc1"

---

# 9. Device Tree Hierarchical Structure

Tree-like structure:

```text
/
 ├── cpus
 ├── memory
 ├── soc
 │    ├── uart
 │    ├── gpio
 │    └── i2c
 └── chosen
``` id="tree1"

---

# 10. Root Node

Top-level node:

```dts
/ {
};
``` id="root1"

---

# 11. Node Structure

Example:

```dts
uart0 {
    compatible = "ns16550";
    reg = <0x10000000 0x1000>;
};
``` id="node1"

---

# 12. Device Tree Properties

| Property | Purpose |
|----------|---------|
| compatible | Driver matching |
| reg | Register address |
| interrupts | IRQ numbers |
| status | Enable/disable device |
| clocks | Clock references |

---

# 13. compatible Property

Most important property.

Example:

```dts
compatible = "ti,omap3-uart";
``` id="comp1"

Kernel uses it to:
```text
match drivers
```

---

# 14. reg Property

Defines hardware register addresses.

Example:

```dts
reg = <0x4806A000 0x1000>;
``` id="reg1"

Meaning:
- base address
- size

---

# 15. interrupts Property

Defines interrupt numbers.

Example:

```dts
interrupts = <72>;
``` id="irq1"

---

# 16. status Property

Enable/disable hardware.

Example:

```dts
status = "okay";
``` id="stat1"

Disabled:

```dts
status = "disabled";
``` id="stat2"

---

# 17. Memory Node Example

```dts
memory@80000000 {
    device_type = "memory";
    reg = <0x80000000 0x40000000>;
};
``` id="mem1"

---

# 18. CPU Node Example

```dts
cpus {
    cpu@0 {
        compatible = "arm,cortex-a53";
    };
};
``` id="cpu1"

---

# 19. UART Example

```dts
uart0: serial@10000000 {
    compatible = "ns16550a";
    reg = <0x10000000 0x1000>;
    interrupts = <5>;
};
``` id="uart1"

---

# 20. GPIO Example

```dts
gpio0: gpio@20000000 {
    compatible = "vendor,gpio";
    reg = <0x20000000 0x1000>;
};
``` id="gpio1"

---

# 21. I2C Bus Example

```dts
i2c0: i2c@30000000 {
    compatible = "vendor,i2c";
};
``` id="i2c1"

---

# 22. SPI Device Example

```dts
spi@40000000 {
    flash@0 {
        compatible = "jedec,spi-nor";
    };
};
``` id="spi1"

---

# 23. chosen Node

Contains boot parameters.

Example:

```dts
chosen {
    bootargs = "console=ttyS0";
};
``` id="chosen1"

---

# 24. aliases Node

Creates short aliases.

Example:

```dts
aliases {
    serial0 = &uart0;
};
``` id="alias1"

---

# 25. Device Tree Include Files (.dtsi)

Shared hardware definitions.

Example:

```dts
#include "soc.dtsi"
``` id="dtsi1"

---

# 26. Kernel and Device Tree Relationship

Kernel:
- parses DTB
- discovers hardware
- loads matching drivers

---

# 27. Driver Matching Process

```text
Device Tree compatible
          ↓
Kernel Driver Table
          ↓
Driver Loaded
``` id="drv1"

---

# 28. Example Driver Match

Driver:

```c
.compatible = "vendor,my-uart"
```

DT:

```dts
compatible = "vendor,my-uart";
``` id="match1"

---

# 29. Device Tree Loading in U-Boot

U-Boot loads:
```text
board.dtb
```

into RAM.

---

## Example

```bash
fatload mmc 0:1 0x82000000 board.dtb
``` id="load1"

---

# 30. Booting Kernel with DTB

Example:

```bash
bootz 0x80000000 - 0x82000000
``` id="bootz1"

---

# 31. Kernel Access to Device Tree

Kernel exposes DT under:

```text
/proc/device-tree/
``` id="proc1"

---

# 32. View Device Tree in Linux

```bash
ls /proc/device-tree
``` id="view1"

---

# 33. Device Tree Source Tree in Kernel

```text
arch/arm/boot/dts/
arch/arm64/boot/dts/
```

---

# 34. Device Tree Compilation in Kernel

Kernel automatically builds:
```text
.dtb files
```

during compilation.

---

# 35. Kernel Build Example

```bash
make dtbs
``` id="build1"

---

# 36. Device Tree Overlay

Overlays modify DT dynamically.

Common in:
- Raspberry Pi
- FPGA systems

---

# 37. Overlay Flow

```text
Base DTB
    ↓
Apply Overlay
    ↓
Modified Hardware Tree
``` id="ov1"

---

# 38. Raspberry Pi Overlay Example

```text
dtoverlay=spi0-1cs
```

in:
```text
config.txt
```

---

# 39. Interrupt Controller Example

```dts
interrupt-controller@1f000000 {
    compatible = "arm,gic";
};
``` id="gic1"

---

# 40. Clock Configuration

Example:

```dts
clocks = <&clk1>;
``` id="clk1"

---

# 41. Pin Control (pinctrl)

Controls:
- pin muxing
- electrical settings

Example:

```dts
pinctrl-0 = <&uart_pins>;
``` id="pin1"

---

# 42. DMA Configuration

Example:

```dts
dmas = <&dma0 5>;
``` id="dma1"

---

# 43. Reserved Memory

Used for:
- framebuffers
- secure memory
- DMA buffers

Example:

```dts
reserved-memory {
};
``` id="res1"

---

# 44. Common Device Tree Errors

---

## Wrong compatible string

Driver not loaded.

---

## Invalid reg address

Hardware inaccessible.

---

## Missing interrupt

Device not functional.

---

# 45. Debugging Device Tree

---

## View kernel logs

```bash
dmesg
``` id="dbg1"

---

## Inspect DTB

```bash
fdtdump board.dtb
``` id="dbg2"

---

## Decompile DTB

```bash
dtc -I dtb -O dts board.dtb
``` id="dbg3"

---

# 46. Device Tree vs ACPI

| Feature | Device Tree | ACPI |
|---------|-------------|------|
| Embedded systems | Common | Rare |
| x86 systems | Rare | Common |
| Simplicity | Simple | Complex |
| Linux embedded | Standard | Limited |

---

# 47. Embedded Linux Device Tree Flow

```text
U-Boot
   ↓
Load Kernel
   ↓
Load DTB
   ↓
Pass DTB Address
   ↓
Kernel Parses Hardware
   ↓
Initialize Drivers
``` id="emb1"

---

# 48. Advantages of Device Tree

- generic kernels
- hardware abstraction
- easier maintenance
- scalable platform support

---

# 49. Device Tree Limitations

- complex syntax
- debugging can be difficult
- hardware mistakes cause boot failures

---

# 50. Complete Embedded Linux Hardware Description Flow

```text
.dts Source
     ↓
dtc Compiler
     ↓
.dtb Binary
     ↓
U-Boot loads DTB
     ↓
Kernel parses DTB
     ↓
Drivers initialized
``` id="final1"

---

# 51. Summary

- Device Tree describes embedded hardware
- Allows generic Linux kernels
- Used heavily in ARM/ARM64 systems
- Bootloader loads DTB before kernel boot
- Kernel uses DTB for driver initialization

---

````
