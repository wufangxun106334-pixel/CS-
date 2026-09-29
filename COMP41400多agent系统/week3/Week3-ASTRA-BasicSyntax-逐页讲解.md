---
course: COMP41400 Multi-Agent Systems
week: 3
topic: ASTRA Basic Syntax
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - AgentSpeak
  - BasicSyntax
---

# Week 3 - ASTRA: Basic Syntax（逐页讲解）

> 来源：ASTRA-BasicSyntax.pptx（6 页）
> 关联笔记：[[Week3-IntroductionToASTRA-知识点]]、[[Week3-ASTRA-扩展计划语法-知识点]]、[[Week3-ASTRA-StaticTypes-逐页讲解]]、[[Week3-ASTRA-Modules-逐页讲解]]、[[Week3-ASTRA-Maven-知识点]]

本讲是「ASTRA 程序长什么样」的最小骨架演示，用的例子是教科书经典 **Blocksworld / Tower（积木塔）**——即 AgentSpeak(L) 论文里的原版例子。

## 一、6 页递进逻辑

```
第2页 文件与类骨架 → 第3页 本体(谓词词汇表) → 第4页 初始状态
                                          ↓
                     第6页 推理规则(inference) ← 第5页 计划规则(rule)
```

---

## 二、第 2 页：文件与 agent 类骨架

```astra
/**
 * 注释和 Java 一样
 */
agent Tower {              // ← 一个文件 = 一个 agent 类
    // 代码写在这里
}
```

| 规则 | 内容 |
|---|---|
| **文件扩展名** | `.astra` |
| **每个文件只能有一个 agent 类** | 用 `agent` 关键字声明 |
| **文件名必须与类名一致** | 上面这段必须存在 `Tower.astra` 里 |
| **注释** | 与 Java 完全相同：`//` 单行、`/** */` 文档注释 |

> 这是 ASTRA「**语法上贴近 Java**」的第一条体现，也是它与 AgentSpeak(L)（`.asl` + 逻辑式语法）最直观的差别。

---

## 三、第 3 页：Ontology（本体）——用 `types` 声明谓词

```astra
agent Tower {
    types tower_ont {                    // ← 本体名（label），可自定义，一个程序可有多个
        formula block(string);           // 积木：一块积木的名字
        formula table(string);           // 桌子
        formula on(string, string);      // on(A,B)：A 放在 B 上
        formula free(string);            // free(A)：A 上面没东西
        formula holding(string);         // holding(A)：机械手抓着 A
    }
    // ...
}
```

**这页在讲什么**：ASTRA 提供「定义 **ontology**（本体）」的机制——按课件原话，本体 = **the set of valid predicates for your program（程序里合法谓词的集合）**。

| 项 | 说明 |
|---|---|
| `types <label> { ... }` | 声明一组**信念模板（belief template）** |
| `formula name(type, ...)` | 声明一个谓词：名字 + **arity（元数，参数个数）** + 每个参数的类型 |
| 作用 | **编译期类型检查**：用到没声明的谓词、类型不匹配 → 直接报错 |
| 副产品 | 逼开发者**先想清楚领域模型**再写行为（对应「领域建模」三活动里的"识别谓词"） |

> 与 LightSwitch 例子里的 `types ls { formula switch(string); formula light(string); }` 完全同构——只是换了本体名与谓词。

---

## 四、第 4 页：初始状态——用 `initial` 声明初始信念与初始目标

```astra
agent Tower {
    types tower_ont { ... }

    initial block("a"), block("b"), block("c"), block("d");
    initial table("table");
    initial on("a", "table"), ...
    initial !tower("a", "b", "c");        // ← 注意开头的 !
    // ...
}
```

| 写法 | 类型 | 含义 |
|---|---|---|
| `initial block("a"), block("b");` | **初始信念**（initial belief） | agent 一出生就相信的事实；逗号可并列多条 |
| `initial table("table");` | 初始信念 | 桌子本身也建模成一个（唯一的）对象 |
| `initial !tower("a", "b", "c");` | **初始目标**（initial goal） | 开头 `!` = 目标；产生 **goal adoption event**，去触发 `+!tower(...)` 规则 |

