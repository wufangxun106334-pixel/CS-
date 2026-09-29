---
course: COMP41400 Multi-Agent Systems
week: 3
topic: ASTRA Static Types
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - StaticTypes
  - TicTacToe
---

# Week 3 - ASTRA: Static Types（逐页讲解）

> 来源：ASTRA-StaticTypes.pptx（7 页）
> 关联笔记：[[Week3-ASTRA-BasicSyntax-逐页讲解]]、[[Week3-ASTRA-Modules-逐页讲解]]、[[Week3-IntroductionToASTRA-知识点]]、[[Week3-ASTRA-扩展计划语法-知识点]]

板书式主线：

> **ASTRA 是静态类型语言 → 所以必须先声明本体 → 用 Tic-Tac-Toe 演示完整程序 → 但动作（`move`）得由 Java module 提供 → 引出 Modules**

## 一、7 页地图

| 页 | 内容 |
|---|---|
| 2 | 静态类型清单 + **为什么需要类型** |
| 3 | Tic-Tac-Toe 的 ontology（8 个谓词） |
| 4 | 同一程序的 **AgentSpeak(L) 原版**语法 |
| 5 | 翻译成 ASTRA：`inference` + `initial`（类型全部显式化） |
| 6 | 翻译成 ASTRA：`rule` 计划规则 + 一处「现在还做不到」 |
| 7 | 列表类型 + 模式匹配（Tower 例子） |

---

## 二、第 2 页：静态类型（Statically Typed）

**ASTRA 是静态类型语言，类型系统基于 Java**：

| ASTRA 类型 | 说明 | Java 映射 |
|---|---|---|
| `short` `int` `long` | 整数（课件列了 `short`；官方 guide 的类型表只列 `int/long`） | `short/int/long` |
| `float` `double` | 实数 | `float/double` |
| `boolean` `char` | 布尔、字符 | `boolean/char` |
| `string` | 字符串 | `java.lang.String` |
| `list` | 逻辑列表（可伸缩、可同质可异质） | `astra.term.ListTerm`（间接实现 `java.util.List`） |
| `funct` | functional term（函数项） | `astra.term.Funct` |
| **任意 Java 类** | object reference（对象引用） | 那个 Java 类本身 |

**为什么这么设计（本节最重要的一段话）**

1. ASTRA 的**所有 primitive action（基本动作）都由 Java 方法实现**（除语言内置的少数几个）；
2. 所以要「**清楚地指定该调用哪一个方法**」——即 **overload resolution（重载决议）**；
3. **静态类型**让编译器在编译期完成这个选择，也让 ASTRA 与 Java 的**集成（integration）**没有歧义。

> **类型系统不是为了优雅，是为了「让 agent 的动作能准确地落到某个 Java 方法上」。** 这也预告了第 6 页那句 `// We cannot do this yet!`。

---

## 三、第 3 页：Tic-Tac-Toe 的 ontology

棋盘编号（1–9）：

```
 1 | 2 | 3        line 组合共 8 条：
---+---+---       横：1-2-3, 4-5-6, 7-8-9
 4 | 5 | 6       竖：1-4-7, 2-5-8, 3-6-9
---+---+---       斜：1-5-9, 3-5-7
 7 | 8 | 9
```

```astra
agent TicTacToe {
    types tictactoe_ont {
        formula token(string);          // X / O 两个玩家筹码
        formula played(int, string);    // 位置 X 上已被筹码 Y 占据
        formula free(int);              // 位置 X 空着
        formula winner(string);         // 筹码 X 的玩家赢了
        formula loser(string);          // 筹码 X 的玩家输了
        formula drawn(string);          // 筹码 X 的玩家平局
        formula line(int, int, int);    // 三条成线的位置（静态事实，共 8 条）
        formula move(int);              // 本回合被选中的落子位置
    }
}
```

**设计要点**：`line/3` 是**领域的静态背景知识**（8 条永不改变的事实），`winner/loser/drawn` 是**推导知识**（靠 `inference` 算出），`move/1` 是**临时信念**（每回合选出、用完即删）。

---

## 四、第 4 页：AgentSpeak(L) 原版语法（用来对照）

