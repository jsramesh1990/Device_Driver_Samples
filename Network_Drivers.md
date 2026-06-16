# Network Drivers (Complete Guide)

# Table of Contents

1. Introduction
2. What is a Network Driver?
3. Why Network Drivers are Needed
4. Network Driver Architecture
5. Network Stack Overview
6. TX (Transmit) Path
7. RX (Receive) Path
8. Network Interface Card (NIC)
9. DMA in Network Drivers
10. Interrupt Handling
11. NAPI (New API)
12. Ring Buffers
13. Socket to NIC Flow
14. User Space vs Kernel Space
15. Linux Network Driver Framework
16. Important Data Structures
17. Synchronization
18. Performance Optimizations
19. Advantages
20. Disadvantages
21. Common Mistakes
22. Applications
23. Interview Questions
24. Summary

---

# 1. Introduction

A **Network Driver** is a device driver that enables the Operating System to communicate with network hardware such as:

* Ethernet Controllers
* WiFi Chips
* CAN Controllers
* Bluetooth Modules
* Cellular Modems
* Fiber NICs

The driver acts as a bridge:

```text
Applications
      ↓
Socket API
      ↓
TCP/IP Stack
      ↓
Network Driver
      ↓
NIC Hardware
      ↓
Network
```

---

# 2. What is a Network Driver?

A Network Driver is:

```text
Software that controls a network interface card (NIC) and transfers packets between the operating system and the network hardware.
```

Responsibilities:

✔ Initialize NIC

✔ Configure DMA

✔ Manage interrupts

✔ Transmit packets

✔ Receive packets

✔ Link management

✔ Error handling

---

# 3. Why Network Drivers are Needed

Without a network driver:

❌ OS cannot send packets

❌ OS cannot receive packets

❌ No internet connectivity

❌ NIC hardware remains inaccessible

---

Network drivers provide:

✔ Hardware abstraction

✔ Packet transfer

✔ Interrupt handling

✔ DMA support

✔ High-speed networking

---

# 4. Network Driver Architecture

```text
+----------------------+

| User Applications    |

| Browser, SSH, FTP    |

+----------------------+

          |

          v

+----------------------+

| Socket API           |

+----------------------+

          |

          v

+----------------------+

| TCP/IP Stack         |

+----------------------+

          |

          v

+----------------------+

| Network Driver       |

+----------------------+

          |

          v

+----------------------+

| DMA Engine           |

+----------------------+

          |

          v

+----------------------+

| NIC Hardware         |

+----------------------+

          |

          v

+----------------------+

| Ethernet/WiFi Cable  |

+----------------------+
```

---

# 5. Network Stack Overview

Linux networking stack:

```text
Application

↓

Socket API

↓

TCP

↓

IP

↓

ARP

↓

Ethernet

↓

Network Driver

↓

NIC

↓

Network
```

---

# 6. TX (Transmit) Path

When application sends data:

```c
send(sock,buf,size,0);
```

Flow:

```text
Application

↓

Socket API

↓

TCP

↓

IP

↓

Ethernet Header Added

↓

Network Driver

↓

DMA Maps Buffer

↓

NIC

↓

Wire
```

---

Driver responsibilities:

✔ allocate descriptors

✔ DMA map memory

✔ update TX ring

✔ notify NIC

---

# 7. RX (Receive) Path

Packet arrives:

```text
Network

↓

NIC

↓

DMA to RAM

↓

Interrupt

↓

ISR

↓

NAPI Poll

↓

TCP/IP Stack

↓

Socket Buffer

↓

Application
```

---

Driver responsibilities:

✔ allocate RX buffers

✔ handle DMA

✔ receive interrupt

✔ pass packet to kernel

---

# 8. Network Interface Card (NIC)

NIC hardware includes:

```text
MAC

↓

PHY

↓

DMA Engine

↓

TX Ring

↓

RX Ring

↓

Interrupt Controller
```

---

### MAC

Responsible for:

✔ framing

✔ CRC

✔ packet transmission

---

### PHY

Responsible for:

✔ electrical signaling

✔ speed negotiation

✔ link detection

---

# 9. DMA in Network Drivers

DMA is essential.

Without DMA:

```text
NIC

↓

CPU copies packet

↓

RAM
```

Slow.

---

With DMA:

```text
NIC

↓

DMA Engine

↓

RAM
```

CPU only:

```text
Setup DMA

↓

Receive Interrupt

↓

Process Packet
```

---

Benefits:

✔ Low CPU usage

✔ High throughput

✔ Zero-copy support

---

# 10. Interrupt Handling

NIC generates interrupt:

```text
Packet Received

↓

NIC IRQ

↓

ISR

↓

Disable IRQ

↓

Schedule NAPI

↓

Return
```

ISR should:

✔ be short

✔ clear interrupt

✔ avoid packet processing

✔ schedule deferred work

---

# 11. NAPI (New API)

Linux uses:

```text
NAPI
```

to reduce interrupt overhead.

---

Traditional:

```text
Packet

↓

Interrupt

↓

ISR

↓

Packet Process
```

