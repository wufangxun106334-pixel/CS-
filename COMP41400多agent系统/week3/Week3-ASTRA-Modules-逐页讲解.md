---
course: COMP41400 Multi-Agent Systems
week: 3
topic: ASTRA Modules
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - Modules
  - AgentSpeak
  - Philosophy
---

# Week 3 - ASTRA: Modules（逐页讲解 + 哲学补充）

> 来源：ASTRA-Modules.pptx（18 页）
> 关联笔记：[[Week3-IntroductionToASTRA-知识点]]、[[Week3-ASTRA-扩展计划语法-知识点]]
> 参考：<https://guide.astralanguage.com/en/latest/learning/#52-creating-your-own-astra-modules>
>
> 本讲是 ASTRA 部分的核心：上一讲 StaticTypes 留下的 `// We cannot do this yet!` 在这里补上——**动作必须由 module 实现**。

---

# 第一部分：逐页讲解

## 组 A：概念与声明（第 2–3 页）

### 第 2 页 — Modules 是什么

> **Modules = Java 类，其方法可以被 ASTRA 代码调用。** 它是 ASTRA 提供的 **extension mechanism（扩展机制）**，用来构建 **domain specific applications（领域特定应用）**；ASTRA 概念与 Java 方法的映射通过 **annotations（注解）**完成。

可实现的**五种**东西：

| 类型 | 注解 | 含义 |
|---|---|---|
| **Actions** | `@ACTION` | 可被调用以完成 agent 基本活动的方法 |
| **Sensors** | `@SENSOR` | 在推理循环的 **belief revision phase（信念修正阶段）** 被调用的方法 |
| **Terms** | `@TERM` | 求值后被替换为常量值的方法 |
| **Formulae** | `@FORMULA` | 可作为一个逻辑表达式的一部分被求值的方法 |
| **Events** | `@EVENT` | 自定义事件模型，可创建额外的事件类型 |

> Modules 是 **agent 连接外部环境的主要方式**（另一份课件的原话）。

### 第 3 页 — 声明与使用

```java
package mypackage;
public class MyModule extends Module { }      // ← 必须 extends astra.core.Module
```

```astra
agent MyAgent {
    module mypackage.MyModule mod;             // ← module 关键字，别名 mod
}
```

---

## 组 B：Actions（第 4–9 页）

### 第 4 页 — 定义动作

```java
public class MyModule extends Module {
    @ACTION
    public boolean helloAction() {
        System.out.println("Hello World!");
        return true;                            // ← 必须返回 boolean
    }
}
```

两条硬规则（考点）：

1. **动作必须返回 `boolean`**，表示**成功 / 失败**；
2. **未处理的异常（unhandled exception）也视为动作失败**。

> 接上 [[Week3-ASTRA-扩展计划语法-知识点]] 里的 `try ... recover`：动作失败 → 计划失败 → 需要 recover 兜底。

### 第 5 页 — 调用动作

```astra
agent MyAgent {
    module mypackage.MyModule mod;
    initial !init();
    rule +!init() { mod.helloAction(); }        // ← 语法：<模块别名>.<方法名>(参数)
}
```

### 第 6–7 页 — 传参与重载决议（overload resolution）

```java
@ACTION public boolean helloAction()             { ... }
@ACTION public boolean helloAction(String name)  { ... }   // ← Java 重载
```

```astra
rule +!init() { mod.helloAction("Rem"); }        // ← ASTRA 侧看不出区别
```

**第 7 页那句话是整份 PPT 的枢纽**：

> *The ASTRA interpreter automatically maps ASTRA types to Java types and identifies the correct method to execute. If no matching method exists, a compilation error occurs.*

- ASTRA 类型 → Java 类型**自动映射**；
- **重载决议**由解释器完成 → 回答了 StaticTypes 那讲「静态类型是为了什么」；
- **没有匹配的方法 = 编译错误**（不是运行时才炸）→ 动作调用的类型安全。

### 第 8–9 页 — 传列表

```java
@ACTION public boolean helloAction(ListTerm list) {
    for (Term term : list) { System.out.println("Hello World, " + term + "!"); }
    return true;
}
```

