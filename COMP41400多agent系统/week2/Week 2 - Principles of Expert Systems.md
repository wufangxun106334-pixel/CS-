---
course: COMP41400 Multi-Agent Systems
week: 2
topic: Principles of Expert Systems（知识点与案例）
tags:
  - COMP41400
  - Multi-Agent-Systems
  - Knowledge-Representation
  - Expert-Systems
  - Logic
  - Uncertainty
---

# Week 2 - Principles of Expert Systems（知识点与案例整理）

> **书目**：Peter J.F. Lucas & Linda C. van der Gaag, *Principles of Expert Systems*, Addison-Wesley, 1991（426 页，7 章正文 + 2 个语言附录）。
> **定位**：undergraduate（本科）层次的 expert systems（专家系统）教材。作者在序言里写明，写这本书就是为了填一个 gap（空缺）—— 当时 **没有任何一本书同时讲清楚 formalisms（形式化方法）和 implementation（实现）**。全书刻意把理论落到代码上。
> **关键特点**：**几乎所有例子都来自同一个问题域** —— cardiovascular medicine（心血管医学）的很小一部分。

---

## 0. 全书地图

| 章 | 主题 | 核心形式化手段 | 一句话核心 |
|---|---|---|---|
| 1 | Introduction | 知识 / 推理 分离 | `expert system = knowledge + inference` |
| 2 | Logic and Resolution（逻辑与消解） | Horn clauses | resolution、unification、SLD |
| 3 | Production Rules and Inference（产生式规则与推理） | if–then 规则 + working memory | top-down vs bottom-up、pattern matching |
| 4 | Frames and Inheritance（框架与继承） | semantic nets / frames | single / multiple inheritance |
| 5 | Reasoning with Uncertainty（不确定推理） | 准概率模型 | 主观贝叶斯、CF 模型、Dempster-Shafer、belief network |
| 6 | Tools for Knowledge and Inference Inspection（检视工具） | 解释设施 | how / why / why-not、rule models |
| 7 | OPS5, LOOPS and CENTAUR | 工业级开发工具 | **rete 算法**、prototype |
| A / B | PROLOG / LISP 附录 | 实现语言 | unification + backtracking / S-expression |

**全书主线**：先讲"知识怎么表示"（2–4 章三种 formalism），再讲"知识不确定怎么办"（5 章），最后讲"怎么让系统可解释、怎么做成工业工具"（6–7 章）。第 2 章的 logic 是理解其余一切的 base（基础）—— 因为 production rules、semantic nets、frames 都能翻译成逻辑。

---

## 第 1 章 Introduction

### 1.1 知识点

- **专家系统 expert system**：能在特定领域给出解答或建议，水平可比肩人类专家的系统。构建它的专门学科叫 **知识工程 knowledge engineering**。
- **知识型系统 knowledge-based system**：更宽的概念 —— 用人类知识的 symbolic representation（符号化表示）、以类似人类推理的方式运行的系统。
- AI 于 1956 年 Dartmouth 会议定名。早期两大方向：
  - **定理证明 theorem proving** —— 用逻辑 + 推理规则从公理推出定理；
  - **问题求解 problem solving** —— 代表是 **GPS (General Problem Solver)**（Newell、Simon、Shaw）。
- **GPS 失败的两个原因**：① 非平凡问题难以表述成 GPS 能处理的形式；② 它是"通用"的，无法利用领域知识挑转换，每步穷举所有 transition，导致 **指数时间复杂度 exponential time complexity**。但 GPS **促成了 AI 从"通用求解器"转向"专用系统"**，这被视为一次 breakthrough（突破）。
- **启发式 heuristics（n. 经验法则）**：专家靠经验习得的 rule of thumb（经验法则）与事实，**高度依赖领域（domain dependent）**。专家系统的成功正来自"能表示启发式知识"。
- **核心架构范式**（1.3 节）：
  ```
  expert system = knowledge base（知识库） + inference engine（推理机）
  ```
  这是全书反复强调的 separation of knowledge and inference（知识/推理分离）。早期直接用 LISP 硬编码，知识与算法缠在一起，**知识一改就全要改**；分离之后才好维护。
- **知识获取 knowledge acquisition**：由 **知识工程师 knowledge engineer** 通过 **knowledge elicitation（知识抽取，即访谈专家）** 完成。
- **专家系统外壳 expert system shell**：剥掉领域知识后剩下的空架子，如 **EMYCIN**（= MYCIN 去掉医学知识）；**builder tools** 如 OPS5、LOOPS。
- **知识表示形式化应满足四个条件**：
  1. **表达力 expressive power** 足够；
  2. 有清晰的 **语义基础 semantic basis**；
  3. 有**高效算法解释**；
  4. 能给出 **解释与论证 explanation / justification**。
- **核心矛盾**：表达力 ↔ 可解释效率 **互相冲突**，实际系统常"牺牲表达力换效率"。

**两种基本推理方向（高频考点）**：

| 维度 | 自顶向下 top-down / 目标驱动 goal-directed | 自底向上 bottom-up / 数据驱动 data-driven |
|---|---|---|
| 起点 | 给定 goal，生成 subgoal 直到被数据满足 | 由已知 facts 反复推新 facts，直到推不动 |
| 别名 | backward chaining（后向链） | forward chaining（前向链） |
| 好用在 | 目标明确、搜索空间窄 | 数据多、目标不唯一 |

- **咨询系统 consultation system** 的四个部件（图 1.1）：推理机 + 知识库 + **用户界面** + **解释设施 explanation facilities**（回答"为什么问 / 怎么得出结论"）+ **追踪设施 trace facilities**（逐步观察推理，主要给调试用）。

### 1.2 案例 —— 经典专家系统（本节是全书案例总源头）

| 系统 | 领域 | 要解决的现实问题 | 核心技术 | 结果 |
|---|---|---|---|---|
| **MYCIN** | 医学（传染病） | 脑膜炎、细菌性败血症。血/尿培养要 24–48 小时才有结果，但**治疗不能等**，否则病人可能死亡 | 在 **incomplete & inexact（不完整且不精确）** 的 patient data 上给出最可能致病菌的临时判断；建议用药，并考虑药物相互作用、毒性反应；能解释结论 | Stanford 出品（E.H. Shortliffe）。**最著名的医学专家系统**，深刻影响了后续医学 KR 研究 |
| **INTERNIST-1 / CADUCEUS** | 内科 internal medicine | 内科有**数百种疾病**，且要考虑**多病共存**造成的症状组合，不可能逐一枚举 | 聚焦"给定症状、体征、化验结果下**最可能**的疾病" | Pople & Myers（Pittsburgh）。重点研究内科诊断的 disease model |
| **HEURISTIC DENDRAL** | 有机化学 | 由化学式 + 光谱数据**推断化合物的 structural formula（结构式）** | **generate-and-test（生成—测试）**。三子系统：**Structure Generator**（DENDRAL 算法生成全部同分异构体 + 启发式约束）、**Predictor**（预测质谱图）、**Evaluation Function**（按相似度比对实测质谱图） | 1965 年 Stanford 启动（Lederberg + Feigenbaum + Buchanan）。常输出多个按证据排序的答案 |
| **XCON（原 R1）** | 计算机配置 | 帮 DEC 按客户订单**配置 VAX / PDP11 / microVAX**。难点不是信息不完整，而是信息**变化极快（rapidly changing）**，且配置需要相当技能 | 按订单生成配置，可能替换或增补组件以得到可运行的系统 | DEC + McDermott（CMU），**1981 年起全面投产**；后配 **XSEL** 协助销售下单 |
| **NEOMYCIN** | 医学 | MYCIN 的重新设计 | 把各类诊断任务**更显式地分开** | MYCIN 的衍生 |
| **METADENDRAL** | 化学 | 减轻领域知识向 DENDRAL 转移的负担 | 能从 example **学习启发式 learn heuristics** | DENDRAL 的衍生 |