**两个关键点**

1. `initial` 的内容按有没有 `!` 分成两类——这就是 BDI 里 **belief（描述世界是什么样）** 与 **goal（描述想要世界变成什么样）** 的分水岭。
2. `!tower("a","b","c")` 是 **declarative goal（声明式目标）**：目标状态本身可以写成信念公式（`on(...)` 之类）。
   > ⚠️ 这 6 页**没有给出 `+!tower(...)` 规则**——没有规则响应的目标会直接失败。这页只演示「目标怎么声明」。

---

## 五、第 5 页：计划规则——用 rule 实现目标

```astra
rule +!holding(string X) : ~holding(string Y) & free(X) & on(X, string Z) {
    +holding(X); -free(X); -on(X, Z);
}
```

| 片段 | 术语 | 说明 |
|---|---|---|
| `rule` | 关键字 | 声明一条 plan rule |
| `+!holding(string X)` | **触发事件**：目标采纳 | 「当有人要求我达成 `!holding(X)` 时」；`string X` 既声明类型，又从事件里绑定参数 |
| `:` | 上下文分隔符 | 左边响应什么，右边什么条件下可用 |
| `~holding(string Y)` | 上下文中的**否定字面量** | `~` = not；`Y` 是**存在性变量（匿名）**——只要"没抓着任何东西"，Y 的值后面用不到 |
| `free(X)` | 上下文 | 引用 X 已绑定的值 |
| `on(X, string Z)` | 上下文 | 第二个参数**在使用处内联声明类型**并绑定 `Z` |
| `&` | 合取（and） | 三个条件同时成立，规则才 applicable（可用） |
| `{ ... }` | **计划体** | 语句顺序执行，各以 `;` 结尾 |

**语义**：「要抓起 X，前提是我手里没东西 + X 上面没东西 + X 正压在 Z 上；那就加入 `holding(X)`、删掉 `free(X)` 和 `on(X,Z)`。」

> 对照 AgentSpeak(L) 原版（几乎逐字翻译）：
> ```prolog
> +!holding(X) : not holding(Y) & free(X) & on(X,Z) <- +holding(X); -free(X); -on(X,Z).
> ```
> 差别只在 `not` → `~`、`<-` → `:` + `{}`、末尾 `.` → `}`。

---

## 六、第 6 页：推理规则——`inference` 定义「推导出来的信念」

```astra
inference free(string X) :- ~on(string Y, X) & ~holding(X);
```

读法：**「如果没有任何 Y 压在 X 上，且我没抓着 X，那么 X 就是 free 的。」**

| 项 | 说明 |
|---|---|
| `inference` | 关键字，声明推理规则 |
| `:-` | 「**如果**」，读作 *if*（注意与计划规则的 `:` 不同，这里是两个字符） |
| 左边 `free(string X)` | **head（头）**：被推出的新信念 |
| 右边 `~on(...) & ~holding(X)` | **body（体）**：成立条件，在**基础信念**上求值 |
| 作用 | 类似 Prolog 的规则、或数据库的**视图（view）**：不存 `free`，用到时才算 |

**为什么要这么写**：

- 在**基础知识（foundational / sensed）**与**推导知识（derived）**之间划线；
- 减少**信念冗余（redundancy）**：不用每动一块积木就手动维护 `free`；
- 让规则上下文更可读：把长条件抽象成一个概念「X 是空闲的」。

⚠️ **一处不一致**：加了 `inference free(...)` 后，第 5 页计划体里的 `-free(X);` 就**语义可疑**了——`free` 已是**派生信念（derived belief）**，不该像基础信念那样被显式撤回。课件未删这一句，属原版 AgentSpeak 例子的遗留；更干净的写法是删掉 `-free(X)`。

---

## 七、把 6 页拼起来（含第 6 页的正确逻辑）

