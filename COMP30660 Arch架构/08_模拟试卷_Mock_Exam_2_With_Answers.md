# COMP30660 Mock Examination 2 With Answers

> **Module**: Computer Architecture & Organisation  
> **Time Allowed**: 120 minutes  
> **Instructions**: Answer any 5 of the 7 questions. All questions carry 20 marks.

---

## Q1. Digital Circuits (20 marks)

### (a) Explain how NMOS and PMOS transistors work as switches. (6 marks)

### (b) With the aid of a diagram, explain the operation of a CMOS inverter. Your answer should refer to the PMOS and NMOS transistors. (8 marks)

### (c) Explain the difference between static power and dynamic power in digital circuits. (6 marks)

---

## Q2. Data Representation (20 marks)

Each part carries 2 marks.

### (a) Convert `10110101₂` to decimal.

### (b) Convert `173₁₀` to 8-bit unsigned binary.

### (c) Does `11110000₂ + 00110000₂` overflow in 8-bit unsigned arithmetic?

### (d) Convert `-37₁₀` to 8-bit signed magnitude.

### (e) Convert `11011001TC` to decimal.

### (f) Convert `-42₁₀` to 8-bit two's complement.

### (g) Compute `00110110TC + 00011100TC`. Give the result in binary and decimal.

### (h) Does `01110000TC + 00110000TC` overflow in 8-bit two's complement?

### (i) Convert `A7₁₆` to binary.

### (j) Convert IEEE 754 single-precision `0xC1200000` to decimal.

---

## Q3. Combinational Logic (20 marks)

### (a) Create the truth table for the following specification. (5 marks)

Inputs: `A`, `B`, `C`  
Output: `Y`  

`Y = 1` iff `A = 1` and at least one of `B` or `C` is `1`.

### (b) Write the Boolean expression for `Y` from the specification. (4 marks)

### (c) Simplify the expression using Boolean algebra rules. State the rules used. (5 marks)

### (d) Draw a gate-level circuit for the simplified expression. (3 marks)

### (e) Explain what a 2-to-1 multiplexer does and give its Boolean expression. (3 marks)

---

## Q4. Sequential Logic (20 marks)

### (a) With the aid of a diagram and truth table, explain the operation of a NOR SR latch. (8 marks)

### (b) Explain how a positive-edge-triggered D flip-flop behaves. Include a small timing example. (6 marks)

### (c) A memory has a 9-bit address and a 16-bit data word. Calculate its capacity in bits and bytes. (6 marks)

---

## Q5. RISC-V Assembly (20 marks)

Write a RISC-V assembly program that counts the number of lowercase letters `a` to `z` in a string.

The string should:

- be stored in memory before execution,
- contain standard ASCII characters,
- be terminated by the full stop character `'.'`.

The final count should be stored in memory as a 32-bit word called `count`.

For example:

```text
Input string:  "RiscV exam 2026."
Lowercase letters: i, s, c, x, a, m
Final count: 6
```

---

## Q6. Microarchitecture (20 marks)

### (a) Draw a simple single-cycle RISC-V datapath containing PC, instruction memory, register file, ALU, data memory, immediate extender, control unit and writeback MUX. (8 marks)

### (b) Explain the role of the register file and ALU in this datapath. (6 marks)

### (c) Explain why a pipelined RISC-V processor can improve throughput compared with a single-cycle processor. (6 marks)

---

## Q7. Memory Systems (20 marks)

### (a) Define the following terms. (5 marks)

i. Memory hierarchy  
ii. Cache  
iii. Cache hit  
iv. Cache miss  
v. Locality of reference

### (b) A cache has the following specification. Calculate the number of lines, sets, offset bits, index bits and tag bits. (10 marks)

```text
Address size: 16 bits
Cache size: 256 bytes
Block size: 16 bytes
Associativity: 2-way set associative
```

### (c) Calculate AMAT. (5 marks)

```text
Hit rate = 92%
Cache access time = 1 clock cycle
Main memory access time = 9 clock cycles
Clock frequency = 1 GHz
```

---

# Answer Key

---

## Q1. Digital Circuits — Answers

### (a) NMOS and PMOS as switches

An NMOS transistor turns ON when its gate input is high (`1`). When ON, it provides a conducting path and can pull an output node down toward ground.

A PMOS transistor turns ON when its gate input is low (`0`). When ON, it provides a conducting path and can pull an output node up toward supply.

In switch terms:

| Gate input | NMOS | PMOS |
|-----------|------|------|
| 0 | OFF | ON |
| 1 | ON | OFF |

### (b) CMOS inverter

```text
        Supply / VDD
             |
           PMOS
Gate A ------|
             |
             +------ Y
             |
           NMOS
Gate A ------|
             |
           Ground
```

