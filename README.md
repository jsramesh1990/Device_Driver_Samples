
<div align="center">

#  Linux Device Driver Samples

### Comprehensive Linux Kernel Module & Device Driver Examples for Learning and Development

<p>
  <img src="https://img.shields.io/badge/Linux-Kernel-yellow?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/Language-C-blue?style=for-the-badge&logo=c" />
  <img src="https://img.shields.io/badge/Build-Make-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-success?style=for-the-badge" />
</p>

<p>
A curated collection of Linux device driver examples covering kernel modules, character drivers, procfs, ioctl, polling, interrupts, PCI, USB, DMA, block drivers, network drivers, TTY, and more.
</p>

</div>

---

##  Overview

This repository provides **hands-on Linux device driver examples** designed for:

- Beginners learning Linux kernel programming
- Embedded Linux developers
- Device driver engineers
- Interview preparation
- Reference implementations for real projects

The examples progress from **basic kernel modules** to **advanced driver development concepts**.

---

##  Features

 32+ practical examples  
 Progressive learning structure  
 Linux kernel module development  
 Character driver implementation  
 procfs/sysfs examples  
 Interrupt handling  
 PCI & USB driver skeletons  
 DMA programming  
 Block device drivers  
 Network device drivers  
 TTY subsystem examples  
 Red-black tree implementation  

---

#  Repository Structure

```bash
Device-Driver-samples/
│
├── 0_index/
├── 1_Introduction_of_Linux_and_Device_Drivers/
├── 2_start_with_driver_development/
│
├── ex_1_hello_world/
├── ex_2_module_parameters/
├── ex_3_scull_basic/
├── ex_4_proc_fs_basic/
├── ex_5_proc_fs_iterator/
├── ex_6_completion/
├── ex_7_ioctl/
├── ex_8_pipe_sleep/
├── ex_9_pipe_sleep/
├── ex_10_polling/
├── ex_11_asynchronous_tech/
├── ex_12_seeking/
├── ex_13_jit/
├── ex_14_jiq/
├── ex_15_short_ioporting/
├── ex_16_short_port_brief/
├── ex_17_short_mmio/
├── ex_18_short_IRQ/
├── ex_19_short_IRQ_probe/
├── ex_20_pci_skel/
├── ex_21_usb_skel/
├── ex_22_lddbus/
├── ex_23_example/
├── ex_24_copy_zero/
├── ex_25_async_io/
├── ex_26_msi_interrupt/
├── ex_27_DMA/
├── ex_28_DMA_sg_/
├── ex_29_block_device/
├── ex_30_netdev/
├── ex_31_TTY/
├── ex_32_rbtree/
│
├── bin/
├── LICENSE
└── README.md
```

---

#  Learning Path

Follow examples in order:

## Beginner Level

| Example | Topic |
|---|---|
| ex_1 | Hello World Module |
| ex_2 | Module Parameters |
| ex_3 | SCULL Character Driver |
| ex_4 | Basic procfs |
| ex_5 | procfs Iterator |

---

## Intermediate Level

| Example | Topic |
|---|---|
| ex_6 | Completion |
| ex_7 | IOCTL |
| ex_8 | Sleep & Wakeup |
| ex_10 | Polling |
| ex_11 | Async Notification |
| ex_12 | Seeking |
| ex_13 | JIT |
| ex_14 | JIQ |

---

## Advanced Level

| Example | Topic |
|---|---|
| ex_17 | MMIO |
| ex_18 | Interrupt Handling |
| ex_19 | IRQ Probe |
| ex_20 | PCI Skeleton |
| ex_21 | USB Skeleton |
| ex_26 | MSI Interrupts |
| ex_27 | DMA |
| ex_28 | Scatter-Gather DMA |

---

## Expert Level

| Example | Topic |
|---|---|
| ex_29 | Block Device Driver |
| ex_30 | Network Device Driver |
| ex_31 | TTY Driver |
| ex_32 | Red-Black Tree |

---

#  Prerequisites

## System Requirements

- Linux Ubuntu 18.04+
- GCC 7+
- Make
- Kernel headers matching running kernel
- Root privileges

---

## Install Dependencies

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r)
```

---

#  Getting Started

## Clone Repository

```bash
git clone https://github.com/jsramesh1990/Device-Driver-samples.git
cd Device-Driver-samples
```

---

## Build Example

Example:

```bash
cd ex_1_hello_world
make
```

---

## Load Module

```bash
sudo insmod hello.ko
```

Check logs:

```bash
dmesg | tail
```

---

## Unload Module

```bash
sudo rmmod hello
```

---

#  Driver Development Workflow

```mermaid
flowchart LR
    A[Write Driver Code] --> B[Build Module]
    B --> C[Insert Module]
    C --> D[Test Driver]
    D --> E[Check dmesg Logs]
    E --> F[Remove Module]
```

---

#  Common Commands

## Module Info

```bash
modinfo module.ko
```

## Loaded Modules

```bash
lsmod
```

## Kernel Logs

```bash
dmesg -w
```

## Device Nodes

```bash
ls /dev
```

---

#  Topics Covered

- Kernel Modules
- Character Drivers
- procfs
- sysfs
- IOCTL
- Polling
- Async Notification
- Interrupt Handling
- MMIO
- PCI Drivers
- USB Drivers
- DMA
- MSI
- Block Drivers
- Network Drivers
- TTY Drivers
- Kernel Data Structures

---

#  Troubleshooting

## Invalid Module Format

```bash
uname -r
```

Ensure headers match kernel.

---

## Permission Denied

Use:

```bash
sudo insmod module.ko
```

---

## Module Already Loaded

```bash
sudo rmmod module_name
```

---

#  References

Recommended resources:

- Linux Device Drivers (LDD3)
- Linux Kernel Documentation
- Kernel source tree documentation

---

#  Contributing

Contributions are welcome.

```bash
Fork → Clone → Improve → Pull Request
```

---

#  Author

Linux Kernel | Embedded Systems | Device Drivers

---

<div align="center">

### ⭐ Star this repository if it helps your Linux kernel journey

</div>
````