| 类 | 说明 |
|---|---|
| `astra.term.ListTerm` | 实现了 `java.util.List<Term>` 接口（可直接 for-each） |
| `astra.term.Term` | **所有 ASTRA term 对象的基类** |

```astra
rule +!init() { mod.helloAction(["Rem", "Joe"]); }     // ASTRA 侧直接传列表字面量
```

---

## 组 C：Sensors 原理（第 10 页）

```java
@SENSOR public void foo() { /* 干活 */ }
```

三个要点（考点）：

1. **无返回值**（`void`）；
2. 是 **self-contained belief revision function（自包含的信念修正函数）**——**必须自己增删信念**，解释器不会替你同步；
3. **由 agent 解释器自动调用**，每轮推理循环都会跑（agent 无法手动"调"传感器）。

> 语义位置：`@SENSOR` 就是 BDI 推理循环里 **感知（perceive）** 那一步。每轮循环：**感知 → 处理事件 → 执行计划**。

---

## 组 D：完整案例——虚拟灯泡（第 11–18 页）

### 第 11–12 页 — 造一条信念

```java
Predicate belief = new Predicate("light", new Term[] {
    Primitive.newPrimitive(state ? "on" : "off")     // ← 三元表达式选字符串
});
agent.beliefs().addBelief(belief);
```

| 类 / 方法 | 位置 | 说明 |
|---|---|---|
| `astra.formula.Predicate` | `astra.formula` 包 | 一条信念（谓词 + 参数数组） |
| `astra.term.Primitive` | `astra.term` 包 | 原始值；`newPrimitive(...)` 是**静态工厂方法（factory method）** |
| `agent.beliefs().addBelief(...)` | 继承自 `Module` 的 `agent` 字段 | 访问 agent 信念库 |
| `agent.beliefs().dropBelief(...)` | 同上 | 删除信念 |

### 第 13 页 — 别忘了声明谓词

```astra
agent MyAgent {
    types lights { formula light(string); }      // ← module 产生的信念同样要在 types 里声明
    module mypackage.MyModule mod;
    rule +light(string S) { /* ... */ }
}
```

> 呼应 StaticTypes：**类型检查在编译期，无论信念来自感知还是推理。**

### 第 14 页 — 虚拟灯需要动作来改状态

```java
@ACTION public boolean setLight(String newState) {
    state = newState.equals("on");
    return true;
}
```

### 第 15–16 页 — sensor 必须「先删后加」

```java
private Predicate belief;                            // ← 记住上次加进去的那条信念

@SENSOR public void foo() {
    if (belief != null) agent.beliefs().dropBelief(belief);   // ← 删掉不再为真的旧信念
    belief = new Predicate("light", new Term[] {
        Primitive.newPrimitive(state ? "on" : "off")
    });
    agent.beliefs().addBelief(belief);                        // ← 再加新的
}
```

**为什么必须 drop？** sensor 每轮都跑，只 add 不 drop 会**同时堆积 `light("on")` 与 `light("off")`** → 信念库不一致（inconsistent），`+light(string S)` 规则被反复触发。这是信念库的「垃圾回收」问题。

### 第 17–18 页 — 效率优化：脏检查（dirty check）

```java
private boolean changed = true;                      // ← 只有状态真的变了才同步

@SENSOR public void foo() {
    if (changed) {
        if (belief != null) agent.beliefs().dropBelief(belief);
        belief = new Predicate("state", new Term[] { ... });   // ⚠️ 谓词名笔误，见下
        agent.beliefs().addBelief(belief);
        changed = false;
    }
}

@ACTION public boolean setLight(String newState) {
    boolean ns = newState.equals("on");
    if (ns != state) { state = ns; changed = true; }   // ← 真变了才置脏标志
    return true;
}
```

**优化的真正意义（深层考点）**：每轮都 drop+add 会产生**信念事件**（`-light(...)`、`+light(...)`），而事件会触发规则 → 形成 **event storm（事件风暴）**，agent 一直忙于处理自己制造的事件，**永不空闲（never idles）**。`changed` 标志把「每轮同步」变成「**变化时才同步**」。

---

## 三处课件笔误（做作业别抄错）

