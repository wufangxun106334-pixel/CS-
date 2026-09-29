# 10 Practice Paper 1 With Solutions

> Course: COMP30660 Computer Architecture and Organisation  
> Based on: `COMP30660 Sample Paper.pdf`  
> Notes source: `/Users/alex/Documents/Obsidian Vault/COMP30660 Arch架构`  
> Date: 2026-05-10  
> Language: English  

---

## Instructions

Answer any **5** of the **7** questions.  
All questions carry **20 marks**.  
The paper is marked out of **100** in total.  
Calculators are not assumed.

This practice paper is designed to match the style and difficulty of the COMP30660 sample paper. Answers and explanations are provided after the question paper.

---

# Question Paper

## 1. Digital Circuits

This question focuses on digital circuits.

### 1(a)
Explain what an integrated circuit is and how binary data is represented in digital circuits.  
**5 marks**

### 1(b)
With the aid of a diagram, explain the operation of a CMOS inverter, describing the roles of the NMOS and PMOS transistors.  
**5 marks**

### 1(c)
With the aid of a diagram or graph, explain static and dynamic power consumption in a CMOS inverter. Your answer should refer to the charging and discharging of the output node.  
**10 marks**

**20 marks total**

---

## 2. Data Representation

This question focuses on data representation. Each part carries **2 marks**.

### 2(a)
Convert the following unsigned binary number to decimal:

```text
1101_2
```

### 2(b)
Convert the following decimal number to binary:

```text
45_10
```

### 2(c)
Does the following calculation overflow 4-bit unsigned binary format?

```text
1001_2 + 1010_2
```

### 2(d)
Convert the following decimal number to 8-bit signed magnitude format:

```text
-13_10
```

### 2(e)
Convert the following 4-bit two's complement number to decimal:

```text
1101_TC
```

### 2(f)
Convert the following decimal number to 8-bit two's complement:

```text
-23_10
```

### 2(g)
Perform the following calculation in 4-bit two's complement format:

```text
0101_TC - 0111_TC
```

### 2(h)
Does the following calculation overflow 4-bit two's complement format?

```text
0100_TC + 0101_TC
```

### 2(i)
Convert the following hexadecimal number to decimal:

```text
3A_16
```

### 2(j)
Convert the following IEEE 754 single-precision floating-point number to decimal:

```text
0xC0A00000
```

**20 marks total**

---

## 3. Combinational Logic

This question focuses on combinational logic.

### 3(a)
State the truth table for a circuit with the following specification:

```text
Inputs: A, B, C
Output: Y
Y is 1 iff A equals 1 and B differs from C.
```

**5 marks**

### 3(b)
Derive a Boolean expression for a circuit that implements the specification in part 3(a).  
**5 marks**

### 3(c)
Draw a circuit diagram that implements the Boolean expression obtained in part 3(b).  
**5 marks**

### 3(d)
Draw a timing diagram matching the specification in part 3(a). The input sequence is:

```text
(A,B,C)(t) = [(0,0,0), (1,0,1), (1,1,0), (1,1,1)]
```

**5 marks**

**20 marks total**

---

## 4. Sequential Logic

This question focuses on sequential logic.

### 4(a)
With the aid of a logic symbol and a timing diagram, explain the logic-level operation of a positive-edge-triggered D-type flip-flop. An explanation of the internal circuitry is not required.  
**8 marks**

### 4(b)
With the aid of diagrams, explain the operation of a NAND SR latch with active-low 
inputs `!S` and `!R`.  
**8 marks**

### 4(c)
Calculate the capacity, in Kibibytes (KiB), of a memory with the following specifications:

```text
12-bit address
16-bit data word
```

**4 marks**

**20 marks total**

---

## 5. Computer Architecture

Write a RISC-V assembly language program that counts the number of lowercase letters in a string.

The string should:

- be stored in memory before program execution,
- consist of standard ASCII characters,
- be terminated by a full stop `.`

The final count should be stored in memory as a 32-bit word.

The program should process the string character-by-character and stop when the terminating full stop is encountered.

For example, given the following input string stored in memory:

```text
Comp30660 exam.
```

after execution, memory should contain the original string and the count:

```text
Comp30660 exam.
7
```

**20 marks total**

---

## 6. Computer Microarchitecture

This question focuses on computer microarchitecture.

### 6(a)
Draw an outline diagram showing the organisation of a single-cycle RISC-V processor. Include the main components and their connections.  
**4 marks**

### 6(b)
Explain the function of each component included in your diagram.  
**8 marks**

### 6(c)
With the aid of a diagram, explain how a pipelined RISC-V implementation improves performance compared to a single-cycle RISC-V design. Your answer should describe the limitations of pipelining.  
**8 marks**

