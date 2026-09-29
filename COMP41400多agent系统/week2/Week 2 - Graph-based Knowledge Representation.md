---
course: COMP41400 Multi-Agent Systems
week: 2
topic: Graph-based Knowledge Representation
source: KR Graphs.pptx
tags:
  - COMP41400
  - Knowledge-Representation
  - Knowledge-Graph
  - RDF
  - Semantic-Web
---

# Week 2 - Graph-based Knowledge Representation

Source: `KR Graphs.pptx` (43 slides)

## Core idea

Graph-based Knowledge Representation（基于图的知识表示）使用 **nodes**（节点）表示 entities / concepts（实体／概念），使用 directed labelled **edges**（有向带标签边）表示 relations（关系）。

```text
LA380 ──from──> Santiago
LA380 ──to────> Arica
LA380 ──company─> LATAM
```

它特别适合 Multi-Agent Systems：多个 Agent 可依据共享 ontology（本体）理解同一 entity、relation 和 data meaning（数据含义）。

---

# Slide-by-slide explanation（逐页讲解）

## Slides 1–8: Graph-based approaches

### Slide 1 — Knowledge Representation: Graph-based Approaches

本课从 Logic-based representation 转向 Graph-based representation。重点不再只是 “某个 proposition 是否 true”，而是 “哪些 entities 彼此 connected，以及 connection 的 meaning 是什么”。

### Slide 2 — Graph-based Approaches

课件列出三类方法：

- **Semantic Networks**：以 graph 表示 concepts 与 relations。
- **Frames**：以 slots（槽）表示典型情境。
- **Semantic Web / Knowledge Graphs**：以可共享的 Web standards 表示现实世界知识。

### Slide 3 — Semantic Networks

Semantic Network 是 graph-based representation。它将 concepts 画作 nodes，将 concepts 间的 relationship 画作 edges，类似 brain map（脑图），但 edge 应具明确 semantic label（语义标签）。

```text
Dog ──is_a──> Animal
Dog ──has_part──> Tail
```

历史背景：人类很早就使用关系图组织知识；课件指出可追溯至 3rd Century AD。AI 中的 Semantic Networks 在 1950s–1960s 成为重要知识表示方法。

### Slide 4 — Semantic Networks: why relations matter

这页延续上一页。关键不在“画图”，而在 relation 允许 reasoning：若 `Dog is_a Animal`，则任何具体 Dog 可 inherit（继承）Animal 的 properties。Graph 使这种 path（路径）与 hierarchy（层级）可见。

### Slide 5 — Frame-based Systems

Frame 是代表 stereotyped situation（典型情境）的 data structure（数据结构），例如 “客厅” 或 “儿童生日派对”。

```text
Frame: BirthdayParty
  location: home
  attendees: children
  activity: games
```

Frame 的 slots 可保存 facts/data、values（称为 facets）、procedures（称为 procedural attachments），也可连接 subframes（子框架）。

### Slide 6 — Frame-based Systems: connection to AI

Frame 让 Agent 在一个 situation 出现时，快速填补合理的 default knowledge（默认知识）。例如进入 restaurant frame，Agent 预期会有 menu、table、ordering 等 slots。

历史背景：Marvin Minsky 在 1974 的 *A Framework for Representing Knowledge* 推广 frame theory。它后来影响 object-oriented modelling 与 modern schema design。

### Slide 7 — Knowledge Graphs

Knowledge Graph 的定义：它是表达现实世界知识的 data graph；nodes 代表 entities of interest（关注实体），edges 代表这些 entities 的 relations。

重点：普通 graph 可只是连接结构；Knowledge Graph 还要求 entity 和 relation 有可解释的 real-world semantics（现实语义）。

### Slide 8 — Knowledge Graphs: Syntax and Semantics

本页区分：

| Syntax（语法） | Semantics（语义） |
|---|---|
| graph of data | knowledge of the real world |
| nodes | entities of interest |
| edges | relations between entities |

