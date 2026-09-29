---
course: COMP41400 Multi-Agent Systems
week: 3
topic: ASTRA Extended Plan Syntax
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - AgentSpeak
  - PlanSyntax
---

# Week 3 - ASTRA and Extended Plan Syntax（扩展计划语法）知识点

> 来源：IntroductionToASTRA.pptx，slide 15（清单）+ slide 16（排序例子）。
> 关联笔记：[[Week3-IntroductionToASTRA-知识点]]
> 参考：ASTRA 官方指南 <https://guide.astralanguage.com/en/latest/reference/>

---

## 一、这一页在讲什么（先定位）

**为什么要"扩展"计划语法？**

纯 AgentSpeak(L) 的计划体（plan body）只有很少几种 plan operator：

| AgentSpeak(L) 原生 | 对应 ASTRA 写法 |
|---|---|
| 加/删信念 | `+belief(...)`、`-belief(...)` |
| 采纳子目标 | `!subgoal(...)` |
| 信念查询 | `query(...)`（AgentSpeak 里是 `?`） |
| 私有动作 / 外部动作 | `module.action(...)` |

它**没有** `if`、`while`、局部变量这些常规编程结构。于是写稍复杂的算法（排序、遍历、状态机）只能靠"拆成一堆 rule + 递归子目标"，规则数量会爆炸（rule proliferation，规则膨胀）。

ASTRA 的做法：**在计划体里直接引入 Java 风格的过程式语句**，同时保留 AgentSpeak 的事件驱动语义。这就是 slide 15 那张清单的意义——它列的是"**比 AgentSpeak(L) 多出来的那一层**"。

一句话概括：

> ASTRA 的计划体骨子里是一个 **procedural block（过程式代码块）**：`rule 触发事件 : 上下文 { 语句序列 }`，语句之间用 `;` 分隔，可以嵌套子块。清单里的 11 个构造就是"这个代码块里允许写什么"。

---

## 二、11 个核心构造（逐个过）

### 2.1 选择（choice）

| 构造 | 作用 | 关键点 |
|---|---|---|
| `if` statement | 最基本的流程控制（flow control） | 也支持 `else if` / `else`；条件是一个 **logical expression（逻辑表达式）**，在 agent 的信念上求值 |

**`if` vs 多规则选择（考点对比）**

| | 用多条 rule | 用 `if` |
|---|---|---|
| 写法 | 每个选项一条 rule，靠 **context（上下文）** 区分 | 一条 rule 内一个 `if / else if` 链 |
| 可读性 | 选项少时清晰，选项多时规则爆炸 | 选项多时紧凑 |
| 可扩展性 | **加选项只要加 rule**（可配合继承 override 单条分支） | 加选项要改这条 rule |
| 建议 | 用 rule 定义**完整（partial）行为** | 在一个行为**内部**用 `if` |

官方建议的类比：**你会用 method/function 的地方 → 用 rule；你会用 `if` 的地方 → 用 `if`。**

---

### 2.2 重复（repetition）—— 四种，别混淆

ASTRA 提供**四种**循环方式（slide 只列了三种，`while` 前面的"递归 rule 版循环"是第四种）：

| 方式 | 形态 | 迭代对象 | 本质 |
|---|---|---|---|
| **Recursive rules** | 一个"循环体 rule" + 一个"终止 rule"，靠子目标 `!print(X-1)` 递归 | 递归深度 | 事件驱动：每次迭代都是一个事件 |
| **`while`** | `while (guard) { ... }` | guard 真假 | 命令式循环；**连续执行，不产生事件** |
| **`foreach`** | `foreach ( belief-pattern ) { ... }` | **公式的所有变量绑定** | 更像 **plan expansion（计划展开）** |
| **`forall`** | `forall ( <变量> : <list> ) { ... }` | **列表里的每个元素** | 真·遍历列表 |

#### `foreach` 要点（易考）

```astra
rule +!init() : rate(double rate) {
    foreach (balance(string name, double amt)) {
        -balance(name, amt);
        +balance(name, amt*rate);
    }
}
```

- **guard 只求值一次**：进入循环前，先枚举出信念 `balance(string, double)` 的**全部绑定**（如 `name="Rem", amt=1000.0` 和 `name="Bob", amt=500.0`）；
- 然后对每个绑定执行一遍循环体；
- **循环体内改信念不影响迭代次数**（因为绑定已经定好了）；
- 因此它"**不是真正的循环**"，而是把循环体**复制 N 份**（每份变量取值不同）——所以叫 plan expansion；
- 语义上非常接近"计划展开/宏替换"。

#### `forall` 要点（易考）

语法：

```
forall ( <variable> : <list> ) <statement>
```

```astra
rule +!main(list args) {
    list names = ["Ringo", "John", "George", "Paul"];
    forall (string name : names) {
        C.println(name);
    }
}
```

