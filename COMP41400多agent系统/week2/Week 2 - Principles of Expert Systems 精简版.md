---
course: COMP41400 Multi-Agent Systems
week: 2
topic: Principles of Expert Systems（精简版）
tags:
  - COMP41400
  - Multi-Agent-Systems
  - Knowledge-Representation
  - Expert-Systems
---

# Week 2 - 专家系统原理（精简版）

> 书目：Lucas & van der Gaag, *Principles of Expert Systems* (1991)。
> 特点：理论落代码，全书所有例子几乎都来自同一个领域——**心血管医学**。

---

## 一、全书主线

**先讲"知识怎么表示"（第2–4章），再讲"不确定怎么办"（第5章），最后讲"可解释 + 工业工具"（第6–7章）。**

| 章 | 主题 | 一句话 |
|---|---|---|
| 1 | 引言 | `专家系统 = 知识库 + 推理机`（知识/推理分离） |
| 2 | 逻辑与消解 | Horn 子句 + resolution + unification + SLD，是其余一切的地基 |
| 3 | 产生式规则 | 规则 + 事实 + 识别-动作循环，擅长启发式分类 |
| 4 | 框架与继承 | 分类体系 + 继承，擅长层次描述，难点在异常/多继承 |
| 5 | 不确定推理 | 五种模型 = 数学严谨性 ↔ 工程可行性的权衡 |
| 6 | 检视工具 | how / why / why-not，可解释 = 记录推理搜索空间 |
| 7 | 工业工具 | OPS5 用 rete 提效，CENTAUR 用 prototype 提结构 |

---

## 二、第1章 核心概念

- **专家系统**：特定领域给出专家级建议的系统；学科叫**知识工程**。
- **知识型系统**：用符号化人类知识、以类人方式推理的系统。
- **架构范式（核心）**：`expert system = knowledge base + inference engine` → 知识/推理分离，便于维护。
- **shell**：剥掉领域知识的空架子（如 EMYCIN = MYCIN − 医学知识）。
- **知识获取**：知识工程师通过**知识抽取**（访谈专家）获取。
- **启发式 heuristics**：专家经验法则，**领域相关**——专家系统成功的来源。

**形式化方法四要求**：
1. 表达力 expressive power
2. 语义基础 semantic basis
3. 高效算法
4. **可解释 explanation**（区别于普通软件的关键）

> 核心矛盾：**表达力 ↔ 可解释效率互相冲突**。

**两种推理方向（高频）**：

| 维度 | top-down（backward chaining） | bottom-up（forward chaining） |
|---|---|---|
| 起点 | goal → subgoal | facts → 推新事实 |
| 别名 | 目标驱动 | 数据驱动 |
| 适用 | 目标明确、空间窄 | 数据多、目标不唯一 |

**深层 vs 浅层知识**：深层=结构与功能（论证强、难获取），浅层=症状—疾病经验关联（快、易错）。

### 经典系统速查（案例源头）

| 系统 | 领域 | 技术/要点 |
|---|---|---|
| MYCIN | 传染病 | 产生式规则 + CF，最著名医学专家系统 |
| INTERNIST-1/CADUCEUS | 内科 | 多病共存，反证"互斥穷尽"假设过强 |
| HEURISTIC DENDRAL | 化学 | generate-and-test（生成-测试） |
| XCON(R1) | VAX 配置 | 1981 投产，第一个商业成功的专家系统 |
| PROSPECTOR | 地质 | 主观贝叶斯 |

---

## 三、第2章 逻辑与消解

### 命题逻辑与 FOL
- 5 联结词：`¬ ∧ ∨ → ↔`，优先级 `¬ > ∧ > ∨ > → > ↔`。
- 公式三分类：**valid（永真）/ unsatisfiable（矛盾）/ satisfiable（可满足）**。
- **逻辑后承** `⊨`：前提为真的解释都令结论为真。
- FOL 加：谓词、函数、量词 ∀ ∃；structure + assignment → interpretation。
- 量词对偶：`¬∃x ≡ ∀x¬`，`¬∀x ≡ ∃x¬`；**注意 `∀x(P∨Q) ≢ ∀xP ∨ ∀xQ`**。