如果 `Santiago` 只是文字，它只是 data；当 ontology 指出它是 `CapitalCity`，`from` 是航班出发地关系时，才形成 knowledge。

## Slides 9–11: Data models for Knowledge Graphs

### Slide 9 — Data Models for Knowledge Graphs

课件以航班领域比较两种 graph data model：Directed Edge-Labelled Multigraph 和 Property Graph。两者都可描述 `LA380`、`LATAM`、`Santiago`、`Arica` 及其 relations。

### Slide 10 — Directed Edge-Labelled Multigraph

此模型有三个特点：

- **Directed**：edge 有方向，例如 `LA380 ─from→ Santiago` 与反向关系不同。
- **Edge-labelled**：edge 的 label 如 `from`、`to`、`company` 表达语义。
- **Multigraph**：同两个 nodes 间可存在多条不同 relations。

它是 RDF 和 Semantic Web 的基本 graph model，优点是 minimal（最小化）且容易标准化。

### Slide 11 — Property Graph

Property Graph 给 nodes/edges 加入 labels、types 和 key-value properties，例如：

```text
(Santiago:CapitalCity {lat: -33.45, long: -70.66})
(LA380:Flight {company: LATAM})
```

Neo4j 是常见 Property Graph database。课件指出它比 Directed Edge-Labelled Multigraph 更 flexible（灵活），并且两种模型可转换。该模块优先讲前者，因为它支撑 Semantic Web。

## Slides 12–15: Semantic Web and Linked Data

### Slide 12 — The Semantic Web

Semantic Web 是让 Web 从 human-readable documents（人类可读文件）进化为 machine-readable linked data（机器可读链接数据）的目标。

### Slide 13 — Tim Berners-Lee’s vision

Tim Berners-Lee 在 1999 提出：computers 应能分析 Web 上的 content、links 与 transactions；最终让 machines talking to machines，支持真正的 intelligent agents。

这不是要求所有网页都改成一个中央 database，而是让不同资源可由 shared identifiers 和 shared vocabulary 连接。

### Slide 14 — RDF

本页把 RDF 放入 Semantic Web 愿景。RDF（Resource Description Framework）提供共享 representation format，使不同 organisation 的 data 可表达为 compatible graph。

### Slide 15 — Linked Data

Linked Data 将 RDF 放在可访问 Web resources 上。理想 workflow：每个 entity 使用 URI（统一资源标识符），Agent 通过 HTTP GET 请求该 URI，得到包含 relations 的 RDF document，并沿 links 继续 retrieve knowledge。

## Slides 16–21: What is RDF?

### Slide 16 — RDF Graph and Turtle syntax

RDF 是基于 Directed Edge-Labelled Multigraph 的 W3C standard。RDF graph 是 triple（三元组）集合：

```text
subject ──predicate──> object
```

课件用 Turtle syntax（Terse RDF Triple Language）表示航班图。

### Slide 17 — RDF components

RDF triple 的合法组成：

- subject：URI 或 blank node（匿名节点）；
- predicate：URI；
- object：URI、blank node 或 literal（字面量，如 number/string/date）。

```turtle
:LA380 :company :LATAM .
```

读作：LA380 的 company 是 LATAM。

### Slide 18 — RDF and directed edges

同一 graph 可写成图形或 Turtle。箭头方向来自 subject 指向 object，箭头 label 来自 predicate：

```turtle
:LA380 :from :Santiago .
:LA380 :to :Arica .
```

不要把 `from` 误认为 node。它是 relationship 的 identifier。

### Slide 19 — RDF triples in practice

Turtle 中同一 subject 的多个 predicates 可用 `;` 串联：

```turtle
:LA380 :company :LATAM ;
       :mode :Flight ;
       :from :Santiago ;
       :to :Arica .
```

`;` 表示 subject 不变，`.` 表示这一组 statements 结束。

### Slide 20 — Subject, Predicate, Object

本页标示三元组位置：

```text
:LA380     :company      :LATAM
subject    predicate     object
```

