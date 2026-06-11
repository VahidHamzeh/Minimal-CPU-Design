# Minimal 10-bit CPU Design & Embedded Implementation

A custom-designed 10-bit CPU architecture engineered and simulated using the **Digital** logic design tool. This project showcases hardware-software co-design, optimized control units, instruction set expansion, and rigorous low-level assembly debugging to solve practical hardware constraints.

---

## 🛠️ System Architecture Overview

* **Data Path:** Custom 10-bit Instruction Set Architecture (ISA).
* **Word Size:** 8-bit registers and ALU operands.
* **Control Unit:** Custom micro-programmed/combinational control unit modified to support dynamic peripheral parsing.
* **Memory Map:** Harvard Architecture inspired separated Instruction ROM and Data memory.

---

## 🚀 Tasks & Implementation Breakdown

### Task I.1: Underflow Protection & ALU Correction
* **Objective:** Read an external stimulus (button input) into a register, subtract a constant threshold ($39_{10}$), and robustly display the output on a 7-Segment display.
* **Engineering Insight:** Handled 8-bit Two's Complement underflow edge cases to guarantee stable physical outputs during negative subtraction boundaries.

### Task I.2: Accumulative Double-and-Add Arithmetic
* **Objective:** Sequential multi-step arithmetic parsing. The program fetches an initial button entry, duplicates its value via ALU addition (`R3 = R0 + R1`), reads a second independent peripheral input, and accumulates them into a final total.
* **Execution Strategy:** Optimized temporary registers to prevent data hazards and arithmetic overwrites before latching the final state onto the display.

### Task I.3 & I.4: Iterative Fibonacci Generation & Overflow Analysis
* **Objective:** Calculate the classic Fibonacci sequence iteratively. Task I.3 computes up to the 5th term, while Task I.4 attempts to loop infinitely ("Fibonacci Forever").
* **The "Hardware Limit" Bug & Discovery:** An architectural analysis reveals that the loop is **not infinite**. Due to the strict 8-bit register layout, an integer overflow triggers at the $13\text{th}$ term ($144 + 233 = 377 \rightarrow 121$ in unsigned 8-bit). 
  * If the comparator behaves as **Unsigned**, the relation $R0 > R1$ ($121 > 233$) evaluates to *False*, instantly breaking the jump loop.
  * If treated as **Signed**, $144_{10}$ turns into $-112_{10}$ via Two's Complement, violating the conditional loop parameter immediately.

### Task I.5: Interactive State Lock ("Wait For Me")
* **Objective:** Initialize the hardware display at `04`. Poll an external input continuously, holding the program counter inside a strict boundary loop until the precise input matches $27_{10}$ ($0x1B$). Upon matching, break the loop, jump to a terminal address, and lock the display on $83_{10}$ (`53` in Hex).
* **Algorithmic Trick:** Since the CPU hardware only implemented a Greater-Than (`>`) comparator, exact equality (`== 27`) was achieved purely in software using a **Double-Boundary Condition Check** (verifying that neither $\text{Input} > 27$ nor $27 > \text{Input}$ is true).

---

## 📊 Live Demos & Simulations

### Task I.4: Fibonacci Sequence Infinite Loop & Overflow Latch
Below is the live execution of the iterative arithmetic sequence running on the 10-bit datapath, showcasing real-time register updates and structural clock cycles:

<p align="center">
  <img src="assets/fibonacci-forever.gif" alt="Fibonacci Forever Simulation Demo" width="800" height="456">
</p>

### Task I.5: Conditional Lock ("Wait For Me") Dynamic Verification
Demonstrating real-time peripheral polling, execution of the double-boundary conditional check, and jumping to the terminal safe state upon registering exact match criteria:

<p align="center">
  <img src="assets/wait-for-me.gif" alt="Wait For Me Simulation Demo" width="800" height="456">
</p>

---

## 📝 Technical Documentation & Memory Maps

### Task I.5 Assembly-to-Machine-Code Register Mapping

| ROM Addr | Hex Code | My Assembly | Explanation | Assembly To Binary |
| :--- | :--- | :--- | :--- | :--- |
| **0x00** | 104 | `IMM 4` | Load initial target value 4 into R2 | `# 0100000100` |
| **0x01** | 220 | `MOV R2, R0` | Move 4 to R0 for display | `# 1000100000` |
| **0x02** | 307 | `ADD R7` | Trigger 7-seg to display 04 | `# 1100000111` |
| **0x03** | 11B | `IMM 27` | Load target threshold 27 (0x1B) into R2 | `# 0100111011` |
| **0x04** | 255 | `MOV R5, R5` | **[Loop Start]** Read input from button into R5 | `# 1001010101` |
| **0x05** | 221 | `MOV R2, R1` | Move target 27 to R1 (for Condition A check) | `# 1000100001` |
| **0x06** | 250 | `MOV R5, R0` | Move input to R0 (for Condition A check) | `# 1001010000` |
| **0x07** | 004 | `CNDR 4` | Check if Input > 27. If true, jump back to PC=0x04 | `# 0000000100` |
| **0x08** | 251 | `MOV R5, R1` | Move input to R1 (for Condition B check) | `# 1001010001` |
| **0x09** | 220 | `MOV R2, R0` | Move target 27 to R0 (for Condition B check) | `# 1000100000` |
| **0x0A** | 004 | `CNDR 4` | Check if 27 > Input. If true, jump back to PC=0x04 | `# 0000000100` |
| **0x0B** | 153 | `IMM 83` | **[Exit Loop]** Input is exactly 27! Load value 83 | `# 0101010011` |
| **0x0C** | 220 | `MOV R2, R0` | Move 83 to R0 | `# 1000100000` |
| **0x0D** | 307 | `ADD R7` | Trigger 7-seg to update and display 53 (83 Dec) | `# 1100000111` |
| **0x0E** | 00E | `CNDR 14` | Infinite loop: Jump to PC=0x0E (Locks CPU on 83) | `# 0000001110` |

---
*Developed as part of the Computer Architecture & Digital Logic Design Laboratory at Sharif University of Technology.*