Both gates are connected to the input `A`. The drains are connected together to form the output `Y`.

| A | PMOS | NMOS | Y |
|---|------|------|---|
| 0 | ON | OFF | 1 |
| 1 | OFF | ON | 0 |

When `A=0`, PMOS is ON and NMOS is OFF, so the output is pulled up to supply. When `A=1`, PMOS is OFF and NMOS is ON, so the output is pulled down to ground. Therefore `Y = NOT A`.

### (c) Static and dynamic power

Static power occurs when the circuit is powered but not switching. Ideally CMOS has no direct path from supply to ground in a stable state, but leakage currents still cause small power consumption.

Dynamic power occurs when transistors switch. It includes short-circuit current, when NMOS and PMOS may briefly conduct at the same time, and capacitive switching, where the output node capacitance is charged and discharged. Capacitive switching is usually the dominant part.

---

## Q2. Data Representation — Answers

### (a)

```text
10110101₂ = 128 + 32 + 16 + 4 + 1 = 181
```

### (b)

```text
173₁₀ = 10101101₂
```

### (c)

```text
11110000₂ = 240
00110000₂ = 48
240 + 48 = 288
```

8-bit unsigned range is `0` to `255`, so this overflows.

### (d)

Signed magnitude uses one sign bit and 7 magnitude bits.

```text
37₁₀ = 0100101₂
-37₁₀ = 10100101
```

### (e)

`11011001TC` is negative because the MSB is `1`.

Invert and add 1:

```text
11011001
00100110
+      1
00100111 = 39
```

Answer: `-39`.

### (f)

```text
42₁₀ = 00101010
Invert: 11010101
Add 1:  11010110
```

Answer: `11010110TC`.

### (g)

```text
00110110 = 54
00011100 = 28
54 + 28 = 82
```

Binary result:

```text
01010010TC
```

### (h)

```text
01110000 = 112
00110000 = 48
112 + 48 = 160
```

8-bit two's complement range is `-128` to `+127`. Adding two positive numbers gives a negative-looking result, so overflow occurs.

### (i)

```text
A7₁₆ = 1010 0111₂
```

### (j)

`0xC1200000` in binary:

```text
1 10000010 01000000000000000000000
```

Sign bit `S = 1`, so the number is negative.  
Exponent `E = 130`, so unbiased exponent is `130 - 127 = 3`.  
Fraction is `1.25`.

```text
Value = -1.25 × 2^3 = -10
```

---

## Q3. Combinational Logic — Answers

### (a) Truth table

| A | B | C | Y |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

### (b) Boolean expression

At least one of `B` or `C` is `1` means `B + C`.

```text
Y = A(B + C)
```

Equivalent SOP form:

```text
Y = AB + AC
```

### (c) Simplification

Starting from SOP:

```text
Y = AB + AC
  = A(B + C)     Distributive Law
```

So the minimized expression is:

```text
Y = A(B + C)
```

### (d) Circuit

```text
B ----\
      OR ----\
C ----/       \
              AND ---- Y
A ------------/
```

### (e) 2-to-1 MUX

A 2-to-1 multiplexer selects one of two data inputs using a select signal `S`.

| S | Y |
|---|---|
| 0 | D0 |
| 1 | D1 |

Boolean expression:

```text
Y = S' D0 + S D1
```

---

## Q4. Sequential Logic — Answers

### (a) NOR SR latch

```text
        +-----+
S ----->| NOR |---- Q'
        +-----+     |
           ^        |
           |        v
        +-----+     |
R ----->| NOR |---- Q
        +-----+     |
           ^________|
```

| S | R | Q(next) | Operation |
|---|---|---------|-----------|
| 0 | 0 | Q | Hold |
| 1 | 0 | 1 | Set |
| 0 | 1 | 0 | Reset |
| 1 | 1 | Invalid | Both outputs become 0 |

The latch stores one bit using feedback. When both inputs are `0`, feedback holds the previous state. For a NOR SR latch, `S=R=1` is invalid because `Q` and `Q'` are no longer complementary.

### (b) Positive-edge D flip-flop

A positive-edge-triggered D flip-flop samples `D` only on the rising edge of the clock. Between rising edges, `Q` holds its previous value.

Example:

```text
Rising edge 1: D = 1 -> Q becomes 1
Rising edge 2: D = 0 -> Q becomes 0
Between edges: Q does not change
```

Characteristic equation:

```text
Q(next) = D
```

### (c) Memory capacity

```text
Address bits = 9
Data word width = 16 bits
Depth = 2^9 = 512 words
Capacity = 512 × 16 = 8192 bits
8192 bits / 8 = 1024 bytes
```

Answer:

