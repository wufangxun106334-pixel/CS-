# COMP30660 Mock Examination

> **Module**: Computer Architecture & Organisation
> **Time Allowed**: 120 minutes
> **Instructions**: Answer any 5 of the 7 questions. All questions carry 20 marks. The paper is marked out of 100 in total.

---

## Q1. Digital Circuits (20 marks)

This question focuses on digital circuits.

### (a) Explain what a **transistor** is and how it functions as a switch in CMOS technology. (5 marks)

### (b) With the aid of a truth table, explain the operation of a 2-input NAND gate. (8 marks)

### (c) Explain static and dynamic power consumption in digital circuits. Refer to leakage current, short-circuit current and capacitive switching. (7 marks)

---

## Q2. Data Representation (20 marks)

This question focuses on data representatio
![[Pasted image 20260510204334.png|464]]
### (a) Convert the following unsigned binary number to decimal: `11010110₂` (2 marks)

### (b) Convert the following decimal number to 8-bit binary: `200₁₀` (2 marks)

### (c) Does the following calculation overflow 8-bit unsigned binary format? Show your working. (2 marks)
`11001000₂ + 01100100₂`

### (d) Convert the following decimal number to 8-bit signed magnitude format: `-25₁₀` (2 marks)

### (e) Convert the following 8-bit two's complement number to decimal: `10110100TC` (2 marks)

### (f) Convert the following decimal number to 8-bit two's complement: `-50₁₀` (2 marks)

### (g) Perform the following calculation in 8-bit two's complement format. Give the result in binary and decimal: (2 marks)
`00101100TC - 00010100TC`

### (h) Does the following calculation overflow 8-bit two's complement format? Show your working. (2 marks)
`10101100TC + 10011000TC`

### (i) Convert the following hexadecimal number to decimal: `3F₁₆` (2 marks)

### (j) Convert the following IEEE 754 single-precision floating-point number to decimal: `0x41200000` (2 marks)

---

## Q3. Combinational Logic (20 marks)

This question focuses on combinational logic.

### (a) State the truth table for a circuit with the following specification: (5 marks)
```
Inputs: A, B, C
Output: Y
Y is 1 iff A equals 1 or (B and C are different)
```

### (b) Derive a minimized Boolean expression for the circuit in part (a) using a Karnaugh map. (5 marks)

### (c) Draw a circuit diagram that implements the Boolean expression obtained for part (b). (5 marks)

### (d) Draw a timing diagram matching the specification in part (a). The input sequence is:
`(A,B,C)(t) = [(1,0,0), (1,1,0), (0,1,1), (0,0,1)]` (5 marks)

---

## Q4. Sequential Logic (20 marks)

This question focuses on sequential logic.

### (a) With the aid of a diagram and a truth table, explain the logic-level operation of an SR latch built from NOR gates. Include the invalid input condition in your explanation. (8 marks)

### (b) With the aid of a diagram, explain the operation of a D latch at the gate level. Explain how it differs from a D flip-flop. (8 marks)

### (c) Calculate the capacity, in Kibibytes (KiB), of a memory with the following specifications: (4 marks)
```
10-bit address
16-bit data word
```

---

## Q5. RISC-V Assembly (20 marks)

This question focuses on computer architecture.

Write a RISC-V assembly language program that counts the number of digit characters (`0`-`9`) in a string.

The string should:
- be stored in memory before program execution,
- consist of standard ASCII characters, and
- be terminated by a hash symbol `#`.

The final count should be stored in memory as a 32-bit word.

The program should process the string character-by-character and stop when the terminating hash is encountered.

For example, given the following input string stored in memory:
```
COMP30660#
```
After execution, the contents of memory should be:
```
COMP30660#
5
```

(20 marks)

---

## Q6. Computer Microarchitecture (20 marks)

This question focuses on computer microarchitecture.

### (a) Draw an outline diagram showing the organisation of a pipelined RISC-V processor with 5 stages. Include the main components and pipeline registers. (4 marks)

### (b) Explain the function of each pipeline stage (IF, ID, EX, MEM, WB) included in your diagram. (8 marks)

