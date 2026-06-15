# Interrupt Stack

# Table of Contents

1. Introduction
2. What is an Interrupt Stack?
3. Why Interrupt Stack is Needed
4. How Interrupt Stack Works
5. Stack Usage During Interrupts
6. Context Saving and Restoring
7. Main Stack vs Process Stack
8. Nested Interrupt Stack Behavior
9. Stack Overflow in Interrupts
10. Stack Size Calculation
11. Interrupt Stack in RTOS vs Bare Metal
12. Advantages
13. Disadvantages
14. Common Mistakes
15. Applications
16. Interview Questions
17. Real-World Examples
18. Summary

---

# 1. Introduction

An **Interrupt Stack** is a dedicated memory stack used by the CPU to save execution context when an interrupt occurs.

It is a core concept in:

* embedded systems
* operating systems
* microcontrollers
* real-time systems

---

# 2. What is an Interrupt Stack?

An interrupt stack is:

```text id="is1"
a stack region used to store CPU state during interrupt handling
```

It stores temporary data required to resume normal execution.

---

# 3. Why Interrupt Stack is Needed

When an interrupt occurs:

* CPU must pause current execution
* save registers, program counter, flags
* execute ISR
* restore state

This requires a stack.

---

# 4. How Interrupt Stack Works

Flow:

```text id="is2"
Interrupt occurs
→ CPU pushes context to stack
→ ISR executes
→ CPU pops context from stack
→ normal execution resumes
```

---

# 5. Stack Usage During Interrupts

The stack stores:

✔ program counter (PC)
✔ CPU registers
✔ status flags
✔ return address

---

# 6. Context Saving and Restoring

### Save (on interrupt entry):

```text id="is3"
Push registers → store CPU state
```

### Restore (on return):

```text id="is4"
Pop registers → restore CPU state
```

---

# 7. Main Stack vs Process Stack

| Type          | Usage                  |
| ------------- | ---------------------- |
| Main Stack    | OS kernel / interrupts |
| Process Stack | User-level tasks       |

Example (ARM systems):

* MSP (Main Stack Pointer) → interrupts
* PSP (Process Stack Pointer) → threads

---

# 8. Nested Interrupt Stack Behavior

When interrupts are nested:

```text id="is5"
Each ISR call pushes new context onto stack
```

This increases stack usage rapidly.

---

# 9. Stack Overflow in Interrupts

Risk:

```text id="is6"
Too many nested interrupts → stack overflow
```

Causes:

❌ deep ISR nesting
❌ large local variables in ISR
❌ insufficient stack allocation

---

# 10. Stack Size Calculation

Factors:

✔ max ISR depth
✔ number of nested interrupts
✔ register save size
✔ RTOS task switching overhead

Rule of thumb:

```text id="is7"
Stack size = worst-case nesting depth × context size
```

---

# 11. Interrupt Stack in RTOS vs Bare Metal

| System        | Stack Behavior                       |
| ------------- | ------------------------------------ |
| Bare Metal    | Single interrupt stack               |
| RTOS          | Separate stacks per task + ISR stack |
| Advanced RTOS | multiple controlled stacks           |

---

# 12. Advantages

✔ ensures safe context switching
✔ supports nested interrupts
✔ enables multitasking
✔ essential for RTOS operation

---

# 13. Disadvantages

❌ memory overhead
❌ stack overflow risk
❌ difficult debugging
❌ complex stack management in RTOS

---

# 14. Common Mistakes

❌ allocating small stack size
❌ using large variables inside ISR
❌ ignoring nested interrupt depth
❌ not accounting for RTOS overhead
❌ stack corruption due to race conditions

---

# 15. Applications

* microcontroller interrupt handling
* real-time operating systems
* embedded device firmware
* robotics control systems
* automotive ECUs
* aerospace systems

---

# 16. Interview Questions

### Q1. What is interrupt stack?

A memory stack used to store CPU state during interrupt handling.

---

### Q2. Why is it important?

It allows safe execution and return after ISR.

---

### Q3. What causes stack overflow in interrupts?

Deep nesting and large ISR context.

---

### Q4. Difference between main stack and process stack?

Main stack is used for interrupts; process stack is used for tasks.

---

### Q5. How to prevent stack overflow?

Limit ISR nesting and optimize stack usage.

---

# 17. Real-World Examples

* ARM Cortex-M interrupt handling
* RTOS task switching systems
* automotive control ECUs
* industrial automation controllers
* sensor-based embedded systems

---

# 18. Summary

The interrupt stack is:

```text id="is8"
a dedicated memory stack used to store CPU state during interrupts
```

Key points:

✔ enables safe ISR execution
✔ supports nested interrupts
✔ critical for RTOS and embedded systems
✔ must be carefully sized

It is essential in:

**embedded systems, real-time operating systems, and low-level hardware control**
