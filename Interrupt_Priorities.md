# Interrupt Priorities (Complete Guide)

# Table of Contents

1. Introduction
2. What are Interrupt Priorities?
3. Why Priorities are Needed
4. Priority Levels Concept
5. Preemption and Priority
6. Hardware vs Software Priorities
7. Nested Interrupts
8. Priority Inversion in Interrupts
9. Priority Assignment Rules
10. Interrupt Priority in ARM Cortex-M (Example)
11. Interrupt Priority in RTOS
12. Advantages
13. Disadvantages
14. Common Mistakes
15. Applications
16. Interview Questions
17. Real-World Examples
18. Summary

---

# 1. Introduction

**Interrupt Priorities** define the order in which multiple interrupts are handled when they occur at the same time or while another ISR is executing.

They are essential in:

* embedded systems
* microcontrollers
* real-time operating systems (RTOS)
* safety-critical systems

---

# 2. What are Interrupt Priorities?

Interrupt priority determines:

```text id="ip1"
which interrupt gets CPU attention first
```

Higher priority interrupts can preempt lower priority ones.

---

# 3. Why Priorities are Needed

Without priorities:

* all interrupts are treated equally
* critical events may be delayed

With priorities:

✔ urgent events handled first
✔ deterministic behavior
✔ better system control

---

# 4. Priority Levels Concept

Typical model:

```text id="ip2"
Higher number → higher urgency (or vice versa depending on architecture)
```

Example:

* Priority 0 → lowest
* Priority 5 → highest

---

# 5. Preemption and Priority

If a higher priority interrupt occurs:

```text id="ip3"
it can interrupt a lower priority ISR (preemption)
```

This is called:

✔ nested interrupt handling

---

# 6. Hardware vs Software Priorities

| Type              | Description                         |
| ----------------- | ----------------------------------- |
| Hardware priority | Defined by CPU/interrupt controller |
| Software priority | Defined by OS/RTOS configuration    |

---

# 7. Nested Interrupts

Higher priority interrupts can interrupt lower ones:

```text id="ip4"
ISR(low) → interrupted by ISR(high)
```

This improves responsiveness.

---

# 8. Priority Inversion in Interrupts

Occurs when:

* high priority interrupt is delayed
* due to system masking or lower ISR blocking

Causes:

❌ latency spikes
❌ missed deadlines

---

# 9. Priority Assignment Rules

Good design principles:

✔ assign highest priority to real-time critical events
✔ keep ISR execution time short
✔ avoid blocking inside ISR
✔ group similar urgency interrupts

---

# 10. Interrupt Priority in ARM Cortex-M (Example)

ARM Cortex-M uses:

* NVIC (Nested Vectored Interrupt Controller)
* configurable priority levels

Example concept:

```text id="ip5"
0 = highest priority
255 = lowest priority (implementation dependent)
```

---

# 11. Interrupt Priority in RTOS

RTOS handles:

* task priorities
* interrupt priorities

Rule:

```text id="ip6"
ISR priority > task priority (for critical events)
```

But must avoid conflicts with scheduler.

---

# 12. Advantages

✔ faster response for critical events
✔ deterministic behavior
✔ better system control
✔ supports real-time constraints

---

# 13. Disadvantages

❌ complex configuration
❌ risk of starvation of low-priority interrupts
❌ priority inversion issues
❌ debugging difficulty

---

# 14. Common Mistakes

❌ assigning all interrupts high priority
❌ long ISR execution at high priority
❌ ignoring nested interrupt risks
❌ improper RTOS configuration

---

# 15. Applications

* automotive ECU systems
* robotics control
* industrial automation
* aerospace systems
* microcontroller-based embedded systems
* communication devices

---

# 16. Interview Questions

### Q1. What are interrupt priorities?

A mechanism to decide which interrupt is handled first.

---

### Q2. What is nested interrupt?

A higher priority interrupt preempting a lower one.

---

### Q3. Why are priorities needed?

To ensure critical events are handled first.

---

### Q4. What happens if priorities are misconfigured?

System delays, missed interrupts, or starvation.

---

### Q5. Hardware vs software priority?

Hardware is CPU-defined; software is OS-configured.

---

# 17. Real-World Examples

* ABS braking system (high priority sensor interrupts)
* airbag deployment system
* industrial emergency stop systems
* network packet interrupts in routers
* motor control systems in robotics

---

# 18. Summary

Interrupt priorities define:

```text id="ip7"
the order in which multiple interrupts are serviced
```

Key points:

✔ ensures deterministic real-time response
✔ supports nested interrupts
✔ critical in embedded systems
✔ must be carefully configured

They are essential in:

**RTOS design, embedded systems, and safety-critical computing environments**