它与 Predicate Logic 的 binary predicate 对应：

```text
company(LA380, LATAM)
```

### Slide 21 — RDF as Web resources

RDF node 可被看作 Web resource。若 Agent 查询资源，例如 `GET /LA380`，server 可返回描述 LA380 的 triples。这是从 standalone data graph 走向 linked data 的关键。

## Slides 22–27: Building RDF Graphs

### Slide 22 — Building RDF Graphs

section divider。接下来不只存 facts，也使用 RDF Schema（RDFS）表达 ontology：classes、properties 和 constraints。

### Slide 23 — Classes and Properties

基本构造：

```turtle
:Person rdf:type rdfs:Class .
:friendOf rdf:type rdf:Property .
```

`Person` 是 class（类别），`friendOf` 是 property（关系）。注意 property 本身也可作为 RDF graph 中的 resource 被描述。

### Slide 24 — Type and hierarchy

instance 与 class：

```turtle
:Rem rdf:type :Person .
```

class hierarchy：

```turtle
:Student rdfs:subClassOf :Person .
```

若 `Alex rdf:type Student`，reasoner 可 infer `Alex rdf:type Person`。

### Slide 25 — Domain and Range

`domain` 限定 relation 的 subject type，`range` 限定 object type：

```turtle
:friendOf rdfs:domain :Person .
:friendOf rdfs:range :Person .
```

重要 caveat：在 RDFS 中 domain/range 主要触发 type inference，不是 database validation。`friendOf(car, Rem)` 更可能使 reasoner 推出 `car` 是 Person，而不是自动报错。

### Slide 26 — RDFS construct table

本页汇总 Class、Property、type、subClassOf、subPropertyOf、domain 与 range。它们共同构成一个轻量 ontology language。

### Slide 27 — Subproperty reasoning

```turtle
:goodFriendOf rdfs:subPropertyOf :friendOf .
```

于是：

```turtle
:Liam :goodFriendOf :Rem .
```

蕴含：

```turtle
:Liam :friendOf :Rem .
```

这说明 graph 的 relation hierarchy 可产生 implicit knowledge（隐含知识）。

## Slides 28–33: Representing Knowledge

### Slide 28 — Ontology versus state

区分两层：

- **Ontology**：哪些 classes/properties 存在，及它们的 meaning。
- **State**：当前 domain 的具体 facts，例如 `Rem` 是 Person。

Ontology 类似共享 schema；state 是 Agent 当前 beliefs 或 world facts。

### Slide 29 — Friend ontology

```turtle
:Person rdf:type rdfs:Class .
:friendOf rdf:type rdf:Property .
:friendOf rdfs:domain :Person .
:friendOf rdfs:range :Person .
```

这定义 vocabulary，但还没有说明谁与谁是朋友。

### Slide 30 — State facts

加入 instances：

```turtle
:Rem rdf:type :Person .
:Liam rdf:type :Person .
```

现在 graph 开始表示具体 world state。

### Slide 31 — Explicit relation

```turtle
:Liam :friendOf :Rem .
```

这是 explicit fact（显式事实）：Liam 是 Rem 的朋友。

### Slide 32 — Inference from ontology

若没写 `Liam rdf:type Person`，但已有 `Liam friendOf Rem`，RDFS domain 仍可推导 Liam 为 Person；range 可推导 Rem 为 Person。这是 ontology 让 data become knowledge 的例子。

### Slide 33 — Ontology, state and implicit fact

本页明确指出：Ontology 提供 class/property semantics；State 提供具体 triples；reasoner 将两者组合，得到 implicit facts。Multi-Agent Systems 可利用这一点共享 vocabulary，同时各 Agent 保持不同 local state。

## Slides 34–39: JSON-LD and schema.org

### Slide 34 — JSON-LD

JSON-LD（JSON for Linked Data）让 RDF/Linked Data 能以熟悉 JSON 写法表达。它适合 Web APIs 与网页 metadata。

### Slide 35 — `@context`

`@context` 将短 field name 映射到正式 vocabulary：

