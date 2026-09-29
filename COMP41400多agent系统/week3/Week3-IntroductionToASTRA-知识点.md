---
course: COMP41400 Multi-Agent Systems
week: 3
topic: Introduction to ASTRA
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - AgentSpeak
  - BDI
---

# Week 3 - Introduction to ASTRA（知识点整理）

> 来源：IntroductionToASTRA.pptx（32 页）
> 主线：**ASTRA 定位 → Modules 机制 → 类型与程序结构 → 领域建模 → 电灯开关案例（含模块版）→ Maven**
> 配套笔记：[[Week3-ASTRA-BasicSyntax-逐页讲解]]、[[Week3-ASTRA-StaticTypes-逐页讲解]]、[[Week3-ASTRA-Modules-逐页讲解]]、[[Week3-ASTRA-Maven-知识点]]、[[Week3-ASTRA-扩展计划语法-知识点]]

---

## 一、ASTRA 是什么

**ASTRA = AgentSpeak(TR) 的实现**，本质是"**有主见的 AgentSpeak(L)**"（opinionated implementation）。

- 集成 **Teleo-Reactive（目的-反应式）** 编程概念（所以叫 AgentSpeak**(TR)**）；
- 语法不同 + 对私有动作等有具体设计。

五个关键设计：

| 特点 | 含义 |
|---|---|
| **链接 Java** | 静态类型 + Java 语法 |
| **易扩展** | Modules 支持添加动作、传感器等 |
| **熟悉的代码结构** | 循环、选择、赋值、局部变量 |
| **多继承复用** | 模块支持多重继承 |
| **Maven 构建** | 标准 `pom.xml` 工程 |

---

## 二、核心机制：Modules（模块）⭐

**Modules = Java 类，其方法可被 ASTRA 代码调用**，靠**注解（annotation）**映射。

- 是 ASTRA 提供的**扩展机制**，用于构建领域特定应用；
- **是 ASTRA agent 连接外部环境的主要方式**（slide 5 原话）。

一个 Module 可实现五种东西：

| 类型 | 作用 |
|---|---|
| **Actions** | 执行 agent 基本活动的可调用方法 |
| **Sensors** | 在推理循环**信念更新阶段**被调用的方法 |
| **Terms** | 求值后替换为常量值的方法 |
| **Formulae** | 作为逻辑表达式一部分求值的方法 |
| **Events** | 自定义事件模型 |

---

## 三、类型系统（接近 Java）

| ASTRA 类型 | 映射到 Java |
|---|---|
| `int` / `long` | `int` / `long`（4/8 字节整数） |
| `float` / `double` | `float` / `double`（4/8 字节浮点） |
| `char` / `boolean` | `char` / `boolean` |
| `string` | `java.lang.String` |
| `list` | `java.util.List` 的自定义实现 |
| `funct` | 函数项（functional terms） |
| **任意 Java 类** | 可直接作类型 |

---

## 四、Hello World 程序骨架

```astra
agent HelloWorld {
    module Console console;
    rule +!main(list args) {
        console.println("Hello World");
    }
}
```

逐部分对应：

| ASTRA 片段 | 含义 | Java 等价 |
|---|---|---|
| `agent HelloWorld` | agent 程序（类）名 | 类名 |
| `module Console console` | 引入模块库 | 引入库/对象 |
| `rule +!main(list args)` | **`+!main` = 目标采纳事件**，运行时自动触发 | `public static void main` |
| `console.println(...)` | 调用模块动作 | `System.out.println` |

> 运行 ASTRA 程序时，像 Java 一样指定"启动类"，触发 `+!main` 事件。

---

## 五、扩展计划语法（比 AgentSpeak(L) 多了啥）

ASTRA 在计划规则体内加入大量**过程式控制结构**：

| 语法 | 作用 |
|---|---|
| `if` | 基础流程控制 |
| `while` | 普通重复 |
| `foreach` | 对公式每个匹配绑定重复 |
| `forall` | 对列表所有值重复 |
| `try ... recover` | 从失败动作中**恢复** |
| 局部变量 + 赋值 | `int j = 0; min = k;` |
| `query` | 把信念值绑定到变量 |
| `wait` | 暂停直到条件为真 |
| `send` | 给其他 agent 发消息 |
| `synchronized` | 临界区互斥 |
| `constant` | 定义常量 |