| 页 | 问题 | 后果 |
|---|---|---|
| 第 14、18 页 | `newState.equals("on”)` —— 弯引号（curly quote） | **编译错误**，必须写直引号 `"on"` |
| 第 17 页 | 谓词名误写成 `"state"`（前面都是 `"light"`） | `+light(string S)` 规则**永不触发**，案例跑不通 |
| 第 5、13 页 | 混用两套启动方式（`initial !init();` vs `rule +!main(...)`） | 都能跑，但别在同一程序里混着用而不自知 |

---

## 本讲考点清单

- [ ] **Modules = Java 类 + 注解**，是 ASTRA 的扩展机制与连接环境的主要方式。
- [ ] 五种可实现类型：**Actions / Sensors / Terms / Formulae / Events**。
- [ ] `extends astra.core.Module` + `module <包名>.<类名> <别名>;`
- [ ] `@ACTION` 方法**必须返回 boolean**；**未处理异常 = 失败**。
- [ ] **重载决议（overload resolution）** + 无匹配方法 = **编译错误**（这是静态类型的理由）。
- [ ] `astra.term.ListTerm` 实现 `java.util.List<Term>`；`astra.term.Term` 是所有 term 的基类。
- [ ] `@SENSOR` **void、无返回值、每轮自动调用、必须自己 add/drop 信念**。
- [ ] 信念用 `astra.formula.Predicate` + `Primitive.newPrimitive(...)` 构造；`agent.beliefs().addBelief/dropBelief`。
- [ ] **只 add 不 drop → 信念库不一致**；**每轮 drop+add → 事件风暴**；解法是 `changed` 脏检查。
- [ ] module 产生的信念也必须在 `types` 中声明。

---

# 第二部分：哲学知识补充

> 把「模块 = 心智与世界之间的接口」放进 AI 哲学与心灵哲学的坐标系。**★ = 与本课程直接相关（可作论述题角度）**，其余为拓展。

## ★ 1. 三层次视角：知识层 / 符号层（Newell 1982）

Newell 在 *The Knowledge Level* 中提出，解释系统可在两个层次进行：

| 层次 | 问的问题 | ASTRA 里的对应 |
|---|---|---|
| **Knowledge level（知识层）** | 「它**知道**什么、**想要**什么、为何这么做」 | **ASTRA agent 程序**（beliefs / goals / plan rules） |
| **Symbol level（符号层）** | 「这些知识**如何被物理实现**」 | **Modules**（Java 代码、对象、I/O） |

> 这解释了为什么两者要**分开写**：agent 程序用「知识」的语言描述行为，module 才碰「符号/物理」的实现。**Modules 不只是工具函数库，它是知识层落地的插槽。**

## ★ 2. 多层描述与 Dennett 的三种立场（Week 3 前文已讲）

| Stance（立场） | 解释依据 | 在本讲中的位置 |
|---|---|---|
| Physical stance（物理立场） | 物理/化学规律 | Java 代码、内存里的 `boolean state` |
| Design stance（设计立场） | 功能、用途 | **Modules**（`@ACTION`/`@SENSOR` 即"这个组件**用来**做什么"） |
| Intentional stance（意向立场） | 相信什么、想要什么、打算做什么 | **agent 的 rule / context / goal** |

**同一个模块可从三个立场描述**，而 agent 程序只活在第三个立场——这就是「模块化」的哲学含义：**让每一层只用它该用的词汇讲话。**

## ★ 3. 符号接地问题（Symbol Grounding Problem, Harnad 1990）

Harnad 的问题：**纯符号系统的「意义」从哪来？** 若符号只由别的符号定义，语义永远悬空。

回看 LightSwitch **版本 1**：agent 用 `-switch("off"); +switch("on")` 自己改写信念——**它可以"相信"灯亮着而灯其实关着**。这是**无指称（ungrounded，悬空）**的符号游戏。

`sensor` 的哲学地位由此浮现：**@SENSOR 是符号指称的锚点（anchor of reference）**——它把符号 `light("on")` 通过**因果耦合（causal coupling）**接到世界的真实状态。对应课程里的划分：

| 课程术语 | 哲学叫法 |
|---|---|
| **foundational / sensed knowledge（基础/感知知识）** | 非推导的**基础信念**（foundationalism，基础主义） |
| **derived knowledge（推导知识，`inference`）** | 在基础之上的**推论** |

