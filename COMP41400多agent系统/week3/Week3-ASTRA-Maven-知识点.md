---
course: COMP41400 Multi-Agent Systems
week: 3
topic: ASTRA Maven
tags:
  - COMP41400
  - Multi-Agent-Systems
  - ASTRA
  - Maven
  - Build
---

# Week 3 - ASTRA 与 Maven（知识点）

> 来源：ASTRA-Maven.pptx（4 页）+ IntroductionToASTRA.pptx 第 30–32 页 + 本地可运行工程 `week3/hello`
> 关联笔记：[[Week3-ASTRA-Modules-逐页讲解]]、[[Week3-IntroductionToASTRA-知识点]]

本讲是 ASTRA 的**工程实现层**：怎么编译、怎么跑。

---

## 一、编译流水线（ASTRA-Maven slide 2 + 官方 deploying 文档 + 本地实据）

```
Main.astra  --[ASTRA Compiler]-->  Main.java  --[javac]-->  Main.class
src/main/astra                     target/gen/java           target/classes
```

**本地 `week3/hello` 工程的真实证据**：

```
./src/main/astra/Main.astra        ← 人写的 ASTRA 源码
./target/gen/java/Main.java        ← 编译器生成的 Java（86 行，头部写着 GENERATED CODE - DO NOT CHANGE）
./target/classes/Main.class        ← 编出的字节码
./target/classes/Main.astra        ← 顺手把源码也拷进去了
```

生成出来的 `Main.java` 把 agent 完整「翻译」成 Java：

```java
public class Main extends ASTRAClass {
    public Main() {
        setParents(new Class[] {astra.lang.Agent.class});
        addRule(new Rule("Main", new int[] {6,9,6,19},        // ← 行号区间，报错时能指回 .astra
                new GoalEvent('+', new Goal(new Predicate("init", new Term[] {}))),
                Predicate.TRUE,                                // ← context
                new Block("Main", new int[] {6,18,8,5}, new Statement[] {
                    new ModuleCall("console", ..., new Predicate("println", ...), ...)
                })));
    }
    public void initialize(astra.core.Agent agent) {
        agent.initialize(new Goal(new Predicate("init", new Term[] {})));   // ← initial !init() 的翻译
    }
    public static void main(String[] args) {                   // ← 编译器自动加的 main()
        Scheduler.setStrategy(new TestSchedulerStrategy());    // ① 设定调度策略
        ...
        String name = java.lang.System.getProperty("astra.name", "main");  // ④ agent 名字
        astra.core.Agent agent = new Main().newInstance(name);
        agent.initialize(new Goal(new Predicate("main", new Term[] { argList })));  // ② 给 initial goal !main(list args)
        Scheduler.schedule(agent);                             // ③ 丢进调度器
    }
}
```

**三个关键结论（考点）**

1. **`+!main(list args)` 不是人写的，是编译器自动加的**——所以写 agent 时既可用 `initial !init();`（如本地 `Main.astra`），也可直接 `rule +!main(list args) { ... }`（如课件 HelloWorld），两者等价。
2. **`[main]` 这个输出前缀**就是 agent 实例名，来自 `astra.name` 属性（默认 `main`）。
3. ASTRA 运行时是**纯 Java**，所以可任意添加 Java 依赖、在任意支持 Maven 的 IDE（IDEA / Eclipse / VS Code 插件）里开发。

---

## 二、目录约定（slide 30）

| 内容 | 位置 |
|---|---|
| **ASTRA 源码** | `/src/main/astra`（**不是** Java 惯例的 `src/main/java`） |
| **支撑用 Java 代码**（写 module 的 `@ACTION`/`@SENSOR`） | `/src/main/java` |
| 构建文件 | `pom.xml` |
| 生成的 Java | `target/gen/java` |
| 编译产物 | `target/classes` |

**包（package）规则**：`src/main/astra` 下 = **默认包（default package）**；`src/main/astra/soccer/Defender.astra` 属于 `soccer` 包，**必须在文件头写 `package soccer;`**。

---

## 三、pom.xml 最小骨架：为什么可以这么短

**本地 `hello/pom.xml` 全文只有 20 行，连 `<build>` 段都没有**：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>examples</groupId>
    <artifactId>hello</artifactId>
    <version>1.0.0</version>
    <parent>
        <groupId>com.astralanguage</groupId>
        <artifactId>astra-base</artifactId>
        <version>1.0.7</version>
        <relativePath></relativePath>     <!-- 空标签 = 别去本地找，强制从 Maven Central 拉 -->
    </parent>
    <properties>
        <astra.main>Main</astra.main>     <!-- 启动类 -->
        <astra.name>hiby</astra.name>     <!-- agent 实例名（会出现在输出前缀） -->
    </properties>
