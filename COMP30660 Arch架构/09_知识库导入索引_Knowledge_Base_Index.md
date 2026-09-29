# 09 知识库导入索引 (Knowledge Base Index)

> 更新日期: 2026-05-09
> 笔记目录: `/Users/alex/Documents/Obsidian Vault/COMP30660 Arch架构`
> 课程资料目录: `/Users/alex/Documents/COMP30660 Arch架构`
> 本索引用于把 week1-week10 的 PDF、ASM、MD、TXT 资料合并到现有 Obsidian 笔记体系中。

---

## 导入原则

- **不重复粘贴相同知识点**: 已在章节笔记中完整覆盖的内容，只在这里记录来源与对应笔记。
- **同一知识点多来源合并**: 讲义、worksheet、solutions、ASM 示例共同指向同一个复习入口。
- **PDF 用于理论与题目来源**: 章节 PDF 是概念主来源，worksheet PDF 是练习主来源。
- **ASM 用于程序模板来源**: `.asm` 文件按程序模式归并到 RISC-V 汇编笔记和补充模板。
- **CIRC/DOCX/XLSX 仍保留为原始实验资料**: 当前索引主要覆盖 PDF、ASM、MD、TXT 正文。

---

## 总入口

| 类型 | 路径 |
|------|------|
| Obsidian 笔记目录 | `/Users/alex/Documents/Obsidian Vault/COMP30660 Arch架构` |
| 本地导入知识库 | `/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base` |
| 导入索引 | `/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base/index.md` |
| 导入清单 | `/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base/manifest.json` |
| PDF 抽取正文 | `/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base/extracted_text` |
| ASM 导入副本 | `/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base/asm` |

导入统计:

- PDF: 20 个
- ASM: 13 个
- Markdown: 8 个
- TXT: 1 个
- 失败: 0 个
- 总文本量: 约 1,186,039 字符

---

## 合并后的知识点地图

| 合并知识点 | Obsidian 主笔记 | 来源资料 | 备注 |
|------------|-----------------|----------|------|
| Digital Circuits / CMOS / Logic Gates | [[01_数字电路_Digital_Circuits]] | `week1/Ch 1 Digital Circuits.pdf` | 现有笔记已覆盖物理基础、晶体管、CMOS、功耗、逻辑门；无需重复粘贴 PDF 文本 |
| Data Representation / Two's Complement / IEEE 754 | [[02_数据表示_Data_Representation]] | `week2/Ch 2 Data Representation.pdf`, `week2/Ws 1 Data Represenetation.pdf`, `week2/Ws 1 Data Representation Solutions.pdf` | 讲义概念 + Worksheet 1 题解合并到 Ch2 与 [[08习题汇总_Worksheets]] |
| Combinational Logic / Boolean Algebra / K-map / Adders | [[03_组合逻辑_Combinational_Logic]] | `week3/Ch 3 Combinational Logic.pdf`, `week3/Ws 2 Combinational Logic.pdf`, `week3/Ws 2 Combinational Logic Solutions.pdf`, `week4/Ws 3 Combinational Logic Solutions.pdf` | Week3/Week4 同属组合逻辑，按“设计方法 + 化简 + 常用电路 + 习题”合并 |
| Timing Diagram / Sample Paper Q3 | [[03_补充_Q3_时序图绘制指南]] | `week3/Ws 3-Lab3-Tasks.pdf`, 样卷解析 | 与组合逻辑输出随输入变化的题型合并，不另开重复章节 |
| Sequential Logic / Latch / FF / Register / Counter / Memory Array | [[04_时序逻辑_Sequential_Logic]] | `week5/Ch 4 Sequential Logic.pdf`, `week5/Ws 4 Mini Processor.pdf`, `week5/Ws 4solutions/w4q4.txt` | Mini Processor 资料归入时序逻辑与处理器基础连接点 |
| Architecture / RISC-V 基础 | [[05_体系结构_Architecture]] | `week7/Ch 5 Architecture.pdf`, `week7/week7_ch5_architecture/*.md`, `week7/solutions/*.asm` | Week7 中文 MD 与 Ch5 主笔记重复较多，以 Ch5 主笔记为 canonical |
| RISC-V 字符串、数组、循环、函数模板 | [[05_补充_Q5_字符串处理模板]], [[05_体系结构_Architecture]] | `week8/*.asm`, `week9/*.asm`, `week10/riscv1.asm`, `week8/Ws 6 Assembly Language.pdf`, `week9/Ws 7 Assembly Language - Copy.pdf`, `week10/Ws 8 Assembly.pdf` | ASM 示例按程序模式合并，见下方“ASM 模板索引” |
| Machine Code 编码/反汇编 | [[05_体系结构_Architecture]] | `week7/Ch 5 Architecture.pdf`, `week7/week7_ch5_architecture/06_机器码编码与反汇编.md` | 统一入口为 Ch5 的指令格式、编码/解码示例 |
| Microarchitecture / Datapath / Control / Pipeline | [[06_微架构_Microarchitecture]] | `week9/Ch 6 Microarchitecture.pdf` | 与 Architecture 区分: Architecture 是程序员可见接口，Microarchitecture 是实现方式 |
| Memory Hierarchy / Cache / Virtual Memory | [[07_存储系统_Memory_Systems]] | `week10/Ch 7 Memory Systems.pdf`, `week10/Ws 8 Assembly.pdf`, `week10/Worksheet 8 Solutions.pdf` | 讲义与 Worksheet 8 合并到 Ch7 和习题汇总 |
| Sample Paper 综合题 | [[08_样卷解析_Sample_Paper_Analysis]] | `COMP30660 Sample Paper.pdf` | 跨章节题型总入口 |
| Worksheets 1-8 | [[08习题汇总_Worksheets]] | Week2-Week10 worksheet PDFs / solutions PDFs | 题目与答案按章节归并，不与理论章节重复 |