> 📎 以上每一项的详细语义、`foreach` vs `forall` 对比、变量 5 种声明途径、slide 16 选择排序例题，见 [[Week3-ASTRA-扩展计划语法-知识点]]。

---

## 六、三个常用模块

| 模块 | 关键方法 |
|---|---|
| `astra.lang.Console` | `println()` 等控制台输出 |
| `astra.lang.Debug` | `dumpBeliefs()` 打印信念、`printEventQueue()` 打印事件队列、`printStackTrace()` 打印当前意图栈 |
| `astra.lang.System` | `exit()` 终止、`fail()` 失败动作、`sleep(ms)` 睡眠 |

> 完整列表见 astra-core 仓库的 astra-apis。

---

## 七、领域建模（Domain Modelling）

三个活动：

1. **对象命名约定**：环境对象映射到领域模型（如积木用 A、B、C）；
2. **识别谓词**：决定需要哪些关系及适用对象（如 `on(X,Y)`、`block(X)`、`holding(X)`、`free(X)`）；
3. **指定知识来源**：区分**感知所得（foundational/sensed）**与**可推导（derived）**。

ASTRA 两个关键字显式表达：

```astra
// 谓词声明：types + formula
types example {
    formula block(string);
    formula on(string, string);
    formula holding(string);
}

// 推理规则：inference
inference free("table") :- true;
inference free(string B) :- block(B) & ~on(string X, B);
```

- `free("table") :- true`：桌子永远空 = **基础知识**；
- `free(B) :- block(B) & ~on(X, B)`：没被压的积木是空的 = **推导知识**（对应 BDI 信念的感知 vs 推理之分）。

---

## 八、案例：ASTRA 版电灯开关

### 版本 1：纯信念操作（对应上一份 PPT 的进阶版）

```astra
agent LightSwitch {
    module Console C;
    types ls {
        formula switch(string);
        formula light(string);
    }
    initial switch("off");
    initial light("off");
    rule +!main(list args) { !switch("on"); }

    rule +!switch("on") : switch("off") { -switch("off"); +switch("on"); }
    rule +!switch("off") : switch("on") { -switch("on"); +switch("off"); }
    rule +switch("on") : light("off") { -light("off"); +light("on"); }
    rule +switch("off") : light("on") { -light("on"); +light("off"); }
    rule +light(string S) { C.println("Light is in state " + S); }
}
```

关键点：

- `initial` = 初始信念；
- `types` + `formula` = 声明谓词；
- 规则 = `rule +事件 : 上下文 { 动作 }`（等价于 AgentSpeak(L) 的 `+事件 : 上下文 <- 动作`）；
- `-`/`+` 撤回/采纳，语义完全继承 BDI；
- `+!switch("on")` 是**目标**，`+switch("on")` 是**信念事件**。

#### 执行轨迹（推理循环逐轮）

| 轮次 | 从事件队列取出的事件 | 匹配到的规则 | 执行的动作与副作用 |
|---|---|---|---|
| 1 | `+!main(args)`（运行时自动生成） | `+!main(list args)` | 采纳子目标 `!switch("on")` → 生成 `+!switch("on")` 事件，`main` 的 intention 挂起等待 |
| — | `+light("off")`（初始信念的**采纳事件**） | `+light(string S)`，`S="off"` | 打印 **Light is in state off** ← 这就是第一行输出来源 |
| 2 | `+!switch("on")` | 第 1 条 switch 规则（context `switch("off")` 成立） | `-switch("off")` 产生信念**撤回**事件（无规则匹配 → 忽略）；`+switch("on")` |
| 3 | `+switch("on")` | `+switch("on") : light("off")` | `-light("off")`（撤回事件，忽略）；`+light("on")` |
| 4 | `+light("on")` | `+light(string S)`，`S="on"` | 打印 **Light is in state on** ← 第二行输出 |
| 5 | 事件队列为空 | — | agent **空转（idle）**，程序不自动退出 |