### (c) With the aid of diagrams, explain the following pipeline hazards and their solutions: (8 marks)
- Data hazard with forwarding
- Load-use hazard
- Control hazard with branch prediction

---

## Q7. Memory Systems (20 marks)

This question focuses on memory systems.

### (a) In terms of computer memory systems, define the following terms: (5 marks)
i. Cache hit
ii. Cache miss
iii. Write-through policy
iv. Write-back policy
v. Locality of reference

### (b) With the aid of a diagram, explain how a Direct Mapped Cache works. Include the address breakdown and lookup process. (10 marks)

### (c) Calculate the average memory access time, in nanoseconds, of a memory with the following specification: (5 marks)
```
Hit rate: 95%
Access time for cache: 2 clock periods
Access time for main memory: 8 clock periods
Clock frequency: 2 GHz
```

---

# Answer Key

---

## Q1. Digital Circuits — Answers

### (a) Transistor as a switch in CMOS (5 marks)

**Answer:**

A transistor is a semiconductor device with three terminals: Gate, Source, and Drain. In CMOS technology, two types of transistors are used:

- **NMOS**: Turns ON when Gate = 1 (high voltage), allowing current to flow between Source and Drain
- **PMOS**: Turns ON when Gate = 0 (low voltage), allowing current to flow between Source and Drain

The Gate voltage acts as a control signal that opens or closes the switch. When ON, the transistor acts as a closed switch (low resistance). When OFF, it acts as an open switch (high resistance).

### (b) 2-input NAND gate (8 marks)

**Answer:**

| A | B | Y = NAND(A,B) |
|---|---|---------------|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

Y = (A · B)' — NAND gate

A NAND gate outputs `1` unless both inputs are `1`. It is the inverse of an AND gate.

### (c) Static and dynamic power consumption (7 marks)

**Answer:**

**Static power** occurs when the circuit is powered on but the transistors are not switching. Ideally there should be no direct path from supply to ground, but small leakage currents still flow.

**Dynamic power** occurs when transistors switch. It has two main causes:

- **Short-circuit current**: for a short moment during switching, both NMOS and PMOS transistors may conduct, creating a temporary path from supply to ground.
- **Capacitive switching**: the output node has parasitic capacitance. When the output changes, this capacitance is charged or discharged. This is the dominant source of dynamic power in digital circuits.

Dynamic power increases when a circuit switches more frequently.

---

## Q2. Data Representation — Answers

### (a) `11010110₂` → decimal (2 marks)

```
1×128 + 1×64 + 0×32 + 1×16 + 0×8 + 1×4 + 1×2 + 0×1
= 128 + 64 + 16 + 4 + 2
= 214
```

**Answer: 214**

### (b) `200₁₀` → 8-bit binary (2 marks)

```
200 / 2 = 100 rem 0
100 / 2 = 50  rem 0
50  / 2 = 25  rem 0
25  / 2 = 12  rem 1
12  / 2 = 6   rem 0
6   / 2 = 3   rem 0
3   / 2 = 1   rem 1
1   / 2 = 0   rem 1

= 11001000
```

**Answer: `11001000₂`**

### (c) Unsigned overflow check (2 marks)

```
  11001000 (200)
+ 01100100 (100)
──────────
	1 00101100

Carry out = 1 → Overflow!
200 + 100 = 300 > 255 (8-bit unsigned max)
```

**Answer: Yes, overflow. 300 exceeds 8-bit unsigned range (0-255).**

### (d) `-25₁₀` → 8-bit signed magnitude (2 marks)

```
+25 = 00011001
-25 = 10011001 (set sign bit to 1)
```

**Answer: `10011001`**

### (e) `10110100TC` → decimal (2 marks)

```
MSB = 1 → negative
Take two's complement: 01001011 + 1 = 01001100
01001100 = 64 + 8 + 4 = 76
Answer: -76
```

**Answer: -76**

### (f) `-50₁₀` → 8-bit two's complement (2 marks)

```
+50 = 00110010
Flip: 11001101
Add 1: 11001110
```

**Answer: `11001110`**

### (g) `00101100TC - 00010100TC` (2 marks)