- 遍历的是**列表**，不是信念绑定；
- **列表必须同质（homogeneous）**——所有元素同一类型；
- 变量**必须在 `forall` 里显式声明**（不能复用已有变量），**作用域仅限该语句**；
- 列表可以是字面量，也可以是变量。

> 记忆口诀：**`foreach` 迭代"信念的匹配"，`forall` 迭代"列表的元素"。**

---

### 2.3 失败与恢复

| 构造 | 作用 | 关键点 |
|---|---|---|
| `try ... recover` | 从**失败的动作**中恢复 | ASTRA 里动作（action，通常来自 module）返回 false 即"失败"，会导致整个 plan **fail**；`recover` 块提供失败后的补救路径 |

```astra
try {
    module.action(...);
} recover {
    // 补救：重试 / 报告 / 改用其它计划
}
```

这在 AOP 里特别重要：**动作失败 ≠ 程序崩溃**，而是"这条计划走不通"，agent 需要有 fallback（后备方案）。传统 AgentSpeak 只能靠"规则匹配不到 → 子目标自动失败"来处理，非过程式、很不好写。

> 语义提示：`recover` 对应 Java 的 `catch`，但注意它不是异常（exception）机制，而是**action failure（动作失败）**机制。

---

### 2.4 变量与状态

| 构造 | 说明 |
|---|---|
| **局部变量声明** | 在 plan rule 内部声明，如 `int j = 0;`、`list names = [...]`；**ASTRA 没有全局变量（no global variables）**，所有变量都在 rule 内局部 |
| **assignment（赋值）** | 改变局部变量的值：`min = k;`、`i = i + 1;` |
| **`query`** | 把信念的值**绑定到变量**：`query(time(int T));` —— 替代 AgentSpeak 的 `?` |
| **`constant`** | 在 agent 类顶层与 `initial`/`rule` 同级声明**常量**：`constant string RED = "red";`（ASTRA v1.4.4 引入），值不可变 |

**变量声明的 5 种途径（ASTRA 是强类型语言，所有变量必须有类型）**：

1. **触发事件里**：`rule +!addition(int X, int Y) { +result(X, Y, X+Y); }`
2. **规则上下文里**：`rule +!addition(int X, int Y) : result(X, Y, int R) { ... }`（从上下文抽取值）
3. **语句 guard 里**：目前只有 `query` 支持，`query(time(int T));`
4. **显式声明**：`double h = math.sqrt(X*X+Y*Y);`（等价于过程式语言的局部变量）
5. **`while` / `if` / `foreach` 的 guard 里**也可引入变量

**`query` 的语义细节（重要）**：

```astra
rule +!init() {
    C.println("starting");
    query(is("rem", "happy"));   // 成功：继续
    C.println("first hurdle passed");
    query(is("rem", "sad"));     // 失败：整个 plan 失败，后面不再执行
    C.println("ending");         // 永远不会打印
}
```

> `query` 失败 = **当前 plan 直接失败**（不是返回 false 让你判断），所以"分支"要靠 `if` 或多规则。

**`constant` 例子**（交通灯）：

```astra
agent TrafficLight {
    module Console C;
    module System S;
    types trafficlight {
        formula light(string);
        formula transition(string, string);
    }
    constant string RED = "red";
    constant string YELLOW = "yellow";
    constant string GREEN = "green";

    initial light(RED);
    initial transition(RED, GREEN), transition(GREEN, YELLOW), transition(YELLOW, RED);

    rule +!main(list args) {
        while (true) {
            S.sleep(1000);
            query(light(string S) & transition(S, string T));
            -+light(T);                 // 替换式更新：先删同谓词旧信念再加新的
        }
    }
    rule +light(GREEN) { C.println("GO GO GO!"); }
    rule +light(RED)   { C.println("STOP!"); }
}
```

要点：用常量替代 "magic number / magic string"；`-+light(T)` 是**替换（replace）**写法，等价"删掉 light 的旧值 + 加新值"。

---

### 2.5 时间、通信与并发（agent 特有的三个）

| 构造 | 作用 | 说明 |
|---|---|---|
| `wait` | **暂停（suspend）**执行，直到条件为真 | 传统编程里没有对应物：线程会停住，但 agent 的**其它 intention 仍可继续执行**（阻塞的是这条 intention，不是整个 agent） |
| `send` | 给另一个 agent 发**消息** | 走 FIPA ACL：`send(performative, "接收者", content)`，例如 `send(request, "agt_0", replicate(1));`；接收端用 `rule @message(request, string sender, replicate(int value)) { ... }` 处理 |
| `synchronized` | 临界区**互斥（mutual exclusion）** | 多个 intention 并发执行，若都要改同一份共享状态，用 `synchronized(lock) { ... }` 保证同一时刻只有一个 intention 进入临界区（critical section） |

**为什么需要 `synchronized`？** 因为 ASTRA 的 intention 是**并行执行（executed in parallel）**的：agent 可以有多个 intention，每轮推理循环挑一个执行一步。这带来真正的并发问题——共享资源（共享信念、外部模块对象）需要锁保护。