```astra
agent Tower {
    types tower_ont {
        formula block(string);
        formula table(string);
        formula on(string, string);
        formula free(string);
        formula holding(string);
    }

    // ---- 推理规则：free 是推导出来的 ----
    inference free(string X) :- ~on(string Y, X) & ~holding(X);

    // ---- 初始信念与初始目标 ----
    initial block("a"), block("b"), block("c"), block("d");
    initial table("table");
    initial on("a", "table"), on("b", "a"), on("c", "b"), on("d", "c");
    initial !tower("a", "b", "c");

    // ---- 计划规则 ----
    rule +!holding(string X) : ~holding(string Y) & free(X) & on(X, string Z) {
        +holding(X);  -on(X, Z);      // 注意：不再 -free(X)
    }
    // （!tower(...) 的规则课件未给出）
}
```

---

## 八、这 6 页引出的 5 个基本概念（整套课件都在复用）

| 概念 | 在 LightSwitch 案例里的对应 |
|---|---|
| `agent <Name>` + 文件名一致 | `agent LightSwitch` / `LightSwitch.astra` |
| `types` + `formula`（ontology） | `types ls { formula switch(string); ... }` |
| `initial` 信念 / `initial !目标` | `initial switch("off");` / `initial !switch("on");` |
| `rule +!goal : context { body }` | `rule +!switch("on") : switch("off") { ... }` |
| `inference head :- body;` | （LightSwitch 没有，可用来推 `free`、`winner` 之类的派生概念） |

---

## 附录 A：单条规则语法逐 token

以 LightSwitch 的这条规则为例：

```astra
rule +!switch("on") : switch("off") { -switch("off"); +switch("on"); }
```

**骨架**

```
rule  <触发事件>  [ : <上下文> ]  { <计划体> }
```

| 片段 | 内容 | 术语 |
|---|---|---|
| `rule` | `rule` | 关键字，声明一条计划规则 |
| 触发事件 | `+!switch("on")` | triggering event |
| 分隔符 + 上下文 | `: switch("off")` | context / guard，**可整段省略**（省略 = 恒真） |
| 计划体 | `{ -switch("off"); +switch("on"); }` | plan body，代码块，语句以 `;` 结尾 |

> `rule` + `:` + `{}` 就是 ASTRA 对 AgentSpeak(L) 的 `+te : ctxt <- body.` 的 Java 化改写；**整条 rule 末尾不加 `;`**（`}` 即结束符）。

**触发事件的四个组成部分**

| 记号 | 名称 | 含义 |
|---|---|---|
| `+` | addition operator | 事件类型是「加入」；对应 `-` 是撤回 |
| `!` | bang operator | 标记这是 **goal（目标）**；`!` = 目标采纳 |
| `switch` | predicate / functor | 谓词名，需在 `types` 中声明 |
| `("on")` | argument list | 字符串字面量；也可写 `(string X)` 做变量绑定 |

**四种触发子对照**

| 触发子 | 事件名 | 何时触发 |
|---|---|---|
| `+!switch("on")` | goal adoption event | 有人 `!switch("on")` |
| `-!switch("on")` | goal deletion event | 有人撤销该目标（很少用） |
| `+switch("on")` | belief addition event | 信念库**加入** `switch("on")` |
| `-switch("off")` | belief retraction event | 信念库**删除** `switch("off")` |

**上下文的可用语法**

```astra
rule +!switch("on") : switch("off") { ... }                    // 单字面量
rule +!tick() : time(int T) { C.println("time=" + T); }        // 带变量绑定，T 可在计划体用
rule +!go()   : ~holding(string X) & free(X) { ... }           // ~ 取反、& 与、| 或、() 分组
rule +!n(int X): X > 0 & X ~= 5 { ... }                        // 比较：> < == ~=(不等于)
```

> context **不是 if 语句**：真 → 本规则可用（applicable）；假 → 被筛掉，事件去匹配别的规则。**若所有规则都被筛掉**，事件被丢弃；若该事件是子目标，则该子目标**失败**，调用它的计划随之失败。

**三个最易混的点**