```text
8192 bits = 1024 bytes = 1 KiB
```

---

## Q5. RISC-V Assembly — Answer

```asm
.data
str:    .ascii "RiscV exam 2026."
count:  .word 0

.text
.globl main
main:
    la   t0, str        # t0 = pointer to current character
    li   t1, 0          # t1 = count
    li   t2, '.'        # terminator
    li   t3, 'a'        # lower bound
    li   t4, 'z'        # upper bound

loop:
    lb   t5, 0(t0)      # load current character
    beq  t5, t2, done   # stop at '.'

    blt  t5, t3, next   # if char < 'a', skip
    blt  t4, t5, next   # if 'z' < char, skip

    addi t1, t1, 1      # count++

next:
    addi t0, t0, 1      # move to next byte
    j    loop

done:
    la   t6, count
    sw   t1, 0(t6)

    li   a7, 10
    ecall
```

Key points:

- `la` loads the address of `str` and `count`.
- `li` loads immediate constants such as `'.'`, `'a'`, `'z'`.
- `lb` loads one byte because characters are one byte.
- `sw` stores the final 32-bit count.

---

## Q6. Microarchitecture — Answers

### (a) Datapath diagram

```text
             +----------------+
 PC -------->| Instruction    |---- instruction ----+
 |           | Memory         |                     |
 |           +----------------+                     v
 |                                               +--------+
 |                                               |Control |
 |                                               | Unit   |
 |                                               +--------+
 |                                                    |
 |                                                    v
 |              +---------------+                +----------+
 | instruction  | Register File |-- rs1 data --->|          |
 | fields ----->|               |-- rs2 data --+ |   ALU    |---- ALU result ---+
 |              +---------------+              | |          |                   |
 |                    ^                        MUX+----------+                   |
 |                    |                         ^       |                         |
 |                    |                         |       v                         |
 |                    |                   Extend imm  Data Memory                 |
 |                    |                               |      |                    |
 |                    +------------- writeback MUX <--+------+--------------------+
 |
 +---- PC+4 / branch target MUX <---- branch adder / PC+imm
```

### (b) Register file and ALU

The instruction contains register fields such as `rs1`, `rs2` and `rd`.

The register file reads the values stored in `rs1` and `rs2`. These are the operands. The first operand usually goes directly to the ALU. The second ALU input is selected by a MUX:

- R-type instruction: use `rs2 data`
- I-type/load/store instruction: use extended immediate

The ALU then performs an operation such as add, subtract, AND, OR, address calculation or branch comparison. The ALU result may be written back to the register file or used as a data memory address.

### (c) Pipelining

A single-cycle processor completes all parts of one instruction in one long clock cycle. The clock period must be long enough for the slowest instruction.

A pipelined processor splits instruction execution into stages such as IF, ID, EX, MEM and WB. Different instructions can occupy different stages at the same time. After the pipeline fills, ideally one instruction completes every clock cycle. This improves throughput, although hazards may require forwarding, stalls or flushing.

---

## Q7. Memory Systems — Answers

### (a) Definitions

| Term | Definition |
|------|------------|
| Memory hierarchy | Organisation of memory into levels, with small fast memory close to the CPU and larger slower memory farther away |
| Cache | Small fast memory that stores recently used blocks from main memory |
| Cache hit | Requested data is found in cache |
| Cache miss | Requested data is not found in cache and must be fetched from lower memory |
| Locality of reference | Programs tend to reuse recently accessed data/instructions or access nearby addresses |

### (b) Cache calculation

Given:

```text
Address size = 16 bits
Cache size = 256 bytes
Block size = 16 bytes
Associativity = 2
```

Number of cache lines:

```text
Lines = Cache size / Block size
      = 256 / 16
      = 16 lines
```

Number of sets:

```text
Sets = Lines / Associativity
     = 16 / 2
     = 8 sets
```

Offset bits:

```text
Offset = log2(Block size)
       = log2(16)
       = 4 bits
```

Index bits:

```text
Index = log2(Sets)
      = log2(8)
      = 3 bits
```

Tag bits:

```text
Tag = Address bits - Index - Offset
    = 16 - 3 - 4
    = 9 bits
```

Address breakdown:

```text
Address = Tag | Set Index | Offset
        = 9 bits | 3 bits | 4 bits
```

### (c) AMAT

```text
Hit rate = 92%
Miss rate = 8% = 0.08
Cache access time = 1 cycle
Main memory access time = 9 cycles
```

AMAT:

```text
AMAT = T_cache + Miss_rate × T_main
     = 1 + 0.08 × 9
     = 1 + 0.72
     = 1.72 cycles
```

Clock frequency is 1 GHz, so one cycle is 1 ns.

```text
AMAT = 1.72 ns
```

