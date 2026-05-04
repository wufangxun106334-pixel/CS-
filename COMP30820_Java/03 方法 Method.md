# 方法 Method

对应课件：

- `06 - Methods`

## 对应源码

- [TestMax.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_06_examples/TestMax.java)
- [MyMath.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_06_examples/MyMath.java)
- [TestPassByValue.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_06_examples/TestPassByValue.java)

## 1. 方法的基本结构

```java
public static int max(int num1, int num2) {
    int result;
    if (num1 > num2)
        result = num1;
    else
        result = num2;
    return result;
}
```

组成：
public      access modifier
static      optional, belongs to the class
int         return type
add         method name
(int a, int b) parameter list
{ ... }     method body


## 2. 形参和实参

- `formal parameters`：定义时写在括号里
- `actual arguments`：调用时传进去的值

## 3. 返回值和 `void`

- 有结果要返回：如 `int`, `double`, `String`
- 没结果只做动作：`void`

## 4. 方法重载 overloading

来源示例：
- `lec_06_examples/MyMath.java`

### Demo

```java
public static double max(double a1, double a2) {
    return (a1 > a2) ? a1 : a2;
}

public static double max(double a1, double a2, double a3) {
    return max(max(a1, a2), a3);
}
```

重载条件：

- 方法名相同
- 参数列表不同

## 5. pass-by-value

Java 只有 pass-by-value。

### primitive 例子

来源示例：
- `lec_06_examples/TestPassByValue.java`

```java
public static void foo(int n) {
    n *= 10;
}
```

调用后，外部变量 `x` 不会变。

### 核心理解

- 传递的是“值的副本”
- 对 primitive 来说，副本就是数值本身

## 6. 方法抽象和复用

为什么要抽方法：

- 减少重复代码
- 提高可读性
- 便于测试和维护

## 7. Demo Code 案例

### TestMax

来源：`lec_06_examples/TestMax.java`

```java
public static int max(int num1, int num2, int num3) {
    return (num1 > num2) && (num1 > num3)
            ? num1
            : (num2 > num3 ? num2 : num3);
}
```

### MyMath

来源：`lec_06_examples/MyMath.java`

```java
public static double min(double a1, double a2, double a3) {
    return min(min(a1, a2), a3);
}
```

## 8. 易错点辨析

### 易错 1：方法重载看返回类型

不对。

重载主要看参数列表，不看返回类型。

### 易错 2：Java 传对象时是 pass-by-reference

不对。

Java 仍然是 pass-by-value，只不过传的是“引用值的副本”。

### 易错 3：`static` 方法能直接访问所有实例变量

不对。

`static` 方法不属于某个具体对象，不能直接用实例字段。

---

## 8. ⭐ 方法歧义调用 (Ambiguous Invocation)

当两个重载方法对同一个调用**匹配度相同**时，编译器无法决定 → **编译错误**。

```java
// AmbiguousInvocation.java
public class AmbiguousInvocation {
    public static double max(int num1, double num2) {
        return (num1 > num2) ? num1 : num2;
    }

    public static double max(double num1, int num2) {
        return (num1 > num2) ? num1 : num2;
    }

    public static void main(String[] args) {
        // max(1, 2) 模棱两可！调用哪个？
        // 编译错误：reference to max is ambiguous
        System.out.println(max(1, 2));
    }
}
```

### 解决方案

```java
// 提供一个精确匹配的版本
public static int max(int num1, int num2) {
    return (num1 > num2) ? num1 : num2;
}
```

### 易错点

- 只有参数列表不同才算不同的重载（返回类型不算）
- `max(int, double)` 和 `max(double, int)` 在传入 `(1, 2)` 时歧义

---

## 9. 局部变量作用域 (Scope of Local Variables)

### 基本规则

- 局部变量的作用域从声明处开始，**到所属代码块的 `}` 结束**
- for 循环头中声明的变量，作用域为**整个 for 循环**（包括循环体）

```java
public class ScopeDemo {
    public static void main(String[] args) {
        // 变量 i 的作用域：整个 for 循环
        for (int i = 0; i < 3; i++) {
            System.out.println(i);  // 0, 1, 2
        }
        // System.out.println(i);  // ❌ 编译错误：i 已出作用域！
    }
}
```

### 非嵌套块可以同名

```java
// ✅ OK：两个独立块可以声明同名变量
for (int i = 0; i < 3; i++) { System.out.println(i); }
for (int i = 3; i < 6; i++) { System.out.println(i); }
```

### 嵌套块不能重名

```java
int x = 10;
{
    // int x = 20;  // ❌ 编译错误：内层不能与外层重名！
}
```

### Sample Paper 陷阱 (Q16)

```java
for (int i = 0; i < x.length; i++) {
    if (x[i] > 3) break;
}
System.out.println(x[i]);  // ❌ 编译错误！i 在 for 循环结束后不可见
```

---

## 10. Stepwise Refinement（逐步求精）

### 概念

用"分而治之"策略，将大问题分解为小问题，每个小问题用一个方法实现。

### 结构图 (Structure Chart)

```
            isValidNumber
           /      |       \
  hasValidLength  |   satisfiesMod10Check
                  |      /            \
         hasValidPrefix  sumOfDoubleEvenPlace  sumOfOddPlace
                                |
                           getSumDigits
```

### Stubs（方法桩）

在自上而下实现时，先用返回默认值的方法"占位"：

```java
// 在实现前先用 stub 测试上层逻辑
public static boolean hasValidLength(String number) {
    return true;  // TODO: 后续实现
}

public static int sumOfDoubleEvenPlace(String number) {
    return 0;     // TODO: 后续实现
}
```

### Top-down vs Bottom-up

| 策略 | 做法 | 优势 |
|------|------|------|
| Top-down | 从顶层方法开始，用 Stubs 逐步替换 | 适合设计阶段，逻辑清晰 |
| Bottom-up | 从最底层工具方法开始，逐一测试 | 适合独立测试每个模块 |

### 案例：信用卡 Luhn 校验

来自 `lec_06_examples/ValidateCCStubs.java`：

```java
// 完整方法签名一览
public static boolean isValidNumber(String number)     // 主入口
public static boolean hasValidLength(String number)     // 13-16 位
public static boolean hasValidPrefix(String number)     // 4,5,37,6 开头
public static boolean satisfiesMod10Check(String number) // Luhn 算法
public static int sumOfDoubleEvenPlace(String number)    // 双倍偶数位求和
public static int sumOfOddPlace(String number)           // 奇数位求和
public static int getSumDigits(int number)               // 两位数拆分求和
```

### 优势

1. 简化复杂程序，便于管理
2. 促进代码复用
3. 方便调试和测试
4. 支持团队协作

### 易错点

- Stub 方法必须返回和方法签名兼容的值
- 方法抽象的核心：调用者不需要知道实现细节（黑盒）