```
00101100 = +44
00010100 = +20

Convert subtraction to addition:
-00010100 = 11101011 + 1 = 11101100

  00101100 (+44)
+ 11101100 (-20)
──────────
1 00011000 (+24)

Carry out ignored in 8-bit TC.
```

**Answer: `00011000` = +24**

### (h) TC overflow check (2 marks)

```
  10101100 (-84)
+ 10011000 (-100)
──────────
1 01000100

MSB of operands: 1 + 1 (both negative)
MSB of result: 0 (positive)
Negative + Negative = Positive → Overflow!
```

**Answer: Yes, overflow. -84 + (-100) = -184, which exceeds 8-bit TC range (-128 to +127).**

### (i) `3F₁₆` → decimal (2 marks)

```
3F = 3 × 16 + 15 = 48 + 15 = 63
```

**Answer: 63**

### (j) `0x41200000` → decimal (2 marks)

```
0x41200000 = 0100 0001 0010 0000 0000 0000 0000 0000

S = 0 (positive)
E = 10000010 = 130, actual exponent = 130 - 127 = 3
F = 01000000000000000000000
Mantissa = 1.01000000000000000000000 = 1 + 0.25 = 1.25

Value = (+1) × 1.25 × 2³ = 1.25 × 8 = 10.0
```

**Answer: 10.0**

---

## Q3. Combinational Logic — Answers

### (a) Truth Table (5 marks)

Specification: Y = 1 iff A = 1 OR B ≠ C

| A | B | C | B≠C | A=1 | Y |
|---|---|---|-----|-----|---|
| 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 | 1 |
| 0 | 1 | 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 1 | 1 | 1 |
| 1 | 1 | 0 | 1 | 1 | 1 |
| 1 | 1 | 1 | 0 | 1 | 1 |

### (b) Karnaugh Map and minimized expression (5 marks)

```
K-map:
       BC
       00  01  11  10
  A=0 | 0 | 1 | 0 | 1 |
  A=1 | 1 | 1 | 1 | 1 |

Groups:
- A=1 row (4 cells): A
- A=0, BC=01: A'B'C
- A=0, BC=10: A'BC'

Minimized: Y = A + A'B'C + A'BC'
         = A + A'(B'C + BC')
         = A + A'(B ⊕ C)
         = A + B ⊕ C

Using absorption: Y = A + B ⊕ C
```

**Answer: Y = A + B ⊕ C**

### (c) Circuit diagram (5 marks)

```
A --------\
            OR ---- Y
B -- XOR --/
C --/

Where XOR = (B'C + BC')
```

Or using basic gates:
```
A --------\
            OR --------\
B -- NOT --\            |
            AND --------|
C ----------/           OR ---- Y
                        |
B ----------\           |
            AND --------/
C -- NOT ---/
```

### (d) Timing diagram (5 marks)

Input: (A,B,C) = [(1,0,0), (1,1,0), (0,1,1), (0,0,1)]

| t | A | B | C | B⊕C | Y = A + B⊕C |
|---|---|---|---|-----|-------------|
| 0 | 1 | 0 | 0 | 0 | 1 |
| 1 | 1 | 1 | 0 | 1 | 1 |
| 2 | 0 | 1 | 1 | 0 | 0 |
| 3 | 0 | 0 | 1 | 1 | 1 |

```
t:  0   1   2   3
A:  ‾‾‾‾‾‾‾‾\________
B:  ____/‾‾‾‾‾‾‾\____
C:  ______/‾‾‾‾‾‾‾‾‾‾
Y:  ‾‾‾‾‾‾‾‾‾\____/‾‾
```

---

## Q4. Sequential Logic — Answers

### (a) SR Latch (NOR implementation) (8 marks)

**Diagram:**
```
        +----------+
S ----->| NOR Gate |---- Q'
        |    X     |      |
        +----------+      |
             ^            |
             |            v
        +----------+      |
R ----->| NOR Gate |---- Q
        |    Y     |      |
        +----------+      |
             ^            |
             |____________|
```

An SR latch stores one bit using feedback between two NOR gates. The outputs are normally complementary: `Q` and `Q'`.

**Truth table:**