### 1.4 案例 —— 贯穿全书的问题域：心血管医学

- **心血管系统** = 心脏 + 血管网络（arteries 动脉 / capillaries 毛细血管 / veins 静脉）。动脉壁厚含平滑肌、运**富氧血**；静脉相反。
- **关键生理量**：主动脉平均血压 ≈ 100 mmHg，尺动脉 ≈ 90 mmHg，静脉 < 10 mmHg；**心输出量 cardiac output `CO = F · SV`**（心率 × 每搏量）。
- **深层知识 deep knowledge vs 浅层知识 shallow knowledge**（重要对比）：

| | 深层知识 deep knowledge | 浅层知识 shallow knowledge |
|---|---|---|
| 内容 | 系统**结构与功能**的详细知识 | 症状—疾病的**经验关联** |
| 例子 | 知道动脉狭窄为何导致腿部供血不足 | "走路腿抽筋、休息缓解 → 动脉狭窄" |
| 优点 | 论证强、能找病因 | 快、写规则容易 |
| 缺点 | 难获取、计算贵 | 论证弱、遇到例外就错 |

- 书中具体诊断示例：
  - 收缩压 > 140 mmHg **且**（舒张期杂音 **或** 心脏增大）→ **aortic regurgitation（主动脉瓣反流）**；
  - 腹痛 + 腹部杂音 + 搏动性包块 → **aortic aneurysm（腹主动脉瘤）**；
  - 小腿痛、走路出现、休息消失 → **arterial stenosis（动脉狭窄）**，常因 **atherosclerosis（动脉粥样硬化）**。

---

## 第 2 章 Logic and Resolution（逻辑与消解）

### 2.1–2.2 命题逻辑与一阶谓词逻辑

- **命题逻辑 propositional logic**：基本单位是 **atom（原子命题）**，真或假。5 个 connective（联结词）：`¬ ∧ ∨ → ↔`，优先级 `¬ > ∧ > ∨ > → > ↔`。
- 术语：**WFF (well-formed formula，合式公式)**、**interpretation（解释，把命题映射到 {true, false}）**、使公式为真的解释叫 **model（模型）**。
- 公式三分类：**valid / tautology（永真式）**、**unsatisfiable / contradiction（不可满足/矛盾式）**、**satisfiable（可满足）**。等价律表要记：**德摩根律 De Morgan**、**分配律 distributive**、双重否定、交换、结合。
- **逻辑后承 logical consequence `⊨`**：所有让前提为真的解释也让结论为真。
- **一阶谓词逻辑 FOL**：加入 predicates（谓词）、variables、functions、quantifiers（量词 ∀ ∃）。
  - **term（项）**、**atomic formula（原子公式）**、**free / bound variable（自由/约束变量）**、**closed formula / sentence（闭公式/句子）**、量词的 **scope（作用域）**。
  - **structure（结构）`S = (D, 函数集, 谓词集)`** + **assignment / valuation（赋值）** → 得到 interpretation。
  - 量词等价律（表 2.5）：`¬∃x ≡ ∀x¬`、`¬∀x ≡ ∃x¬`、`∀` 对 `∧` 分配。**注意 `∀x(P∨Q) ≢ ∀xP ∨ ∀xQ`**（易错点）。

### 2.3 子句形式 Clausal form（重点）

- **literal（文字）**：正文字或负文字。**clause（子句）**：`∀x₁…∀xₛ (L₁ ∨ … ∨ Lₘ)` —— 变量的全称量词隐式省略。**空子句 □ 恒假**。
- 子句的规则化写法（PROLOG 风格）：`A₁, …, Aₖ ← B₁, …, Bₙ`（左边是结论的 disjunction，右边是条件的 conjunction）。
- **化为子句的 8 步**（Example 2.16，考试常考流程）：
  1. 消去 `→`（用 `¬F ∨ G`）
  2. 否定内移（De Morgan + 量词对偶）
  3. **重命名变量**（产生 **variants（变体）**，避免同名冲突）
  4. 消去存在量词 → 引入 **Skolem 函数 Skolem function**（丢掉等价性，但**保持可满足性**）
  5. 前束化（所有量词提到最前，得 **prenex normal form**）
  6. 化为 CNF（合取范式，用分配律）
  7. 去掉量词前缀（变量默认为全称）
  8. 拆成子句集
- **Horn 子句**：至多含**一个正文字**，三种形式：
  - `A ←` —— unit clause（单位子句，即事实）
  - `← B₁…Bₙ` —— goal clause（目标子句）
  - `A ← B₁…Bₙ` —— 规则
  - **PROLOG 就建立在 Horn 子句之上**（这是逻辑与工程的关键接口）。

### 2.4 推理规则与两大元性质

- **Modus ponens**：`A, A→B ⊢ B`。推理规则是**纯语法操作**（只看符号形状，不管含义）。
- **可靠性 soundness**：`S ⊢ F ⇒ S ⊨ F`（能推出的都是真的）。
- **完备性 completeness**：`S ⊨ F ⇒ S ⊢ F`（真的都能推出）。
- **一阶谓词逻辑不可判定 undecidable**（Church & Turing, 1936）—— 不存在通用程序能判定任意公式是否可满足；**命题逻辑是可判定 decidable 的**。

### 2.5–2.6 消解与合一

- **反证法 refutation**：要证 `S ⊢ G`，等价于证 `W = S ∪ {¬G}` **不可满足**（推出空子句 □ 即成功）。这是所有自动定理证明的骨架。
- **消解规则 resolution**：找一对互补文字（`L` 与 `¬L`），删掉它们，把剩余部分取析取得 **resolvent（消解式）**。**推导树 refutation tree** 的叶是原始子句、根是 □。
- **替换 substitution** `σ = {t₁/x₁, …, tₙ/xₙ}`：把变量 x 换成项 t。**例 instance**：施加替换后的结果；不含变量的例叫 **ground instance**。
- **合一 unification**：找替换 σ 使两个表达式完全相同。**最一般合一元 mgu (most general unifier)**：任何其他 unifier 都能由它复合得到 —— 它是"最省约束"的那个。
- **合一算法**：反复取**最左不一致子式**组成 **disagreement set（不一致集）**，逐步构造 mgu；必须做 **occur check（出现检查）**，否则会把 `x` 绑成含 `x` 自身的项，造成死循环。
- **FOL 消解**用 **binary resolvent（二元消解式）** + **factor（因子，同一子句内互补文字先合并）** 得到 **general resolvent**。

### 2.7 消解策略（重点对比）

| 策略 | 做法 | 可靠性 / 完备性 |
|---|---|---|
| **Semantic resolution** | 用一个 interpretation I 把子句集分成"假集 S₁"和"真集 S₂"，父子句分别取自两侧 | 缩小搜索面 |
| **Set-of-support** | 把可满足的 S 与待证的 T（support set）分开，**每步至少一个父句来自 T** | 可靠且完备；风格类似 top-down |
| **Linear / Input resolution** | Linear：上一步的消解式必作下一步的父句；Input：必与原始子句消解 | 一般子句**不完备**；Horn 子句下 input resolution 完备 |
| **SLD resolution** | Horn 子句 + input resolution + **selection rule（选择规则，选定目标中的某个文字）** | **Horn 子句下可靠且完备** —— 这就是 PROLOG 的理论基础 |