```json
"@context": "https://schema.org"
```

因此 `name`、`jobTitle`、`address` 不只是任意 JSON keys，而带有 schema.org 定义的 shared meaning。

### Slide 36 — `@type`

```json
"@type": "Person"
```

`@type` 表示当前 object 是哪个 class 的 instance。它对应 RDF 的：

```turtle
:JaneDoe rdf:type schema:Person .
```

### Slide 37 — JSON-LD Person example

课件使用 Jane Doe：`name`、`jobTitle`、`email`、`telephone`、`colleague` 和 nested address。JSON nesting 可转换为 graph nodes 与 edges。

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Jane Doe",
  "jobTitle": "Professor"
}
```

### Slide 38 — Blank nodes and nested objects

嵌套 `address` 可能创建 blank node：它有 properties，但不必拥有 global URI。

```json
"address": {
  "@type": "PostalAddress",
  "addressLocality": "Seattle"
}
```

Blank node 适合 address 这类局部结构；若 entity 需要跨 documents 连接，URI 通常更合适。

### Slide 39 — schema.org/Person

schema.org 是常用 Web vocabulary。`schema:Person` 标准化了 `name`、`email`、`jobTitle` 等 relation 的 meaning，使不同网站与 Agent 更容易 interoperability（互操作）。

## Slides 40–43: Back to Knowledge Graphs

### Slide 40 — Back to Knowledge Graphs

这页把 RDF/JSON-LD 拉回 Knowledge Graph：RDF 提供存储、链接和交换 graph knowledge 的 standard format。

### Slide 41 — Semantic Web: consistent logical web of data

“consistent logical web of data” 指不同资源以 shared identifiers 与 ontologies 相连。目标不是保证现实资料永远无冲突，而是让 machines 可 identify、query、combine 与 reason over the data。

### Slide 42 — HTTP retrieval example

本页展示：Agent 对 `LA380` 执行 HTTP GET，服务器返回航班 RDF；再对 `Santiago` 执行 GET，取得城市 RDF。每个 resource 都能导向更多 linked knowledge。

```text
GET /LA380 → airline, origin, destination
GET /Santiago → city type, latitude, longitude
```

这就是 Semantic Web 的 traversal（遍历）思想。

### Slide 43 — Summary

Graph-based approach 的优势：

- 比 relational database 或 fixed frame schema 更 flexible；
- 可 mash up（组合）多个 ontologies，表示 heterogeneous data；
- 适合 Enterprise Knowledge Graph、Semantic Data Lake/Warehouse；
- 支持 Semantic RAG for Agentic Systems。

RDF 是 W3C graph representation standard，JSON-LD 是适合 Web 的一种 RDF representation format。

---

# Semantic RAG connection

Vector RAG 通常通过 similarity（相似度）找 text chunks；Semantic RAG 同时利用 entities、relations 与 ontology constraints。

```text
Question
→ identify entities and relations
→ retrieve relevant RDF subgraph
→ traverse graph / apply ontology inference
→ provide evidence to LLM
→ generate grounded answer
```

例如查询 “LATAM 从 CapitalCity 出发、飞往 PortCity 的航班”：Agent 可连接 `company`、`from`、`to`、`rdf:type` edges，而不只依赖关键词匹配。

# Exam checklist

- 区分 Semantic Network、Frame、Knowledge Graph。
- 解释 node、edge、label、entity 与 relation。
- 对比 Directed Edge-Labelled Multigraph 与 Property Graph。
- 写出 RDF triple 的 subject、predicate、object。
- 理解 URI、blank node、literal、Turtle 的 `;` 与 `.`。
- 解释 `rdf:type`、`rdfs:subClassOf`、`rdfs:subPropertyOf`、`domain`、`range`。
- 区分 ontology、state、explicit fact 与 implicit fact。
- 说明 JSON-LD 的 `@context`、`@type` 和 schema.org。
- 说明 Semantic Web 与 Semantic RAG 如何支持 Multi-Agent Systems。