**20 marks total**

---

## 7. Memory Systems

This question focuses on memory systems.

### 7(a)
In terms of computer memory systems, define the following terms:

1. Memory hierarchy
2. Cache
3. Write-back
4. Page Table
5. Translation Lookaside Buffer

**5 marks**

### 7(b)
With the aid of a diagram, explain how a 2-way set-associative cache works. Your answer should describe the roles of tag, set index, offset, valid bit, set, and way.  
**10 marks**

### 7(c)
Calculate the average memory access time, in microseconds, of a memory system with the following specification:

```text
Hit rate: 80%
Access time for cache: 2 clock periods
Access time for main memory: 8 clock periods
Clock frequency: 2 MHz
```

**5 marks**

**20 marks total**

---

# Answers and Explanations

## Answer 1. Digital Circuits

### 1(a)
An integrated circuit is a small electronic circuit fabricated on a semiconductor chip. It contains many transistors and wires that implement digital logic. Digital data is represented using voltage levels: a high voltage represents logic `1`, and a low voltage represents logic `0`.

### 1(b)
A CMOS inverter uses a PMOS transistor connected to the supply and an NMOS transistor connected to ground.

```text
             VDD / Supply
                 |
               PMOS
Input A --------| |
                 |
                 +------ Output Y
                 |
Input A --------| |
               NMOS
                 |
               Ground
```

When `A = 0`, the PMOS is ON and the NMOS is OFF, so the output is connected to supply and `Y = 1`.  
When `A = 1`, the PMOS is OFF and the NMOS is ON, so the output is connected to ground and `Y = 0`.

Therefore:

```text
Y = NOT A
```

### 1(c)
In the stable states of an ideal CMOS inverter, there is almost no direct path from supply to ground, so static power is very small. When `A = 0`, PMOS is ON and NMOS is OFF. When `A = 1`, NMOS is ON and PMOS is OFF.

Most power is dynamic power during switching. The output node has capacitance. When the output changes from `0` to `1`, the PMOS charges the capacitance from the supply. When the output changes from `1` to `0`, the NMOS discharges the capacitance to ground.

```text
Output Y:
0 ____/^^^^\____/^^^^\____
      charge discharge
```

Dynamic power increases with capacitance, supply voltage, and switching frequency:

```text
P_dynamic is proportional to C * VDD^2 * f
```

---

## Answer 2. Data Representation

### 2(a)

```text
1101_2 = 8 + 4 + 1 = 13_10
```

### 2(b)

```text
45_10 = 32 + 8 + 4 + 1 = 101101_2
```

### 2(c)

```text
1001_2 = 9
1010_2 = 10
9 + 10 = 19
```

4-bit unsigned range is `0` to `15`, so this overflows. In 4 bits:

```text
1001
+1010
=0011 with carry out
```

Overflow occurs.

### 2(d)

Signed magnitude uses the first bit as the sign bit. `13_10 = 0001101_2` in 7 magnitude bits.

```text
-13_10 = 10001101
```

### 2(e)

`1101_TC` is negative because the MSB is `1`.

Invert and add 1:

```text
1101 -> 0010 + 1 = 0011 = 3
```

So:

```text
1101_TC = -3
```

### 2(f)

`23_10 = 00010111`. Invert and add 1:

```text
00010111 -> 11101000 + 1 = 11101001
```

So:

```text
-23_10 = 11101001_TC
```

### 2(g)

```text
0101_TC - 0111_TC = 5 - 7 = -2
```

In 4-bit two's complement:

```text
-2 = 1110_TC
```

### 2(h)

```text
0100_TC = 4
0101_TC = 5
4 + 5 = 9
```

4-bit two's complement range is `-8` to `+7`, so `9` cannot be represented. Overflow occurs.

### 2(i)

```text
3A_16 = 3 * 16 + 10 = 58_10
```

### 2(j)

```text
0xC0A00000
Binary sign bit = 1, so the number is negative.
Exponent = 10000001_2 = 129
Unbiased exponent = 129 - 127 = 2
Mantissa = 1.25
```

Therefore:

```text
value = -1.25 * 2^2 = -5.0
```

---

## Answer 3. Combinational Logic

Specification:

```text
Y = 1 iff A = 1 and B differs from C.
```

This means:

```text
Y = A * (B XOR C)
```

### 3(a)

| A | B | C | B differs from C | Y |
|---|---|---|------------------|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 | 1 |
| 1 | 1 | 0 | 1 | 1 |
| 1 | 1 | 1 | 0 | 0 |

### 3(b)

```text
Y = A * (B XOR C)
Y = A * (B' C + B C')
Y = A B' C + A B C'
```

### 3(c)