### 子句形式（重点）
- **literal**（文字）×正负；**clause**（子句）= 全称量词隐式省略的文字析取；**空子句 □ 恒假**。
- **Horn 子句**（至多一个正文字）三种形式：
  - `A ←` 事实（unit clause）
  - `← B₁…Bₙ` 目标（goal clause）
  - `A ← B₁…Bₙ` 规则
  - **PROLOG 就建立在 Horn 子句之上**。

**子句化 8 步**（常考）：
1. 消 `→`（`¬F∨G`）→ 2. 否定内移 → 3. 重命名变量 → 4. **消存在量词引入 Skolem 函数**（保可满足、不保等价）→ 5. 前束化 → 6. CNF → 7. 去量词 → 8. 拆子句集。

### 推理规则与元性质
- **Modus ponens**：`A, A→B ⊢ B`（纯语法操作）。
- **soundness 可靠性**：`⊢ ⇒ ⊨`；**completeness 完备性**：`⊨ ⇒ ⊢`。
- **FOL 不可判定**（Church/Turing 1936）；**命题逻辑可判定**。

### 消解与合一
- **反证法**：证 `S ⊢ G` ⇔ 证 `S ∪ {¬G}` 不可满足（推出 □）。
- **resolution**：消去互补文字 `L` 与 `¬L`，剩余取析取得 resolvent。
- **substitution / instance / ground instance**。
- **mgU（最一般合一元）**：最省约束的 unifier；**occur check** 防 `x` 绑成含 `x` 的项导致死循环。

### 消解策略（重点对比）

| 策略 | 特点 | 性质 |
|---|---|---|
| Semantic resolution | 用解释 I 分真假两集 | 缩小搜索面 |
| Set-of-support | 每步至少一父句来自待证集 T | 可靠完备，类 top-down |
| Linear / Input | Input 必与原始子句消解 | 一般不完备；Horn 下完备 |
| **SLD** | Horn + input + selection rule | **Horn 下可靠完备 = PROLOG 理论基础** |

- **SLD 树**分支：success / failure / **infinite**。
- PROLOG 工程三妥协：① 最左文字 + 书写顺序 + **深度优先（不完整）**；② 省略 occur check；③ **negation as failure + 封闭世界假设**（证不出 P 就当 ¬P）。

---

## 四、第3章 产生式规则

### 表示
- 变量分 **single-valued / multi-valued**；成立"变量-值"对叫 **fact**，集合叫 **working memory（工作内存）**。
- `unknown` = 元常量（"没推出"，≠ "值为假"）。
- 规则：`if <antecedent> then <consequent>`；动作为 `add` / `remove`。
- **关键：`add` 单调（近逻辑蕴含），`remove` 破坏单调 → 超出标准逻辑**。
- **o-a-v 三元组**（对象-属性-值），配 object schema。

### 推理
**recognise-act cycle（核心循环）**：
```
规则库+事实 → match → select（conflict set）→ apply → 循环
```

**冲突消解 4 策略**：

| 策略 | 规则 |
|---|---|
| forward chaining | 来一条用一条 |
| prioritization | 按优先级 |
| **specificity** | **条件更多者优先** |
| **recency** | 按 **time tag**，最近加入者优先 |

- **rule instance** `(R,M)`：规则+匹配集，防止重复应用同一事实。

### 模式识别
- pattern 变量：`?` 单值、`!` 多值、纯 `?`/`!` 为无关变量。
- **matching ≠ unification**：匹配不对称（事实不反向替换模式变量），换效率。

### 局限
① 描述性知识难表达；② 问题知识与元知识混用同形式；③ 操作性强；④ 难模块化。

---

## 五、第4章 框架与继承

### 语义网
- 顶点=概念，弧=二元关系（`part-of`、`is-a`）。
- **高频考点——两种链接**：
  - **subset-of**：类→类（`large-artery → artery`）
  - **member-of**：个体→类（`aorta → large-artery`）
  - 均构成**偏序**，支撑属性继承。

### 框架
- **class frame / instance frame**；链接 `instance-of` / `superclass`。
- **facet（槽的元信息）——最实用机制**：

| facet | 含义 |
|---|---|
| `value` | 确定值，不可覆盖 |
| `default` | 缺省值，可覆盖 |
| `demon` | if-needed / if-added / if-removed（**过程附着**，frame↔rule 的桥） |

