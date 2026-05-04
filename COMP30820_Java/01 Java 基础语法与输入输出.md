# Java 基础语法与输入输出

对应课件：

- `01 - Introduction`
- `02 - Elementary Programming`
- `04 - Mathematical Functions, Characters, and Strings`

## 对应源码

- [ComputeAverage.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_02_examples/ComputeAverage.java)
- [HexDigit2Dec.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_04_examples/HexDigit2Dec.java)

## 1. 程序最小结构

Java 程序最常见入口：

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

关键点：

- `class` 定义类
- `main` 是程序入口
- `String[] args` 表示命令行参数

## 2. 变量、类型、赋值

### 常见 primitive types

- `byte`
- `short`
- `int`
- `long`
- `float`
- `double`
- `char`
- `boolean`

### 标准声明形式

```java
int x;
double y = 3.14;
char ch = 'A';
boolean ok = true;
```

## 3. 标识符规则

- 可以包含字母、数字、`_`、`$`
- 不能以数字开头
- 不能是保留字
- 不能是 `true`、`false`、`null`

## 4. Scanner 输入

来源示例：
- `lec_02_examples/ComputeAverage.java`
- `lec_04_examples/HexDigit2Dec.java`

### Demo

```java
Scanner input = new Scanner(System.in);
double num1 = input.nextDouble();
String text = input.nextLine();
input.close();
```

### 易错点

- `nextInt()` / `nextDouble()` 后面接 `nextLine()` 时，容易读到残留换行
- `Scanner` 用完最好 `close()`
- 最好都用nextLine 然后做类型转化:
- int age = Integer.parseInt(sc.nextLine());
- double score = Double.parseDouble(sc.nextLine());

## 4.5 printf 格式化输出

`System.out.printf()` 不会自动换行，需要手动加 `\n`。
建议写法:
System.out.printf("age = %d\n", age);
System.out.printf("score = %.1f\n", score);

| 格式符    | 含义                 | 示例                                 |
| ------ | ------------------ | ---------------------------------- |
| `%d`   | 整数 (int)           | `printf("%d", 42)`                 |
| `%f`   | 浮点数 (float/double) | `printf("%f", 3.14)`               |
| `%.2f` | 保留 2 位小数           | `printf("%.2f", 3.14159)` → `3.14` |
| `%s`   | 字符串                | `printf("%s", "hello")`            |
| `%c`   | 字符                 | `printf("%c", 'A')`                |
| `%b`   | 布尔值                | `printf("%b", true)`               |
| `%n`   | 平台无关换行             | 同 `\n`                             |

### Demo

```java
double radius = 5.0;
double area = radius * radius * Math.PI;
System.out.printf("Area is %.3f\n", area);  // Area is 78.540
```

### 易错点

- `printf` **不会自动换行**，不加 `\n` 或 `%n` 则后续输出会粘连在同一行

## 5. 字符与字符串

### `char`

- 单个字符
- 用单引号：`'A'`

### `String`

- 字符串对象
- 用双引号：`"A"`

### 常用操作

- `length()` -- 返回字符串长度
- `charAt(i)` -- 返回索引 i 处的字符
- `equals()` / `equalsIgnoreCase()` -- 内容比较
- `compareTo()` / `compareToIgnoreCase()` -- 字典序比较
- `indexOf(ch)` / `indexOf(ch, fromIndex)` / `indexOf(s)` -- 查找字符/子串
- `lastIndexOf(ch)` / `lastIndexOf(s)` -- 从后往前查找
- `substring(beginIndex)` / `substring(beginIndex, endIndex)` -- 截取子串
- `startsWith(prefix)` / `endsWith(suffix)` -- 前缀/后缀判断
- `concat(s)` -- 拼接（等同于 `+`）
- `toUpperCase()` / `toLowerCase()` -- 大小写转换
- `trim()` -- 去除首尾空白

### 字符串与数字互转

```java
// 字符串 → 数字
int i = Integer.parseInt("123");          // 123
double d = Double.parseDouble("3.14");    // 3.14

// 数字 → 字符串
String s1 = String.valueOf(100);          // "100"
String s2 = Integer.toString(200);        // "200"
```

### 转义字符 (Escape Sequences)

| 转义序列 | 含义 |
|----------|------|
| `\n` | 换行 |
| `\t` | 制表符 (Tab) |
| `\\` | 反斜杠 |
| `\"` | 双引号 |
| `\'` | 单引号 |
| `\r` | 回车 |

### 读取字符

```java
Scanner input = new Scanner(System.in);
String s = input.next();       // 读取一个单词
char ch = s.charAt(0);         // 取第一个字符
```

## 6. Character 和 Math

来源示例：
- `lec_04_examples/HexDigit2Dec.java`
- `lec_06_examples/MyMath.java`

### Demo: 十六进制字符转十进制