要点：**连锁反应（chain reaction）**——一条规则的动作产生的信念事件，正好触发下一条规则，直到 `+light(...)` 的打印规则才「落地」。

注意：`initial` 声明的初始信念**同样会产生信念采纳事件**（所以启动时就打印了 `off`），这也是输出两行而不只是一行的原因。

#### 易踩的坑

1. **引号**：slide 上的 `“off”` 是弯引号（curly quote），真实代码必须用直引号 `"off"`，否则编译报错；
2. **没有退出**：事件队列空后 agent 只是空闲，要停下需 `module System S;` + `S.exit()`；
3. **类型检查**：`formula switch(string)` 等模板是编译期检查用的，谓词/参数类型不匹配直接报错；
4. **规则顺序即优先级**：同一事件的多条规则，**写在前面且 context 成立**的那条胜出（后面那条相当于 default 行为）；
5. **文件名**必须是 `LightSwitch.astra` 且与 `agent LightSwitch` 一致。

### 版本 2：把物理设备建模成 Module

**Switch 模块（Java）**：

```java
public class Switch extends Module {
    private boolean on = false;
    @ACTION public boolean set(String on) {
        this.on = (on == "on"); return true;
    }
    private Predicate belief;
    private boolean lastState = true;
    @SENSOR public void sense() {
        if (lastState == on) return;
        if (belief != null) agent.beliefs().dropBelief(belief);
        agent.beliefs().addBelief(
            belief = new Predicate("switch",
                new Term[]{ Primitive.newPrimitive(on ? "on" : "off") })
        );
        lastState = on;
    }
}
```

**修订后的 agent 代码**：

```astra
agent LightSwitch {
    module Console C;
    module Switch switch;
    module Light light;
    types ls {
        formula switch(string);
        formula light(string);
    }
    rule +!switch("on") : switch("off") { switch.set("on"); }
    rule +!switch("off") : switch("on") { switch.set("off"); }
    rule +switch("on") : light("off") { light.set("on"); }
    rule +switch("off") : light("on") { light.set("off"); }
    rule +light(string S) { C.println("Light is in state " + S); }
}
```

**两个版本的核心变化（考点）**：

| 维度 | 版本 1（纯信念） | 版本 2（模块化） |
|---|---|---|
| **状态从哪来** | `initial` 硬编码，agent 自己维护 | **`@SENSOR` 感知真实设备**，信念由环境推导 |
| **如何改变世界** | `-switch("off"); +switch("on")`（只改自己的信念） | `switch.set("on")`（调用 module 动作，改真实世界） |
| **世界的反馈** | 无 | 下一轮**感知阶段**由 `@SENSOR` 同步回信念 |
| **信念 vs 现实** | 可能脱节（belief 可以「撒谎」） | 由传感器保证一致（**grounding，接地**） |
| **决策逻辑（goal 规则 + context）** | 完全一样 | 完全一样 |
| **连锁反应规则 `+light(string S)`** | 一样 | 一样 |
| **物理因果（switch → light）** | agent 用规则**自己模拟** | 拆给 module，agent 只下发命令 |

> 一句话：**两版的「策略（policy）」没变，被换掉的只是「执行」与「感知」两处**——这正是封装（encapsulation）的意义。

- `@ACTION`：标注可被 ASTRA 调用的动作；返回 `boolean`，返回 `false` 即 **action failure**（计划失败）——此时正是 extended plan syntax 里 `try ... recover` 的用武之地；
- `@SENSOR`：在推理循环的**感知／信念修正阶段（belief revision phase）**运行，状态变化时 `dropBelief`（删旧）+ `addBelief`（加新）；
- 这演示了 ASTRA 精髓：**`agent`（ASTRA，策略）↔ `Module`（Java，环境）** 的分工。

**执行轨迹对比（版本 2 每步多一个感知周期）**