### 四种继承算法

| 算法 | 结构 | 优先级 | 要点 |
|---|---|---|---|
| Single | 树 | 先见者优先 | 下层覆盖上层 |
| N-inheritance | 树+facet | value > demon > default | 确切值最可靠 |
| **Z-inheritance** | 树+facet | 本框架 value→demon→default 再上行 | **来源越具体越可靠（本书主推）** |
| Multiple | DAG | 继承链 + preclusion | 结论集必须一致，否则 taxonomy 不一致 |

### 多继承
- 多个 superclass → 图（DAG）。
- **subtyping**：`y₁ ≤ y₂` ⇔ `D(y₂)⊆D(y₁)` 且类型函数满足。
- **inheritance chain → conclusion set**；不一致则引入 **intermediary + preclusion**。
- type lattice 有 **meet(∧)/join(∨)**：例 `vein ∨ artery = AV-anastomosis`。

> **贯穿案例异常**：`left-pulmonary-artery` 属 artery 却运缺氧血——同时用来讲继承失效、异常、preclusion。

---

## 六、第5章 不确定推理

规则 `if e then h [x]`；需 4 个组合函数：**fprop / fand / for / fco**。

### 概率论
- `P(h|e) = P(e|h)·P(h)/P(e)`（Bayes）。
- **瓶颈**：需指数级概率 → 引出下列 4 种准概率模型。

### 5.3 主观贝叶斯（PROSPECTOR）
- **odds**：`O(h) = P(h)/(1−P(h))`
- **似然比**：`λ = P(e|h)/P(e|¬h)`（λ>1 支持 h）
- **核心公式**：`O(h|e) = λ · O(h)`
- 多规则共证：`O(h|∩eᵢ) = Πλᵢ · O(h)` → **顺序无关**。

### 5.4 确定性因子 CF（MYCIN，考试重灾区）
- `MB(h,e) = max{0, (P(h|e)−P(h))/(1−P(h))}`
- `MD(h,e) = max{0, (P(h)−P(h|e))/P(h)}`
- 复合：and 取 min、or 取 max（MD 相反）。
- **CF = (MB−MD) / (1−min{MB,MD})**，∈ [−1,1]。
- **共证合并三情形（必背）**：
  - 皆正：`CF₁ + CF₂(1−CF₁)`
  - 异号：`(CF₁+CF₂)/(1−min{|CF₁|,|CF₂|})`
  - 皆负：`CF₁ + CF₂ + CF₁CF₂`
- CF **不是真实概率**，是实用近似。

### 5.6 Dempster-Shafer
- 辨识框架 Θ；`m`（基本概率分配）、`Bel(x)=Σ_{y⊆x}m(y)`、`Pl(x)=Σ_{x∩y≠∅}m(y)`。
- **区间 `[Bel, Pl]` 表"无知"** 是精髓。
- Dempster 组合：`m₁⊕m₂(x) = Σ_{y∩z=x} m₁(y)m₂(z) / Σ_{y∩z≠∅} m₁(y)m₂(z)`。

### 5.7 信念网络
- DAG（定性）+ CPT（定量）。
- **威力 = 条件独立**：`P(V₁∧…∧Vₙ) = Πᵢ P(Vᵢ|parents(Vᵢ))` —— 局部概率定义全局联合分布。
- 证据传播：causal（前驱）/ diagnostic（后继）。
- **Kim & Pearl**：限 causal polytree；π（自上而下）/ λ（自下而上）消息，本地重算。
- **Lauritzen & Spiegelhalter**：转可分解图（＞4 环路加弦）→ clique 树局部传播。

### 五种方法对比

| 方法 | 优点 | 缺点 | 代表 |
|---|---|---|---|
| 纯概率 | 严谨 | 指数级概率难获取 | — |
| 主观贝叶斯 | 顺序更新直观 | 仍需先验 | PROSPECTOR |
| CF | 计算极简 | 无严格概率基础 | MYCIN |
| D-S | 表达无知 | 冲突归一化 | — |
| 信念网络 | 条件独立降维 | 建 CPT 成本高 | Kim&Pearl / L&S |

---

## 七、第6章 解释设施（必考）