---

## ASM 模板索引

这些 `.asm` 文件不按文件名孤立记忆，而按常见程序结构合并到 RISC-V 模板中。

| 程序模式 | 典型指令/结构 | 来源 ASM | 合并到 |
|----------|---------------|----------|--------|
| 两数加载、相加、存储 | `.data`, `.text`, `lw`, `add`, `sw`, `ecall` | `week7/solutions/w5q3.asm`, `week8/firstprogram.asm`, `week10/riscv1.asm` | [[05_体系结构_Architecture#6. 首个完整程序与汇编指示]] |
| 数组逐项读取与比较 | `la`, `lw`, `addi`, `bne`, `bgt`, `sw` | `week9/Q1.asm`, `week9/Q11.asm`, `week9/solutions-2/w7q1.asm`, `week9/solutions-2/w7q2.asm` | [[05_体系结构_Architecture#8. 分支、循环与函数调用]] |
| 字符串复制 | `lb`, `sb`, 指针递增, null/终止符判断 | `week8/solutions/w6q1.asm`, `week8/solutions/w6q2.asm` | [[05_补充_Q5_字符串处理模板]] |
| 字符大小写转换 | ASCII 范围判断, `blt`, `bgt`, `addi -32` | `week8/solutions/w6q3.asm`, `week8/solutions/w6q4.asm` | [[05_补充_Q5_字符串处理模板#常见变体]] |
| 函数调用与返回 | `jal`, `jalr`, `ra`, `a0`, 函数标签 | `week9/solutions-2/w7q2.asm` | [[05_体系结构_Architecture#8.5 函数调用]] |
| 系统调用退出 | `li a7, 10`, `ecall` | 多数 ASM 示例 | [[05_体系结构_Architecture#14.10 `ecall` (Environment Call)]] |

### ASM 去重规则

- 同样的 `.data` / `.text` / `ecall` 结构只保留在 Ch5 基础模板中解释一次。
- `lw/sw/lb/sb` 的区别只在 Ch5 的内存访问表和字符串模板中分别说明，不在每个 ASM 示例重复解释。
- `bgt/ble` 作为伪指令的提醒统一保留在 Ch5“易混淆概念”中。
- 字符范围判断统一归入 Q5 字符串模板；数组比较统一归入分支/循环模板。

---

## Worksheet 合并索引

| Worksheet | 核心知识点 | 对应主笔记 |
|-----------|------------|------------|
| Worksheet 1 | 进制转换、Two's Complement、IEEE 754、ASCII | [[02_数据表示_Data_Representation]], [[08习题汇总_Worksheets]] |
| Worksheet 2 | 真值表、布尔表达式、逻辑图、组合逻辑设计 | [[03_组合逻辑_Combinational_Logic]], [[08习题汇总_Worksheets]] |
| Worksheet 3 | 布尔代数化简、K-map、Logisim 组合电路 | [[03_组合逻辑_Combinational_Logic]], [[08习题汇总_Worksheets]] |
| Worksheet 4 | Sequential Logic、Mini Processor、寄存器/控制 | [[04_时序逻辑_Sequential_Logic]], [[06_微架构_Microarchitecture]] |
| Worksheet 5 | RARS / RISC-V 基础程序 | [[05_体系结构_Architecture]] |
| Worksheet 6 | Assembly Language、字符串/字节操作 | [[05_体系结构_Architecture]], [[05_补充_Q5_字符串处理模板]] |
| Worksheet 7 | 数组、循环、函数调用、分支 | [[05_体系结构_Architecture]], [[06_微架构_Microarchitecture]] |
| Worksheet 8 | Memory Systems、Cache、Assembly 综合 | [[07_存储系统_Memory_Systems]], [[08习题汇总_Worksheets]] |

---

## 缺口与后续更新建议

当前笔记已经覆盖主要知识点。后续如果继续精修，优先做这三类低重复、高收益更新:

1. **在 [[08习题汇总_Worksheets]] 中按题号补来源路径**: 让每道题能回到原 PDF/solution。
2. **在 [[05_补充_Q5_字符串处理模板]] 增加“数组比较/函数调用”小节**: 复用 Week9 ASM，不重复已有字符串扫描模板。
3. **在 [[07_存储系统_Memory_Systems]] 增加 Worksheet 8 题型索引**: 把 cache 地址划分、AMAT、虚拟内存题型单独列出。

---

## 检索提示

在终端或 Obsidian 外部检索时，可直接搜导入知识库:

```bash
rg -n "Two's Complement|Floating Point|K-map|jalr|AMAT|Page Table" "/Users/alex/Documents/COMP30660 Arch架构/.workbuddy/knowledge_base"
```

在 Obsidian 中复习时，优先从以下入口进入:

- [[00_课程总览_Overview]]
- [[08习题汇总_Worksheets]]
- [[05_体系结构_Architecture]]
- [[05_补充_Q5_字符串处理模板]]
- [[06_微架构_Microarchitecture]]
- [[07_存储系统_Memory_Systems]]