- **SLD 推导**：目标子句 `G₀ = ← A₁…A_q` 与输入子句 `C = B ← B₁…B_p` 经 mgu θ 得到 `G_{i+1}`。所有可能路径构成 **SLD 树**，分支分三类：**success / failure / infinite**。
- **深度优先 depth-first（PROLOG 采用）不完整**（可能掉进无限分支）；**广度优先 breadth-first 完整**（但费内存）。这是 PROLOG 的经典取舍。
- **PROLOG 的三个工程妥协**：① 最左文字 + 子句书写顺序 + 深度优先；② **省略 occur check**（损失可靠性）；③ **negation as failure（失败即否定）** + **closed-world assumption（封闭世界假设）** —— 证不出 `P` 就当 `¬P`。
- 处理例外的小技巧：`oxygenrich(X) :- artery(X), not(exception(X)).` —— 用 `not` 表达"非 Horn"的知识。

### 2.8–2.10 实现与逻辑的定位

- **结构共享 structure sharing**（Boyer & Moore, 1972）：消解时不复制整个子句，只存**变量绑定 + 环境指针**，大幅降空间复杂度。
- **environment（环境）**：带 **level number（层号）** 的绑定集合，用来区分同名变量。书中 LISP 实现由 `Prove → Resolution → ResolveUnit → Unify` 构成。
- **等式与序的处理**：等式公理 E1–E4（自反、对称、传递、**置换性 substitutivity**）；实际用 **paramodulation（等式推理规则）** 提效；**唯一名字假设 unique name assumption**（不同名字 = 不同对象）。序公理 O1–O4 在 PROLOG 里做成内建谓词。
- **逻辑的定位**（2.10）：逻辑语义清晰、可解释性强，但**表达力与效率的张力**使它不适合直接当工程语言；PROLOG 就是"把逻辑工程化"的产物。

### 2.x 案例

- **Example 2.14 —— 用谓词逻辑建模心血管知识**：一元谓词 `Artery / Large / Wall / Oxygenrich / Exception` + 常量 `a`（主动脉）、`p`（肺动脉）。形式化"每条动脉都有肌壁""肺动脉是大动脉但不含富氧血 → 它是 exception"。
- **Example 2.16 —— 子句化 8 步示范**：把 `∀x(∃yP(x,y) ∨ ¬∃y(¬Q(x,y) → R(f(x,y))))` 一步步化为子句集，中途演示 Skolem 函数的引入。
- **Example 2.39 / 2.40 —— SLD 推导树**：对 Horn 子句集与目标 `← P(u,b)` 画出完整 SLD 树；再演示**子句顺序不当会产生 infinite derivation（无限推导）**，换个顺序就能得反证。
- **Example 2.51 —— 患者诊断咨询**：把 Ann（12 岁女，发热 + 舒张期杂音，血压 150/60）与 John（60 岁男，腹痛 + 腹部杂音 + 搏动性包块，血压 130/90）的数据与诊断规则译成 Horn 子句，跑 SLD 程序推断所患疾病 —— 直观展示"**逻辑本身就是专家系统**"。
- 扩展技巧：把 `=`、`>` 做成 **evaluable（可求值）目标**，在匹配到时调用 LISP 的 `eval` 求值。

---

## 第 3 章 Production Rules and Inference（产生式规则与推理）

### 3.1 知识表示

- **变量与事实**：变量分 **single-valued（单值）** 与 **multi-valued（多值）**；所有变量声明构成 **domain declaration（领域声明）**。运行时成立的"变量-值"对叫 **fact（事实）**，全体事实放进 **fact set / working memory（事实集 / 工作内存）**。`unknown` 是 **meta-constant（元常量）**，表示"还没推出值"（注意：不等于"值为假"）。
- **规则形态**：`if <antecedent> then <consequent>`。
  - 条件由 **predicate（谓词）** + 变量 + 常量组成：`same / notsame / greaterthan / lessthan / known / notknown`（后两个是 **meta-predicate（元谓词）**）。
  - 结论由 **action（动作）** 组成：`add`（加入值）、`remove`（删除值）。
  - **关键差异**：`add` 近似逻辑蕴含（单调）；**`remove` 破坏单调性** —— 这让产生式系统超出标准逻辑的范围。
- **object-attribute-value 三元组（o-a-v triple）**：把谓词/动作改写成 `(对象, 属性, 值)`，并配 **object schema（对象模式）**，里面可有 **subobject（子对象）**。
- **产生式 vs 一阶谓词逻辑**：多值属性 → 二元关系 `a(o,v)`；单值属性 → 函数 `a(o)=v`；`notsame` 靠 **negation by absence（缺席即否定）**；`unknown` 和 `remove` 这类**元信息标准 FOL 表达不了**。

### 3.2 推理

- **recognise-act cycle（识别-动作循环，全书最核心的循环）**：
  ```
  规则库 + 事实集  →  match（匹配）  →  select（选择，得到 conflict set）  →  apply（执行，add/remove 事实）  →  循环
  ```
- **Top-down = backward chaining（自顶向下/后向链）**：
  - 始于 **goal variable（目标变量）**；推不出就向用户询问 **askable variable（可问变量）**。
  - 主流程 `TraceValues → Infer → Select → Apply`。
  - 两类不确定性：**nondeterminism of the first / second kind（一/二类非确定性）**，靠冲突消解策略处理。
  - 两个优化技巧：标记 **traced（已追溯）**、标记 **used（已用过）** —— 防止自引用造成无限递归；**look-ahead（前瞻）** 提前剔除必然失败的规则。
- **Bottom-up = forward chaining（自底向上/前向链）**：由事实集触发。**每轮只应用一条规则**（因为 `remove` 会改变事实集，冲突集必须重算）。
- **冲突消解策略 conflict resolution（重点）**：

| 策略 | 规则 |
|---|---|
| **forward chaining** | 最简单 —— 来一条用一条 |
| **prioritization（优先级）** | 按人为设定的优先级 |
| **specificity（特异性）** | **条件更多者优先**（更具体的规则先跑） |
| **recency（近期性）** | 用 **time tag（时间标签）** 排序，最近加入的事实优先 |

- **rule instance（规则实例 `(R, M)`）**：规则 + 匹配集合的配对，用来防止同一条规则反复应用同一组事实。
- **rete algorithm**（Forgy 提出，OPS5 采用）：把规则编译成网络、缓存匹配结果，把"匹配"从每次重算变成增量更新 —— 详见第 7 章。

### 3.3 模式识别 Pattern Recognition

- **pattern（模式）**：有序的元素序列。变量以 `?` 开头表示**单值**、`!` 开头表示**多值**；纯 `?` / `!` 是 **don't-care variable（无关变量）**，不保留绑定。
- **binding（绑定）/ substitution（替换）**：匹配成功后变量的取值。
- **matching ≠ unification（易考）**：匹配是**不对称的** —— 事实不会反向替换模式变量。这个不对称性换来效率。

### 3.4 产生式作为表示形式的局限

1. 描述性知识（如心血管系统的结构）难表达；
2. **problem knowledge（问题知识）与 meta-knowledge（元知识）被迫用同一形式**，混在一起；
3. 操作性太强 —— 使用者必须懂执行模型（规则顺序会影响结果）；
4. 大规模知识库难 module（模块化）。

---

## 第 4 章 Frames and Inheritance（框架与继承）

### 4.1 语义网 Semantic Nets

- **顶点 = 概念**；**带标签的弧 = 二元关系**，典型如 `part-of`、`is-a`。
- **务必区分两种链接**（高频考点）：
  - **subset-of（子类包含）**：类 → 类，如 `large-artery` → `artery`
  - **member-of（成员属于）**：个体 → 类，如 `aorta` → `large-artery`
  - 二者都构成**偏序 partial order**，支撑 **property inheritance（属性继承）**。