</project>
```

原因：`astra-base` 的父 POM 里已经**替子工程配好了一切**（查看 Maven Central 上的 `astra-base-1.0.7.pom`）：

| 父 POM 干的事 | 内容 |
|---|---|
| `<packaging>pom</packaging>` | 只是「模板父工程」，不产出 jar |
| `<dependencies>` | 自动带上 `astra-core`、`astra-interpreter`、`astra-apis`、`astra-compiler` |
| `<build><plugins>` | 默认激活 `astra-maven-plugin` → 子工程不用再声明 |
| `<pluginManagement>` 绑定阶段 | `astra:compile` → **compile 阶段**；`astra:testCompile` → test-compile；`astra:test` → test |

> 所以 **`mvn compile` 就已经触发 ASTRA 编译**（先生成 Java 再 javac），不需要单独敲 `astra:compile`。这就是 slide 30「Compiling ASTRA: mvn compile」的原理。

---

## 四、两个属性

| 属性 | 含义 | 默认 |
|---|---|---|
| **`astra.main`** | 用哪个 agent 当**启动类** | `Main` |
| **`astra.name`** | 第一个 agent 的**实例名**（输出前缀） | `main` |

---

## 五、⚠️ `mvn astra:deploy` 到底干什么（最容易误解）

Maven 标准生命周期里 `deploy` 阶段是「**上传 artifact 到远程仓库**」。但 `astra:deploy` 是**插件目标（plugin goal）**，含义完全不同——**编译并运行 agent**：

- 它**假定**默认包（`src/main/astra`）下有一个叫 `Main` 的 agent 程序作为启动类；
- 可用 `astra.main` 属性覆盖。

```bash
mvn clean compile astra:deploy        # 官方给出的完整写法
mvn                                    # 若 pom 里配了 defaultGoal，可直接 mvn
mvn astra:deploy -Dastra.main=Namey -Dastra.name=George
```

预期输出（官方文档原文）：

```
[INFO] --- astra-maven-plugin:0.1.0:deploy (default-cli) @ hello ---
[main]Hello World, ASTRA
```

改成 `-Dastra.name=George` 后：

```
[George]Hello World, George
```

---

## 六、archetype 命令逐参数解释（slide 32）

```bash
mvn archetype:generate \
  -DarchetypeGroupId=com.astralanguage \
  -DarchetypeArtifactId=astra-archetype \
  -DarchetypeVersion=2.0.13 \
  -DgroupId=examples \
  -DartifactId=light-me \
  -Dversion=0.1.0
```

`archetype`（原型模板）是 Maven 的「项目骨架生成器」：`archetype*` 三个参数指明**用哪个模板**，`groupId/artifactId/version` 是新工程的**坐标**。生成后自动带上 `src/main/astra`、`src/main/java`、`pom.xml`，`astra.main` 默认指向 `Main`。

---

## 七、易踩的坑

| 坑 | 说明 |
|---|---|
| **版本号不一致** | slide 30 说 `2.0.13`；老奴查过 Maven Central：**1.0.7 与 2.0.13 都存在**（2.0.13 是最新一档）；但本地 `hello/pom.xml` 用 **1.0.7**，`ASTRA-Maven.pptx` 也写 1.0.7——**别混用父 POM 与插件版本** |
| **`mvn deploy` ≠ `mvn astra:deploy`** | 前者是 Maven 生命周期阶段（上传仓库），后者是 ASTRA 插件的运行目标 |
| **`astra:deploy` 假定 `Main.astra`** | 启动类不叫 `Main` 就必须设 `astra.main`，否则报找不到 |
| **包声明** | `src/main/astra/soccer/Defender.astra` 必须写 `package soccer;` |
| **JDK 要求** | 官方文档写 JDK 1.8+、Maven 3.3+（本机为 Java 17，编译老版 1.0.7 可能踩兼容问题） |
| **生成代码别改** | `target/gen/java/**` 头一行写着 `GENERATED CODE - DO NOT CHANGE`，改了下次编译被覆盖 |

---

## 八、一键速查表

```bash
mvn compile                    # 编译：.astra → .java → .class
mvn astra:compile              # 只跑 ASTRA 编译器（等同 compile 阶段里的那步）
mvn clean compile astra:deploy # 编译 + 运行（官方推荐的一条龙）
mvn astra:deploy -Dastra.main=Namey -Dastra.name=George   # 换启动类 + 换实例名
```

## 九、考点清单

- [ ] `.astra → .java → .class` 的编译链，`target/gen/java` 与 `target/classes`。
- [ ] 编译器**自动生成 `main()`**，并给出 `!main(list args)` 初始目标。
- [ ] `[agentName]` 输出前缀来自 `astra.name`。
- [ ] ASTRA 源码在 `src/main/astra`；Java 代码在 `src/main/java`。
- [ ] 父 POM `astra-base` 已配好依赖 + 插件 + 阶段绑定 ⇒ 子工程 pom 可以极简。
- [ ] **`mvn compile` 即触发 ASTRA 编译**（`astra:compile` 绑在 compile 阶段）。
- [ ] `astra.main` / `astra.name` 两个属性的作用与默认值。
- [ ] **`astra:deploy` = 运行 agent**（不是上传仓库）。
- [ ] archetype 生成工程的命令与参数含义。
- [ ] 包规则：默认包 vs 命名包（要写 `package`）。
