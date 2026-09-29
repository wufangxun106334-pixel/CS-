# 08 样卷解析 -- COMP30660 Sample Paper

> 来源: `COMP30660 Sample Paper.pdf`
> 用途: 对照样卷 7 道题补齐复习笔记中缺少的考试型模板。

---

## 总体结构

样卷共 7 题，考试时任选 5 题作答。每题 20 分，总分 100。

| 题号 | 主题 | 核心能力 |
|---|---|---|
| Q1 | Digital Circuits | IC、CMOS inverter、CMOS 功耗解释 |
| Q2 | Data Representation | 二进制、signed magnitude、two's complement、hex、IEEE754 |
| Q3 | Combinational Logic | 真值表、布尔表达式、电路图、timing diagram |
| Q4 | Sequential Logic | D flip-flop、SR latch、memory capacity |
| Q5 | RISC-V Assembly | 字符串逐字节扫描、大写字母计数、结果存内存 |
| Q6 | Microarchitecture | single-cycle datapath、pipeline、hazards |
| Q7 | Memory Systems | memory hierarchy、cache、VM、TLB、AMAT |

---

## Q1 Digital Circuits

> **知识点定位**: [[01_数字电路_Digital_Circuits]]

### (a) Integrated Circuit 与数据表示

> **对应笔记**: [[01_数字电路_Digital_Circuits#四、电子计算机的构成 (Electronic Computer Components)|IC 定义与数据表示]] · [[01_数字电路_Digital_Circuits#4.3 数据表示 (Data Representation)|电压水平与二进制]]

答题要点:

- Integrated Circuit (IC) 是在一片半导体材料上制造大量微小电子元件形成的电路。
- 计算机芯片中的主要元件包括 transistor、wire、logic gate、memory cell 等。
- 数据以二进制 `0/1` 表示。
- 物理上，二进制由导线上的电压水平表示。
- High voltage / Supply 表示 `1`，Low voltage / Ground 表示 `0`。

标准答法:

```text
An integrated circuit is an electronic circuit in which many tiny components are fabricated together on a single piece of semiconductor material, usually silicon. In a computer IC, data is represented using binary values. These binary values are physically represented by voltage levels on wires: a voltage close to supply represents logic 1, and a voltage close to ground represents logic 0.
```

### (b) CMOS Inverter / NOT Gate

> **对应笔记**: [[01_数字电路_Digital_Circuits#六、NOT 门的 CMOS 实现|CMOS NOT 门结构与工作原理]] · [[01_数字电路_Digital_Circuits#5.3 NMOS vs PMOS 开关行为|NMOS/PMOS 开关行为]]

核心结构:

```text
Supply
  |
 PMOS
  |
  Y
  |
 NMOS
  |
Ground

Input A connects to both PMOS gate and NMOS gate.
```

工作过程:

| A | PMOS | NMOS | Y |
|---|---|---|---|
| 0 | ON | OFF | 1 |
| 1 | OFF | ON | 0 |

解释:

- 当 `A=0` 时，PMOS 导通，NMOS 截止，输出节点 `Y` 被连接到 Supply，所以 `Y=1`。
- 当 `A=1` 时，PMOS 截止，NMOS 导通，输出节点 `Y` 被连接到 Ground，所以 `Y=0`。
- 因此输出总是输入的反相，形成 NOT gate。

### (c) CMOS Inverter Power Consumption

> **对应笔记**: [[01_数字电路_Digital_Circuits#十、功耗 (Power Consumption)|静态功耗与动态功耗]] · [[01_数字电路_Digital_Circuits#10.4 充放电过程详解|寄生电容充放电]]

考试必须提到两类功耗:

| 类型 | 原因 | 何时发生 |
|---|---|---|
| Static Power | 漏电流 leakage current | 电路稳定、不切换时 |
| Dynamic Power | 输出节点寄生电容充放电 + 短路电流 | 输入切换时 |

充放电过程:

| 输入变化 | 输出变化 | 导通晶体管 | 电流路径 | 过程 |
|---|---|---|---|---|
| `1 -> 0` | `0 -> 1` | PMOS ON, NMOS OFF | Supply -> Y | Charging |
| `0 -> 1` | `1 -> 0` | NMOS ON, PMOS OFF | Y -> Ground | Discharging |

功耗图答题模板:

```text
Y voltage
Supply  ____      ____
           |    |
           |    |
Ground  ___|____|____ time
          discharge charge
```

关键句:

```text
During switching, the parasitic capacitance at the output node is charged and discharged. Charging draws current from the supply through the PMOS transistor. Discharging releases the stored energy through the NMOS transistor to ground. This energy is dissipated as heat, giving dynamic power consumption. Dynamic power increases with switching frequency.
```

---

## Q2 Data Representation

> **知识点定位**: [[02_数据表示_Data_Representation]]

### 逐题答案速查

| 小题  | 题目                                         | 答案                                  | 对应笔记                                                                                   |
| --- | ------------------------------------------ | ----------------------------------- | -------------------------------------------------------------------------------------- |
| (a) | `1010_2` -> decimal                        | `10`                                | [[02_数据表示_Data_Representation#进制转换方法\|二进制→十进制]]                                        |
| (b) | `57_10` -> binary                          | `111001_2`                          | [[02_数据表示_Data_Representation#进制转换方法\|十进制→二进制]]                                        |
| (c) | `0110_2 + 1100_2` unsigned 4-bit overflow? | `6 + 12 = 18 > 15`，overflow         | [[02_数据表示_Data_Representation#二进制加法 (Binary Addition)\|无符号溢出]]                         |
| (d) | `-7_10` -> 8-bit signed magnitude          | `10000111`                          | [[02_数据表示_Data_Representation#符号-幅度表示法 (Sign-Magnitude)\|Sign-Magnitude]]              |
| (e) | `1001_TC` 4-bit -> decimal                 | `-7`                                | [[02_数据表示_Data_Representation#二补码 (Two's Complement)\|Two's Complement]]               |
| (f) | `-18_10` -> 8-bit two's complement         | `11101110`                          | [[02_数据表示_Data_Representation#Two's Complement 取负 (Negation)\|取负方法]]                   |
| (g) | `1010_TC - 0010_TC` 4-bit                  | `1010 + 1110 = 1000`，即 `-8`         | [[02_数据表示_Data_Representation#Two's Complement 减法\|TC 减法]]                             |
| (h) | `0110_TC - 1100_TC` overflow?              | `6 - (-4) = 10` 超出 `-8..7`，overflow | [[02_数据表示_Data_Representation#Two's Complement 溢出 (Overflow)\|TC 溢出检测]]                |
| (i) | `29_16` -> decimal                         | `41`                                | [[02_数据表示_Data_Representation#十六进制 ↔ 十进制\|Hex→Decimal]]                                |
| (j) | `0xc0600000` floating-point -> decimal     | `-3.5`                              | [[02_数据表示_Data_Representation#IEEE 754 单精度浮点数格式 (Single Precision, 32-bit)\|IEEE 754]] |
|     |                                            |                                     |                                                                                        |

### (j) IEEE754 `0xc0600000` 详细步骤

```text
0xc0600000
= 1100 0000 0110 0000 0000 0000 0000 0000

S = 1
E = 10000000 = 128
F = 11000000000000000000000

Value = (-1)^S * 1.F * 2^(E-127)
      = (-1)^1 * 1.11_2 * 2^(128-127)
      = -1 * 1.75 * 2
      = -3.5
```

### Two's Complement 易错点

- 4-bit two's complement 范围是 `-8` 到 `+7`。
- 减法先转成加法: `A - B = A + (-B)`。
- two's complement overflow 判断: 两个同号数相加得到异号结果，或数学结果超出可表示范围。

---

## Q3 Combinational Logic

> **知识点定位**: [[03_组合逻辑_Combinational_Logic]] · [[03_补充_Q3_时序图绘制指南]]

题目规格:

```text
Inputs: A, B, C
Output: Y
Y is 1 iff A equals 0 or (B and C are equal)
```

### (a) Truth Table

> **对应笔记**: [[03_组合逻辑_Combinational_Logic#2.2 真值表 (Truth Table)|真值表构造方法]]

逻辑条件:

```text
Y = 1 if A=0 OR B=C
```

| A   | B   | C   | B=C | A=0 | Y   |
| --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 1   | 1   | 1   |
| 0   | 0   | 1   | 0   | 1   | 1   |
| 0   | 1   | 0   | 0   | 1   | 1   |
| 0   | 1   | 1   | 1   | 1   | 1   |
| 1   | 0   | 0   | 1   | 0   | 1   |
| 1   | 0   | 1   | 0   | 0   | 0   |
| 1   | 1   | 0   | 0   | 0   | 0   |
| 1   | 1   | 1   | 1   | 0   | 1   |

### (b) Boolean Expression

> **对应笔记**: [[03_组合逻辑_Combinational_Logic#5. 布尔表达式 (Boolean Expressions)|布尔表达式]] · [[03_组合逻辑_Combinational_Logic#8. SOP与POS形式|SOP 形式]]

直接表达:

```text
Y = A' + (B XNOR C)
```

只用 AND/OR/NOT:

```text
Y = A' + BC + B'C'
```

### (c) Circuit Diagram

> **对应笔记**: [[03_组合逻辑_Combinational_Logic#3. 逻辑图 (Logic Diagrams)|逻辑图绘制]]

门级结构:

```text
A -- NOT ---------------------\
                               OR ---- Y
B -- AND -- BC ---------------/
C --/

B -- NOT --\
            AND -- B'C' ------/
C -- NOT --/
```

如果允许 XNOR，可更简单:

```text
A -- NOT --\
            OR ---- Y
B -- XNOR --/
C --/
```

### (d) Timing Diagram

> **对应笔记**: [[03_补充_Q3_时序图绘制指南|时序图绘制完整指南]] · [[03_组合逻辑_Combinational_Logic#2.3 时序图 (Timing Diagram)|时序图定义]]

输入序列:

```text
(A,B,C)(t) = [(0,1,1), (0,1,0), (1,0,1), (1,0,0)]
```

逐段计算:

| t   | A   | B   | C   | Y   |
| --- | --- | --- | --- | --- |
| 0   | 0   | 1   | 1   | 1   |
| 1   | 0   | 1   | 0   | 1   |
| 2   | 1   | 0   | 1   | 0   |
| 3   | 1   | 0   | 0   | 1   |

可画成:

```text
t:  0   1   2   3
A:  0   0   1   1
B:  1   1   0   0
C:  1   0   1   0
Y:  1   1   0   1
```

---

## Q4 Sequential Logic

> **知识点定位**: [[04_时序逻辑_Sequential_Logic]]

### (a) D-Type Flip-Flop

> **对应笔记**: [[04_时序逻辑_Sequential_Logic#5. D Flip-Flop（D 触发器）|D Flip-Flop 工作原理]] · [[04_时序逻辑_Sequential_Logic#5.2 符号与定时图|时序图与符号]]

核心定义:

- D flip-flop 是 edge-sensitive 存储元件。
- 在有效时钟边沿采样 `D`。
- 边沿之间 `Q` 保持不变。
- Positive-edge D FF 的特征方程: `Q(n+1) = D`。

符号说明:

```text
       +-----+
D ---->| D Q |---- Q
CLK -->|>    |
       +-----+
```

`>` 三角表示 edge-triggered clock input。

Timing diagram 模板:

```text
CLK: __/‾‾\__/‾‾\__/‾‾\__
D:   0___1_____0___1_____
Q:   x___1_____0___1_____
        ^     ^   ^
      sample D only at rising edges
```

### (b) SR Flip-Flop / SR Latch Level Operation

> **对应笔记**: [[04_时序逻辑_Sequential_Logic#2. 基本存储单元 -- SR Latch|SR Latch 工作原理]] · [[04_时序逻辑_Sequential_Logic#2.1 SR Latch (NOR Gate Implementation)|NOR 实现]]

NOR SR latch 表:

| S | R | Q(next) | 说明 |
|---|---|---|---|
| 0 | 0 | Q | Hold |
| 1 | 0 | 1 | Set |
| 0 | 1 | 0 | Reset |
| 1 | 1 | invalid | 禁止 |

结构:

```text
      +-----+        
S --->| NOR |--- Q' --\
      +-----+         \
                       > feedback
      +-----+         /
R --->| NOR |--- Q ---/
      +-----+
```

关键句:

```text
The SR latch stores one bit using feedback between two cross-coupled NOR gates. Set makes Q=1, reset makes Q=0, and when S=R=0 the feedback maintains the previous state. The input S=R=1 is invalid because both outputs are forced to 0, so Q and Q' are no longer complementary.
```

### (c) Memory Capacity

> **对应笔记**: [[04_时序逻辑_Sequential_Logic#10.2 参数定义|存储器容量计算]] · [[04_时序逻辑_Sequential_Logic#10.3 数据单位|KiB 单位换算]]

题目:

```text
8-bit address
8-bit data word
```

计算:

```text
Depth = 2^8 = 256 words
Width = 8 bits = 1 byte
Capacity = 256 * 1 byte = 256 bytes
256 bytes = 256 / 1024 KiB = 0.25 KiB
```

答案: `0.25 KiB`

---

## Q5 RISC-V Assembly

> **知识点定位**: [[05_体系结构_Architecture]] · [[05_补充_Q5_字符串处理模板]]

任务:

```text
Count uppercase letters in a string.
String is stored in memory before execution.
String consists of standard ASCII characters.
String is terminated by a full stop '.'.
Final count stored in memory as a 32-bit word.
```

### 关键 ASCII 值

> **对应笔记**: [[02_数据表示_Data_Representation#ASCII (American Standard Code for Information Interchange)|ASCII 编码]] · [[05_补充_Q5_字符串处理模板#ASCII 值速查|ASCII 速查表]]

| Character | Decimal |
|---|---:|
| `.` | 46 |
| `A` | 65 |
| `Z` | 90 |

### Native-Instruction-Friendly 程序

> **对应笔记**: [[05_体系结构_Architecture#8. 分支、循环与函数调用|分支与循环]] · [[05_补充_Q5_字符串处理模板#完整程序模板|字符串处理模板]]

尽量避免 `bgt`，因为样卷 reference table 只列了 `beq`, `bne`。如果要比较 `char > 'Z'`，可写成 `'Z' < char`。

```asm
.data
str:    .ascii "COMP 30660."
count:  .word 0

.text
.globl main
main:
    la   t0, str        # t0 = pointer to current character
    li   t1, 0          # t1 = count
    li   t2, '.'        # t2 = terminator
    li   t3, 'A'        # lower bound for uppercase
    li   t4, 'Z'        # upper bound for uppercase

loop:
    lb   t5, 0(t0)      # t5 = current character
    beq  t5, t2, done   # if char == '.', stop

    blt  t5, t3, next   # if char < 'A', not uppercase
    blt  t4, t5, next   # if 'Z' < char, not uppercase

    addi t1, t1, 1      # otherwise A <= char <= Z

next:
    addi t0, t0, 1      # move pointer to next byte
    j    loop

done:
    la   t6, count
    sw   t1, 0(t6)      # store final count as 32-bit word

    li   a7, 10
    ecall
```

如果考试只允许 reference table 中的指令，`j loop` 可展开为:

```asm
jal x0, loop
```

`li` 和 `la` 是伪指令。一般汇编编程题允许使用伪指令；若题目要求 machine code encoding，必须先展开。

### 程序逻辑

```text
count = 0
p = address of str
while (*p != '.'):
    if 'A' <= *p <= 'Z':
        count++
    p++
store count
```

---

## Q6 Microarchitecture

> **知识点定位**: [[06_微架构_Microarchitecture]]

### (a) Single-Cycle RISC-V Processor Outline

> **对应笔记**: [[06_微架构_Microarchitecture#3.2 数据通路组件 Datapath Components|单周期数据通路组件]] · [[06_微架构_Microarchitecture#3.2.1 各组件详解|组件详解]]

考试手绘版结构:

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
 |              | Register File |-- rs data ---->|          |
 |              |               |-- rs data --+  |   ALU    |---- ALU result ---+
 |              +---------------+             |  |          |                   |
 |                    ^                       MUX +----------+                   |
 |                    |                        ^        |                        |
 |                    |                        |        v                        |
 |                    |                  Extend imm  Data Memory                 |
 |                    |                              |      |                    |
 |                    +------------ writeback MUX <--+------+--------------------+
 |
 +---- PC+4 / branch target MUX <---- branch adder / PC+imm
```

必须包含:

- PC
- Instruction Memory
- Control Unit
- Register File
- ALU
- Data Memory
- Extend / Sign Extend Unit
- MUXes
- PC+4 adder / branch target adder

### (b) Component Functions

> **对应笔记**: [[06_微架构_Microarchitecture#3.2.1 各组件详解|各组件功能详解]] · [[06_微架构_Microarchitecture#3.8 控制单元 Control Unit|控制信号]]

| Component          | Function                                                                                                                            |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| PC                 | Holds **address** of current **instruction**                                                                                        |
| Instruction Memory | **Outputs** **instruction** at PC address                                                                                           |
| Control Unit       | Decodes instruction and generates control signals                                                                                   |
| Register File      | **Stores** the **data** that the **CPU** **is** currently working on. Data can be passed between main memory and the register file. |
| ALU                | Performs arithmetic and logical operations on data held in the register file.                                                       |
| Extend Unit        | Sign-extends immediate fields to 32 bits                                                                                            |
| main Memory        | Looks up the address and outputs the machine instruction stored at that address.\|                                                  |
| MUX                | Selects between alternative datapath values                                                                                         |
| Adders             | Compute `PC+4` and branch target address                                                                                            |

### (c) Pipeline Performance Explanation

> **对应笔记**: [[06_微架构_Microarchitecture#4. 流水线处理器 Pipelined Processor|五级流水线]] · [[06_微架构_Microarchitecture#5. 流水线冒险 Hazards|流水线冒险与限制]]

五级流水线:

| Stage | Meaning |
|---|---|
| IF | Instruction Fetch |
| ID | Instruction Decode / register read |
| EX | Execute / ALU |
| MEM | Data memory access |
| WB | Write back |

核心解释:


**A single-cycle** processor completes **all steps** of **one instruction** in **one long clock cycle**. The clock period must be long enough for the slowest **instruction**. A **pipelined** processor splits instruction execution into stages **separated** by **pipeline** **registers**. **Different** **instructions** occupy different **stages** at the **same time**. After the pipeline fills, ideally one **instruction completes** every **clock** cycle, improving throughput.


流水线限制:

| Limitation                 | Explanation                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------- |
| Structural hazards         | Two instructions need the same hardware resource                                   |
| Data hazards               | Instruction needs a result not yet written back                                    |
| Load-use hazard            | Even with forwarding, load data is available only after MEM; usually needs 1 stall |
| Control hazards            | Branch target/path unknown until branch is resolved                                |
| Pipeline register overhead | Clock period cannot be reduced perfectly by number of stages                       |

---

## Q7 Memory Systems

> **知识点定位**: [[07_存储系统_Memory_Systems]]

### (a) Definitions

> **对应笔记**: [[07_存储系统_Memory_Systems#2. 存储层次结构 (Memory Hierarchy)|Memory Hierarchy]] · [[07_存储系统_Memory_Systems#5.1 Cache 基础概念|Cache]] · [[07_存储系统_Memory_Systems#6.1 核心理念|Virtual Memory]] · [[07_存储系统_Memory_Systems#6.3.3 Page Table 结构|Page Table]] · [[07_存储系统_Memory_Systems#6.6 Translation Lookaside Buffer (TLB)|TLB]]

| Term             | Definition                                                                                                                                                     |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Memory hierarchy | Organisation of memories in levels: small/fast/expensive close to CPU, large/slow/cheap farther away                                                           |
| Cache            | Small, fast memory close to CPU storing recently/frequently used blocks from main memory                                                                       |
| Virtual Memory   | System that gives programs the illusion of a larger address space than physical memory                                                                         |
| Page Table       | Data **structure** **mapping** virtual page numbers (VPNs) to physical frame numbers (PFNs)                                                                    |
| TLB              | Translation Lookaside Buffer:A small, fast **cache** for the page table that stores recently used VPN-to-PFN **mappings** to speed up address **translation**. |



### (b) Set Associative Cache

> **对应笔记**: [[07_存储系统_Memory_Systems#5.4.2 2-Way Set Associative Cache 详解|Set Associative Cache 结构]] · [[07_存储系统_Memory_Systems#5.8 N-Way Set Associative Cache 访问流程|访问流程]]

核心结构:

```text
Address = Tag | Set Index | Offset

Set 0: Way 0 [V Tag Data]   Way 1 [V Tag Data]
Set 1: Way 0 [V Tag Data]   Way 1 [V Tag Data]
Set 2: Way 0 [V Tag Data]   Way 1 [V Tag Data]
...
```

访问流程:

1. CPU 发出地址。
2. 地址被拆成 `Tag | Set | Offset`。
3. `Set` 字段选择 cache 中的一组。
4. 同时比较该 set 中所有 ways 的 tag。
5. 若某个 way 的 valid bit 为 1 且 tag 匹配，则 cache hit。
6. 用 offset 选出 block 内的目标 byte/word。
7. 若没有 way 匹配，则 cache miss，需要从 main memory 载入 block。
8. 若 set 已满，根据 LRU/FIFO/random 选择一个 way 替换。
9. 若被替换行 dirty，需要先写回 main memory。
10. The **CPU** sends an **address** to the **cache**.
11. The cache uses the **index** bits to select one ****set**.
12. Inside that set, the **cache** compares the **tag** with the tags in **all ways** at the same time.
13. If one **tag** matches and the **valid** bit is set, it is a **cache hit**.
14. The **offset** bits select the exact **byte**/word inside the **block**.
15. If no tag matches, it is a **cache miss**. The **block** is fetched from main memory and placed into one of the ways in that **set**.
16. If the selected **set** is full, choose one **cache** **line** to replace using a replacement policy such as LRU, FIFO, or random.
17. If the cache **line** being replaced is **dirty**, write it back to main **memory** before **overwriting** it.

优点:

- 比 direct-mapped cache 冲突缺失少。
- 比 fully-associative cache 硬件成本低。
- 实际 CPU cache 常用 N-way set associative。

### (c) AMAT Calculation

> **对应笔记**: [[07_存储系统_Memory_Systems#5.11.2 AMAT 计算示例|AMAT 计算公式与示例]]

题目:

```text
Hit rate = 90%
Cache access time = 1 clock period
Main memory access time = 5 clock periods
Clock frequency = 1 MHz
```

计算:

```text
Miss rate = 1 - 0.90 = 0.10

AMAT = T_cache + Miss_rate * T_main
     = 1 + 0.10 * 5
     = 1.5 clock periods

Clock frequency = 1 MHz
Clock period = 1 / 1,000,000 s = 1 microsecond

AMAT = 1.5 microseconds
```

答案: `1.5 microseconds`

---

## 覆盖率与补强清单

| 样卷题目 | 覆盖率 | 主要笔记来源 | 关键知识点链接 | 补强状态 |
|---|---:|---|---|---|
| Q1 Digital Circuits | 95% | [[01_数字电路_Digital_Circuits]] | [[01_数字电路_Digital_Circuits#4.2 集成电路 (IC - Integrated Circuit)\|IC]] · [[01_数字电路_Digital_Circuits#六、NOT 门的 CMOS 实现\|CMOS]] · [[01_数字电路_Digital_Circuits#十、功耗 (Power Consumption)\|功耗]] | 已补考试答题模板 |
| Q2 Data Representation | 90% | [[02_数据表示_Data_Representation]] | [[02_数据表示_Data_Representation#二补码 (Two's Complement)\|TC]] · [[02_数据表示_Data_Representation#IEEE 754 单精度浮点数格式\|IEEE754]] · [[02_数据表示_Data_Representation#十六进制数 (Hexadecimal Numbers)\|Hex]] | 已补样卷答案和 IEEE754 `0xc0600000` |
| Q3 Combinational Logic | 85% | [[03_组合逻辑_Combinational_Logic]] | [[03_组合逻辑_Combinational_Logic#2.2 真值表 (Truth Table)\|真值表]] · [[03_组合逻辑_Combinational_Logic#5. 布尔表达式\|布尔表达式]] · [[03_补充_Q3_时序图绘制指南\|时序图]] | 已补同款完整解析 |
| Q4 Sequential Logic | 95% | [[04_时序逻辑_Sequential_Logic]] | [[04_时序逻辑_Sequential_Logic#5. D Flip-Flop\|D FF]] · [[04_时序逻辑_Sequential_Logic#2. 基本存储单元 -- SR Latch\|SR Latch]] · [[04_时序逻辑_Sequential_Logic#10.2 参数定义\|容量计算]] | 已补容量计算答案 |
| Q5 RISC-V Assembly | 75% | [[05_体系结构_Architecture]], [[08习题汇总_Worksheets]] | [[05_体系结构_Architecture#8. 分支、循环与函数调用\|循环]] · [[05_补充_Q5_字符串处理模板\|字符串模板]] | 已补完整程序模板 |
| Q6 Microarchitecture | 90% | [[06_微架构_Microarchitecture]] | [[06_微架构_Microarchitecture#3.2 数据通路组件\|Datapath]] · [[06_微架构_Microarchitecture#4. 流水线处理器\|Pipeline]] · [[06_微架构_Microarchitecture#5. 流水线冒险 Hazards\|Hazards]] | 已补手绘版 datapath |
| Q7 Memory Systems | 95% | [[07_存储系统_Memory_Systems]] | [[07_存储系统_Memory_Systems#5.4.2 2-Way Set Associative Cache\|Set Assoc]] · [[07_存储系统_Memory_Systems#5.11.2 AMAT 计算示例\|AMAT]] · [[07_存储系统_Memory_Systems#6. 虚拟内存\|VM]] | 已补 AMAT 同款计算 |

---

## 最后复习建议

优先背熟以下 5 个模板:

1. **CMOS inverter**: `A=0 -> PMOS ON -> Y=1`; `A=1 -> NMOS ON -> Y=0`。→ [[01_数字电路_Digital_Circuits#六、NOT 门的 CMOS 实现|详细解析]]
2. **IEEE754**: `S/E/F` 拆分，`(-1)^S * 1.F * 2^(E-127)`。→ [[02_数据表示_Data_Representation#IEEE 754 单精度浮点数格式|详细解析]]
3. **Q3 逻辑表达式**: `Y = A' + BC + B'C'`。→ [[03_组合逻辑_Combinational_Logic#8.1 Sum-of-Products (SOP)|SOP 推导]]
4. **RISC-V uppercase count loop**: `lb -> beq '.' -> range check A/Z -> count++ -> pointer++`。→ [[05_补充_Q5_字符串处理模板#完整程序模板|完整模板]]
5. **AMAT**: `T_cache + miss_rate * T_main`，最后用 clock frequency 换算时间。→ [[07_存储系统_Memory_Systems#5.11.2 AMAT 计算示例|计算示例]]