- **exception（异常）问题**：如左肺动脉运**缺氧血**却属于 artery —— 继承会得出错误结论。
- **扩展语义网 extended semantic net**（Deliyanni & Kowalski）：把子句系统翻译成"二元谓词作顶点 + 弧"的图（条件弧、结论虚弧），可切成 **subnet**。

### 4.2 框架与单继承 Frames & Single Inheritance

- **frame（框架）**放在 **taxonomy（分类体系）**里，树状。分 **class frame（类框架）** 与 **instance frame（实例框架）**；链接分 **instance-of** 与 **superclass**（对应上面的 member-of / subset-of）。
- 框架内部是 **slot / attribute（槽 / 属性）**，内含 `attribute-type-pair` 与 `attribute-value-pair`。
- **facet（面）** —— 槽的"元信息"，是本章最实用的机制：

| facet | 含义 |
|---|---|
| `value` | 确定的、不可覆盖的值 |
| `default` | 缺省值，可被下层覆盖 |
| `demon`（过程型） | **if-needed**（需要时才算）/ **if-added**（加入时触发）/ **if-removed**（删除时触发） |

  这三种 demon 合称 **procedural attachment（过程附着）** —— 它是 frames 与 production rules 的桥。

- **异常的两种逻辑处理**：**McDermott & Doyle 的 non-monotonic logic（非单调逻辑）**（模态算子 `M`）或 **Reiter 的 default logic（缺省逻辑）**。

**四种继承算法对比（本章最值得画表的地方）**：

| 算法 | 结构 | 遍历与优先级 | 异常处理 | 适用场景 |
|---|---|---|---|---|
| **Single inheritance** | 树状 taxonomy | 自叶向根收集，先见者优先 | 同名属性下层"覆盖"上层（surpassing） | 单父类、无 facet 的简单层次 |
| **N-inheritance** | 树状 + facet | 先查本/上层 `value` → `if-needed` demon → `default` | 确定值 > 推导值 > 缺省值 | 强调"确切值最可靠" |
| **Z-inheritance** | 树状 + facet | 先在**本框架**查 `value` → demon → `default`，再上行 | 同框架内：确定值 > 推导 > 缺省 | 强调"来源越具体越可靠"（**本书主推**） |
| **Multiple inheritance** | 图状（DAG）+ preclusion | 构建 inheritance chain，用 intermediary / preclusion 取舍 | **inheritable conclusion set 必须一致**，否则 taxonomy 本身不一致 | 多父类，如 AV-anastomosis 既属静脉又属动脉 |

### 4.3 多继承 Multiple Inheritance

- 类框架可有**多个 superclass**，结构从此变成**图（DAG）**。
- **subtyping（子类型化）**：每个类关联一个 domain `D(y)`（属性序列集合）与 **type function（类型函数）τ**。`y₁ ≤ y₂` 当且仅当 `D(y₂) ⊆ D(y₁)` 且 `τ₁(a) ≤ τ₂(a)` —— 称 **correctly typed**。
- **属性值多继承的算法**：
  1. 构建 **inheritance chain（继承链）** → **conclusion set（结论集）**；
  2. 若结论集里同一属性有不同值 → **inconsistent（不一致）**；
  3. 引入 **intermediary（中介类）** 与 **preclusion（排除）** 得到 **inheritable conclusion set（可继承结论集）**；
  4. 若仍不一致 → 说明 taxonomy 设计本身有问题。
- **graph-shaped subtyping**：正确类型构成 **type lattice（类型格）**，有 **meet（∧，最大下界）** 与 **join（∨，最小上界）**。例：`vein ∨ artery = AV-anastomosis`（动静脉吻合）。

### 4.x 案例

- **心血管分类体系（书的贯穿示例）**：
  - 语义网：`heart part-of cardiovascular-system`；`large-artery is-a artery`、`aorta is-a large-artery`；`aorta` 经继承得到 `oxygen-rich blood`。
  - 框架树：根 `blood-vessel`（form = tubular, contains = blood-fluid）→ 子类 `artery`（wall = muscular, blood = oxygen-rich）/ `vein`；实例 `aorta`（diameter = 2.5）、`left-brachial-artery`（diameter = 0.4, location = arm）、`left-pulmonary-artery`（blood = oxygen-poor —— **异常覆盖**）。
  - 多继承：`pulmonary-artery` 同时 `≤ artery` 与 `≤ oxygen-poor-artery`，用 **preclusion** 只继承 oxygen-poor；`AV-anastomosis` 的 superclass = {artery, vein}（join 示例）。
- **规则片段（Example 3.3 / 3.7 / 3.8 / 3.12）**：
  ```text
  if same(complaint, abdominal-pain)
     and same(auscultation, abdominal-murmur)
     and same(palpation, pulsating-mass)
  then add(disorder, aortic-aneurysm)

  if greaterthan(systolic-pressure, 140)
     and greaterthan(pulse-pressure, 50)
     and (same(auscultation, diastolic-murmur) or same(percussion, enlarged-heart))
  then add(disorder, aortic-regurgitation)
  ```
  用它说明：goal variable 是 `disorder`；`complaint` / `sex` 是 askable variable，而 `disorder` 不可问；o-a-v 表示里 `patient` / `pain` 是 object schema。
- **扩展语义网实例**：`Wall(x, muscular) ← Isa(x, artery)` 对应到弧图。
- 书中提到的相关系统：**HEURISTIC DENDRAL**（最早用产生式做预测）、**MYCIN / EMYCIN**（top-down 诊断 + o-a-v 的代表）、**OPS5**（bottom-up 代表，含 rete）、**CENTAUR**、**LOOPS**（面向对象，demon 叫 method）。学者：Quillian（语义网）、Minsky（框架）、Forgy（rete）、McDermott & Doyle（非单调逻辑）、Reiter（缺省逻辑）。

---

## 第 5 章 Reasoning with Uncertainty（不确定推理）

### 5.1 问题的提出

- 规则形如 `if e then h [x]`，`x` 是不确定性度量。后向链推理铺开成 **inference network（推理网络）**。
- 需要四类 **combination function（组合函数）**：
  - **fprop** —— 证据 e 本身只被部分确认时，把不确定性传播给 h；
  - **fand / for** —— 合取 / 析取组合；
  - **fco** —— 多条规则共同支持同一假设（co-concluding）时怎么合并。

### 5.2 概率论

- **概率函数**：`P: 2^Ω → [0,1]`，满足 `P(e) ≥ 0`、`P(Ω) = 1`、互斥事件可加。
- **条件概率**：`P(h|e) = P(h ∩ e) / P(e)`
- **Bayes 定理**：`P(h|e) = P(e|h) · P(h) / P(e)`
- 假设互斥且穷尽（mutually exclusive and collectively exhaustive）时：
  `P(hᵢ|e) = P(e|hᵢ)·P(hᵢ) / Σⱼ P(e|hⱼ)·P(hⱼ)`
- 再加**条件独立 conditional independence** 假设，可写成乘积形式。
- **瓶颈**：要直接套概率论，需要**指数级数量**的 `P(e|hᵢ)` 与 `P(hᵢ)`，实际领域拿不到 → 这才引出各种 **quasi-probabilistic model（准概率模型）**。本章其余方法的共同动机就在这。

### 5.3 主观贝叶斯方法（PROSPECTOR 用）

- **odds（几率）**：`O(h) = P(h) / (1 − P(h))`，反解 `P(h) = O(h) / (1 + O(h))`
- **后验几率**：`O(h|e) = P(h|e) / (1 − P(h|e))`
- **似然比 likelihood ratio**（正向，level of sufficiency）：`λ = P(e|h) / P(e|¬h)`
  **负似然比**（level of necessity）：`λ̄ = (1 − P(e|h)) / (1 − P(e|¬h))`
  λ > 1 支持 h，λ < 1 支持 ¬h。