Many packets:

```text
100000 packets

↓

100000 interrupts
```

Expensive.

---

NAPI:

```text
Interrupt

↓

Disable IRQ

↓

Poll Packets

↓

Enable IRQ
```

---

Benefits:

✔ fewer interrupts

✔ high throughput

✔ lower CPU usage

---

# 12. Ring Buffers

NIC uses:

---

## TX Ring

```text
Head

↓

[P1][P2][P3][ ]

          ↑

         Tail
```

Packets waiting to send.

---

## RX Ring

```text
Head

↓

[ ][ ][ ]

↓

DMA fills

↓

[P1][P2][P3]
```

Packets received.

---

Benefits:

✔ lock-free

✔ efficient

✔ continuous streaming

---

# 13. Socket to NIC Flow

Example:

```c
send(sock,buf,1500,0);
```

Flow:

```text
User App

↓

send()

↓

Socket Layer

↓

TCP

↓

IP

↓

Ethernet

↓

Network Driver

↓

DMA Descriptor

↓

NIC

↓

Cable
```

---

# 14. User Space vs Kernel Space

---

## User Space

Applications:

```text
Browser

SSH

FTP

MQTT Client
```

Cannot:

❌ access NIC registers

---

## Kernel Space

Contains:

✔ TCP/IP Stack

✔ Network Driver

✔ DMA

✔ Interrupts

---

# 15. Linux Network Driver Framework

Driver registers:

```c
struct net_device
```

and:

```c
struct net_device_ops
```

Example:

```c
ndo_open()

ndo_stop()

ndo_start_xmit()

ndo_set_mac_address()
```

---

Register:

```c
register_netdev();
```

Unregister:

```c
unregister_netdev();
```

---

# 16. Important Data Structures

---

## struct net_device

Represents:

```text
eth0

wlan0

can0
```

Contains:

✔ MAC address

✔ MTU

✔ queues

✔ flags

---

## struct sk_buff (skb)

Packet container.

Contains:

```text
Ethernet Header

IP Header

TCP Header

Payload
```

Linux packets are stored in:

```text
skb
```

---

# 17. Synchronization

Network drivers use:

---

## Spinlock

Used for:

✔ TX queue

✔ RX queue

✔ interrupt protection

---

## Atomic Variables

Used for:

✔ packet counters

✔ statistics

---

## RCU

Used for:

✔ routing tables

✔ neighbor tables

---

# 18. Performance Optimizations

Modern NICs support:

---

### Interrupt Coalescing

Combine interrupts.

```text
100 Packets

↓

1 Interrupt
```

---

### Checksum Offloading

NIC computes:

✔ IP checksum

✔ TCP checksum

✔ UDP checksum

CPU saved.

---

### TSO

TCP Segmentation Offload.

NIC splits packets.

---

### LRO/GRO

Combine packets.

Less CPU overhead.

---

### RSS

Receive Side Scaling.

Distributes packets:

```text
CPU0

CPU1

CPU2

CPU3
```

Improves multicore performance.

---

# 19. Advantages

✔ High throughput

✔ DMA support

✔ Interrupt driven

✔ Scalable

✔ Low CPU overhead

✔ Multi-queue support

---

# 20. Disadvantages

❌ Complex implementation

❌ Race conditions

❌ DMA synchronization issues

❌ Interrupt storms

❌ Difficult debugging

---

# 21. Common Mistakes

❌ Not freeing skb

❌ Missing DMA unmap

❌ Improper ring handling

❌ Long ISR

❌ Missing memory barriers

❌ Race conditions

❌ TX timeout handling errors

---

# 22. Applications

Network drivers are used in:

* Ethernet Controllers
* WiFi Adapters
* CAN Controllers
* Bluetooth Modules
* LTE Modems
* Fiber NICs
* Industrial Ethernet
* Routers
* Switches
* IoT Devices
* Automotive ECUs

---

# 23. Interview Questions

### Q1. What is a Network Driver?

A driver that transfers packets between OS and NIC hardware.

---

### Q2. What is NAPI?

Linux mechanism to reduce interrupt overhead by polling packets.

---

### Q3. Why DMA is used?

To transfer packets directly between NIC and RAM without CPU copying.

---

### Q4. What is skb?

Linux packet buffer structure.

---

### Q5. What is TX Ring?

Queue of packets waiting for transmission.

---

### Q6. What is RX Ring?

Queue of buffers receiving packets from NIC.

---

### Q7. What is Checksum Offload?

NIC computes checksums instead of CPU.

---

# 24. Summary

Network Drivers are:

```text
Device drivers that transfer network packets between the operating system and network hardware.
```

Core responsibilities:

✔ NIC initialization

✔ Packet TX/RX

✔ DMA management

✔ Interrupt handling

✔ NAPI polling

✔ Ring buffer management

✔ Checksum offloading

✔ Link management

They are fundamental to:

**Ethernet, WiFi, CAN, Bluetooth, LTE Modems, Linux Networking Stack, Routers, Switches, Data Centers, Cloud Servers, Automotive ECUs, and Embedded Networking Systems.**
