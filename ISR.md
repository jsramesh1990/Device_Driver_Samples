# Interrupt Service Routine (ISR)

# Table of Contents

1. Introduction
2. What is an ISR?
3. Why ISRs are Used
4. Interrupt vs Polling
5. How ISR Works
6. ISR Flow (Step-by-Step)
7. Types of Interrupts
8. ISR Design Rules
9. ISR vs Normal Function
10. Nested Interrupts
11. Common Mistakes in ISR Design
12. Advantages
13. Disadvantages
14. Applications
15. Interview Questions
16. Real-World Examples
17. Summary

---

# 1. Introduction

An **Interrupt Service Routine (ISR)** is a special function in embedded systems and operating systems that executes automatically in response to an interrupt event.

It is a core concept in:

* embedded systems
* microcontrollers
* operating systems
* real-time systems

---

# 2. What is an ISR?

An ISR is:

```text id="isr1"
a function that runs when an interrupt occurs
```

It temporarily pauses the main program to handle urgent events.

---

# 3. Why ISRs are Used

ISRs are used to handle:

✔ time-critical events
✔ hardware signals
✔ external inputs
✔ system exceptions

Without ISR:

```text id="isr2"
CPU would continuously check devices (inefficient polling)
```

---

# 4. Interrupt vs Polling

| Feature     | Interrupt (ISR) | Polling           |
| ----------- | --------------- | ----------------- |
| CPU usage   | Efficient       | Wastes CPU cycles |
| Response    | Fast            | Delayed           |
| Power usage | Low             | High              |
| Complexity  | Higher          | Lower             |

---

# 5. How ISR Works

Basic idea:

```text id="isr3"
Hardware event → Interrupt signal → CPU pauses → ISR executes → Resume main program
```

---

# 6. ISR Flow (Step-by-Step)

1. Interrupt occurs
2. CPU completes current instruction
3. Context is saved
4. ISR is executed
5. Interrupt flag cleared
6. Context restored
7. Program resumes

---

# 7. Types of Interrupts

## 1. Hardware Interrupt

Triggered by external devices:

* keyboard
* sensors
* timers

## 2. Software Interrupt

Triggered by software instructions.

---

# 8. ISR Design Rules

✔ keep ISR short
✔ avoid heavy computation
✔ avoid blocking calls
✔ avoid dynamic memory allocation
✔ use flags for deferred processing

---

# 9. ISR vs Normal Function

| Feature          | ISR        | Normal Function |
| ---------------- | ---------- | --------------- |
| Trigger          | Automatic  | Manual call     |
| Execution time   | Very short | Can be long     |
| Priority         | High       | Normal          |
| Blocking allowed | No         | Yes             |

---

# 10. Nested Interrupts

Some systems allow:

```text id="isr4"
higher priority interrupt interrupts current ISR
```

Used in real-time systems.

---

# 11. Common Mistakes in ISR Design

❌ doing heavy computation
❌ using printf/logging inside ISR
❌ enabling blocking operations
❌ forgetting to clear interrupt flag
❌ sharing unsafe global data

---

# 12. Advantages

✔ fast response to events
✔ efficient CPU usage
✔ reduces polling overhead
✔ essential for real-time systems

---

# 13. Disadvantages

❌ complexity in design
❌ debugging difficulty
❌ race conditions possible
❌ stack/context overhead

---

# 14. Applications

* embedded controllers
* microcontrollers (ARM, AVR, PIC)
* operating systems kernel
* robotics systems
* automotive systems
* communication devices

---

# 15. Interview Questions

### Q1. What is an ISR?

A function executed in response to an interrupt.

---

### Q2. Why are ISRs important?

They handle real-time hardware events efficiently.

---

### Q3. Can ISR call normal functions?

Yes, but only lightweight ones.

---

### Q4. Difference between ISR and polling?

ISR is event-driven; polling is continuous checking.

---

### Q5. Why must ISR be short?

To avoid delaying other interrupts and system tasks.

---

# 16. Real-World Examples

* pressing a keyboard key
* receiving network packet
* timer tick in OS
* sensor trigger in robots
* car airbag deployment signal

---

# 17. Summary

An Interrupt Service Routine (ISR) is:

```text id="isr5"
a special function that handles hardware/software interrupts immediately
```

Key properties:

✔ event-driven execution
✔ very fast response
✔ critical for real-time systems
✔ must be short and efficient

It is widely used in:

**embedded systems, operating systems, and hardware control applications**