```java
char c = hexStr.charAt(0);
boolean isDigit = Character.isDigit(c);
boolean isHex = (c >= 'A' && c <= 'F') || (c >= 'a' && c <= 'f');

if (isDigit) {
    System.out.println(c - '0');
} else if (isHex) {
    System.out.println(Character.toUpperCase(c) - 'A' + 10);
}
```

这个例子同时考：

- `charAt`
- `Character.isDigit`
- `Character.toUpperCase`
- 字符和整数的转换

## 7. Demo Code 案例

### 案例 1：平均值计算

来源：`lec_02_examples/ComputeAverage.java`

```java
Scanner input = new Scanner(System.in);
double num1 = input.nextDouble();
double num2 = input.nextDouble();
double num3 = input.nextDouble();
double average = (num1 + num2 + num3) / 3;
System.out.println("The average is " + average);
```

### 案例 2：HexDigit2Dec

来源：`lec_04_examples/HexDigit2Dec.java`

```java
if (hexStr.length() != 1) {
    System.out.println("Invalid input");
    return;
}

char c = hexStr.charAt(0);
if (Character.isDigit(c)) {
    System.out.println(c - '0');
} else {
    System.out.println(Character.toUpperCase(c) - 'A' + 10);
}
```

## 8. 易错点辨析

### 易错 1：`'A'` 和 `"A"` 是一样的

不是。

- `'A'` 是 `char`
- `"A"` 是 `String`

### 易错 2：`==` 可以比较所有字符串内容

不对。

比较字符串内容应优先用 `equals()`。

### 易错 3：`char` 不能参与算术

不对。

`char` 本质上有数值编码，可以做减法，例如 `c - '0'`。

---

## 9. ⭐ 浮点数精度陷阱（高频考点）

**浮点数在 Java 中是近似值，不能精确存储。**

```java
System.out.println(1.0 - 0.9);
// 输出 0.09999999999999998，而不是 0.1

double x = 1.0 - 0.1 - 0.1 - 0.1 - 0.1 - 0.1;
System.out.println(x == 0.5);
// false！x 实际是 0.5000000000000001
```

### 正确做法：使用 epsilon 比较

```java
final double EPSILON = 1E-14;       // double 用 1E-14
// final float EPSILON = 1E-7;      // float 用 1E-7

if (Math.abs(x - 0.5) < EPSILON) {
    System.out.println("x 约等于 0.5");
}
```

### 易错点

- `==` 比较两个浮点数几乎总是错的
- 整数可以用 `==`，浮点数必须用 epsilon
- Sample Paper Q6 曾考此陷阱

---

## 10. Math 类常用方法速查

| 方法               | 说明                             | 示例                     |
| ---------------- | ------------------------------ | ---------------------- |
| `Math.PI`        | π 常量                           | `3.14159...`           |
| `Math.E`         | e 常量                           | `2.71828...`           |
| `Math.pow(a, b)` | a 的 b 次方                       | `pow(2,3)` → `8.0`     |
| `Math.sqrt(x)`   | 平方根                            | `sqrt(4)` → `2.0`      |
| `Math.abs(x)`    | 绝对值（重载于 int/long/float/double） | `abs(-2.1)` → `2.1`    |
| `Math.exp(x)`    | e 的 x 次方                       | `exp(1)` → `~2.718`    |
| `Math.log(x)`    | 自然对数 (ln)                      | `log(E)` → `1.0`       |
| `Math.log10(x)`  | 以 10 为底的对数                     | `log10(100)` → `2.0`   |
| `Math.ceil(x)`   | 向上取整                           | `ceil(10.1)` → `11.0`  |
| `Math.floor(x)`  | 向下取整                           | `floor(10.9)` → `10.0` |
| `Math.round(x)`  | 四舍五入，返回 long                   | `round(9.5)` → `10`    |
| `Math.max(a,b)`  | 较大值                            | `max(2,5)` → `5`       |
| `Math.min(a,b)`  | 较小值                            | `min(2,5)` → `2`       |

### 保留 n 位小数的方法

```java
double x = 2.0 / 3;                           // 0.666666...
double r = Math.round(x * 100.0) / 100.0;      // 0.67
```

### 易错点

- `Math.round(-1.5)` → `-1`（向正无穷取整，不是对称四舍五入！）
- `ceil` 和 `floor` 返回 `double`，`round` 返回 `long`

---

## 11. XOR 运算符 (`^`)

逻辑异或：两边相同为 `false`，不同为 `true`。

| a | b | a ^ b |
|---|---|-------|
| true | true | false |
| true | false | true |
| false | true | true |
| false | false | false |

```java
System.out.println(true ^ true);   // false
System.out.println(true ^ false);  // true
```

### 典型考题

"数字能被 2 整除或被 3 整除，但**不同时**被两者整除" → 用 `^`

```java
boolean result = (n % 2 == 0) ^ (n % 3 == 0);
```

### 易错点

- `^` 和 `||` 容易混淆：`||` 是"或"，`^` 是"排他或"
- Source: `lec_03_examples/TestLogicalOperators.java`