| S | R | Q(next) | Q'(next) | Operation |
|---|---|---------|----------|-----------|
| 0 | 0 | Q | Q' | Hold previous state |
| 1 | 0 | 1 | 0 | Set |
| 0 | 1 | 0 | 1 | Reset |
| 1 | 1 | 0 | 0 | Invalid |

When `S=1` and `R=0`, the latch is set, so `Q=1`. When `S=0` and `R=1`, the latch is reset, so `Q=0`. When `S=R=0`, the feedback keeps the previous stored value. The case `S=R=1` is invalid for a NOR SR latch because both outputs become `0`, so `Q` and `Q'` are no longer complementary.

### (b) D Latch vs D Flip-Flop (8 marks)

**D Latch (gate level):**
```
        +---+
D ----->|   |
        | AND|---> S --->|    |
E ----->|   |     |     | SR  |---> Q
        +---+     |     | Latch|
D --+-->|   |     |     |    |---> Q'
    |   | AND|---> R --->|    |
NOT-+-->|   |
        +---+
E ----->|
```

| E | D | Q(next) | Mode |
|---|---|---------|------|
| 0 | X | Q (hold) | Memory |
| 1 | 0 | 0 | Transparent (follows D) |
| 1 | 1 | 1 | Transparent (follows D) |

**Key differences:**

| Feature | D Latch | D Flip-Flop |
|---------|---------|-------------|
| Trigger | Level-sensitive | Edge-sensitive |
| Transparency | Transparent when E=1 | Opaque (samples only at edge) |
| Output changes | Any time E=1 and D changes | Only at clock edge |
| Noise immunity | Lower (glitches pass through) | Higher (samples once per edge) |

### (c) Memory capacity (4 marks)

```
10-bit address → 2¹⁰ = 1024 locations
16-bit data word → 2 bytes per location

Capacity = 1024 × 16 bits
         = 1024 × 2 bytes
         = 2048 bytes
         = 2048 / 1024 KiB
         = 2 KiB
```

**Answer: 2 KiB**

---

## Q5. RISC-V Assembly — Answers

### Complete program (20 marks)

```asm
.data
str:    .ascii "COMP30660#"
count:  .word 0

.text
.globl main
main:
    la   t0, str        # t0 = pointer to current character
    li   t1, 0          # t1 = count
    li   t2, '#'        # t2 = terminator
    li   t3, '0'        # t3 = lower bound for digits
    li   t4, '9'        # t4 = upper bound for digits

loop:
    lbu  t5, 0(t0)      # t5 = current character (unsigned)
    beq  t5, t2, done   # if char == '#', stop

    blt  t5, t3, next   # if char < '0', not a digit
    blt  t4, t5, next   # if '9' < char, not a digit

    addi t1, t1, 1      # count++ (char is a digit)

next:
    addi t0, t0, 1      # move pointer to next byte
    j    loop

done:
    la   t6, count
    sw   t1, 0(t6)      # store final count as 32-bit word

    li   a7, 10
    ecall
```

**Explanation:**

1. **Initialization**: Load string address, set count=0, load ASCII constants
2. **Loop**: Load byte, check terminator, check digit range ('0' to '9')
3. **Range check**: Two `blt` instructions test if character is within '0'-'9'
4. **Store result**: Save count to memory
5. **Exit**: System call to terminate program

**ASCII values used:**
- `#` = 35
- `0` = 48
- `9` = 57

---

## Q6. Microarchitecture — Answers

### (a) 5-stage pipelined processor diagram (4 marks)

```
         +------+   +------+   +------+   +------+   +------+
Instr -->|  IF  |-->|  ID  |-->|  EX  |-->| MEM  |-->|  WB  |--> Reg
Memory   |      |   |      |   |      |   |      |   |      |   File
         +------+   +------+   +------+   +------+   +------+
             |          |          |          |          |
             +--IF/ID---+--ID/EX---+--EX/MEM--+--MEM/WB--+
               Reg        Reg        Reg        Reg

         +------+   +------+   +------+
         | PC   |   |Reg   |   | ALU  |
         +------+   |File  |   +------+
                     +------+

         +------+   +------+
         |Data  |   |Control|
         |Memory|   | Unit  |
         +------+   +------+
```

### (b) Pipeline stage functions (8 marks)