- **核心公式（考点）**：`O(h|e) = λ · O(h)`；`O(h|¬e) = λ̄ · O(h)`
- **多条规则共证**（条件独立下）：`O(h | ∩eᵢ) = (Π λᵢ) · O(h)` —— 乘法可交换，所以**更新顺序无关**，这是它比纯概率好用的一大原因。
- **前提不确切时**：用插值函数把 `P(e|e')` 映射成 `P(h|e')`，再算 **effective likelihood ratio（有效似然比）`λ' = O(h|e') / O(h)`**（落在 λ 与 λ̄ 之间）。复合证据用 `min{}` 近似 and、`max{}` 近似 or。

### 5.4 确定性因子模型（MYCIN 用，考试重灾区）

- **MB（measure of belief，信任测度）**：
  `MB(h,e) = max{0, (P(h|e) − P(h)) / (1 − P(h))}`（P(h) = 1 时取 1）
- **MD（measure of disbelief，不信任测度）**：
  `MD(h,e) = max{0, (P(h) − P(h|e)) / P(h)}`（P(h) = 0 时取 1）
- 性质：同一对 `(h, e)` 上，**MB 与 MD 至多一个为正**。
- **传播**：`MB(h,e') = MB(h,e) · MB(e,e')`，`MD(h,e') = MD(h,e) · MB(e,e')`
- **复合**：`MB(e₁ and e₂) = min{MB(e₁), MB(e₂)}`，`MB(e₁ or e₂) = max{·}`；MD 则 and 取 max、or 取 min。
- **共证**：`MB(h, e'₁ co e'₂) = MB(h,e'₁) + MB(h,e'₂)·(1 − MB(h,e'₁))`（MD = 1 时取 0），MD 对称。
- **CF 函数**：`CF(h,e) = (MB − MD) / (1 − min{MB, MD})`，取值 **[-1, 1]**。
- **CF 组合函数（三种情形，必背）**：
  - 皆正：`CF = CF₁ + CF₂·(1 − CF₁)`
  - **异号**：`CF = (CF₁ + CF₂) / (1 − min{|CF₁|, |CF₂|})`
  - 皆负：`CF = CF₁ + CF₂ + CF₁·CF₂`
- **CF 与概率的关系**：CF **不是真实概率**，它是由 MB/MD（本身从概率派生）构造出来的**实用近似** —— 没有严格概率基础，但计算极简单。这就是它当年被 MYCIN 选中的原因。

### 5.6 Dempster-Shafer 理论

- **frame of discernment（辨识框架）Θ**：所有互斥假设的集合。
- **basic probability assignment（基本概率分配）m**：`m(x) ≥ 0`、`m(∅) = 0`、`Σ_{x⊆Θ} m(x) = 1`。`m(x) > 0` 的 x 叫 **focal element（焦点元素）**，其全体叫 **core κ(m)**。
- **belief function（信度函数）**：`Bel(x) = Σ_{y⊆x} m(y)`
- **plausibility（似真度函数）**：`Pl(x) = Σ_{x∩y≠∅} m(y)`，且 `Pl(x) = 1 − Bel(¬x)`
- 区间 `[Bel(x), Pl(x)]` 叫 **belief interval** —— D-S 的精髓就是**用区间表示"无知"**。若 core 只含单点集，就退化为 Bayesian belief function。
- **Dempster 组合规则**：`m₁ ⊕ m₂ (x) = Σ_{y∩z=x} m₁(y)·m₂(z) / Σ_{y∩z≠∅} m₁(y)·m₂(z)`（分母是归一化因子，处理冲突）。

### 5.7 网络模型 Belief Networks

- **结构** = 有向无环图 DAG（定性）+ 各节点 **CPT（conditional probability table，条件概率表）**（定量）。有 m 个前驱就要 `2^m` 个概率；无前驱只需先验 `P(vᵢ)`。
- **条件独立假设**是威力所在：图结构本身就编码了独立关系，于是
  `P(V₁ ∧ … ∧ Vₙ) = Πᵢ P(Vᵢ | Cρ(Vᵢ))` —— **只需少量局部概率就能定义全局概率函数**。
- **证据传播**分两类：**causal support（因果支持，来自前驱）** 与 **diagnostic support（诊断支持，来自后继）**。
- **Kim & Pearl 模型**：限定于 **causal polytree（因果多叉树，任意两点间至多一条路径）**。节点间传两类消息：
  - **π（causal evidence parameter）** `π_{V₀}(vⱼ) = P(vⱼ | c̃_V(Gⱼ))` —— 自上而下
  - **λ（diagnostic evidence parameter）** `λ_{Vᵢ}(v₀) = P(c̃_V(Gᵢ) | v₀)` —— 自下而上
  每个节点收到邻居消息后**本地重算**自身概率；证据一次遍历即收敛。源节点 `π = P(Vᵢ)`，汇节点 `λ = 1`。
- **Lauritzen & Spiegelhalter 模型**：先把有向图转成 **decomposable graph（可分解图 —— 长度 ≥ 4 的初等环必有弦）**，步骤：① 给非相邻前驱加边；② 去掉方向；③ 给长环加弦。然后在 **clique（团 / 极大完全子图）** 上做局部计算，靠 **running intersection property（运行交集性质）**：
  `P(C_V(G)) = Πᵢ P(C_V(Clᵢ)) / Πᵢ P(C_Sᵢ)`
  证据进来后，以证据节点重新排序团，逐团本地更新边际，即 **junction tree / clique tree（联合树/团树）** 消息传播。

**五种不确定性方法总对比（本章最重要的表）**：

| 方法 | 优点 | 缺点 | 适用场景 | 代表系统 |
|---|---|---|---|---|
| **纯概率** | 数学严谨 | 需穷尽 `P(e\|h)` 与 `P(h)`，指数级，难获取 | 假设少、证据条件独立的小规模诊断 | — |
| **主观贝叶斯** | odds/λ 顺序更新直观；容许可不一致的专家评估 | 仍需先验；组合函数只是近似 | 专家能给出似然比的中型系统 | **PROSPECTOR** |
| **确定性因子 CF** | 计算极简、易实现，可用阈值过滤弱证据 | 无严格概率基础；共证后 MB/MD 可能同正 | 规则多、追求效率的医疗诊断 | **MYCIN** |
| **Dempster-Shafer** | 能表达"无知"与集合级不确定，Bel–Pl 区间 | 组合规则遇冲突要归一化；缺完整组合函数 | 证据部分、需显式不确定区间 | Ishizuka / Gordon & Shortliffe 的补充 |
| **信念网络** | 图形化、条件独立降维、局部计算高效 | 建/学 CPT 成本高；Kim-Pearl 限 polytree，L&S 要先转可分解图 | 变量多、依赖结构清晰的大领域 | Kim & Pearl / Lauritzen & Spiegelhalter |

### 5.x 案例

- **CF 抽象算例（Example 5.7，带完整数字推导）** —— 5 条规则：
  `R1: a and (b or c) → h [0.80]`、`R2: d and f → b [0.60]`、`R3: f or g → h [0.40]`、`R4: a → d [0.75]`、`R5: i → g [0.30]`
  用户给定 `CF(a) = 1.00, CF(c) = 0.50, CF(f) = 0.70, CF(i) = −0.40`。
  推导链：`CF(d) = 1.00 × 0.75 = 0.75` → `CF(d and f) = min{0.75, 0.70} = 0.70` → `CF(b) = 0.60 × 0.70 = 0.42` → `CF(b or c) = max{0.42, 0.50} = 0.50` → `CF(a and (b or c)) = min{1.00, 0.50} = 0.50` → R1 给 h `0.80 × 0.50 = 0.40`；R3 给 h `0.40 × max{0, 0.70} = 0.28`（g 因 i = −0.40 而 CF 归零）→ **共证净 `CF(h) = 0.40 + 0.28 × (1 − 0.40) = 0.568`**。