| 设施 | 问什么 | 实现 |
|---|---|---|
| **how** | 值怎么得出的？ | 查事实集→显示规则+前提事实 |
| **why** | 为什么问我？ | 沿推理链**反向回溯**到 goal |
| **why-not** | 为什么没推出？ | 找含该结论但失败/未触发的规则，指首个失败条件 |

- 依赖：规则库 + 事实集 + **推理搜索空间**。
- PROLOG 版：fact/rule 第四参记来源（`from_user`/规则名）；LISP 版：`*history*` 栈。
- **rule model（TEIRESIAS）**：元知识，对同结论规则做高层归纳，辅助知识获取。

---

## 八、第7章 工业工具

### OPS5
- 声明 `literalize`；规则 `(p <name> <lhs> --> <rhs>)`；动作为 make/remove/modify。
- 冲突消解：**LEX**（recency，按 time tag 字典序）/ **MEA**（目标驱动）。
- **rete 算法（重点）**：规则编译成网络（椭圆=class 测试、矩形=alpha 节点、圆=beta 两输入节点）；相同条件共享；token 增量传播。
- **为什么快**：编译一次 + 共享测试 + 每轮只重算受影响 token（朴素匹配可占 90% 时间）；不适合实时刷新场景。

### CENTAUR
- **出发点是产生式三个局限**：① 问题结构隐含不显式；② 控制策略与启发式知识混同形式；③ 疾病表现分散。
- **prototype = frames + rules**：component slot（PEV/PV/inference rules/importance 0–5）+ meta 槽（moregeneral/morespecific）+ control 槽（to-fill-in/if-confirmed 等）。
- 推理：**hypothesize-and-match + agenda（LIFO 深度优先）**；match measure 与阈值比较 → confirm/disconfirm。
- 成就：提问随病例动态变化、比 PUFF 更聚焦。

---

## 九、附录 PROLOG vs LISP

| 维度 | PROLOG | LISP |
|---|---|---|
| 范式 | 逻辑（Horn 子句） | 函数式+命令式 |
| 核心 | unification + backtracking | S-expression 求值 |
| 作用域 | 子句内 | lexical（默认）/ dynamic special |
| 知识表示 | 子句/事实/规则 | 结构/框架/列表/宏 |
| 易错点 | cut/fail、子句顺序影响结果、negation as failure | **lexical vs dynamic scope** |

- PROLOG 关键：`cut`（禁止回溯）、`not`（证不出即假）、`asserta/retract`、`algorithm = logic + control` 但实践偏过程式。
- LISP 关键：quote、lambda、eval/apply、宏在求值前展开、defstruct。

---

## 十、易考点清单（压缩版）

- ⭐ `expert system = knowledge base + inference engine`
- top-down vs bottom-up 适用场景
- 子句化 8 步；为何引入 Skolem 函数（保可满足不保等价）
- Horn 子句三形式；为何 PROLOG 选它
- mgU + occur check；matching ≠ unification
- SLD 可靠完备；深度优先不完整；negation as failure + CWA
- soundness vs completeness；FOL 不可判定
- recognise-act cycle；冲突消解（specificity / recency）
- **add 单调 vs remove 破坏单调**
- subset-of vs member-of；Z-inheritance facet 优先级
- 多继承 preclusion；结论集必须一致
- Bayes 公式；**`O(h|e) = λ·O(h)`** + 顺序无关
- **MB/MD/CF 公式 + CF 共证三情形**
- D-S 的 m/Bel/Pl/belief interval
- belief network 局部分解；Kim-Pearl（π/λ）vs L&S（团树）
- **how/why/why-not** 方向
- **rete 为何快**；LEX vs MEA
- PROLOG cut/fail；LISP lexical vs dynamic scope

---

## 十一、一句话总结

1. **专家系统本质 = 知识库 + 推理机分离**。
2. **逻辑（Horn + SLD）是地基**，PROLOG 是其工程化身。
3. **产生式**靠规则+事实+识别-动作循环，擅长分类、弱于结构化。
4. **框架**靠分类+继承，难点在异常与多继承一致性。
5. **不确定推理**沿"严谨↔可行"演进：概率→贝叶斯/CF→D-S→信念网络。
6. **可解释性 = 记录推理搜索空间**。
7. **OPS5 用 rete 提效，CENTAUR 用 prototype 提结构**。