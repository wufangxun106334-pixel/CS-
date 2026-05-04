# Java 期末复习整合版

> 合并来源：`15 补充知识点合辑.md`、`16 字符串不可变性 & String Pool 详解.md`、`22 补充知识点（代码检查补全）.md`、`快速参考卡.md`、`题目详细解析.md`。
>
> 整理原则：同一知识点只保留最完整的一版；速查表保留结论，详细解析保留推导，补充专题保留代码 Demo。

---

## 目录

- [[#一、考前速查]]
- [[#二、基础与控制流易错点]]
- [[#三、字符串、常量池与对象复用]]
- [[#四、补充代码模式]]
- [[#五、样卷 Q1-Q40 详细解析]]
- [[#六、综合复习框架]]

---

# 一、考前速查

## 📋 40道题快速答案表

| 题号 | 答案 | 题号 | 答案 | 题号 | 答案 | 题号 | 答案 |
|------|------|------|------|------|------|------|------|
| 1 | **c** | 11 | **d** | 21 | **c** | 31 | **b** |
| 2 | **d** | 12 | **e** | 22 | **b** | 32 | **c** |
| 3 | **b** | 13 | **c** | 23 | **d** | 33 | **e** |
| 4 | **c** | 14 | **e** | 24 | **b** | 34 | **e** |
| 5 | **c** | 15 | **d** | 25 | **b** | 35 | **e** |
| 6 | **a** | 16 | **e** | 26 | **e** | 36 | **c** |
| 7 | *无确定答案* | 17 | **d** | 27 | **b** | 37 | **c** |
| 8 | **c** | 18 | **a** | 28 | **e** | 38 | **d** |
| 9 | **a** | 19 | **d** | 29 | **e** | 39 | **d** |
| 10 | **d** | 20 | **a** | 30 | **c** | 40 | **c** |

---

## 🔄 最容易混淆的对比表

### 1️⃣ 值传递 vs 参考传递 (Q20)

```
❌ 错误理解: Java有"引用传递"
✅ 正确理解: Java只有值传递

对基本类型的参数:
  void modify(int x) { x = 100; }  → 对方法内x的修改不影响外部
  
对对象参数的理解:
  void modify(MyObject obj) { 
      obj.field = 100;     // ✅ 有效 - 修改对象内容
      obj = new MyObject(); // ❌ 无效 - 只修改参数引用，不影响外部,like ojb=new int{1,2,3}
  }
  
为什么? 传的是引用的复制(值)，不是引用本身
```

### 2️⃣ 强制转换 (int)value (Q4)

```
❌ (int)11.9 会四舍五入为12
✅ (int)11.9 直接舍去小数得到11

(int)11.0252175 * 100 / 100
= 1102 / 100
= 11  ← 整数除法

11.0252175 * 100 / 100  
= 11.0252175  ← 浮点圆整
```

### 3️⃣ 前缀 vs 后缀递增 (Q13)

```
i++;  先返回i的当前值，后递增
++j;  先递增，后返回新值

int i = 0;
int j = i++;    // j=0, i=1
int k = ++j;    // j=1, k=1
```

### 4️⃣ While vs Do-While (Q10)

```
While: 先判断后执行(可能不执行)
Do-While: 先执行后判断(至少执行一次)

i = 1
while (i <= 2) {        // do-while也是
    i++;
    sout("Java");
}
// i=1: 1<=2 ✓ → i=2, Print
// i=2: 2<=2 ✓ → i=3, Print  
// i=3: 3<=2 ✗ → 退出
// 打印2次
```

### 5️⃣ Switch Fall-Through (Q14)

```
❌ 常见错误: 认为case之间相互独立
✅ 正确理解: 没有break会向下执行

switch(ch) {
    case 'c': k = 2;      // ← 匹配！
    default: k += 4;      // ← 继续执行(Fall-Through)
}
// 结果: k = 2 + 4 = 6
```

### 6️⃣ 数组索引边界检查顺序 (Q15)

```
❌ for (int i = 0; a[i] != value && i < a.length; i++)
   // 先访问a[i]，可能越界！

✅ for (int i = 0; i < a.length && a[i] != value; i++)
   // 先检查边界，再访问元素
   // 利用 && 的短路求值
```

### 7️⃣ 方法签名的定义 (Q1)

```
❌ 方法签名 = 返回类型 + 方法名 + 参数
✅ 方法签名 =          方法名 + 参数列表
  method signature = method name + parameter list
返回类型不是签名的一部分！
这导致：不能仅因返回类型不同而重载

public int add(int x, int y) { }
public double add(int x, int y) { }  
// ❌ 编译错误！签名相同
```

### 8️⃣ 引用赋值 vs 对象复制 (Q24)

```
Circle c1 = new Circle(1.0);
Circle c2 = new Circle(5.0);
c2 = c1;

内存图:
c1 ──┐
     └→ Circle(1.0)
c2 ──┴→ Circle(1.0)  
        Circle(5.0) 成为垃圾

这是引用赋值，不是deep copy
```

### 9️⃣ 基本类型 vs 引用类型 (Q25)

```
基本类型(int, boolean, char等):
  存储: 实际值
  赋值: 复制值
  内存: 通常在栈

引用类型(Object, Array, String等):
  存储: 对象的引用(地址)  ← 不是对象本身！
  赋值: 复制引用
  内存: 对象在堆，引用在栈/堆

❌ 引用类型直接存储对象
✅ 引用类型存储对象的地址
```

### 🔟 向上转型 vs 向下转型 (Q30, Q32)

```
向上转型(Upcasting): 子→父 ✓ 自动
Object obj = new Circle(5.0);  // ✓ 编译通过

向下转型(Downcasting): 父→子 ✗ 需要强制
Circle c = obj;                // ❌ 编译错误
Circle c = (Circle) obj;       // ✓ 必须显式转型

运行时可能抛出ClassCastException
if (obj instanceof Circle) {
    Circle c = (Circle) obj;   // 安全做法
}
```

### 1️⃣1️⃣ 抽象类 vs 接口 (Q35, Q37, Q39)

```
              抽象类      接口
构造方法      ✓          ✗
实现方法      ✓          ✓(Java 8+)
被继承        ✓ 单继承   (无)
被实现        (无)       ✓ 多实现
作为数据类型  ✓          ✓ (都能!)
扩展/实现     extends    implements

❌ 接口不能用作数据类型
✅ 接口也能用作引用类型!
✅ 抽象类实现接口/继承抽象类时，可以暂时不实现全部抽象方法；普通类必须全部实现。
```

### 1️⃣2️⃣ 静态变量 vs 实例变量 (Q26)

```
静态变量:
  - 属于类，不属于实例
  - 所有实例共享一个副本
  - 在类加载时初始化(一次)
  - ClassName.staticVar 访问
  
实例变量:
  - 属于每个实例
  - 每个实例有一个副本
  - 在对象创建时初始化
  - obj.instanceVar 访问

class A {
    static int count = 0;      // 所有实例共享
    int id;                    // 每个实例独有
    
    A() { count++; }
}
A a1 = new A();  // count = 1
A a2 = new A();  // count = 2
```

### 1️⃣3️⃣ This vs Super (Q22, Q23)

```
this: 当前对象本身
super: 父类

用途:
- this.field: 区分参数和字段
- this(): 调用其他构造方法
- super(): 调用父类构造方法
- super.method(): 调用父类方法

class Car {
    private String model;
    
    public Car(String model) {
        this.model = model;  // ✓ 正确
        model = model;       // ❌ 没有意义
    }
}
```

### 1️⃣4️⃣ For循环变量作用域 (Q16)

```
❌ for (int i = 0; i < 5; i++) { ... }
   System.out.println(i);  // 编译错误!

✅ int i;
   for (i = 0; i < 5; i++) { ... }
   System.out.println(i);  // 循环外可访问

for循环内的变量声明作用域仅限于循环体
```

### 1️⃣5️⃣ 整数运算 vs 浮点运算 (Q3)

```
2.0 + 3 * 5 / 2 / 5.0

执行过程:
相同优先级从左到右: 3 * 5 = 15
                  15 / 2 = 7.5  (有浮点数!)
                  7.5 / 5.0 = 1.5
                  2.0 + 1.5 = 3.5

关键: ❌ 15 / 2 = 7 (整数)
      ✅ 15 / 2 = 7.5 (有浮点数参与)
```

---

## 💾 高频知识点速记表

### 运算符优先级 (从高到低)
```
1. () [] . {}          // 括号、数组、点、代码块
2. ! ~ ++ -- + - (type)  // 逻辑非、按位非、递增/递减、强制转换
3. * / %              // 乘、除、取模
4. + -                // 加、减
5. << >> >>>          // 位移
6. < <= > >= instanceof // 关系与类型检查
7. == !=              // 相等比较
8. &                  // 按位与
9. ^                  // 按位异或
10. |                 // 按位或
11. &&                // 逻辑与
12. ||                // 逻辑或
13. ? :               // 条件(三元)
14. = += -= *= /= ... // 赋值
```

### Java关键字速记: 修饰符家族

```
访问修饰符:      public | protected | (package) | private
类修饰符:        abstract | final | strictfp
变量/方法修饰符: static | final | transient | volatile
继承相关:        extends | implements | super | this
```

### 循环关键语句

```
for (init; condition; update) { }
while (condition) { }
do { } while (condition);
for (type element : collection) { }  // Enhanced for

break;       // 退出最近的循环或switch
continue;    // 跳到下一次迭代
```

### 方法定义规则

```
[访问修饰符] [返回类型] 方法名(参数列表) [throws异常] {
    方法体
}
[access modifier] [return type] methodName(parameter list) [throws exceptions] {
    method body
}
**signature** 只包括 methodName(parameter list)
示例:
public int add(int x, int y) { return x + y; }
public static void main(String[] args) { }
public abstract void process(int value);
private synchronized void criticalSection() { }
```

---

## 🧠 记忆技巧(Mnemonics)

### Java五大关键点

```
APRIM:
A - Abstract类可以部分实现接口(A类不用全实现)
P - Polymorphism通过向上转型实现
R - Reference类型(类、数组)按引用传递值(简单变量值不能传递出方法,复杂变量可以传出)
I - Interface设计优先(多实现)
M - Method签名只有名字和参数(没返回类型)
```

### 循环陷阱记忆法

```
"ESB" - Empty Statement Break
E - Empty statement: for(...);  ← 分号导致循环体为空
S - Scope: for(int i=...) 循环外i不可用
B - Break position: case后必须有break,不然fall-through
```

### 类型转换简易判断

```
向上转型(子→父): ✅ 自动安全
向下转型(父→子): ⚠️ 需要显式强制转型
               可能ClassCastException

内存法则:
- 栈: 基本类型和引用
- 堆: 对象实体
```

---

## 📝 考前30分钟速过清单

### 必须复习的5个概念
- [ ] 参数传递：值传递vs对象引用值
- [ ] 引用赋值：指向同一对象的后果
- [ ] 数组越界：边界检查顺序
- [ ] 循环执行：初始→判断→执行→更新
- [ ] 强制转换：(int)舍弃小数，不四舍五入

### 必须做的5道题
- [ ] Q20 参数传递
- [ ] Q4 类型转换
- [ ] Q15 数组边界
- [ ] Q30 向下转型
- [ ] Q35 抽象类实现接口

### 必须避免的5个陷阱
- [ ] Q9 空语句` for(...);`
- [ ] Q16 for循环变量作用域
- [ ] Q18 字符串与字符拼接
- [ ] Q32 必须显式强制转型
- [ ] Q14 记住fall-through

---

**最后提醒**: 
- ⏰ 60分钟做40题 ≈ 1.5分钟/题，建议1分钟/题，预留查看
- 📌 不会的先跳过，最后再做逻辑题(Q30-Q40)
- ✍️ 按标记答案: a b c d e 用5个字母格式

祝考试顺利! 🎯

---

# 二、基础与控制流易错点

## 1. printf 格式化输出

```java
System.out.printf("整数 %d, 浮点 %.2f, 字符串 %s\n", 42, 3.14159, "Java");
// 输出：整数 42, 浮点 3.14, 字符串 Java
```

| 格式符 | 用途 | 示例 |
|--------|------|------|
| `%d` | 整数 | `printf("%d", 42)` |
| `%f` | 浮点 | `printf("%f", 3.14)` |
| `%.nf` | n位小数 | `printf("%.3f", 3.14159)` → `3.142` |
| `%s` | 字符串 | `printf("%s", "Hi")` |
| `%c` | 字符 | `printf("%c", 'A')` |

⚠️ `printf` **不自动换行**，需手动加 `\n`。

---

## 2. 浮点数精度 & Epsilon 比较

```java
System.out.println(1.0 - 0.9);
// 0.09999999999999998（不是 0.1！）

// ✅ 用 epsilon 比较
final double EPSILON = 1E-14;
if (Math.abs(x - 0.5) < EPSILON)
    System.out.println("约等于");
```

⚠️ 永远不要用 `==` 比较浮点数！

---

## 3. 字符串与数字互转

```java
int i = Integer.parseInt("123");
double d = Double.parseDouble("3.14");
String s1 = String.valueOf(100);
String s2 = Integer.toString(200);
```

---

## 4. 转义字符

| 序列 | 含义 |
|------|------|
| `\n` | 换行 |
| `\t` | Tab |
| `\\` | 反斜杠 |
| `\"` | 双引号 |

---

## 5. Math 类常用方法

| 方法 | 示例 |
|------|------|
| `pow(a,b)` | `pow(2,3)` → `8.0` |
| `sqrt(x)` | `sqrt(4)` → `2.0` |
| `abs(x)` | `abs(-2.1)` → `2.1` |
| `ceil(x)` | `ceil(10.1)` → `11.0` |
| `floor(x)` | `floor(10.9)` → `10.0` |
| `round(x)` | `round(9.5)` → `10` (返回 long) |
| `max(a,b)` / `min(a,b)` | 较大/较小值 |

⚠️ `round(-1.5)` → `-1`（向正无穷取整，非对称）

---

## 6. XOR 运算符 (`^`)

```java
true ^ true   → false
true ^ false  → true
```

典型考题："能被 2 或被 3 整除，但**不同时**被两者整除"

---

## 7. 短路求值 (Short-Circuit)

`&&` 左侧为 `false` → 右侧不求值
`||` 左侧为 `true` → 右侧不求值

```java
// ❌ 数组越界！a[i] 在 i<a.length 之前求值
for (int i = 0; a[i] != value && i < a.length; i++)

// ✅ 正确顺序
for (int i = 0; i < a.length && a[i] != value; i++)
```

---

## 8. continue 在 for vs while 中的差异

- **for** 中的 `continue` → 仍执行 `i++`（安全）
- **while** 中的 `continue` → 跳过 `i++`（容易死循环！）

```java
// ❌ while 中的 continue 陷阱
while (i < 4) {
    if (i % 3 == 0) { continue; }  // 跳过 i++ → 死循环！
    sum += i;
    i++;
}

// ✅ 修正
while (i < 4) {
    if (i % 3 == 0) { i++; continue; }
    sum += i;
    i++;
}
```

---

## 9. 分号陷阱

```java
for (i = 0; i < 10; i++);  // ← 空循环体！后面 { } 是独立块
{ System.out.println(i + 4); }  // 输出 14

while (i < 10);  // ← 死循环！
{ i++; }

do { ... } while (condition);  // ← 这个分号必须加！
```

---

## 10. 方法歧义调用

```java
max(int, double) 和 max(double, int)
→ max(1, 2) 调用哪个？歧义 → 编译错误！
```

---

## 11. 局部变量作用域

- for 循环头声明的变量，作用域仅限循环内
- 循环结束后不可见

```java
for (int i = 0; i < 5; i++) { ... }
System.out.println(i);  // ❌ 编译错误！
```

---

## 12. foreach 循环

```java
for (double x : arr) { sum += x; }
```

⚠️ foreach 不能**修改数组元素**、不能逆序、不能用索引。

---

## 13. Arrays.sort() & Arrays.toString()

```java
int[] arr = {6, 4, 1, 9};
Arrays.sort(arr);                            // [1, 4, 6, 9]
System.out.println(Arrays.toString(arr));     // [1, 4, 6, 9]
```

字符按 Unicode 排序：数字 < 大写 < 小写。

---

## 14. 线性搜索

```java
public static int linearSearch(int[] list, int key) {
    for (int i = 0; i < list.length; i++)
        if (key == list[i]) return i;
    return -1;
}
```

---

## 15. 数组拷贝陷阱

```java
int[] b = a;      // ❌ 拷贝引用，修改 b 影响 a,浅拷贝
int[] b = new int[a.length];
for (...) b[i] = a[i];  // ✅ 元素拷贝,深拷贝
```

---

## 16. 静态绑定 vs 动态绑定

|     | 静态绑定                     | 动态绑定     |
| --- | ------------------------ | -------- |
| 时机  | 编译时                      | 运行时      |
| 适用  | private/static/final/构造器 | 可覆盖的实例方法 |
| 依据  | 声明类型                     | 实际类型     |
static 方法看“引用类型/类名”
实例方法才看“实际对象类型”
static：看引用类型 / 类名，编译期决定了实际运行调用方法是引用类型
instance：先看引用类型能不能调用，运行时看实际对象类型

![[Pasted image 20260502213205.png]]
a.speak()
   |
   v
[编译器先检查]
   |
   +-- 看引用类型 Animal
   |
   +-- Animal 有 speak() ? ----否----> 编译错误
   |
   +-- 是
   |
   v
[程序运行时]
   |
   +-- 看真实对象 Dog
   |
   +-- Dog 有重写 speak() ? ----是----> 调 Dog.speak()
   |
   +-- 否
   |
   v
   调 Animal.speak()


---

## 17. private 方法不能覆盖  & static 方法只能隐藏
private 方法：不能 override
final 方法：不能 override
static 方法：不能 override
普通实例方法：可以 override

子类同名 private 方法 = 与新方法无关（不是覆盖）。
子类同名 static 方法 = 隐藏父类方法（调用看声明类型）:
```java
Parent p = new Child();
p.show();// call parent method
Child c = new Child();
c.show(); // call child method

```

`callShow()` 里调用的是父类自己的 `private show()`，不是子类的同名方法：

```java
class Parent {
    private void show() {
        System.out.println("Parent private show");
    }

    public void callShow() {
        show(); // 这里调用的是 Parent 自己的 private show()
    }
}

class Child extends Parent {
    public void show() {
        System.out.println("Child public show");
    }
}

Child c = new Child();
c.callShow(); // Parent private show
```

原因：

```text
private 方法只在当前类内部可见
Parent 里的 show() 只能被 Parent 自己调用
Child 里的 show() 是另一个新方法，不是 override
```

---

## 17.1 类内部方法调用规则

类内部不只可以调用 `static` 方法；普通实例方法、抽象方法也可以调用。

```java
class A {
    static void s() {
        System.out.println("static");
    }

    void m() {
        System.out.println("instance");
    }

    void test() {
        s(); // 调用静态方法
        m(); // 调用实例方法，等价于 this.m()
    }
}
```

但在 `static` 方法里没有 `this`，所以不能直接调用实例方法：

```java
class A {
    void m() {}

    static void test() {
        // m(); // 编译错误：static context 中没有 this
        A a = new A();
        a.m(); // 必须通过对象调用
    }
}
```

`static` 方法也不能直接访问实例字段，因为实例字段属于对象，不属于类本身：

```java
class A {
    int x = 10;

    static void test() {
        // System.out.println(x); // 编译错误
        A a = new A();
        System.out.println(a.x); // 必须先有对象
    }
}
```

抽象方法也可以在抽象类内部被调用，运行时执行子类实现：

```java
abstract class Animal {
    abstract void sound();

    void test() {
        sound(); // 实际执行子类的 sound()
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("woof");
    }
}
```

速记：

```text
实例方法中：static / 普通实例方法 / 抽象方法 都能直接调
static 方法中：只能直接调 static；调实例方法必须先有对象
```

---

## 18. 构造函数链

子类构造器必须先调父类构造器（隐式 `super()` 或显式）。
执行顺序：最顶层父类 → 逐级向下。

---

## 19. protected 关键字

`protected` = 同包 + 任何包的子类可访问。

---

## 20. equals() 5 步覆盖模式

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;                // 1. 同一引用
    if (obj == null) return false;               // 2. null 检查,喂了后续能正常转型
    if (getClass() != obj.getClass()) return false; // 3. 类型检查
    MyClass other = (MyClass) obj;               // 4. 安全转型
    return this.field == other.field;            // 5. 字段比较
}
```

⚠️ 参数必须是 `Object`，不能用子类类型（否则不是覆盖）！

---

## 21. Checked vs Unchecked 异常

| | Checked | Unchecked |
|---|---|---|
| 父类 | Exception (非 RuntimeException) | RuntimeException |
| 强制处理 | ✅ | ❌ |
| 代表 | IOException | NullPointerException |
| 处理方式 | try-catch 或 throws | 编码时预防（不用 try-catch） |

---

## 22. finally 总是执行

即使 try 中有 `return`，finally 也会先执行。

---

## 23. 接口隐式修饰符

- 变量 → `public static final`
- 方法 → `public abstract`

接口中的方法默认是 `public abstract`，不能有方法体。
如果要在接口里写带方法体的方法，必须显式声明为 `default` 或 `static`：

```java
interface Test {
    void run();              // 等价于 public abstract void run();

    default void log() {     // 有方法体，必须 default
        System.out.println("log");
    }

    static void help() {     // 有方法体，必须 static
        System.out.println("help");
    }
}
```

接口可 `extend` 多个接口。抽象类可以没有抽象方法。

---

---

# 三、字符串、常量池与对象复用

## 1. String 的不可变性 (Immutability)

String 对象一旦创建，**内容不可更改**。

```java
String s = "Java";
s = "HTML";  // s 指向了新对象，原来的 "Java" 没变（等待 GC）
```

### 为什么 String 拼接效率低

```java
// ❌ 低效：每次 `+` 都创建新的 String 对象
String result = "";
for (int i = 0; i < 1000; i++) {
    result += "x";  // 每次循环创建新 String！
}

// ✅ 高效：用 StringBuilder (append)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("x");  // 在原对象上修改
}
String result = sb.toString();
```

---

## 2. String Constant Pool (字符串常量池)

Java 对字符串字面量使用**常量池**来节省内存。

```java
String s1 = "Java";           // 放入常量池
String s2 = "Java";           // 复用常量池中的同一个对象
System.out.println(s1 == s2); // true（同一对象！）

String s3 = new String("Java");  // 强制在堆上创建新对象
	System.out.println(s1 == s3);    // false（不同对象！）
	System.out.println(s1.equals(s3)); // true（内容相同）
```

### 关键结论

| 比较方式 | 比较内容 | 何时用 |
|---------|---------|--------|
| `==` | 引用地址 | 基本不用 |
| `.equals()` | 字符串内容 | **始终用这个** |

```java
String a = "hello";
String b = "hel" + "lo";  // 编译器优化 → "hello"，常量池
System.out.println(a == b);  // true！

String c = "hel";
String d = c + "lo";       // 运行时拼接 → 在堆上创建新对象
System.out.println(a == d);  // false！
```

---

## 3. StringBuilder 完整方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `append(s)` | 尾部追加 | `sb.append("World")` |
| `insert(idx, s)` | 指定位置插入 | `sb.insert(0, "Hello ")` |
| `deleteCharAt(idx)` | 删除指定字符 | `sb.deleteCharAt(0)` |
| `delete(start, end)` | 删除子串 | `sb.delete(0, 3)` |
| `reverse()` | 反转字符串 | `sb.reverse()` |
| `replace(start, end, s)` | 替换子串 | `sb.replace(0, 2, "XY")` |
| `setCharAt(idx, ch)` | 设置指定位置字符 | `sb.setCharAt(0, 'A')` |
| `toString()` | 转为 String | `sb.toString()` |
| `length()` | 长度 | `sb.length()` |

### Demo

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");         // "Hello World"
sb.insert(5, " Java");       // "Hello Java World"
sb.replace(0, 5, "Hi");      // "Hi Java World"
sb.delete(0, 3);             // "Java World"
sb.reverse();                // "dlroW avaJ"
String result = sb.toString();
```

### String vs StringBuilder 选择

| 场景 | 用什么 |
|------|--------|
| 字符串不常变化 | `String`（可享常量池优化） |
| 频繁拼接/修改 | `StringBuilder`（性能好） |
| 多线程环境 | `StringBuffer`（线程安全版） |

---

## 4. String 创建方式对比总结

```java
// 方式 1：字面量（推荐，用常量池）
String s1 = "Hello";

// 方式 2：new（强制堆分配，不推荐）
String s2 = new String("Hello");

// 方式 3：从 char 数组
char[] chars = {'H', 'i', ' ', 'A', 'l'};
String s3 = new String(chars);  // "Hi Al"
```

### 易错点

- `s1 == s2` 只能判断**是否同一个对象**（内存地址），不能判断内容是否相同
- 内容比较用 `equals()`
- 忽略大小写比较用 `equalsIgnoreCase()`
- 字面量拼接 (编译时常量) 走常量池；变量拼接 (运行时) 在堆上创建新对象

---

## 5. String 变体题模板

这类题常考三件事：

```text
1. 字面量拼接 vs 变量拼接
2. `==` 比地址，`equals()` 比内容
3. `intern()` 是否回到常量池
```

### 基础模板

```java
String a = "ab";
String b = "a" + "b";
String c = new String("ab");

System.out.println((a == b) + " " + (a == c) + " " + a.equals(c));
```

输出：

```text
true false true
```

原因：

```text
"a" + "b" 是编译期常量拼接，b 进入常量池
new String("ab") 一定创建堆上的新对象
equals() 比较内容，不比地址
```

### 常见变体 1：变量拼接

```java
String a="ab"
String x = "a";
String y = x + "b";

System.out.println(a == y);      // false
System.out.println(a.equals(y)); // true
```

`x + "b"` 是运行时拼接，通常会产生新对象。

### 常见变体 2：`intern()`

```java
String d = y.intern();

System.out.println(a == d);      // true
System.out.println(a == y);      // false
```

`intern()` 返回字符串常量池里的引用。

### 常见变体 3：`final` 参与常量折叠

```java
final String p = "a";
String q = p + "b";

System.out.println(a == q); // true
```

`final` 字符串在编译期可被折叠，常走常量池。

### 常见变体 4：包装类也有缓存陷阱

```java
Integer m = 100;
Integer n = 100;
System.out.println(m == n); // true，缓存范围内

Integer u = 200;
Integer v = 200;
System.out.println(u == v); // false，通常超出缓存范围
```

### 速记

```text
字面量 + 字面量：编译期合并，进常量池
变量 + 字面量：运行时拼接，常是新对象
new String(...)：一定是新对象
intern()：去常量池拿引用
```

---

## 包装类缓存、数组与普通对象对比

这一节只补 `String Pool` 之外的对象复用规则，避免和前面的字符串常量池重复。

```text
String: 有字符串常量池
Integer/Long/Short/Byte/Character/Boolean: 有包装类缓存
Double/Float: 没有常用缓存
数组: 没有常量池，每次创建新对象
普通对象: 没有常量池，每次 new 新对象
```

### 包装类缓存

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // true，-128 到 127 范围内通常复用缓存对象

Integer x = 200;
Integer y = 200;
System.out.println(x == y); // false，超出缓存范围，通常是不同对象
```

| 类型 | 缓存范围 |
|------|----------|
| `Byte` | 全部缓存 |
| `Short` | `-128` 到 `127` |
| `Integer` | `-128` 到 `127` |
| `Long` | `-128` 到 `127` |
| `Character` | `0` 到 `127` |
| `Boolean` | `true` / `false` |

`Double` 和 `Float` 没有这种常用缓存：

```java
Double d1 = 1.0;
Double d2 = 1.0;

System.out.println(d1 == d2);      // false
System.out.println(d1.equals(d2)); // true
```

### 数组和普通对象不会进常量池

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};

System.out.println(a == b);              // false，不同数组对象
System.out.println(a.equals(b));         // false，数组默认 equals 仍比地址
System.out.println(Arrays.equals(a, b)); // true，逐个比较元素
```

普通对象也是每次 `new` 一个新对象：

```java
Circle c1 = new Circle(1.0);
Circle c2 = new Circle(1.0);

System.out.println(c1 == c2); // false
```

速记：

```text
基本类型 == 比值
引用类型 == 比地址
String.equals() 比内容
数组比内容用 Arrays.equals()
```


---

# 四、补充代码模式

## 缺口 5：`java.util.Random` 随机数类

笔记仅有 `Math.random()`，但课程代码大量使用 `Random` 类。

### 基本用法

```java
import java.util.Random;

Random rnd = new Random();       // 无种子：每次运行结果不同
int x = rnd.nextInt(26);         // 返回 [0, 26) 范围内的随机 int
double d = rnd.nextDouble();     // 返回 [0.0, 1.0) 范围内的随机 double
boolean b = rnd.nextBoolean();   // 返回 true 或 false
int any = rnd.nextInt();         // 返回任意 int（含负数）
```

### 带种子构造器

```java
Random rnd = new Random(42);     // 带种子：每次运行结果相同（可复现，便于调试）
// 种子为 42，每次生成的随机数序列完全一致
```

### Demo：随机生成小写字母

```java
Random rnd = new Random(1);
char[] chars = new char[20];
for (int i = 0; i < chars.length; i++) {
    int offset = rnd.nextInt(26);          // [0, 26)
    chars[i] = (char) ('a' + offset);     // int → char 强制转型
}
```

### `Math.random()` vs `Random` 对比

| | `Math.random()` | `Random` 对象 |
|---|---|---|
| 返回类型 | `double` [0.0, 1.0) | 多种（`nextInt`, `nextDouble`, `nextBoolean`）|
| 范围控制 | `(int)(Math.random() * n)` | 直接 `rnd.nextInt(n)` |
| 可设种子 | ❌ | ✅ `new Random(seed)` |
| 批量生成 | 每次调用静态方法 | 同一对象反复调用 |
| 性能 | 每次都要初始化 | 复用对象更高效 |

### 易错点

- `nextInt(n)` 返回范围是 **`[0, n)`**，不包括 n
- 不带参数的 `nextInt()` 返回任意 `int`（含负数），**不是** `[0, n)`
- 设种子主要是为了**调试可复现**，正常使用不要传种子

---

## 缺口 6：`ex.printStackTrace()` + 多异常 throws

### `printStackTrace()`

```java
try {
    int result = divide(x, y);
} catch (ArithmeticException ex) {
    ex.printStackTrace();  // 打印完整异常栈到控制台
}
```

输出内容示例：

```
java.lang.ArithmeticException: Divisor cannot be zero
    at lec_12_examples_1.Divide3.divide(Divide3.java:7)
    at lec_12_examples_1.Divide3.main(Divide3.java:16)
```

### `toString()` vs `printStackTrace()` vs `getMessage()`

| 方法 | 输出内容 | 用途 |
|------|---------|------|
| `ex.toString()` | 异常类型 + 消息 | 简洁信息 |
| `ex.getMessage()` | 仅消息字符串 | 提取关键信息 |
| `ex.printStackTrace()` | 异常类型 + 消息 + **完整调用栈** | 调试定位 |

```java
catch (ArithmeticException ex) {
    System.out.println(ex.toString());      // java.lang.ArithmeticException: Divisor cannot be zero
    System.out.println(ex.getMessage());    // Divisor cannot be zero
    ex.printStackTrace();                   // 完整栈（调试用）
}
```

### 方法签名声明多个异常

可以在 `throws` 后面列出多个异常类型，用逗号分隔：

```java
public static void badCode() throws ArithmeticException, IndexOutOfBoundsException {
    System.out.println(0 / 1);        // 不会抛异常（1/0 才会）
    int[] array = new int[]{1, 2};
    System.out.println(array[3]);     // 可能抛 IndexOutOfBoundsException
}
```

调用方可以只 catch 其中一个：

```java
try {
    badCode();
} catch (IndexOutOfBoundsException e) {
    System.out.println("数组越界：" + e);
}
// ArithmeticException 未被捕获，如果抛出则继续向上传播
```

### 易错点

- `printStackTrace()` 是 **void**，不返回字符串，不能用于拼接输出
- 多个异常在 `throws` 中可以是 Checked 和 Unchecked 混用
- catch 多个异常时，子类异常必须在前（否则编译错误）

---

## 缺口 7：`String.format()` — 格式化字符串

`printf` 直接输出到控制台，`String.format()` **返回格式化后的字符串**，可用于拼接、toString 等。

### 基本用法

```java
// printf：直接输出
System.out.printf("Value: %.2f\n", 3.14159);

// String.format：返回字符串
String s = String.format("Value: %.2f", 3.14159);
// s = "Value: 3.14"
```

### 格式符与 printf 完全相同

| 格式符 | 含义 | 示例 |
|--------|------|------|
| `%d` | 整数 | `String.format("%d", 42)` → `"42"` |
| `%f` | 浮点数 | `String.format("%f", 3.14)` → `"3.140000"` |
| `%.2f` | 保留 2 位小数 | `String.format("%.2f", 3.14159)` → `"3.14"` |
| `%s` | 字符串 | `String.format("%s", "Hi")` → `"Hi"` |
| `%c` | 字符 | `String.format("%c", 'A')` → `"A"` |
| `%n` | 平台换行 | 同 `\n` |

### 常见用途 1：在 `toString()` 中格式化输出

```java
@Override
public String toString() {
    return String.format("Car [Brand: %s, Model: %s, Year: %d]", brand, model, year);
    // 输出：Car [Brand: Nissan, Model: bx4t, Year: 2020]
}
```

```java
@Override
public String toString() {
    return id + ", " + title + ", " + department + ", " + String.format("%.2f", salary);
    // 输出：1, Manager, Sales, 50000.00
}
```

### 常见用途 2：构造带格式的中间字符串

```java
String res = String.format("%.2f", getPower());
return getBrand() + ", " + res + ", " + speedLevel;
// 输出：Dyson, 50.00, 3
```

### `printf` vs `String.format()` 对比

| | `System.out.printf()` | `String.format()` |
|---|---|---|
| 作用 | 格式化并输出到控制台 | 格式化并返回字符串 |
| 返回值 | `void`（无返回值） | `String` |
| 自动换行 | ❌ 需手动加 `\n` | ❌ 需手动加 `\n` |
| 用于 toString | ❌ 不能 | ✅ 最常用 |
| 格式符语法 | 完全相同 | 完全相同 |

### 易错点

- `String.format()` **不会自动换行**
- 与 `printf` 使用完全相同的格式语法
- 常用于 `toString()`、`return` 语句中，不适合打印场景

---

## 缺口 8：`char[]` 频率计数模式（Bucket Pattern）

将字符映射为数组索引，实现 O(1) 计数。这是数组最经典的算法模式之一。

### 核心思路

```java
char[] chars = ...;              // 源数据：小写字母数组
int[] counts = new int[26];      // 26 个计数桶，默认全 0

for (int i = 0; i < chars.length; i++) {
    int idx = chars[i] - 'a';    // 'a'→0, 'b'→1, ..., 'z'→25
    counts[idx]++;               // 对应桶计数 +1
}
```

### 过程演示

```java
// 假设 chars = {'c', 'a', 'b', 'a', 'c', 'c'}
// counts 变化：
// 'c' → idx=2 → counts[2]++ → [0,0,1,0,...]
// 'a' → idx=0 → counts[0]++ → [1,0,1,0,...]
// 'b' → idx=1 → counts[1]++ → [1,1,1,0,...]
// 'a' → idx=0 → counts[0]++ → [2,1,1,0,...]
// 'c' → idx=2 → counts[2]++ → [2,1,2,0,...]
// 'c' → idx=2 → counts[2]++ → [2,1,3,0,...]
```

### 反向映射：索引 → 字符

```java
for (int i = 0; i < counts.length; i++) {
    System.out.println((char)(i + 'a') + ": " + counts[i]);
    // 0+'a'='a', 1+'a'='b', ..., 25+'a'='z'
}
```

### 完整 Demo：随机生成 20 个小写字母并统计频率

```java
import java.util.Random;

public class CountLettersInArray {
    public static void main(String[] args) {
        char[] chars = createArray(20);
        displayArray(chars);
        int[] counts = countLetters(chars);
        displayCounts(counts);
    }

    public static char[] createArray(int n) {
        char[] chars = new char[n];
        Random rnd = new Random(1);
        for (int i = 0; i < chars.length; i++) {
            chars[i] = (char)('a' + rnd.nextInt(26));
        }
        return chars;
    }

    public static int[] countLetters(char[] chars) {
        int[] counts = new int[26];
        for (int i = 0; i < chars.length; i++) {
            counts[chars[i] - 'a']++;
        }
        return counts;
    }

    public static void displayCounts(int[] counts) {
        for (int i = 0; i < counts.length; i++) {
            System.out.println((char)(i + 'a') + ": " + counts[i]);
        }
    }

    public static void displayArray(char[] chars) {
        for (char c : chars) System.out.print(c + " ");
        System.out.println();
    }
}
```

### 扩展：其他字符集

```java
// 大写字母：offset = 'A'
int idx = chars[i] - 'A';

// 数字字符：offset = '0'
int digit = chars[i] - '0';    // '5' → 5

// 任意 ASCII：直接用 int 值做索引（需要更大的数组）
int[] ascii = new int[256];
ascii[chars[i]]++;
```

### 易错点

- `chars[i] - 'a'` 的前提是字符必须是 'a'–'z'，不包含大写或非字母
- counts 数组的默认值全是 0，不需要手动初始化
- 映射和反向映射的公式要配对：`idx = c - 'a'` ↔ `c = (char)(idx + 'a')`

---

## 缺口 9：对象数组模式

数组的元素可以是**引用类型**（对象）。每个元素存的是**引用**，不是对象本体。

### 创建对象数组

```java
Circle[] circleArray = new Circle[10];  // 创建数组，元素全为 null
```

此时数组有 10 个空位，每个位置存的是 `null`，**还没有任何 Circle 对象**。

### 逐个初始化

```java
for (int i = 0; i < circleArray.length; i++) {
    circleArray[i] = new Circle(i + 1);  // 半径 1, 2, ..., 10
}
```

### 对象数组作为方法参数

```java
public static double sumArea(Circle[] circleArray) {
    double sum = 0;
    for (int i = 0; i < circleArray.length; i++) {
        sum += circleArray[i].getArea();  // 通过引用调方法
    }
    return sum;
}

// 调用
Circle[] circles = new Circle[10];
for (int i = 0; i < circles.length; i++) {
    circles[i] = new Circle(i + 1);
}
double total = sumArea(circles);
```

### 内存模型理解

```
circleArray (引用) → [ ref0 ][ ref1 ][ ref2 ] ... [ ref9 ]
                         ↓       ↓       ↓            ↓
                      Circle  Circle  Circle  ...  Circle
                      (r=1)   (r=2)   (r=3)        (r=10)
```

- `circleArray` 存的是数组对象的引用
- 数组的每个元素是 `Circle` 对象的引用
- 通过 `circleArray[i].getArea()` 沿引用链两次跳转

### `Circle[]` vs `Circle` 参数传递

```java
public static void changeRadius(Circle c) {
    c.setRadius(100);          // 修改对象内容 → 影响外部
}

public static void sumArea(Circle[] arr) {
    arr[0].getArea();          // 通过数组访问对象的方法
}
```

两者本质上一样：都是传引用副本，副本指向同一个对象。

### 易错点

- `Circle[] arr = new Circle[10];` 后每个元素是 `null`，**不能直接调用方法**
- 必须逐个 `arr[i] = new Circle(...)` 初始化后才可使用
- 传对象数组进方法，修改 `arr[i]` 的内容会影响外部（和普通对象传参相同）

---

## 缺口 10：杂项知识点合集

### 10.1 多级继承（三层继承）

```java
// 基类
public class zoo {
    private String name;
    private int id;
    public void shout() { System.out.println("the animal can shout"); }
}

// 中间层
public class animal extends zoo {
    private String type;
    public void swim() { System.out.println("the " + type + " can swim"); }
    public void fly()  { System.out.println("the " + type + " can fly"); }
}

// 叶子层
public class bird extends animal {
    @Override
    public void shout() { System.out.println("the bird chirps"); }
    @Override
    public void swim()  { System.out.println("the bird can swim a little"); }
    @Override
    public void fly()   { System.out.println("the bird can fly"); }
}

// 多态使用
animal bird1 = new bird();
bird1.shout();  // the bird chirps（动态绑定）
bird1.fly();    // the bird can fly（继承自 animal，被 bird 覆盖）
```

关键点：
- `bird` 继承 `animal`，`animal` 继承 `zoo` → 三层继承链
- 覆盖可以在任意层发生
- 多态支持跨层级的动态绑定

---

### 10.2 枚举 + switch 实用案例

```java
enum Membership {
    NORMAL, SILVER, GOLD, DIAMOND
}

public static void mealCalculation(Membership memberType, int mealPrice) {
    int payment;
    switch (memberType) {
        case GOLD:
        case DIAMOND:
            payment = mealPrice * 8 / 10;   // 80折
            break;
        case SILVER:
            payment = mealPrice * 9 / 10;   // 90折
            break;
        case NORMAL:
        default:
            payment = mealPrice;             // 原价
            break;
    }
    System.out.printf("Member: %s, Price: $%d, Pay: $%d%n",
                      memberType, mealPrice, payment);
}
```

关键点：
- `case GOLD` 和 `case DIAMOND` 共享同一代码块（fall-through 的合理使用）
- `switch` 表达式的类型是 `Membership` 枚举
- 枚举与 `switch` 结合是期末常见考题

---

### 10.3 模式打印（嵌套循环）

```java
// 打印数字三角形（n=4）：
//    1
//   1 2
//  1 2 3
// 1 2 3 4

int n = 4;
for (int i = 1; i <= n; i++) {            // 行
    for (int k = 1; k <= n - i; k++)       // 前导空格
        System.out.print("  ");
    for (int j = 1; j <= i; j++)           // 数字
        System.out.print(j + " ");
    System.out.println();
}
```

关键模式：
- 外层循环控制行数
- 内层循环 1：打印空格（数量递减）
- 内层循环 2：打印内容（数量递增）
- 边界计算：`n - i` 是第 i 行的空格数

---

### 10.4 `File.createNewFile()` 和文件操作补充

```java
import java.io.File;
import java.io.IOException;

File file = new File("test.txt");
// 此时只是 Java 对象，磁盘上没有实际文件

if (file.createNewFile()) {           // 需要在 try-catch 中
    System.out.println("File created: " + file.getName());
} else {
    System.out.println("File already exists.");
}

```
```java
import java.io.File;
import java.io.IOException;

try {
	File file = new File("test.txt");
	file.createNewFile();
}catch(IOException o){
	e.printStackTree();
}
```

**`createNewFile()` 返回值**：
- `true`：文件不存在，成功创建
- `false`：文件已存在，没有创建新文件

**补充 File 类方法**：

| 方法 | 说明 | 返回类型 |
|------|------|---------|
| `file.lastModified()` | 最后修改时间戳 | `long`（毫秒）|
| `new Date(file.lastModified())` | 转为可读日期 | `Date` |
| `file.isHidden()` | 是否隐藏文件 | `boolean` |
| `file.mkdir()` | 创建目录 | `boolean` |
| `file.delete()` | 删除文件/目录 | `boolean` |

---

### 10.5 包私有类（默认访问级别）

类本身也可以没有 `public` 修饰符：

```java
// CircleWithStaticMembers.java — 类名前无 public
class CircleWithStaticMembers {
    double radius;                        // 包私有字段
    static int numberOfObjects = 0;

    CircleWithStaticMembers() { ... }     // 包私有构造器
    double getArea() { ... }              // 包私有方法
    static int getNumberOfObjects() { ... }
}
```

规则：
- 没有 `public` 的类——**包私有类**（package-private / default）
- 只能被**同包的**其他类访问
- 一个 `.java` 文件最多有一个 `public` 类，但可以有多个包私有类
- 包私有类的字段/方法默认也是包私有

```java
// 同包内可以访问
CircleWithStaticMembers c = new CircleWithStaticMembers();
c.radius = 10;              // ✅ 同包可访问包私有字段
c.numberOfObjects = 100;    // ✅ 同包可访问包私有静态字段

// 不同包 ❌ 无法访问
```

### 10.6 一个文件包含多个非 public 类

```java
// TestAnimal.java — public 类只有一个
public class TestAnimal {
    public static void main(String[] args) { ... }
}

// 以下类和 TestAnimal 在同一文件，但都不是 public
abstract class Animal { ... }
class Chicken extends Animal { ... }
class Tiger extends Animal { ... }
class snake extends Animal { ... }
class bird extends Animal { ... }
```

规则：
- 文件名必须与 `public` 类名一致（`TestAnimal.java`）
- 其他非 public 类可以出现在同一文件中
- 这些类只能被同包访问（包私有）
- 实际使用中不推荐放太多类在一个文件（可读性差），但考试有可能出现

---

---

# 五、样卷 Q1-Q40 详细解析

## 第一部分：基础语法与表达式求值 (Q1-Q7)

### Q1: 方法签名定义

**题目**: 在Java中，方法的签名(signature)包括什么？

**选项分析**:
- a. 仅方法名 ❌
- b. 仅参数列表 ❌
- c. **方法名和参数列表** ✅ **[正确答案]**
- d. 返回类型、方法名和参数列表 ❌
- e. 以上都不是 ❌

**详细解释**:
Java中方法的签名由**方法名**和**参数列表**组成，不包括返回类型。返回类型不是签名的一部分。
- 正确示例：`void calculate(int x, double y)` 的签名是 `calculate(int, double)`
- 返回类型不同但签名相同的方法会导致编译错误（因为方法重载基于签名）

**知识点位置**: Java基础 → 方法定义与重载
**易混淆点**: 
- ⚠️ 返回类型不是方法签名的一部分
- ⚠️ 不能因为只改变返回类型而重载方法

---

### Q2: Void方法的应用场景

**题目**: 以下哪一项最适合声明为void方法？

**选项分析**:
- a. 转换小写字母为大写 - 需要返回转换后的字符 ❌
- b. 返回0到99之间的随机整数 - 需要返回值 ❌
- c. 检验一个数字是否为偶数 - 需要返回布尔值 ❌
- d. **打印0到99之间的随机整数** ✅ **[正确答案]**
- e. 以上都不是 ❌

**详细解释**:
Void方法不返回任何值，用于执行操作而不需要返回结果。打印操作正是这样的场景。

**知识点位置**: Java基础 → 方法返回类型设计
**易混淆点**:
- ⚠️ void vs 返回值的区分：如果需要结果供后续使用，就不能用void
- ⚠️ 打印操作是副作用(side effect)，与返回值是两个概念

---

### Q3: 浮点数表达式求值

**题目**: 表达式 `2.0 + 3 * 5 / 2 / 5.0` 的值是多少？

**步骤求解**:
```
2.0 + 3 * 5 / 2 / 5.0
= 2.0 + (3 * 5 / 2 / 5.0)    // 乘除同级，从左到右
= 2.0 + (15 / 2 / 5.0)
= 2.0 + (7.5 / 5.0)           // 注意：15/2 = 7.5（因为有浮点数）
= 2.0 + 1.5
= 3.5
```

**答案**: b. 3.5 ✅

**知识点位置**: Java基础 → 算术运算符优先级与浮点运算
**易混淆点**:
- ⚠️ 整数除以浮点数得到浮点结果
- ⚠️ 乘除法优先级相同，从左向右计算
- ⚠️ `3 * 5 / 2` 中，如果都是整数得15/2=7(舍入)，但这里有浮点数影响全局

---

### Q4: 类型转换与舍入

**题目**: 表达式 `(int)(11.0252175 * 100) / 100` 的值是多少？

**步骤求解**:
```
(int)(11.0252175 * 100) / 100
= (int)(1102.52175) / 100
= 1102 / 100           // 强制转换为int，直接舍去小数部分
= 11                   // 整数除法
```

**答案**: c. 11 ✅

**关键区别**:
- `(int)(11.0252175 * 100) / 100` → 先转int再除 → 结果是11
- `11.0252175 * 100 / 100` → 浮点运算 → 结果是11.0252175

**知识点位置**: Java基础 → 类型转换与强制转换
**易混淆点**:
- ⚠️ (int)强制转换会直接舍去小数部分，不是四舍五入
- ⚠️ 操作顺序很重要：先转换再运算 vs 先运算再转换
- ⚠️ 整数除以整数得整数，100不被转换回浮点

---

### Q5: 三元运算符返回值类型

**题目**: 表达式 `('a' <= 'b') ? 1 : 0` 的值是多少？

**步骤求解**:
```
('a' <= 'b') ? 1 : 0
= (true) ? 1 : 0       // 'a'(97) <= 'b'(98) 是true
= 1                     // 返回1
```

**答案**: c. 1 ✅

**知识点位置**: Java基础 → 条件(三元)运算符
**易混淆点**:
- ⚠️ 三元运算符返回的是操作数(1或0)，不是布尔值
- ⚠️ 虽然条件部分是布尔值，但返回值类型由两个操作数决定
- ⚠️ 此处返回的是int类型的1，不是布尔值true

---

### Q6: 布尔逻辑表达式求值

**题目**: 给定变量，求 `a && !b || c && !(a && b)` 的值
```java
boolean a = true;
boolean b = false;
boolean c = true;
```

**步骤求解**:
```
a && !b || c && !(a && b)

// 先计算每个部分
= true && !false || true && !(true && false)
= true && true || true && !(false)
= true || true && true
= true || true
= true
```

**答案**: a. true ✅

**优先级记忆**: `&&` (AND) > `||` (OR)

**知识点位置**: Java基础 → 布尔逻辑与运算符优先级
**易混淆点**:
- ⚠️ `!` (非) 优先级最高，`&&` 优先级高于 `||`
- ⚠️ 不要忘记先处理括号
- ⚠️ 短路求值(short-circuit)：`||`前为true则后不计算，但此处需要完全计算

---

### Q7: 复利公式的编程实现

**题目**: 复利公式 $A = P(1 + \frac{r}{100k})^{nk}$ 的哪个表达式**不正确**？

```java
double A, P = 100.0;
int r = 10, n = 10, k = 12;
```

**选项分析**:
- a. `A = P * Math.pow(1 + 0.01 * r / k, n * k);` ✅ 正确
  - `0.01` 是浮点字面量 → 强制浮点运算
  - `0.01 * 10 / 12 = 0.1 / 12 ≈ 0.00833` = $\frac{r}{100k}$
  
- b. `A = P * Math.pow(1.0 + r * 0.01 / k, (n * k));` ✅ 正确
  - `1.0` 是浮点字面量 → 强制浮点运算
  - `10 * 0.01 / 12 = 0.1 / 12 ≈ 0.00833` = $\frac{r}{100k}$
  
- c. **`A = P * Math.pow(1 + r / 100 / k, n * k);` ❌ 不正确**
  - 🔴 **整数除法陷阱！**
  - `10 / 100 = 0` (都是int，整数除法舍去小数!)
  - `0 / 12 = 0`
  - 结果: `1 + 0 = 1` ❌ 完全错误
  
- d. `A = P * Math.pow(1.0 + r / 100.0 / k, n * k);` ✅ 正确
  - `100.0` 是浮点字面量 → 强制浮点运算
  - `10 / 100.0 = 0.1`，再 `0.1 / 12 ≈ 0.00833` = $\frac{r}{100k}$

**正确答案**: c ✅

**关键知识点**：整数除法的陷阱
```
10 / 100 = 0     (两个int相除，向下取整，舍去小数)
10 / 100.0 = 0.1 (有浮点参与，结果是浮点数)
```

**知识点位置**: Java基础 → 数学运算、类型转换、Math库
**易混淆点**:
- ⚠️ 整数与浮点数混合运算的类型转换
- ⚠️ 浮点数精度问题
- ⚠️ 除法运算顺序对结果的影响(虽然结合律在理论成立，但精度不同)

---

## 第二部分：循环与控制流 (Q8-Q11)

### Q8: 循环死循环分析

**题目**: 以下代码结束时n的值是多少？
```java
int n;
for (n = 0; n <= 2; n--)
    n += 3;
```

**执行跟踪**:
```
初始: n = 0
第1次: n <= 2? (0 <= 2) ✓ → n += 3 → n = 3 → n-- → n = 2
第2次: n <= 2? (2 <= 2) ✓ → n += 3 → n = 5 → n-- → n = 4
第3次: n <= 2? (4 <= 2) ✗ → 退出循环
最终: n = 4
```

**答案**: c. 4 ✅

**知识点位置**: Java基础 → for循环执行顺序
**易混淆点**:
- ⚠️ for循环的执行顺序：初始 → 判断 → 执行体 → 更新
- ⚠️ `n--`在循环结束时执行，在下一轮判断前
- ⚠️ 即使条件为false，循环变量已经被修改

---

### Q9: 空语句的陷阱

**题目**: 以下代码的输出是什么？
```java
int i = 0;
for (i = 0; i < 10; i++);  // ← 注意这里有分号！
System.out.println(i + 4);
```

**执行分析**:
- `for (i = 0; i < 10; i++);` 循环体是空语句`;`
- 循环执行：i从0到10（当i=10时条件i<10失败，退出）
- 最后i的值是10
- 输出：`10 + 4 = 14`

**答案**: a. 14 ✅

**知识点位置**: Java基础 → 循环陷阱、空语句
**易混淆点**:
- ⚠️ `for(...);` 中的`;`是空语句，循环体为空
- ⚠️ 循环仍然执行，只是每次什么都不做
- ⚠️ 最常见的bug之一：不小心加了分号
- ⚠️ `System.out.println(i + 4);` 不在循环体内，i的最终值是10

---

### Q10: Do-While循环执行

**题目**: 以下代码打印"Java"多少次？
```java
int i = 1;
do {
    i++;
    System.out.println("Java");
} while (i <= 2);
```

**执行跟踪**:
```
初始: i = 1
第1次: i++ → i = 2 → 打印"Java" → 判断 i <= 2? (2 <= 2) ✓ → 继续
第2次: i++ → i = 3 → 打印"Java" → 判断 i <= 2? (3 <= 2) ✗ → 退出
```

**答案**: d. 2次 ✅

**关键区别**: do-while vs while
- do-while：先执行后判断（至少执行一次）
- while：先判断后执行（可能不执行）

**知识点位置**: Java基础 → do-while循环
**易混淆点**:
- ⚠️ 初始值i=1，判断条件i<=2
- ⚠️ 第一次执行后i变成2，仍满足条件
- ⚠️ 第二次执行后i变成3，不满足条件，退出
- ⚠️ 与while循环的关键区别是执行时机

---

### Q11: While循环的逻辑条件

**题目**: 以下代码做什么？
```java
Scanner input = new Scanner(System.in);
int n = input.nextInt();
while (n < 30 || n > 90)
    n = input.nextInt();
```

**逻辑分析**:
- 条件：`n < 30 || n > 90`（n小于30或大于90）
- 当条件为true时继续循环，当条件为false时退出
- 循环退出条件：`!(n < 30 || n > 90)` → `n >= 30 && n <= 90`
- 即：**继续获取输入直到n在[30, 90]范围内**

**答案**: d. 继续获取输入直到输入在[30, 90]区间内（包含边界） ✅

**知识点位置**: Java基础 → 循环条件逻辑、德摩根定律
**易混淆点**:
- ⚠️ `||` (或) 的否定是 `&&` (与)
- ⚠️ `<` 的否定是 `>=`，`>` 的否定是 `<=`
- ⚠️ 容易混淆"继续条件"和"退出条件"

---

## 第三部分：基础数据操作 (Q12-Q14)

### Q12: 整数除法与取模运算

**题目**: 以下代码的输出是什么？
```java
int i = 2;
int j = i / 10 + i % 10;
int p = 22;
int q = p / 10 + p % 10;
System.out.println(j + " " + q);
```

**执行计算**:
```
i = 2:
  j = 2 / 10 + 2 % 10
  j = 0 + 2
  j = 2

p = 22:
  q = 22 / 10 + 22 % 10
  q = 2 + 2
  q = 4

输出: 2 4
```

**答案**: e. 2 4 ✅

**知识点位置**: Java基础 → 整数运算(除法、取模)
**易混淆点**:
- ⚠️ 整数除法向下取整：2/10=0，22/10=2
- ⚠️ 取模运算得到余数：2%10=2，22%10=2
- ⚠️ 此模式(除以10的商 + 除以10的余数)用于分离数字各位

---

### Q13: 前缀与后缀递增操作

**题目**: 以下代码的输出是什么？
```java
int i = 0;
int j = i++;    // 后缀递增
int k = ++j;    // 前缀递增
System.out.println(i + " " + j + " " + k);
```

**执行跟踪**:
```
初始: i = 0

j = i++;  
    → j被赋予i的当前值0
    → i在之后递增为1
    → 结果：i=1, j=0

k = ++j;
    → j先递增为1
    → k被赋予递增后的值1
    → 结果：j=1, k=1

输出: 1 1 1
```

**答案**: c. 1 1 1 ✅

**知识点位置**: Java基础 → 前缀(++)与后缀(++)递增运算符
**易混淆点**:
- ⚠️ `i++` (后缀)：先返回原值，再递增
- ⚠️ `++j` (前缀)：先递增，再返回新值
- ⚠️ 涉及赋值语句时区别明显

---

### Q14: Switch语句与Fall-Through

**题目**: 以下代码后k和ch的值是什么？
```java
int k = 3;
char ch = 'b';
switch (++ch) {              // ch先递增为'c'
    case 'a': k = 0; break;
    case 'b': k += 1; break;
    case 'c': k = 2;         // 匹配！但没有break
    default: k += 4;         // Fall-through！
}
```

**执行跟踪**:
```
k = 3, ch = 'b'
++ch → ch = 'c'
switch('c'):
    case 'c': k = 2;         ← 匹配
              (没有break，继续执行下一个case!)
    default: k += 4;         ← Fall-through执行
             k = 2 + 4 = 6
```

**最终**: k = 6, ch = 'c'

**答案**: e. 6 and c ✅

**知识点位置**: Java基础 → Switch语句、Fall-Through、Break
**易混淆点**:
- ⚠️ switch语句必须使用break来阻止fall-through
- ⚠️ 没有break会继续执行后续case
- ⚠️ default不一定在末尾，fall-through也会影响它
- ⚠️ `++ch` 是前缀递增，立即生效

---

## 第四部分：数组操作 (Q15-Q19)

### Q15: 数组边界与短路求值

**题目**: 以下代码的输出是什么？
```java
int[] a = {1, 2, 3, 4, 5};
int value = 0;
for (int i = 0; a[i] != value && i < a.length; i++)
    System.out.print(a[i] + " ");
System.out.println(6);
```

**执行跟踪**:
```
i = 0: a[0] != 0? (1 != 0) ✓ && 0 < 5? ✓ → 打印1，输出"1 "
i = 1: a[1] != 0? (2 != 0) ✓ && 1 < 5? ✓ → 打印2，输出"2 "
i = 2: a[2] != 0? (3 != 0) ✓ && 2 < 5? ✓ → 打印3，输出"3 "
i = 3: a[3] != 0? (4 != 0) ✓ && 3 < 5? ✓ → 打印4，输出"4 "
i = 4: a[4] != 0? (5 != 0) ✓ && 4 < 5? ✓ → 打印5，输出"5 "
i = 5: a[5] != 0? 数组越界！
```

**关键**: 条件顺序很重要！应该是 `i < a.length && a[i] != value`（先检查边界）

此题中 `a[i] != value && i < a.length` 先访问a[i]再检查边界，会抛出ArrayIndexOutOfBoundsException

**答案**: d. 运行时错误发生 ✅

**知识点位置**: Java基础 → 数组、索引界限、短路求值的重要性
**易混淆点**:
- ⚠️ 条件顺序很关键：应该**先检查索引有效性**
- ⚠️ `&&` 的短路求值：只有第一个条件为false时才不计算第二个
- ⚠️ 正确写法：`i < a.length && a[i] != value`
- ⚠️ 此题测试对顺序的理解

---

### Q16: 变量作用域与数组访问

**题目**: 以下代码的输出是什么？
```java
int[] x = {1, 2, 3, 4, 5};
System.out.print("[ ");
for (int i = 0; i < 5; i++) {
    if (x[i] > 3)
        break;
}
System.out.printf("%d ", x[i]);  // i是什么值？
System.out.println("]\n");
```

**执行分析**:
```
循环执行：
i = 0: x[0] = 1 > 3? ✗
i = 1: x[1] = 2 > 3? ✗
i = 2: x[2] = 3 > 3? ✗
i = 3: x[3] = 4 > 3? ✓ → break，i = 3

循环外访问i：编译错误！
因为i是for循环内的局部变量，循环外不可访问
```

**答案**: e. 编译错误 ✅

**知识点位置**: Java基础 → 变量作用域、For循环变量的生命周期
**易混淆点**:
- ⚠️ `for (int i = 0; ...)` 中声明的i作用域仅限于循环
- ⚠️ C语言中this would be different，但Java中i循环外不可用
- ⚠️ 如果要循环外使用i，需要在循环前声明

---

### Q17: 数组元素复制与遍历

**题目**: 以下代码的输出是什么？
```java
int[] a = {5, 4, 3, 2, 1, 0};
for (int i = a.length - 2; i >= 0; i--)
    a[i + 1] = a[i];           // 将a[i]复制到a[i+1]
for (int i: a)
    System.out.print(i + " ");
```

**执行跟踪**:
```
初始: a = {5, 4, 3, 2, 1, 0}
i = 4: a[5] = a[4] → a[5] = 1 → {5, 4, 3, 2, 1, 1}
i = 3: a[4] = a[3] → a[4] = 2 → {5, 4, 3, 2, 2, 1}
i = 2: a[3] = a[2] → a[3] = 3 → {5, 4, 3, 3, 2, 1}
i = 1: a[2] = a[1] → a[2] = 4 → {5, 4, 4, 3, 2, 1}
i = 0: a[1] = a[0] → a[1] = 5 → {5, 5, 4, 3, 2, 1}

输出: 5 5 4 3 2 1
```

**答案**: d. 5 5 4 3 2 1 ✅

**知识点位置**: Java基础 → 数组操作、循环里的数组修改
**易混淆点**:
- ⚠️ 数组操作要理解索引的含义
- ⚠️ 从后向前遍历时，后面的元素被修改后，之后的访问会用到新值
- ⚠️ 这是一个"数组左移"操作，第一个元素被复制到第二个位置

---

### Q18: 字符串操作与字符拼接

**题目**: 以下代码的输出是什么？
```java
String s1 = "hello";
String s2 = "";
for (int i = 0; i < s1.length(); i+=2)
    s2 = s1.charAt(i) + s2;    // 在s2前面插入字符
System.out.println(s2);
```

**执行跟踪**:
```
s1 = "hello" (长度5)
i = 0: s2 = 'h' + "" = "h"
i = 2: s2 = 'l' + "h" = "lh"
i = 4: s2 = 'o' + "lh" = "olh"
循环结束 (i=6 >= length)

输出: olh
```

**答案**: a. olh ✅

**知识点位置**: Java基础 → String操作、charAt()方法、字符串拼接
**易混淆点**:
- ⚠️ `s1.charAt(i)` 返回单个字符，与字符串拼接变成String
- ⚠️ 每次在s2的前面插入，所以顺序反向
- ⚠️ `i+=2` 表示跳过一个字符，提取索引0,2,4的字符

---

### Q19: 二维数组与引用赋值

**题目**: 以下代码的输出是什么？
```java
int[][] arr1 = new int[3][2];
int[] arr2 = {1, 2, 3};
arr1[1] = arr2;           // arr1的第二行指向arr2
arr1[1][0] = 10;          // 修改arr2[0]
arr2[1] = 20;             // 修改arr2[1]
for (int i: arr2)
    System.out.print(i + " ");
```

**执行跟踪**:
```
createorig arr2 = {1, 2, 3}
arr1[1] = arr2 → arr1[1]和arr2指向同一个数组
arr1[1][0] = 10 → arr2[0] = 10 → arr2 = {10, 2, 3}
arr2[1] = 20 → arr2[1] = 20 → arr2 = {10, 20, 3}

输出: 10 20 3
```

**答案**: d. 10 20 3 ✅

**知识点位置**: Java基础 → 数组作为引用类型、对象引用传递
**易混淆点**:
- ⚠️ 数组是引用类型，赋值是引用赋值，不是复制
- ⚠️ `arr1[1] = arr2` 后，两者指向同一数组
- ⚠️ 通过任意一个引用修改都会影响实际的数组数据
- ⚠️ 尤其要注意二维数组中的行可以是不同长度的数组

---

## 第五部分：方法与对象基础 (Q20-Q25)

### Q20: 参数传递与对象引用

**题目**: 以下代码的输出是什么？
```java
public static void main(String[] args) {
    int i = 11;
    boolean[] arr1 = new boolean[4];
    char[] arr2 = {'x', 'y', 'z'};
    foo(i, arr1, arr2);
    
    System.out.print(i + " ");
    System.out.print(arr1[0] + " " + arr1[1] + " ");
    System.out.println(arr2[0]);
}

public static void foo(int i, boolean[] b, char[] c) {
    i = 222;                    // 修改参数i（不影响原始变量）
    c = new char[4];            // 创建新数组赋给c（不影响原始arr2）
    b[0] = true;                // 修改数组内容（影响原始arr1）
    c[0] = 'a';                 // 修改新数组（不影响arr2）
}
```

**执行分析**:
- Java中基本类型是值传递，修改参数不影响原始变量
- Java中对象是引用传递：
  - 修改引用本身(赋新对象)不影响原始引用
  - 修改引用指向的对象内容会影响

**最后状态**:
- `i = 11` (值传递，foo中的修改不影响)
- `arr1[0] = true`, `arr1[1] = false` (修改了数组内容)
- `arr2[0] = 'x'` (foo中的c指向新数组，不影响arr2)

**答案**: a. 11 true false x ✅

**知识点位置**: Java中级 → 参数传递、值传递vs引用传递
**易混淆点**:
- ⚠️ **重要**: Java没有真正的"引用传递"，只有值传递
- ⚠️ 对象参数传递的是引用的复制(值)
- ⚠️ 修改参数变量本身(赋新引用)不影响原始变量
- ⚠️ 但修改引用指向对象的内容会影响
- ⚠️ 这是很多人混淆的关键点

---

### Q21: 类的基本定义

**题目**: 以下哪个正确定义了一个名为Student的Java类？

**选项分析**:
- a. `public Student { private int id; }` ❌ 没有关键字class
- b. `public Student() { private int id; }` ❌ Student()是构造方法，不能这样定义
- c. **`public class Student { private int id; }`** ✅ 正确
- d. `public class Student() { private int id; }` ❌ 类定义后不能有()
- e. None ❌

**答案**: c ✅

**知识点位置**: Java中级 → 类的定义语法
**易混淆点**:
- ⚠️ 类定义必须使用class关键字
- ⚠️ 类名不能带括号
- ⚠️ 不要混淆类定义和构造方法定义

---

### Q22: This关键字的作用

**题目**: Java中this关键字的作用是什么？

**选项分析**:
- a. 指向超类 ❌ 用super指向超类
- b. **指向当前调用的对象** ✅ 
- c. 指向静态成员 ❌
- d. 用于创建新实例 ❌ 用new关键字创建
- e. None ❌

**答案**: b ✅

**知识点位置**: Java中级 → This关键字
**易混淆点**:
- ⚠️ this代表当前对象实例(当前方法被调用的对象)
- ⚠️ super代表父类
- ⚠️ 主要用于区分变量名和参数名

---

### Q23: This关键字的实际应用

**题目**: 以下构造方法应该用哪个语句初始化data field model？
```java
public class Car {
    private String model;
    public Car(String model) {
        ... // 填空
    }
}
```

**选项分析**:
- a. `model = model;` ❌ 两个都是参数，赋值无意义
- b. `Car.model = model;` ❌ model是实例变量，不是类变量
- c. `super.model = model;` ❌ Car没有超类(或超类没有model)
- d. **`this.model = model;`** ✅ 左边是实例变量，右边是参数
- e. None ❌

**答案**: d ✅

**知识点位置**: Java中级 → This关键字解决命名冲突
**易混淆点**:
- ⚠️ 当参数名与实例变量名相同时，必须用this区分
- ⚠️ `this.model` 指实例变量，`model` 指参数
- ⚠️ 不使用this会导致参数被赋值给自己

---

### Q24: 对象引用赋值

**题目**: 以下操作后c1和c2指向什么？
```java
Circle c1 = new Circle(1.0);
Circle c2 = new Circle(5.0);
c2 = c1;
```

**执行分析**:
```
创建两个不同对象: c1→Circle(1.0), c2→Circle(5.0)
c2 = c1;  → c2的引用改为指向c1所指向的对象
最终: c1和c2都指向Circle(1.0)
      Circle(5.0)对象变成unreachable，被垃圾收集
```

**答案**: b. 两个都指向半径为1.0的Circle对象 ✅

**知识点位置**: Java中级 → 引用类型与对象
**易混淆点**:
- ⚠️ 赋值是引用赋值，不是对象复制
- ⚠️ 原来c2指向的对象不会被修改，而是被丢弃
- ⚠️ 这是垃圾回收的触发条件

---

### Q25: Java类型系统的正确理解

**题目**: 以下哪个关于Java基本类型和引用类型的说法**不正确**？

**选项分析**:
- a. 基本类型直接存储实际值 ✅ 正确
- b. **引用类型直接存储实际对象** ❌ **这是错的！** 引用类型存储的是对象的地址，不是对象本身
- c. 类是引用类型 ✅ 正确
- d. 原始类型不能直接调用方法 ✅ 正确
- e. 数组是引用类型 ✅ 正确

**答案**: b ✅

**关键区别**:
| | 基本类型 | 引用类型 |
|---|---|---|
| 存储内容 | 实际值 | **对象的地址/引用** |
| 内存位置 | 栈 | 对象在堆，引用可在栈 |
| 复制 | 复制值 | 复制引用 |

**知识点位置**: Java中级 → Java类型系统
**易混淆点**:
- ⚠️ 这是很多初学者的误解
- ⚠️ 引用类型存储的是**指针/地址**，不是对象本身
- ⚠️ 这就是为什么两个引用可以指向同一对象

---

## 第六部分：静态变量与方法 (Q26-Q29)

### Q26: 静态变量的特性

**题目**: 以下关于Java静态变量的说法，哪个**正确**？

**选项分析**:
- a. 静态变量属于类的一个实例 ❌ 属于类本身
- b. 静态变量不能被构造方法访问 ❌ 能访问
- c. 静态变量不能被实例方法访问 ❌ 能访问
- d. 静态变量只能被静态方法访问 ❌ 也能被实例方法访问
- e. **静态变量被类的所有实例共享** ✅

**答案**: e ✅

**关键特性**:
- 静态变量属于类，不属于实例
- 所有实例共享同一个静态变量
- 可以通过类名或实例访问（不推荐用实例）

**知识点位置**: Java中级 → 静态成员
**易混淆点**:
- ⚠️ 静态变量初始化一次，在类加载时
- ⚠️ 修改静态变量对所有实例都有影响
- ⚠️ 应该用类名访问静态变量（ClassName.staticVar）

---

### Q27: 高内聚类的设计

**题目**: 以下哪个最好地描述了高内聚(cohesive)类？

**选项分析**:
- a. 仅包含静态方法 ❌
- b. **将相关功能组织在一起** ✅
- c. 仅包含构造方法 ❌
- d. 能被多个类继承 ❌
- e. 所有私有变量和相应的getter/setter ❌ (这不是定义)

**答案**: b ✅

**内聚性的含义**:
- 强内聚：类的所有成员都紧密相关
- 弱内聚：类的成员关联不紧密

**知识点位置**: Java进阶 → 设计原则、类设计
**易混淆点**:
- ⚠️ 内聚性不等于私有/公共的组织
- ⚠️ 不等于继承关系
- ⚠️ 是类内部功能的相关程度

---

### Q28: Setter方法的正确实现

**题目**: 给定Account类，以下哪个**正确**实现了id的setter方法？
```java
public class Account {
    private int id;
}
```

**选项分析**:
- a. `public int getId() { return id; }` ❌ 这是getter，不是setter
- b. `public void setId(int id) { id = id; }` ❌ 没有用this，赋值无意义
- c. `public int setId() { return id; }` ❌ setter应该返回void
- d. `public void setId() { id = id; }` ❌ 没有参数，无法设置值
- e. **`public void setId(int id) { this.id = id; }`** ✅

**答案**: e ✅

**知识点位置**: Java中级 → 访问器/修改器模式
**易混淆点**:
- ⚠️ Setter方法返回void
- ⚠️ Setter必须有参数来接收新值
- ⚠️ 必须使用this.id避免混淆

---

### Q29: 构造方法的正确定义

**题目**: 以下哪个**正确**定义了Person类的构造方法？

```java
public class Person {
    private String name;
    private int age;
}
```

**选项分析**:
- a. 构造方法使用`Person.name = name;` ❌ 应该使用this
- b. `public void Person(...)` ❌ 构造方法不能有返回类型void
- c. `this.name = name;` 但是 `void` ❌ 同样的问题
- d. `public Person(...) { name = name; ... }` ❌ 没有this，赋值无意义
- e. **`public Person(String name, int age) { this.name = name; this.age = age; }`** ✅

**答案**: e ✅

**构造方法的规则**:
1. 必须与类名相同
2. 不能有返回类型(不是void，是没有)
3. 在创建对象时自动调用
4. 用于初始化对象的状态

**知识点位置**: Java中级 → 构造方法
**易混淆点**:
- ⚠️ 初学者容易添加void返回类型
- ⚠️ 必须使用this.field区分参数和字段
- ⚠️ 如果不定义，会有默认无参构造方法

---

## 第七部分：对象模型与类型系统 (Q30-Q35)

### Q30: 多态性与向上转型

**题目**: 以下代码是否正确？
```java
public class Example {
    private int x;
    public Example() { x = 5; }
    public int getX() { return x; }
}

public class TestExample {
    public static void main(String[] args) {
        Object obj = new Example();
        System.out.println(obj.getX());
    }
}
```

**分析**:
- `Object obj = new Example();` 是合法的(向上转型/upcasting)
- 但Object类没有getX()方法
- 编译器看obj是Object类型，找不到getX()方法
- 编译错误！

**答案**: c. 编译错误发生在TestExample类 ✅

**知识点位置**: Java中级 → 多态、向上转型、静态类型vs动态类型
**易混淆点**:
- ⚠️ 虽然运行时obj指向Example对象，但编译时是Object类型
- ⚠️ 编译器只看编译时类型(Object)，没有getX()
- ⚠️ 必须进行向下转型：`((Example)obj).getX()`
- ⚠️ 向下转型本身不改变对象；只改变“引用的看法”
- ⚠️ `Parent p = new Child(); Child c = (Child) p;` 是合法的，因为 p 实际指向 Child
- ⚠️ `Parent p = new Parent(); Child c = (Child) p;` 会在运行时报 `ClassCastException`

---

### Q31: 多态性的定义

**题目**: 以下哪个最好地描述了Java中的多态性？

**选项分析**:
- a. 方法能动态改变返回类型 ❌
- b. **对象能以超类或接口类型出现，并表现出子类特性** ✅
- c. 能在单个变量存储多个值 ❌
- d. 基本数据类型的存储方式 ❌
- e. Java代码压缩效率 ❌

**答案**: b ✅

**多态性的三个应用场景**:
1. 方法重载(编译时多态)
2. 方法重写(运行时多态/动态绑定)
3. 接口/抽象类实现

**知识点位置**: Java进阶 → 多态性、面向对象特性
**易混淆点**:
- ⚠️ 多态性 ≠ 运行时类型改变
- ⚠️ 是关于类型系统和方法绑定
- ⚠️ 核心是能用父类引用指向子类对象

---

### Q32: 类型转换与方法调用

**题目**: 以下代码是否正确？
```java
Object o = new Circle(5.0);
Circle c = o;  // ← 向下转型
System.out.println(c.getRadius());
```

**分析**:
```
Object o = new Circle(5.0);  // ✓ 向上转型，合法
Circle c = o;                 // ✗ Object类型赋给Circle
                              // 编译错误！需要强制转型
```

**正确方式**:
```java
Circle c = (Circle) o;  // 显式强制转型
```

**答案**: c. 编译错误发生 ✅

**知识点位置**: Java中级 → 向上/向下转型
**易混淆点**:
- ⚠️ 向上转型(子→父)：自动，合法
- ⚠️ 向下转型(父→子)：需要显式强制转型
- ⚠️ ClassCastException发生在运行时
- ⚠️ 强转不会把 Parent 对象变成 Child 对象；它只是在告诉编译器“把它当成 Child 看”

---

### Q33: 抽象类的特性

**题目**: 以下关于Java抽象类的说法，哪个**正确**？

**选项分析**:
- a. 抽象类能被实例化 ❌ 不能
- b. 抽象类不能用作数据类型 ❌ 能用(作为引用类型)
- c. 抽象类不能有构造方法 ❌ 能有(用于初始化子类)
- d. 抽象方法必须有方法体 ❌ 不能有
- e. **抽象类能包含抽象方法和具体方法** ✅

**答案**: e ✅

**抽象类的规则**:
- 包含至少一个抽象方法，就必须是抽象类
- 包含0个或多个抽象方法的class声明为abstract
- 不能被实例化，只能被继承
- 能有构造方法、静态方法、final方法

**知识点位置**: Java进阶 → 抽象类
**易混淆点**:
- ⚠️ 抽象类虽然不能实例化，但能作为数据类型
- ⚠️ 能有构造方法是重要的特性
- ⚠️ 混淆抽象类和接口的异同

---

### Q34: 抽象方法的声明

**题目**: 以下哪个**正确**声明了一个返回int、接收两个int参数的抽象方法max？

**选项分析**:
- a. `public abstract max(int x, int y);` ❌ 没有返回类型
- b. `public abstract void max(int x, int y);` ❌ 返回类型应该是int
- c. `public int max(int x, int y);` ❌ 没有abstract关键字，没有方法体(语法错)
- d. `public abstract int max(int x, int y) { }` ❌ 抽象方法不能有方法体
- e. **`public abstract int max(int x, int y);`** ✅

**答案**: e ✅

**抽象方法的语法**:
```java
public abstract ReturnType methodName(Parameters);
// - 必须有abstract
// - 必须在抽象类中
// - 没有方法体(终止于分号)
```

**知识点位置**: Java进阶 → 抽象方法
**易混淆点**:
- ⚠️ 抽象方法没有方法体
- ⚠️ 不能是final(不能同时用abstract和final)
- ⚠️ 只能在抽象类或接口中声明

---

### Q35: 抽象类实现接口的部分方法

**题目**: 以下代码是否正确？
```java
public interface Test {
    void p();
    void q();
}

public abstract class A implements Test {
    @Override
    public void p() {
        System.out.println("here");
    }
}
```

**分析**:
- 接口Test定义了两个抽象方法：p()和q()
- 类A是抽象类，实现了接口Test
- A实现了p()方法，但**没有实现q()方法**
- 由于A是抽象类，不需要实现所有方法
- 子类如果不是抽象类，则必须实现q()

**答案**: e. 代码可以编译 ✅

**关键规则**:
- **抽象类可以部分实现接口方法**
- 具体类必须实现所有接口方法
- 抽象类中未实现的接口方法仍然是抽象的

**同理：抽象类方法也可以继续不实现，但子类必须也是抽象类**:
```java
abstract class Parent {
    public abstract void run();
}

abstract class Child extends Parent {
    // 可以不实现 run()
}

class Dog extends Parent {
    @Override
    public void run() {
        System.out.println("Dog runs");
    }
}
```

**判断口诀**:
```text
普通类：必须实现所有继承来的抽象方法
抽象类：可以实现一部分，也可以不实现，继续留给子类
```

**知识点位置**: Java进阶 → 接口、抽象类、继承关系
**易混淆点**:
- ⚠️ 抽象类的特权：可以不完全实现接口
- ⚠️ 具体类没有这个特权
- ⚠️ 未实现的方法必须在具体子类中实现

---

## 第八部分：继承与类修饰符 (Q36-Q40)

### Q36: Final类的特性

**题目**: 以下关于Java final类的说法，哪个**正确**？

**选项分析**:
- a. Final类不能被实例化 ❌ 能被实例化
- b. Final类不能有静态方法 ❌ 能有
- c. **Final类不能被其他类继承** ✅
- d. Final类不能有final方法 ❌ 能有
- e. Final类不能有实例变量 ❌ 能有

**答案**: c ✅

**Final修饰符的作用**:
- `final class`: 不能被继承
- `final method`: 不能被重写
- `final variable`: 不能被修改

**知识点位置**: Java进阶 → Final修饰符
**易混淆点**:
- ⚠️ final class禁止继承，常见例子：String、Integer
- ⚠️ final method禁止重写，但实例方法
- ⚠️ 这些限制提供安全性和性能(不需要动态绑定)

---

### Q37: 接口与抽象类的区别

**题目**: 以下关于接口和抽象类的说法，哪个**不正确**？

**选项分析**:
- a. 接口不能定义构造方法，抽象类能 ✅ 正确
- b. 两者都不能用new直接实例化 ✅ 正确
- c. **接口不能用作数据类型，抽象类能** ❌ **这是错的！** 两者都能
- d. 两者都能用作数据类型 ✅ 正确

**答案**: c ✅

**接口vs抽象类对比**:
| 特性    | 接口 | 抽象类 |
|---|---|---|
| 实例化 | 否 | 否 |
| 作为数据类型 | **是** | **是** |
| 构造方法 | 否 | 是 |
| 有具体方法 | Java 8+ | 是 |
| 多继承 | 是(多接口) | 否(单继承) |

**知识点位置**: Java进阶 → 接口vs抽象类
**易混淆点**:
- ⚠️ 接口也能用作引用类型
- ⚠️ `InterfaceType ref = implementingObject;` 是合法的
- ⚠️ 不要混淆"能作为数据类型"和"能实例化"

---

### Q38: 逻辑错误(语义错误)

**题目**: 以下哪个是逻辑(语义)错误的例子？

**选项分析**:
- a. 使用未声明的变量 ❌ 编译错误
- b. 省略语句末尾的分号 ❌ 编译错误  
- c. 将字符串赋给int变量 ❌ 编译错误
- d. **使用了不正确的公式，导致输出错误** ✅ 逻辑错误
- e. 访问越界的数组元素 ❌ 运行时错误

**答案**: d ✅

**错误类型区分**:
| 类型 | 时机 | 原因 |
|---|---|---|
| 编译错误 | 编译 | 语法违规 |
| 运行时错误 | 运行 | 非法操作(如NPE) |
| **逻辑错误** | **程序运行** | **程序错误地实现要求** |

**知识点位置**: Java基础 → 调试、错误分类
**易混淆点**:
- ⚠️ 逻辑错误最难发现：程序运行但结果错误
- ⚠️ 需要理解需求和验证
- ⚠️ 举例：计算平均值时用了求和而非平均

---

### Q39: 类和接口的继承关系

**题目**: 以下关于Java类和接口的说法，哪个**不正确**？

**选项分析**:
- a. 类只能继承一个超类 ✅ 正确
- b. 类能实现多个接口 ✅ 正确
- c. 接口能扩展多个接口 ✅ 正确(Java允许)
- d. **接口只能扩展一个接口** ❌ **错误！接口能扩展多个接口**
- e. 接口能扩展其他接口但不能扩展类 ✅ 正确

**答案**: d ✅

**继承关系**:
```java
class A extends B { }                    // 单继承
class C extends B implements I1, I2 { }  // 单继承，多实现

interface I1 extends I2, I3 { }          // 接口可多重继承(扩展)
```

**知识点位置**: Java进阶 → 继承关系、接口设计
**易混淆点**:
- ⚠️ 类只能单继承
- ⚠️ 接口能多重继承
- ⚠️ "实现接口"和"扩展接口"是不同的概念

---

### Q40: 关系模型in面向对象设计

**题目**: 以下关于面向对象关系的说法，哪个**正确**？

**选项分析**:
- a. 聚合模型表示is-a关系 ❌
- b. 组合模型表示is-a关系 ❌
- c. **继承模型表示is-a关系** ✅
- d. 继承模型表示has-a关系 ❌
- e. 关联模型表示is-a关系 ❌

**答案**: c ✅

**OOP关系模型**:
| 关系 | 含义 | 示例 |
|---|---|---|
| **继承(is-a)** | 子类是父类的一种 | Student is-a Person | Inheritance
| **组合(has-a)** | 整体包含部分(强关系) | Car has-a Engine | Composition
| **聚合(has-a)** | 整体包含部分(弱关系) | Team has-a Player | Aggregation
| **关联(uses-a)** | 两个类有关联 | Student uses-a Book | Association

**知识点位置**: Java进阶 → 面向对象分析与设计
**易混淆点**:
- ⚠️ 是-有关系的区分
- ⚠️ 组合vs聚合：谁拥有所有权
- ⚠️ 继承是特殊化，其他是依赖/组织关系

---

---

# 六、综合复习框架

## 综合知识体系总结

### Java知识体系架构

```
Java基础层
├── 基本语法 (Q1-Q7)
│   ├── 方法签名
│   ├── 表达式求值(算术、布尔、类型转换)
│   └── 运算符优先级
│
├── 控制流 (Q8-Q14)
│   ├── For/While/Do-While循环
│   ├── 条件语句(If/Switch)
│   ├── 循环陷阱(空语句、Fall-Through)
│   └── 前缀/后缀递增
│
└── 数据结构基础 (Q15-Q19)
    ├── 一维数组操作
    ├── 二维数组与引用
    ├── 字符串操作
    └── 数组边界安全

Java面向对象基础层
├── 类与对象 (Q20-Q29)
│   ├── 对象创建与引用
│   ├── 参数传递(值vs引用)
│   ├── 访问修饰符
│   ├── This关键字
│   ├── 构造方法
│   └── 静态成员
│
├── 类型系统 (Q30-Q40)
│   ├── 多态性(向上/下转型)
│   ├── 继承关系
│   ├── 接口与抽象类
│   ├── 关系建模
│   └── 修饰符体系(Final)
│
└── 设计原则 (Q27, Q38)
    ├── 高内聚设计
    ├── 错误分类
    └── 调试策略
```

---

## 核心易混淆点速查表

### 🔴 最常见的错误点

1. **方法签名** - 返回类型不是签名的一部分
2. **Void方法** - 用于执行操作(副作用)，不返回值
3. **类型转换** - (int)强制转换舍去小数，不四舍五入
4. **For循环变量作用域** - `for(int i=...)` 中i超出范围后不可用
5. **参数传递** - Java中只有值传递！对象参数传的是引用的值，不是真正的"引用传递"
6. **引用赋值** - `c2 = c1` 只是引用复制，两个变量指向同一对象
7. **静态变量** - 属于类不属于实例，所有实例共享
8. **抽象类vs接口** - 两者都能用作数据类型，但继承关系和作用不同
9. **Switch Fall-Through** - 没有break会继续执行下一个case
10. **数组索引安全** - 条件判断顺序很重要：先检查边界

### 🟡 需要深化理解的概念

- **多态性** = 对象、引用、方法调用的三层含义
- **继承关系** = is-a，vs 聚合/组合的has-a
- **访问修饰符** = public/private/protected/package-private的合理使用
- **内聚性** = 类内部功能相关程度的设计指标
- **类型转换链** = 隐式向上转型 vs 显式向下转型

---

## 按知识领域的习题分类

### 表达式求值 (考查计算能力)
- Q3, Q4, Q5 → 优先级、类型转换
- Q12 → 整数运算(除法、取模)
- Q13 → 前缀/后缀递增

### 控制流逻辑 (考查流程理解)
- Q6 → 布尔运算符
- Q8, Q10 → 循环执行顺序
- Q9 → 空语句陷阱
- Q14 → Switch Fall-Through

### 数组与字符串 (考查集合操作)
- Q15 → 短路求值与边界
- Q17 → 数组元素操作
- Q18 → 字符串拼接
- Q19 → 二维数组引用

### 方法与对象 (考查OOP核心)
- Q20 → 参数传递
- Q21-Q23 → 类定义、This关键字
- Q24 → 引用赋值
- Q28-Q29 → 访问器/构造方法

### 类型系统 (考查深度理解)
- Q25 → 基本vs引用类型
- Q26 → 静态成员
- Q30, Q32 → 向上/下转型
- Q31 → 多态性定义

### 高级概念 (考查应用广度)
- Q33-Q35 → 抽象类与接口
- Q36 → Final修饰符
- Q37, Q39 → 继承关系
- Q40 → 设计模式(IS-A vs HAS-A)

---

## 备考建议

### 🎯 高频易错题型
- **参数传递**(Q20) - 反复强化"值传递"概念
- **引用赋值**(Q24) - 画图理解引用指向
- **数组操作**(Q15-Q19) - 边界条件最容易出错
- **类型转换**(Q4, Q32) - 向上/下转型规则
- **多态性**(Q30-Q32) - 编译时类型vs运行时类型

### 📊 学习重点投入比例
- 基础语法：20% - 快速过
- 控制流：15% - 掌握陷阱
- 数据结构：15% - 重点是边界安全
- OOP基础：25% - 这是难点
- OOP高级：25% - 这是考点

### 💡 实战训练方法
1. **逐题编写测试代码** - 在IDE中实际运行
2. **画内存图**
   - 对象创建与引用关系
   - 数组元素变化过程
   - 参数传递的值流
3. **总结对比**
   - Is-a vs Has-a
   - 抽象类 vs 接口
   - 值传递 vs 引用传递

---

**最后更新**: 2026年4月22日
**难度评估**: 初级(Q1-Q19) - 中级(Q20-Q30) - 进阶(Q31-Q40)
**通过目标**: 80%以上(32/40)正确率