- **心血管 CF 规则 + PROLOG（5.5 节）**：
  ```prolog
  add(patient, disorder, aortic_aneurysm, cf(0.8, [CF1, CF2, CF3])) :-
       same(patient, complaint, abdominal_pain, CF1),
       same(patient, auscultation, murmur, CF2),
       same(patient, palpation, pulsating_mass, CF3).
  ```
  Example 5.14：取 `CF1 = 0.5, CF2 = 0.7, CF3 = 0.9` → 复合取 `min = 0.5` → 结论 `0.8 × 0.5 = 0.40`。另有析取式 `add(patient, disorder, aortic_regurgitation, cf(0.7, [...]))`。
  **阈值机制**：谓词 `same` 只在 `cf > 0.2` 时返回 true —— 这是 Shortliffe & Buchanan 设的 0.2 门槛，用来**过滤弱证据**（Table 5.1）。共证合并由 `case/3` 实现，皆正时用 `CF_old + CF_new − CF_old · CF_new`。
- **D-S 医学算例（Example 5.17 / 5.19）**：辨识框架 `Θ = {heart-attack, pericarditis, pulmonary-embolism, aortic-dissection}`。
  `m1: m1(Θ) = 0.6, m1({h-a, peri}) = 0.4`；`m2: m2(Θ) = 0.3, m2({h-a, pe, ad}) = 0.7`。
  交集表：`{h-a,peri} ∩ Θ → Θ (0.12)`、`{h-a,peri} ∩ {h-a,pe,ad} → {h-a} (0.28)`、`Θ ∩ Θ → Θ (0.18)`、`Θ ∩ {h-a,pe,ad} → {h-a,pe,ad} (0.42)`。
  于是 `m1 ⊕ m2({h-a}) = 0.28`，`Bel({h-a}) = 0.28`，`m1 ⊕ m2(Θ) = 0.30`，合计 1.0。
- **信念网络算例（5.7 节，"胸科"网络）**：节点与弧 —— `V1 → V2`；`V3(smoke) → V4(lung cancer)`、`V3 → V5(bronchitis)`；`V2 → V6(SOB)`、`V4 → V6`、`V5 → V6`；`V6 → V7`、`V6 → V8`。
  **只需 18 个局部概率**就能定义这个网络，而完整联合分布需要 `2^8 = 256` 个 —— 这就是降维的威力。
  分解式：`P(V1∧…∧V8) = P(V8|V6)·P(V7|V5∧V6)·P(V6|V2∧V4)·…·P(V1)`。
  Lauritzen 示例的 6 个团：`{V1,V2}, {V2,V4,V6}, {V4,V5,V6}, {V3,V4,V5}, {V5,V6,V7}, {V6,V8}`。证据 V8 = true 时重排团并本地更新：`P*(V6) = P(V6|v8)`、`P*(V5∧V6∧V7) = P(V5∧V6∧V7) · P*(V6) / P(V6)`。
- 书中提到的其他系统：**CASNET**（青光眼）、**PUFF**、**DIPMETER ADVISOR**、**INTERNIST/CADUCEUS**（用以反衬 Bayes 的"互斥穷尽"假设过强，因为内科疾病常**不互斥**）。

---

## 第 6 章 Tools for Knowledge and Inference Inspection（知识/推理检视工具）

### 6.1 知识点

- 专家系统区别于普通软件的关键特征之一：具备**检视知识库与推理过程的工具**。
- 对话形式三类：**user-initiated（用户主动）**、**computer-initiated（系统主动）**、**mixed-initiative（混合主动）** —— 多数系统采用混合式。
- **三类解释设施（本章核心，必考）**：

| 设施 | 回答什么问题 | 实现思路 |
|---|---|---|
| **how** | "这个属性值是怎么得出的？" | 查事实集，展示推出该值的**产生式规则**与作为前提的事实；区分值来自用户（`from_user`）还是规则推导（规则名） |
| **why** | "为什么要问我这个问题？" | 从当前子目标沿**推理链反向回溯**到最初的 goal 属性，显示导致提问的规则与子目标 |
| **why-not** | "这个值为什么没被推出来？" | 与 how 互补：找结论包含该 o-a-v 元组、但失败或未触发的规则，并指出**第一个失败的条件** |

- 三者都依赖：规则库 + 事实集 + 咨询期间记录的 **推理搜索空间（inference search space）**。
- **quasi-natural language interface（准自然语言界面）**：把 fact/rule 机械地翻译成句子，如 `the complaint of the patient is fever` —— 便宜且够用。

### 6.2–6.3 实现要点

- **PROLOG 版**：给每条 fact 与 rule 增加**第四个参数** —— 事实记录来源（`from_user` / 规则名 / `not_asked`），规则增加唯一名。
  - `interpret_command` 识别 `how(O,A)`、`why_not(O,A,V)`、`show(Rule)` 或值列表；
  - how 设施 `process_how` / `show_how` 按第四参打印来源；
  - why-not 设施 `process_why_not` / `show_why_not` / `evaluate_rules` / `evaluate_conditions` —— 用内置 `clause` 取出规则体，逐条件 `call` 找首个失败者。
- **LISP 版**（受 **EMYCIN** 启发）：object specification 含 `trans / prompt / constraint / class / legal / rules` 等槽。
  - **`*history*` 是一个栈**：每步 `PushHistory` 压入 `explain-step`（object / attribute / rule）；回溯用 `PopHistory`；
  - **why 设施 `ExplainWhy`** 递归回溯 history 栈直到 `TopExplanation`，打印整条推理链；
  - 另有 `facts` / `?` / `legal` / `trace` / `notrace` 命令。

### 6.4 Rule models（规则模型，源自 TEIRESIAS）

- **rule model 是元知识（meta-knowledge）**：对"结论相同 object-attribute"的规则子集做**高层归纳**。
- 构成：**supermodel / submodel**（树状分类）、**antecedent / consequent**（用 **singlet** 与 **correlation** 概括）、**based-on**（列出所基于的规则及 certainty factor）。
- **阈值规则**：singlet 要求该 object-attribute 出现在 ≥ 30% 的规则条件里、谓词覆盖率 ≥ 75%；correlation 要求 ≥ 80% 的规则中组合出现。
- **价值**：在大型知识库里提示"**缺失的条件/结论**"，辅助知识获取（knowledge acquisition）。

### 6.x 案例

- **PROLOG 对话实例（Ex 6.5，HEPAR 系统子集）**：
  ```text
  Enter ultrasound findings.
  |: how(patient, cholestasis).
  Attribute value extrahepatic has been derived using the rule named chol_1
  |: why_not(patient, cholestasis, intrahepatic).
  Rule chol_2 has failed due to condition same(patient, complaint, purpura)
  ```
- **LISP why 对话（Ex 6.8）**：
  ```text
  > why
  The following rule is being applied to infer the cholestasis of the patient
  If the complaint of the patient is abdominal-pain, and the nature of the is colicky, ...
  then conclude that the cholestasis of the patient is extrahepatic
  -> why          (继续回溯到 goal: diagnosis)
  -> legal        (显示合法值) -> (fever abdominal-pain) ...
  Final conclusion: ("Mirizzi's syndrome" "common-bile-duct stone")
  ```
- **rule model 实例（Ex 6.12）**：`model patient-diagnosis`，含 antecedent `(patient complaint same −0.4)`、consequent `(patient diagnosis add 0.2)`、based-on `(diag-1 −0.4) (diag-2 0.6)`；再特化为 `pos-diagnosis` / `neg-diagnosis`（按 CF 正负分）。