```prolog
free(L) :- ~played(T, L)
winner(T) :- line(L1, L2, L3) & played(T, L1) & played(T, L2) & played(T, L3).
loser(T)  :- winner(T2) & token(T) & T != T2.
drawn(T)  :- token(T) & ~free(L) & ~winner(T2)

line(1,2,3)  line(1,5,9)  ...  line(7,8,9)     /* 8 条事实 */

+turn(T) : player(T) <- !move(); ?move(L); -move(L); move(T, L).

+!move() : free(1) <- +move(1).
...                                            /* 9 条，每个格子一条 */
```

**ASTRA 与它的语法差异表**（这页的价值就在这里）

| AgentSpeak(L) | ASTRA | 说明 |
|---|---|---|
| `head :- body` | `inference head :- body;` | 需要显式 `inference` 关键字 |
| 事实 `line(1,2,3)`（裸写） | `initial line(1,2,3);` | ASTRA 需要显式 `initial` |
| `+te : ctxt <- body.` | `rule +te : ctxt { body; }` | `<-` → `:` + `{}` |
| `?move(L)` | `query(move(L));` | `?` → `query` 关键字 |
| `T != T2` | `T ~= T2` | 不等号不同 |
| 裸原子 `table` | `"table"` | **ASTRA 强类型 → 没有无类型原子，用字符串或 Java 对象顶上** |

---

## 五、第 5 页：ASTRA 版（`inference` + `initial`）

```astra
inference free(int L)      :- ~played(string T, L);
inference winner(string T) :- line(int L1, int L2, int L3) &
                              played(T, L1) & played(T, L2) & played(T, L3);
inference loser(string T)  :- winner(string T2) & token(T) & T ~= T2;
inference drawn(string T)  :- token(T) & ~free(int L) & ~winner(string T2);

initial line(1,2,3), line(1,5,9), line(1,4,7);
initial line(2,5,8), line(3,6,9), line(3,5,7);
initial line(4,5,6), line(7,8,9);
```

**与第 4 页逐句对照，唯一的实质变化就是：每个变量在使用处都补上了类型标注。**

- `~played(string T, L)`：`T` 在这里**新引入** → 必须就地声明类型；`L` 是 head 里已绑定的 → 不用重复标注；
- `line(int L1, int L2, int L3)`：三个变量都首次出现，全部标注；
- `T ~= T2`：比较运算符，两边类型都已知。

**深层含义**：AgentSpeak(L) 的「无类型逻辑」在 ASTRA 里变成「**每个变量、每个绑定都有确定类型**」。

### ⚠️ 一处需要标注的坑

```astra
inference drawn(string T) :- token(T) & ~free(int L) & ~winner(string T2);
```

这里的 `~free(int L)` 用的是**未绑定变量（unbound variable）**，直觉读法是「**不存在任何一个空格**」（negation as failure + 存在量化）。但官方 ASTRA guide 明确警告过：

> "you cannot use an unbound variable here - you must ask is location X free (e.g. `free(5)` would work, but `free(int x)` would not)."

也就是说，**推导规则里对未绑定变量取否定，行为容易与直觉不符**。更安全的做法是显式枚举，或引入 `full()` 之类的概念。

---

## 六、第 6 页：计划规则 + 那句 `// We cannot do this yet!`

```astra
rule +turn(string T) : player(T) {
    !move();  query(move(L));  -move(L);  move(T, L);   // <-- We cannot do this yet!
}

// 9 条走子策略：用「规则顺序」表达优先级
rule +!move() : free(1) { +move(1); }
rule +!move() : free(2) { +move(2); }
... 一直到 free(9)
```

**四步的语义（值得背）**

| 步骤 | 作用 |
|---|---|
| `!move();` | 采纳子目标，让 9 条策略规则中**第一条 context 成立**的来选一个格子 |
| `query(move(L));` | 从信念库**取出**刚选中的位置，绑定到 `L`（失败则整条计划失败） |
| `-move(L);` | **删掉**这个临时信念，好让下一回合重新选择 |
| `move(T, L);` | **真正落子**——这是一个**动作（action）** |