One implementation uses one XOR gate and one AND gate:

```text
B ----\
       XOR ----\
C ----/         AND ---- Y
                /
A -------------/
```

Equivalent AND/OR/NOT implementation:

```text
B' C and B C' are formed with NOT and AND gates.
Their outputs are ORed.
The result is ANDed with A.
```

### 3(d)

Input sequence:

| t | A | B | C | Y |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 1 | 1 | 0 | 1 | 1 |
| 2 | 1 | 1 | 0 | 1 |
| 3 | 1 | 1 | 1 | 0 |

Timing diagram:

```text
t:    0   1   2   3
A:    0___1___1___1
B:    0___0___1___1
C:    0___1___0___1
Y:    0___1___1___0
```

---

## Answer 4. Sequential Logic

### 4(a)
A positive-edge-triggered D flip-flop samples the input `D` only on the rising edge of the clock. Between rising edges, `Q` holds its previous value.

```text
       +---------+
D ---->| D     Q |---- Q
CLK -->| >       |
       +---------+
```

Timing example:

```text
CLK: __/^^\__/^^\__/^^\__
        ^     ^     ^
D:   0__1_____0__1______
Q:   0__1_____0_____1___
        ^     ^     ^
```

At each rising edge:

```text
Q_next = D
```

### 4(b)
A NAND SR latch has active-low inputs `!S` and `!R`.

```text
!S ----+------[NAND X]---- Q
       |          ^        |
       |          |        |
       |          +--------+
       |
!R ----+------[NAND Y]---- !Q
                  ^        |
                  |        |
                  +--------+
```

Truth table:

| !S | !R | Q next | Meaning |
|----|----|--------|---------|
| 1 | 1 | Hold | Previous state is stored |
| 0 | 1 | 1 | Set |
| 1 | 0 | 0 | Reset |
| 0 | 0 | Invalid | Both outputs forced high |

The inputs are active-low, so `!S = 0` sets `Q = 1`, and `!R = 0` resets `Q = 0`.

### 4(c)

Address width:

```text
12-bit address -> 2^12 = 4096 memory locations
```

Data word:

```text
16 bits = 2 bytes
```

Capacity:

```text
4096 locations * 16 bits = 65536 bits
65536 bits / 8 = 8192 bytes
8192 bytes / 1024 = 8 KiB
```

Answer:

```text
8 KiB
```

---

## Answer 5. Computer Architecture

The task is to count lowercase ASCII letters. Lowercase letters are in the range:

```text
'a' = 97
'z' = 122
'.' = 46
```

One possible RISC-V program:

```asm
        .data
str:    .ascii "Comp30660 exam."
count:  .word 0

        .text
        .globl main
main:
        la   t0, str          # t0 = pointer to current character
        li   t1, 0            # t1 = lowercase count
        li   t2, 46           # t2 = '.'
        li   t3, 97           # t3 = 'a'
        li   t4, 122          # t4 = 'z'

loop:
        lb   t5, 0(t0)        # load current character
        beq  t5, t2, done     # stop at full stop

        blt  t5, t3, next     # if char < 'a', not lowercase
        blt  t4, t5, next     # if 'z' < char, not lowercase
        addi t1, t1, 1        # lowercase letter found

next:
        addi t0, t0, 1        # move to next character
        beq  zero, zero, loop

done:
        la   t6, count
        sw   t1, 0(t6)

        li   a7, 10
        ecall
```

Explanation:

- `lb` reads one byte at a time.
- The program stops when it reads `.`.
- It checks whether the character lies between ASCII `a` and `z`.
- The count is stored as a 32-bit word using `sw`.

---

## Answer 6. Computer Microarchitecture

### 6(a)
Outline single-cycle RISC-V datapath:

```text
             +----------------+
PC -------->| Instruction    |---- instruction ----+
 |          | Memory         |                     |
 |          +----------------+                     v
 |                                               +--------+
 |                                               |Control |
 |                                               | Unit   |
 |                                               +--------+
 |                                                    |
 |                                                    v
 |              +---------------+                +----------+
 | instruction  | Register File |-- rs1 data --->|          |
 | fields ----->|               |-- rs2 data -+  |   ALU    |---- ALU result ----+
 |              +---------------+             |  |          |                    |
 |                    ^                       MUX +----------+                    |
 |                    |                        ^        |                         |
 |                    |                        |        v                         |
 |                    |                  Extend imm  Data Memory                  |
 |                    |                              |      |                     |
 |                    +------------ writeback MUX <--+------+---------------------+
 |
 +---- PC+4 / branch target MUX <---- branch adder / PC+imm
```

### 6(b)