| 步骤 | 版本 1 | 版本 2 |
|---|---|---|
| 1 | 读 `initial switch("off"), light("off")` | 感知阶段 `Switch.sense()` → `+switch("off")`；`Light.sense()` → `+light("off")` |
| 2 | `+light("off")` → 打印 **off** | same → 打印 **off** |
| 3 | `!switch("on")` → `-switch;+switch` | `!switch("on")` → `switch.set("on")`（只下命令，信念**暂不变**） |
| 4 | `+switch("on")` → `-light;+light` | **下一轮感知**才产生 `+switch("on")` → `light.set("on")` |
| 5 | `+light("on")` → 打印 **on** | **再下一轮感知**产生 `+light("on")` → 打印 **on** |

**三个容易考到的深层差异**

1. **Grounding（接地）**：版本 1 里信念可以直接被规则改写，与现实毫无约束；版本 2 里信念只能由传感器「看」到的事实产生，**agent 失去了「凭空造事实」的能力**。
2. **异步与活锁风险（live-lock）**：版本 2 中 `switch.set("on")` 是异步的，信念要等下一轮才更新。若设备没真正切换，context `switch("off")` 仍成立 → 同一条规则会**反复匹配、无限重试**。版本 1 因为同步改信念，不存在这个问题。
3. **因果链归属**：「开关动 → 灯动」这个物理因果，版本 1 由 agent 的规则显式表达，版本 2 被藏进 module 里，agent 只负责「开关动 → 叫灯动」。

**⚠️ 课件这一页省略了两行**：slide 27 的代码里**没有 `initial`，也没有 `+!main`（或 `initial !switch("on")`）**。照原样运行只会打印 `off` 然后空转；要复现 slide 29 的两行输出，需自己补上启动目标，例如：

```astra
rule +!main(list args) { !switch("on"); }
```

输出（slide 29）：

```
[main]Light is in state off
[main]Light is in state on
```

---

## 九、Maven 构建

| 项 | 内容 |
|---|---|
| ASTRA 源码目录 | `/src/main/astra` |
| Java 代码目录 | `/src/main/java` |
| 构建文件 | `pom.xml` |
| 基础父工程 | `astra:astra-base:2.0.13` |
| 构建插件 | `astra-maven-plugin` |
| 编译 | `mvn compile` |
| 部署 | `mvn astra:deploy` |

- `pom.xml` 里 `astra.main` 属性指定启动 agent（如 `LightSwitch`）。
- 也可用 archetype 生成模板工程：`mvn archetype:generate -DarchetypeArtifactId=astra-archetype -DarchetypeVersion=2.0.13 …`

---

## 十、易考点汇总

- [ ] ASTRA 与 AgentSpeak(L) 的关系（AgentSpeak(TR) 的实现、TR = Teleo-Reactive）。
- [ ] ASTRA 的五个设计特点（Java 链接、Modules 扩展、熟悉代码结构、多继承、Maven）。
- [ ] **Module 的五个可实现类型**：Actions / Sensors / Terms / Formulae / Events。
- [ ] **`@ACTION` vs `@SENSOR`** 的作用与调用时机。
- [ ] **Modules 是 agent 连接环境的主要方式**。
- [ ] `agent / module / rule / types / formula / inference / initial` 各关键字含义。
- [ ] `+!main(list args)` 与 Java `main` 的对应。
- [ ] 扩展语法：`if/while/foreach/forall/try...recover/query/wait/send/synchronized/constant`。
- [ ] 领域建模三活动 + `inference` 推导规则的写法（`:-` 与 `~`）。
- [ ] Light Switch 两版本区别：纯信念 vs 模块化（动作 + 传感器同步）。
- [ ] Maven 目录结构：astra 代码在哪、java 代码在哪、怎么编译部署。

---

## 十一、一句话总结

> ASTRA 是 AgentSpeak(L) 的 Java 化工程实现：语法逼近 Java（静态类型、控制流、Maven），核心仍是 BDI（信念/目标/计划规则 + 事件队列 + 感知-深思-行动循环）；最大的创新是 **Modules**——用 `@ACTION/@SENSOR` 注解把 Java 代码接进 agent，成为连接外部环境的桥梁，从而把"策略（agent）"与"环境（module）"干净地分离。