| Stage | Name | Function | Components Used |
|-------|------|----------|-----------------|
| **IF** | Instruction Fetch | Fetch instruction from memory, compute PC+4 | PC, Instruction Memory |
| **ID** | Instruction Decode | Decode instruction, read registers, sign-extend immediate | Control Unit, Register File, Extend |
| **EX** | Execute | Perform ALU operation or calculate address | ALU |
| **MEM** | Memory Access | Read from or write to data memory | Data Memory |
| **WB** | Write Back | Write result back to register file | Register File |

### (c) Pipeline hazards and solutions (8 marks)

**Data hazard with forwarding:**
```
add x1, x2, x3    # EX produces x1
sub x4, x1, x5    # EX needs x1

Without forwarding: 3 stall cycles
With forwarding:    0 stall cycles

Forwarding path: EX/MEM register → EX stage ALU input
```

**Load-use hazard:**
```
lw  x1, 0(x2)    # Data available after MEM stage
sub x4, x1, x5   # EX needs x1, but it's not ready

Solution: 1 stall cycle + forwarding

Timeline:
lw:  IF | ID | EX | MEM | WB
sub:      IF | ID | stall| EX | MEM | WB
                      ↑ data forwarded from MEM to EX
```

**Control hazard with branch prediction:**
```
beq x1, x2, L1    # Branch decision in EX
add x3, x4, x5    # Already fetched

Solution: Branch prediction
- Predict taken (backward branches = loops)
- Predict not taken (forward branches = if-else)
- If prediction wrong: flush pipeline (2 cycle penalty)
```

---

## Q7. Memory Systems — Answers

### (a) Definitions (5 marks)

| Term | Definition |
|------|------------|
| **Cache hit** | The requested data is found in the cache; fast access |
| **Cache miss** | The requested data is not in the cache; must fetch from main memory |
| **Write-through** | Write policy where data is written to both cache and main memory simultaneously |
| **Write-back** | Write policy where data is written only to cache; main memory updated when block is evicted |
| **Locality of reference** | Property where programs tend to access the same or nearby memory locations repeatedly (temporal and spatial) |

### (b) Direct Mapped Cache (10 marks)

**Address breakdown:**
```
32-bit address:
[    Tag    |  Index  | Offset ]
  (t bits)   (s bits)  (b bits)

- Offset: log₂(block size) bits
- Index: log₂(number of lines) bits
- Tag: 32 - s - b bits
```

**Lookup process:**
```
1. CPU generates 32-bit address
2. Extract Index bits → select cache line
3. Compare Tag from address with stored Tag
4. Check Valid bit = 1
5. If Tag matches AND Valid = 1 → HIT
   - Use Offset to select byte/word from block
6. If miss → fetch block from main memory
```

**Diagram:**
```
Address:  [Tag|Index|Offset]
              |
              v
         +---------+
Index -->| Cache   |
         | Line    |
         +---------+
         | V | Tag | Data |
         +---+-----+------+
              |
         Compare----> Hit/Miss
```

### (c) AMAT calculation (5 marks)

```
Given:
  Hit rate = 95% = 0.95
  Miss rate = 1 - 0.95 = 0.05
  T_cache = 2 clock periods
  T_main = 8 clock periods
  Clock frequency = 2 GHz
  Clock period = 1 / (2 × 10⁹) = 0.5 ns

AMAT = T_cache + Miss_rate × T_main
     = 2 + 0.05 × 8
     = 2 + 0.4
     = 2.4 clock periods

In nanoseconds:
AMAT = 2.4 × 0.5 ns = 1.2 ns
```

**Answer: 1.2 ns**

---

## Summary of Key Formulas

| Topic | Formula |
|-------|---------|
| Dynamic Power | Power consumed when transistors switch, mainly due to capacitive switching |
| Two's Complement range | -2^(N-1) to +2^(N-1)-1 |
| IEEE 754 | Value = (-1)^S × 1.F × 2^(E-127) |
| Memory capacity | 2^address_bits × data_width |
| AMAT | T_cache + Miss_rate × T_main |
| Clock period | T = 1/f |

---

*Mock exam created for COMP30660 Computer Architecture & Organisation revision.*