| Component          | Function                                                                                                    |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| PC                 | Holds the address of the current instruction.                                                               |
| Instruction Memory | Hold current machine instruction.                                                                           |
| Control Unit       | Decodes the **instruction** and generates control **signals**.                                              |
| Register File      | Stores CPU working **data** and provides source **operands**.                                               |
| Immediate Extender | Extends **immediate** **fields** from the **instruction** to the required width.                            |
| ALU                | Performs arithmetic and logical operations                                                                  |
| Data Memory        | **Reads or writes data** for `lw` and `sw`.                                                                 |
| MUXes              | Select between alternative inputs, such as register/immediate, ALU/memory writeback, or PC+4/branch target. |
| Adders             | Compute `PC+4` and branch target addresses.                                                                 |

### 6(c)
A single-cycle processor **completes** all **stages** of one **instruction** in one long clock **cycle**. Its clock period must be long enough for the **slowest instruction.**

A pipelined processor divides execution into stages:

```text
IF  ID  EX  MEM  WB
```

				Example:

```text
Cycle:  1   2   3   4   5   6   7
I1:     IF  ID  EX  MEM WB
I2:         IF  ID  EX  MEM WB
I3:             IF  ID  EX  MEM WB
```

After the pipeline fills, ideally one **instruction** **completes** every clock **cycle**. This improves throughput because several instructions are overlapped.

Limitations:

| Limitation        | Meaning                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Data hazard       | An instruction needs a **result** that has not yet been written back. **Forwarding** or stalls may be required.           |
| Control hazard    | A **branch** or jump changes the PC, so instructions **fetched** along the **wrong** path may **need** to be **flushed**. |
| Structural hazard | Two stages need the same **hardware** resource at the same time.                                                          |
| Pipeline overhead | Pipeline registers and hazard logic add complexity and delay.                                                             |

---

## Answer 7. Memory Systems

### 7(a)

| Term | Definition |
|---|---|
| Memory hierarchy | Organisation of memory into levels, with faster, smaller, more expensive memory closer to the CPU and slower, larger, cheaper memory farther away. |
| Cache | A small, fast memory close to the CPU that stores recently used blocks from main memory. |
| Write-back | A write policy where modified data is written only to cache first and written back to main memory later when the cache block is evicted. |
| Page Table | A data structure that maps virtual page numbers to physical frame numbers and stores status bits such as valid and dirty. |
| TLB | A small, fast cache of page table entries used to speed up virtual-to-physical address translation. |

### 7(b)
A 2-way set-associative cache divides the cache into sets. Each set contains two ways. Each way stores a valid bit, tag, and data block.

```text
Address = Tag | Set Index | Offset

Set 0: Way 0 [V Tag Data]   Way 1 [V Tag Data]
Set 1: Way 0 [V Tag Data]   Way 1 [V Tag Data]
Set 2: Way 0 [V Tag Data]   Way 1 [V Tag Data]
Set 3: Way 0 [V Tag Data]   Way 1 [V Tag Data]
```

Operation:

1. The `set index` selects one set.
2. The `tag` from the address is compared with the tags in both ways of that set.
3. A hit occurs if one way has `valid = 1` and a matching tag.
4. The `offset` selects the required byte or word inside the block.
5. If no tag matches, a miss occurs. The block is fetched from main memory and placed into one of the ways in the selected set.

This reduces conflict misses compared to a direct-mapped cache, because each set has two possible locations for a block.

### 7(c)

Formula:

```text
AMAT = T_cache + miss_rate * T_main
```

Given:

```text
Hit rate = 80%
Miss rate = 20% = 0.2
T_cache = 2 clock periods
T_main = 8 clock periods
```

Calculation:

```text
AMAT = 2 + 0.2 * 8
     = 2 + 1.6
     = 3.6 clock periods
```

Clock frequency:

```text
2 MHz -> 1 clock period = 0.5 microseconds
```

Therefore:

```text
AMAT = 3.6 * 0.5 us = 1.8 us
```

Answer:

```text
1.8 microseconds
```

---

## Coverage Map

| Practice Question | Main Knowledge Source |
|---|---|
| Q1 Digital Circuits | [[01_数字电路_Digital_Circuits]] |
| Q2 Data Representation | [[02_数据表示_Data_Representation]] |
| Q3 Combinational Logic | [[03_组合逻辑_Combinational_Logic]], [[03_补充_Q3_时序图绘制指南]] |
| Q4 Sequential Logic | [[04_时序逻辑_Sequential_Logic]] |
| Q5 RISC-V Assembly | [[05_体系结构_Architecture]], [[05_补充_Q5_字符串处理模板]] |
| Q6 Microarchitecture | [[06_微架构_Microarchitecture]] |
| Q7 Memory Systems | [[07_存储系统_Memory_Systems]] |