> ASTRA 的 `types` / `inference` / `@SENSOR` 三分法，其实是**基础主义认识论（foundationalist epistemology）的工程化版本**。

## ★ 4. 框架问题（Frame Problem）与「世界就是它自己最好的模型」

McCarthy & Hayes（1969）的 **frame problem**：agent 要推理「做动作 A 后世界变成什么样」，就得知道**哪些事实变了、哪些没变**——逐条枚举会爆炸。

| 做法 | 立场 |
|---|---|
| 版本 1：手动 `-switch; +switch`，心里跑一个**微缩世界模型** | 必然撞上框架问题 |
| Modules：不维护，**每轮直接感知** | Brooks: *the world is its own best model* |

立场之争：

| 传统立场 | 具身/情境立场（本讲） |
|---|---|
| **representationalism（表征主义）**：认知 = 内部表征的运算 | **situated / embodied cognition（情境化、具身认知）**：认知 = agent 与环境的实时耦合 |
| sense–model–plan–act | perceive–act 闭环（`@SENSOR` ↔ `@ACTION`） |
| 关心「心里有没有正确的世界模型」 | 关心「行为是否在世界中行得通」 |

> ASTRA **不是**纯反应式系统——它有 BDI 的慎思（deliberation），是 **hybrid architecture（混合架构）**：BDI 提供慎思层，Modules 提供与世界的**反应式耦合（reactive coupling）**。

## 5. 功能主义与多重可实现性（Functionalism, Putnam）

**functionalism（功能主义）**主张：心理状态由**功能角色**定义，不由物质载体定义；推论是 **multiple realizability（多重可实现性）**。

```astra
module Switch switch;     // 可以是真开关，也可以是虚拟 boolean
module Light light;       // 可以是真灯，也可以是 println
```

**同一份 agent 程序 + 不同 module = 同一「心智」、不同「身体」**。第 14 页那句「*In a physical system, the state would be based on some sensed value (not an internal field)*」正说明：虚拟灯只是**实现层**的替身，心智层不该知道它的存在。

## 6. 海德格尔：上手状态 vs 现成在手（ready-to-hand / present-to-hand）

《存在与时间》：工具在**顺畅使用**中不成为对象（**Zuhandenheit, ready-to-hand，上手状态**）；只有**坏掉**时才变成被注视的对象（**Vorhandenheit, present-to-hand，现成在手**）。

- 动作成功时，module 是**透明的**——agent 无需"思考" `light.set("on")` 是 Java 代码；
- 一旦 `@ACTION` 返回 `false` 或抛异常（**breakdown，故障**），module 立刻从"透明工具"变成"需要注意的对象" → 触发 `try ... recover`。

> **`@ACTION` 的 `boolean` 返回值，就是「上手 / 不上手」这一现象学区分的工程化标志。** 传感器亦然：感知正常时其产物被直接当真，只有出问题才"浮出水面"。

## 7. 语言哲学的一小口：类型 = 把语义约束外显为语法约束

`syntax（句法）` vs `semantics（语义）`：

- 纯逻辑式语言（AgentSpeak(L)）里，`on("a", 5)` 是**句法合法、语义错误**——只能运行时发现；
- ASTRA 用 `types` + 静态类型 + 「无匹配方法 = 编译错误」，把一部分语义约束**提前固化进语法**。

> 哲学上：**类型系统是"把意义的一部分搬进形式"的尝试**。这也解释了 ASTRA 为何舍弃 AgentSpeak(L) 的简洁——它**用表达自由度换取了可检查性（checkability）与 Java 互操作**。

---

## 一句话总结

> **技术**：Modules 是 Java 类 + 注解（`@ACTION` / `@SENSOR` / `@TERM` / `@FORMULA` / `@EVENT`），是 agent 连接外部环境的唯一桥梁；动作返回 `boolean` 表成败，传感器每轮自动运行且必须自己增删信念（否则信念库污染、事件风暴），生产级写法要加脏检查。
>
> **哲学**：agent 程序活在**知识层 / 意向立场**，modules 活在**符号层 / 设计立场**；`@SENSOR` 解决**符号接地**，`perceive–act` 闭环回避**框架问题**，同一程序可挂不同 module 体现**功能主义的多重可实现性**，而 `boolean` 返回值承担了**上手 / 现成在手**的分界。