**为什么 `move(T, L)` 是「现在做不到」的？**

1. 它在 ontology 里**没有被声明**（本体只声明 `move(int)` 一个参数，这里是两个参数 → 不是信念）；
2. 所以它是 **primitive action**，按第 2 页的说法——**必须由某个 Java module 的方法实现**；
3. 而 module 还没讲 → 课件写下 `// We cannot do this yet!`。

> **这就是整份 PPT 的教学闭环**：静态类型不是为了好看，而是因为**动作 = Java 方法**，类型是 ASTRA 与 Java 之间的「接头规格」。下一份课件（Modules）补上 `move` 的实现。

**两个观察**

- 那 9 条 `rule +!move() : free(N) { +move(N); }` 是 **selection by rule（用规则表达选择）**：靠**规则书写顺序**当优先级，天然构成 if-else 链；因为 `free` 是推理出来的，每回合都会**重新求值**。
- `!move()` 与 `move(T,L)` 只差一个 `!` 和一个参数——**目标 vs 动作**，形态极像但语义完全不同，常考。

---

## 七、第 7 页：列表类型（List）+ 模式匹配

```astra
agent Tower {
    initial !tower(["c", "b", "a"]);

    rule +!tower(list L) { !build(L, "table"); }
    rule +!build([], string B) { }                                  // 基例：空表
    rule +!build([string H | list T], string B) {                   // 递归步：拆头尾
        !on(H, B);
        !build(T, H);
    }
}
```

对照 AgentSpeak(L)：

```prolog
!tower([c, b, a]).
+!tower(L)   <- !build(L, table).
+!build([], B) <- .
+!build([H|T], B) <- !on(H,B); !build(T, H).
```

**四个要点**

1. **列表字面量** `["c","b","a"]`，类型是 `list`；可作为目标/动作的参数整体传递；
2. **模式匹配（pattern matching）在触发事件里完成**：`[]` 匹配空表，`[H | T]` 匹配非空表（`|` 是 head/tail 分隔符）。**匹配失败 = 该规则不适用**，所以两条规则天然构成「if 空表 else 非空」；
3. **类型要写进模式里**：`[string H | list T]`；
4. **递归实现循环**：`!build(T, H)` 是子目标递归，即"递归 rule 循环"；`!on(H,B)` 则是动作（同样待 module 实现）。

---

## 八、类型速查表

| 类型 | 字面量示例 | Java 映射 |
|---|---|---|
| `int` | `5, -11` | `int` |
| `long` | `55l` | `long` |
| `float` | `12.3f` | `float` |
| `double` | `9.876` | `double` |
| `char` | `'a'` | `char` |
| `boolean` | `true` | `boolean` |
| `string` | `"animal"` | `java.lang.String` |
| `list` | `[1,2,3]`、`["the", 4, 'a', true]` | `astra.term.ListTerm` |
| `funct` | `fatherOf("rem")` | `astra.term.Funct` |
| **Java 类** | 对象引用（打印成 `1244defw@Ball` 之类） | 那个类 |

## 九、易考点

- [ ] ASTRA 是**静态类型**语言，类型系统基于 Java；**所有 primitive action 都是 Java 方法**。
- [ ] 静态类型的核心理由：**准确地决定调用哪个（重载的）Java 方法**。
- [ ] AgentSpeak(L) → ASTRA 的语法对照（`:-`→`inference`、`<-`→`rule : {}`、`?`→`query`、`!=`→`~=`、裸原子→`"string"`）。
- [ ] 类型标注**可就地写在使用处**，且**只需在变量首次出现处**声明。
- [ ] `!move()`（目标）vs `move(T,L)`（**动作**，须由 module 提供）。
- [ ] `// We cannot do this yet!` 的原因：动作需要 Java module 实现。
- [ ] 列表模式 `[]` 与 `[H | T]`；`[string H | list T]`；递归子目标实现循环。
- [ ] 推导规则里**未绑定变量的否定**（`~free(L)`）是已知陷阱。
- [ ] `line/3` 静态背景知识、`winner` 等推导知识、`move/1` 临时信念——三种知识的分工。