| 易混点 | 规则 |
|---|---|
| `+` / `-` 的双重身份 | 写在 `rule` 后 = **事件触发子**；写在 `{}` 内 = **语句** |
| `!` 的有无 | 有 `!` = 目标（不占信念库，靠规则实现）；无 `!` = 信念（存在信念库里） |
| `;` 与 `}` | 计划**体内**语句必须 `;` 结尾；**整条 rule 末尾不加 `;`** |

---

## 附录 B：`formula on(string,string)` 是什么意思

**是信念模板声明（belief template / predicate declaration），不是"创建了一条 `on` 信念"。**

```astra
formula   on   (string, string)  ;
   ↑      ↑         ↑           ↑
 关键字  谓词名   参数类型列表   结束符
```

| 部分 | 名称 | 含义 |
|---|---|---|
| `formula` | 关键字 | 「这是一条**谓词公式模板**」——有真假可言的谓词（区别于 `funct`：有返回值但无真假） |
| `on` | predicate / functor | 关系名，约定**小写开头** |
| `(string, string)` | arity 2 的**类型签名** | 第 1、2 个参数都必须是 `string` |

**合法 / 非法实例**

```astra
on("a", "table")   // ✓
on("b", "a")       // ✓
on("a", 5)         // ✗ 5 是 int，不是 string
on("a")            // ✗ 元数不对（on/1 未声明）
onTop("a","b")     // ✗ 谓词根本没声明过
```

**它「不」干什么**

| 语句 | 干什么 | 类比 |
|---|---|---|
| `formula on(string,string);` | 声明**签名** | 数据库 **DDL**：`CREATE TABLE on(a TEXT, b TEXT)` |
| `initial on("a","table");` | 插入一条具体信念 | 数据库 **INSERT** |
| `+on(...)` / `-on(...)` | 运行时加入 / 撤回信念 | INSERT / DELETE |
| `rule +on(...) {...}` | 响应「加入该信念」这个**事件** | 触发器 trigger |

**声明（declaration）与定义（definition）是分开的**——`free` 就是最好的例子：

| 代码 | 角色 | 回答的问题 |
|---|---|---|
| `formula free(string);` | **声明** | 「这个谓词存在吗？参数什么类型？」 |
| `inference free(string X) :- ...;` | **定义** | 「它的真假怎么算出来？」 |
| `initial free("d");` / `+free("d");` | **实例化** | 「此刻我相信它吗？」 |

**三个附加要点**

1. **类型可内联写在使用处**：`on(X, string Z)`、`~holding(string Y)`；声明的类型只约束最终值。
2. **元数是谓词身份的一部分**：`on/2` 与 `on/1` 是两个不同谓词。
3. **`types` 块可有多个、标签名随意取**（`tower_ont`、`ls`、`example`）；类型检查在**编译期**完成，运行时不再检查。

可选类型：`int / long / float / double / char / boolean / string / list / funct` + **任意 Java 类**。

---

## 考点清单

- [ ] `.astra` 文件命名规则、一个文件一个 agent 类。
- [ ] `types` / ontology 的作用是**编译期类型检查的谓词词汇表**。
- [ ] `initial` 有 `!` = 初始**目标**，无 `!` = 初始**信念**。
- [ ] 计划规则三要素：**触发事件 + 上下文 + 计划体**；`:` 与 `{}` 对应 AgentSpeak 的 `<-`。
- [ ] `~` = 否定，`&` = 合取，`|` = 析取；`~=` = 不等于。
- [ ] **变量可在触发事件、上下文、`query` 三处绑定**；上下文中不用的变量（如 `Y`）起「存在性判断」作用。
- [ ] `inference head :- body;` 的读写；**derived belief 不应被 `-` 撤回**。
- [ ] 类型可**内联写在使用处**（`on(X, string Z)`）。
- [ ] 一个目标 = 一条可写成信念的公式 ⇒ **declarative goal**。
- [ ] `+`/`-`/`!` 在"触发子位置"与"语句位置"的双重身份。

> 注：课件里的 `“off”` 是弯引号，真实代码必须用直引号 `"off"`。