---

## 第 7 章 OPS5, LOOPS and CENTAUR

### 7.1 OPS5

- **production memory / working memory**：声明段用 `literalize` 定义对象（类）；运行时实例化成 working-memory element（带整数 **time tag**，用 `(wm)` 查看）。
- **规则语法**：`(p <name> <lhs> --> <rhs>)`
  - LHS 是 **condition element（条件元素）**：class + 属性-谓词-值三元组，如 `^age < 70`；
  - RHS 动作：`make` / `remove` / `modify`；
  - 支持变量 `<X>`、元素变量 `<pers>`、否定（`-` 前缀）、合取 `{...}`、析取 `<<...>>`。
- **recognise-act cycle**：识别（从 production memory 选出可触发的实例化，构成 **conflict set**）+ 动作（执行 RHS）。
- **冲突消解策略**：
  - **LEX**：按 time tag 序列的字典序 —— **recency（近期性）优先**；
  - **MEA**：means-ends analysis —— 优先选"首个条件元素"time tag 最新的实例化，实现**目标驱动**；
  - 仍不唯一就任选。
- **rete algorithm（本节重点）**：
  - 把规则**编译成 rete graph**：class 测试画成**椭圆**；属性-谓词-值测试画成**矩形（one-input / alpha node）**；多条件连接处画成**圆（two-input / beta node，存变量绑定 + 局部记忆）**；叶子是 **production vertex**。
  - **相同条件元素只表示一次**（共享测试）。
  - working memory 以 **token**（`<+ wme>` / `<- wme>`）沿图**增量传播**，只在两输入节点做 join。
  - **为什么快**：网络**编译一次**、测试**共享**、每轮只重算受影响的那部分 token —— 避免了"所有事实 × 所有规则"的朴素重复匹配（作者指出朴素匹配可占 90% 的时间）。
  - **不适用**于 non-temporally-redundant（如实时刷新）的应用。

### 7.3 CENTAUR

- **产生式规则的三个局限**（CENTAUR 的出发点）：
  1. 规则只**局部**给出适用上下文，整体问题结构隐含在规则库里（"uniformity" 问题）；
  2. **领域控制策略被迫与启发式知识用同一形式表达**，控制不可显式；
  3. 某个疾病的**典型表现分散在众多规则**里，难以集中获取。
- **prototype（原型）= frames + rules 的混合体**：
  - **component slot（对象知识部分）** 的 facets：name、actual value、**PEV（possible error values）**、**PV（plausible values）**、surprise values、**inference rules**（相当于 if-needed demon）、default value、**importance measure（0–5，用于匹配打分）**；
  - **meta 槽**：name / author / hypothesis / **moregeneral（is-a）** / morespecific / alternate；
  - **control 槽**：to-fill-in、if-confirmed、if-disconfirmed、action；
  - **rule 槽**：summary / fact-residual / refinement；
  - 另有 match measure、certainty measure、intrigger、origin。
  - 22 个 prototype 组成**树状分类法**。
- **fact（事实）**：全局工作记忆，六字段 —— `fname` / `fact value` / `certainty factor` / `where from` / `classification`（对每个原型标记 PV/PEV/SV）/ `accounted for`。
- **推理机制**：**hypothesize-and-match + agenda 驱动**（栈，LIFO，深度优先遍历分类法）。triggering rule 设 certainty measure → 挑出 relevant prototypes → 按 certainty 排序的 hypotheses list → 当前 prototype 填槽 → 用 match measure 与阈值比较，决定 confirm / disconfirm。
- **成就**：与 **PUFF** 相比，CENTAUR 的提问顺序**随病例动态变化**、聚焦更强；100 个病例中医生的判断更常与 CENTAUR 一致。强点是**对象知识与控制知识的显式分离**。
- **LOOPS**：支持多范式的**面向对象**专家系统构建工具；class-variable / instance-variable（含 default）、**method**、**demon**、多重继承算法。

### 7.x 案例

- **OPS5 祖先问题（Ex 7.16）**：`literalize person / kept / start`；规则 `ancestors` 用 `kept` 栈模拟递归。运行输出：
  ```text
  Enter name of a person: Apollo
  Leto and Zeus are parents of Apollo
  Rhea and Cronus are parents of Zeus
  ...  end -- no production true
  2 productions (9 // 9 nodes)
  ```
- **rete 图（Ex 7.17–7.19）**：单条件 `(patient ^age < 40 ^complaint = fever)` 译为"class 椭圆 + 两个矩形 alpha 节点"；两条规则共享 `^name = <n>` 时该测试只画一次；两输入节点（圆）存变量绑定并做一致性连接。
- **CENTAUR 肺病病例（Ex 7.24–7.27）**：
  - component `fev1`（forced expired volume，用力呼气量）importance = 5；`reversibility` 的 inference rules 指向 `RULE019…025`，actual value = 43。
  - fact `tlc`：value = 126、cf = 0.8、where from = USER、classification = `((PV OAD)(SV NORMAL))`、accounted for = OAD。
  - prototype `oad` 含 moregeneral `(DOMAIN pulmonary-disease)`、morespecific 各 DEGREE/SUBTYPE；if-confirmed 先定 **degree** 再定 **subtype**。

---

## 附录 A / B：PROLOG 与 LISP

### A. PROLOG 要点

- **unification（合一）** 求 **mgu**；**backtracking（回溯）** 撤销实例化寻找替代解。
- **cut（`!`，剪枝）**：提交当前所选子句，**禁止回溯**；常与 **fail** 配合表达"互斥 / 例外"。
- **negation as failure**：`not` 即"证不出就算假"。
- **声明式 vs 过程式语义**：口号是 `algorithm = logic + control`，但 PROLOG 的**子句顺序会影响结果**，实践上偏过程式。
- 数据库操作：`asserta` / `assertz` / `retract`（运行时改库）；`consult` 读程序。
- 检查实例化：`var` / `nonvar`；算术 `is` / `=:=`；I/O `write` / `read` / `nl`。
- 书中代码示例：`member(X,[X|_]).` / `member(X,[_|Y]) :- member(X,Y).`（列表成员，演示回溯）；二叉树 `path` / `branch`；`number_of_parents`（演示 cut 的副作用）。

### B. LISP 要点

