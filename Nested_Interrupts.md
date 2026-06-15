# Nested Interrupts 

# Table of Contents

1. Introduction
2. What are Nested Interrupts?
3. Why Nested Interrupts are Needed
4. Basic Working Principle
5. Interrupt Nesting Flow
6. Priority-Based Nesting
7. Hardware Support for Nesting
8. Software Handling
9. Nested Interrupt Example
10. Advantages
11. Disadvantages
12. Common Mistakes
13. Applications
14. Interview Questions
15. Real-World Examples
16. Summary

---

# 1. Introduction

**Nested Interrupts** are a mechanism where an interrupt can be interrupted by another interrupt of higher priority.

This is widely used in:

* embedded systems
* microcontrollers
* real-time operating systems (RTOS)
* safety-critical applications

---

# 2. What are Nested Interrupts?

Nested interrupts allow:

```text id="ni1"
a higher-priority interrupt to interrupt a currently executing lower-priority ISR
```

---

# 3. Why Nested Interrupts are Needed

Without nesting:

* high-priority events must wait for low-priority ISR to finish
* response time becomes slow

With nesting:

✔ critical events are handled immediately
✔ better real-time responsiveness
✔ improved system determinism

---

# 4. Basic Working Principle

When an interrupt occurs:

```text id="ni2"
CPU starts ISR (low priority)
→ higher priority interrupt arrives
→ CPU pauses current ISR
→ executes higher priority ISR
→ resumes previous ISR
```

---

# 5. Interrupt Nesting Flow

Example:

```text id="ni3"
Main program
  ↓
ISR (Timer interrupt)
  ↓ (paused)
ISR (UART interrupt - higher priority)
  ↓
Return to Timer ISR
  ↓
Return to Main
```

---

# 6. Priority-Based Nesting

Nesting is controlled by priority levels:

| Interrupt | Priority |
| --------- | -------- |
| UART      | High     |
| Timer     | Medium   |
| GPIO      | Low      |

Rule:

```text id="ni4"
Higher priority interrupts can preempt lower priority ISRs
```

---

# 7. Hardware Support for Nesting

Most modern CPUs support:

✔ interrupt priority registers
✔ nested vectored interrupt controller (NVIC)
✔ automatic context saving

Example:

* ARM Cortex-M uses NVIC for nesting

---

# 8. Software Handling

Software responsibilities:

* enable interrupt nesting
* manage shared resources safely
* avoid race conditions
* minimize ISR execution time

---

# 9. Nested Interrupt Example

Scenario:

1. Timer ISR is running
2. UART interrupt occurs
3. UART ISR preempts Timer ISR
4. UART ISR completes
5. Timer ISR resumes

```text id="ni5"
This ensures UART (urgent) is handled immediately
```

---

# 10. Advantages

✔ faster response for critical interrupts
✔ better real-time performance
✔ improved system efficiency
✔ supports priority-based scheduling

---

# 11. Disadvantages

❌ complex system design
❌ harder debugging
❌ risk of stack overflow
❌ race conditions in shared data
❌ increased latency for low-priority ISR completion

---

# 12. Common Mistakes

❌ deep nesting without limits
❌ long ISR execution
❌ not protecting shared resources
❌ ignoring stack usage
❌ enabling nesting without priority design

---

# 13. Applications

* real-time operating systems
* automotive ECUs
* robotics control systems
* aerospace systems
* industrial automation
* communication systems

---

# 14. Interview Questions

### Q1. What are nested interrupts?

Interrupts that can interrupt other lower-priority interrupts.

---

### Q2. Why are they used?

To improve response time for high-priority events.

---

### Q3. What controls nesting?

Interrupt priority levels.

---

### Q4. What is the risk?

Stack overflow and race conditions.

---

### Q5. Which systems use them?

Embedded systems and RTOS-based systems.

---

# 15. Real-World Examples

* airbag deployment interrupt overriding sensor ISR
* CPU handling network packet interrupt during timer ISR
* motor control emergency stop interrupt
* robotic collision detection interrupt
* industrial safety shutdown systems

---

# 16. Summary

Nested interrupts allow:

```text id="ni6"
higher-priority interrupts to preempt lower-priority ISRs
```

Key points:

✔ improves responsiveness
✔ priority-based execution
✔ widely used in RTOS and embedded systems
✔ must be carefully designed

They are essential in:

**real-time, safety-critical, and embedded computing systems**