> 语法细节（`wait` / `try...recover` / `synchronized` 的具体形式）slide 未给出，考试若要求写代码，以官方 guide 和课件 lab 为准；概念层面的作用必须记住。

---

## 三、slide 16 例题精讲：用扩展语法写排序

```astra
rule +!sort(list L) {
    int j = 0;
    while (j < P.size(L)) {
        int min = j;
        int k = j+1;
        while (k < P.size(L)) {
            if (P.valueAsInt(L, min) > P.valueAsInt(L, k))
                min = k;
            k++;
        }
        if (min ~= j) {
            P.swap(L, min, j);
        }
        j++;
    }
}
```

**先纠一个常见误读**：这段代码是 **selection sort（选择排序）**，不是 bubble sort（冒泡排序）——它每轮找最小值下标 `min`，再与位置 `j` 交换。

**逐点说明**

| 代码 | 含义 |
|---|---|
| `rule +!sort(list L)` | 处理目标事件 `+!sort(...)`，参数是一个 list |
| `P.size(L)` | `P` 是 **Prelude 模块**（`module Prelude P;`），提供列表操作；`size` 返回长度 |
| `int j = 0;` | **局部变量显式声明**（这是 AgentSpeak(L) 里做不到的） |
| `while (...)` | 普通命令式循环；**不使用事件**，迭代连续执行 |
| `P.valueAsInt(L, k)` | 把 list 中第 k 个元素**当作 int 取出**（因为 list 是泛型/无类型的，需要 cast） |
| `min = k;` `j++;` | **assignment**（还有自增写法） |
| `if (min ~= j)` | `~=` 是 ASTRA 的"**不等于**"比较符（不是 Java 的 `!=`） |
| `P.swap(L, min, j)` | 调用模块动作交换两个下标的值 |

**它演示了什么（考点）**：证明了 ASTRA 的 plan body **足够强，可以直接写标准算法**——`while` + 局部变量 + 赋值 + `if` + 模块动作。换成纯 AgentSpeak(L)，同样的排序需要一大堆递归 rule，可读性差得多。

---

## 四、延伸对比：`while` 与"递归 rule 循环"的语义差别

同样打印 1..5：

**递归 rule 版**

```astra
rule +!print(int X) : X > 1 { console.println(X); !print(X-1); }
rule +!print(int X) : X == 1 { console.println(X); }
```

**while 版**

```astra
rule +!print(int X) {
    int i = 0;
    while (i < X) { console.println(i); i = i + 1; }
}
```

| | 递归 rule 版 | `while` 版 |
|---|---|---|
| 迭代机制 | 生成**事件**并交给推理循环处理 | 同一条 intention 内连续执行 |
| 是否可被打断 | **可以**：期间别的环境事件/内部事件会插进来 | **不会**：循环一次跑完 |
| 完成时间 | 无保证（要看事件队列） | 相对可预测（无事件调度开销） |
| AgentSpeak 味道 | 更"正宗"（AgentSpeak 原本就是这么写的） | 更"命令式"，但更好写 |

这是理解"**ASTRA = AgentSpeak 语义 + Java 语法**"这个张力最好的例子。

---

## 五、易考点清单

- [ ] 为什么要有 extended plan syntax？——补足 AgentSpeak(L) 只有 4 种 plan operator 的短板，减少 rule proliferation。
- [ ] **`foreach` vs `forall`** 的区别（迭代对象 + guard 求值次数 + 变量声明要求 + 同质性要求）。
- [ ] `foreach` 为什么被说成 "plan expansion" 而非循环。
- [ ] `while` 循环与递归 rule 循环在**事件交织**上的差别。
- [ ] `if` 选择与多规则选择的取舍建议。
- [ ] `query` 失败会导致**整个 plan 失败**。
- [ ] ASTRA **没有全局变量**；变量 5 种声明途径；`constant` 在第 5 种之外顶层声明。
- [ ] `try ... recover` 处理的是 **action failure**，不是异常。
- [ ] `send` 使用 FIPA ACL，配合 `@message` 事件规则接收。
- [ ] `synchronized` 存在的原因是 **intention 并行执行**。
- [ ] slide 16 排序是**选择排序**；`P` = Prelude 模块；`~=` = 不等于。
- [ ] `-+light(T)` 替换式更新。

---

## 六、一句话总结

> Slide 15 那张清单就是 ASTRA 相对 AgentSpeak(L) 的"**过程式增强层**"：`if/while/foreach/forall` 补齐控制流，`try...recover` 补齐失败处理，局部变量/赋值/`query`/`constant` 补齐数据操作，`wait/send/synchronized` 补齐时间、通信与并发；slide 16 的选择排序则证明——**ASTRA agent 的 plan body 已经可以直接写标准算法，而不必把它拆成一堆递归规则**。