- **S-expression**：atom 或 list，底层是 cons / dotted pair，配 `car` / `cdr`。
- **quote（`'`）**：关闭求值。
- **作用域**：默认是 **lexical scope（词法/静态作用域，局部）**；用 `defvar` 声明的是 **special / dynamic scope（动态作用域，运行期全局）** —— 这是 LISP 的经典易错点。
- **lambda**：无名函数；**eval / apply / funcall** 用于强制求值或接口调用。
- **宏与 backquote**：`` ` `` 关闭求值、`,` 打开求值；**宏展开发生在求值之前**（Example B.41 的 `other-if` 就说明为什么函数版会先求值出错）。
- **defstruct**：记录式结构，构造器形如 `make-person`。
- 书中代码示例：`element`（集合成员，递归）；`((lambda (a b) (sqrt (+ (* a a) (* b b)))) 3 4) => 5`；`(defstruct person (name) (age 30) (married 'no))`；`dolist` / `do` 迭代。

### PROLOG vs LISP 作为专家系统实现语言（对比表）

| 维度 | PROLOG | LISP |
|---|---|---|
| 范式 | 逻辑程序设计（Horn 子句） | 函数式 + 命令式 |
| 核心机制 | unification + backtracking | S-expression 求值、substitution model |
| 推理风格 | 声明式驱动，但对顺序敏感（偏过程） | 过程式，控制流显式 |
| 知识表示 | 子句 / 事实 / 规则 | 结构、框架、列表、宏 |
| 作用域 | 子句内（无全局变量） | lexical（默认）/ dynamic special |
| 典型用途 | 规则与关系、专家系统原型 | 符号操作、复杂数据结构、构建工具 |
| 解释设施怎么实现 | `clause` 取规则体、`asserta` 记来源 | `*history*` 栈、`defstruct` 记状态 |

---

## 附录一：全书命名系统一览（背这一张表就够）

| 系统 | 领域 | 核心技术 | 书里用它说明什么 |
|---|---|---|---|
| **MYCIN** | 传染病诊断 | 产生式规则 + **CF 模型** | 不确定推理的起点；"专家不愿接受 P(¬h\|e) = 1 − P(h\|e)"的动机 |
| **EMYCIN** | — | MYCIN 去掉医学知识后的**外壳 shell** | 知识/推理分离的直接产物 |
| **NEOMYCIN** | 医学 | 显式区分诊断任务 | MYCIN 的重设计 |
| **INTERNIST-1 / CADUCEUS** | 内科 | 多病共存的假设竞争 | 说明"互斥穷尽"假设在真实内科里不成立 |
| **HEURISTIC DENDRAL** | 有机化学 | **generate-and-test** | 最早的专家系统之一 |
| **METADENDRAL** | 化学 | 从例子**学习启发式** | 知识获取的自动化尝试 |
| **XCON / R1** | 计算机配置 | 产生式规则 | 第一个真正商业成功的专家系统（1981 投产） |
| **XSEL** | 销售下单 | — | XCON 的补充 |
| **PROSPECTOR** | 地质勘探 | **主观贝叶斯（odds + 似然比）** | 主观贝叶斯方法的代表 |
| **CASNET** | 青光眼 | 因果网络 | 早期网络式推理 |
| **PUFF** | 肺功能 | 产生式规则 | 与 CENTAUR 对比的 baseline（基线） |
| **CENTAUR** | 肺病 | **prototype（frame + rule）** | 对象知识与控制知识分离 |
| **OPS5** | 通用 shell | 前向链 + **rete 算法** | 工业级产生式系统 |
| **LOOPS** | 通用 | 面向对象 + method/demon | 多范式构建工具 |
| **HEPAR** | 肝病 | how / why-not | 第 6 章解释设施的实现示例 |
| **TEIRESIAS** | — | **rule models** | 元知识辅助知识获取 |
| **DIPMETER ADVISOR** | 石油测井 | 不确定性推理 | 同类系统列举 |

---

## 附录二：贯穿案例 —— 心血管医学域知识库

全书绝大多数示例集中在这一个小领域，值得单独记住：

| 层面 | 内容 |
|---|---|
| **解剖结构** | 心血管系统 = 心脏 + 动脉 / 毛细血管 / 静脉；`heart part-of cardiovascular-system` |
| **典型疾病** | `aortic-aneurysm`（主动脉瘤）、`aortic-regurgitation`（主动脉瓣反流）、`arterial-stenosis`（动脉狭窄）、`atherosclerosis`（动脉粥样硬化） |
| **症状/体征字段** | complaint、auscultation（听诊）、palpation（触诊）、percussion（叩诊）、systolic-pressure、pulse-pressure |
| **逻辑表示（Ch2）** | 一元谓词 `Artery / Large / Wall / Oxygenrich / Exception`；常量 `a`=主动脉、`p`=肺动脉 |
| **产生式表示（Ch3）** | `same / notsame / greaterthan / lessthan + add / remove`；o-a-v 三元组 |
| **框架表示（Ch4）** | 树：`blood-vessel` → `artery` / `vein`；实例 `aorta`(diameter 2.5)、`left-brachial-artery`(0.4, arm)、`left-pulmonary-artery`（blood = oxygen-poor，**异常**） |
| **不确定表示（Ch5）** | `add(patient, disorder, aortic_aneurysm, cf(0.8, [...]))` |
| **经典患者** | Ann（12 岁女，发热 + 舒张期杂音，150/60）、John（60 岁男，腹痛 + 腹部杂音 + 搏动包块，130/90） |

**最重要的一条异常**：`left-pulmonary-artery` 属于 artery，却运 **oxygen-poor blood** —— 这一条同时被用来讲语义网的继承失效、框架的 exception、以及多继承的 preclusion。**考到继承就一定会提到它。**

---

## 易考点汇总（Exam checklist）

- [ ] `expert system = knowledge base + inference engine`，以及 shell / builder tool 的区别。
- [ ] 四种知识表示形式化（logic / production rules / semantic nets / frames）各自的**表达力与效率**取舍。
- [ ] top-down（backward chaining）vs bottom-up（forward chaining）的适用场景。
- [ ] 子句化 8 步，尤其是**为什么引入 Skolem 函数**（保可满足、不保等价）。
- [ ] **Horn 子句三种形式**，以及为什么 PROLOG 选它。
- [ ] **mgu 与 occur check**；matching ≠ unification（不对称性）。
- [ ] **SLD** 与深度优先不完整、negation as failure、封闭世界假设。
- [ ] **reliability soundness vs completeness**；一阶逻辑不可判定。
- [ ] recognise-act cycle；四种冲突消解策略（尤其 **specificity** 与 **recency/time tag**）。
- [ ] `add` 单调 vs **`remove` 破坏单调性** —— 为什么会超出标准逻辑。
- [ ] subset-of vs member-of；**single inheritance 的"就近覆盖" vs Z-inheritance 的 facet 优先级**。
- [ ] 多继承的 **preclusion** 与"可继承结论集必须一致"。
- [ ] Bayes 三公式；**odds-likelihood `O(h|e) = λ·O(h)`** 及顺序无关性。
- [ ] **MB / MD / CF 三公式 + CF 共证三种情形的合并公式**（最常考）。
- [ ] D-S 的 **m / Bel / Pl / belief interval** 与组合规则。
- [ ] belief network 的**局部概率分解**（18 vs 256）、causal vs diagnostic support、Kim-Pearl（polytree + π/λ）vs Lauritzen-Spiegelhalter（可分解图 + clique tree）。
- [ ] **how / why / why-not** 三者的推理链方向（how 与 why-not 查规则+事实、why 反向回溯到 goal）。
- [ ] **rete 为什么快**（编译一次 + 共享测试 + 增量 token）。
- [ ] CENTAUR 的三个局限与 prototype 结构；OPS5 的 **LEX vs MEA**。
- [ ] PROLOG 的 `cut` / `fail` / unity；LISP 的 **lexical vs dynamic scope**。

---

## 一句话总结（每章）

| 章 | 一句话 |
|---|---|
| 1 | 专家系统的本质是"**知识库 + 推理机**"的分离。 |
| 2 | 逻辑（尤其 Horn 子句 + SLD 消解）语义清晰，是理解其他一切表示形式的**地基**，PROLOG 是它的工程化身。 |
| 3 | 产生式用"**规则 + 事实 + 识别-动作循环**"擅长启发式分类，但弱于结构化描述、且操作性强。 |
| 4 | 语义网 / 框架用"**分类体系 + 继承**"擅长描述性层次知识，难点在**异常与多继承的一致性**。 |
| 5 | 五种不确定推理模型是沿"**数学严谨性 ↔ 工程可行性**"这条线演进出来的：概率论严谨但难用 → 主观贝叶斯与 CF 用近似换效率 → D-S 容纳"无知" → 信念网络用条件独立实现大规模高效推理。 |
| 6 | 可解释性 = 记录**推理搜索空间**（PROLOG 第四参 / LISP `*history*` 栈）；rule model 用元知识帮知识获取。 |
| 7 | OPS5 用 **rete 增量匹配网络**解决效率，CENTAUR 用 **prototype + agenda** 解决结构清晰度。 |
