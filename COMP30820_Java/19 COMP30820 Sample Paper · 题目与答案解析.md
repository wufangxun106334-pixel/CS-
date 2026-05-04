# COMP30820 Sample Paper · 题目与答案解析

COMP30820 Java Programming (Conv) Sample Paper，共 40 题。按知识点分类整理，每题包含答案与详细解析。

---

## 一、方法基础（Q1–Q2）

### Q1
In Java, the signature of a method consists of:

A. The method name
B. The parameter list
C. The method name and parameter list
D. The return type, method name, and parameter list
E. None of the above

> [!answer]- 答案：C
> Java 中方法签名 (method signature) 只包含**方法名**和**参数列表**，不包括返回类型。
>
> - 返回类型不是方法签名的一部分
> - 例子：`public int add(int a, int b)` 的签名是 `add(int, int)`

---

### Q2
Which one of the following is most appropriately declared as a **void** method?

A. A method that converts a lowercase letter to uppercase
B. A method that returns a random integer between 0 and 99 (inclusive)
C. A method that checks whether a number is even
D. A method that prints a random integer between 0 and 99 (inclusive)
E. None of the above

> [!answer]- 答案：D
> - `void` 方法不返回值，用于执行操作而非计算返回结果
> - A (转换大小写) → 返回 `char`
> - B (返回随机整数) → 返回 `int`
> - C (判断偶数) → 返回 `boolean`
> - D (**打印**随机整数) → 只打印不返回值，适合 `void`

---

## 二、表达式与类型转换（Q3–Q7）

### Q3
The expression `2.0 + 3 * 5 / 2 / 5.0` evaluates to:

A. `3.0`
B. `3.5`
C. `3.6`
D. `3.4`
E. None of the above

> [!answer]- 答案：D
> 运算符优先级：`*`, `/` 同级，高于 `+`，左结合求值：
>
> 1. `3 * 5 = 15`
> 2. `15 / 2 = 7`（整数除法！）
> 3. `7 / 5.0 = 1.4`
> 4. `2.0 + 1.4 = 3.4`
>
> 陷阱：`15 / 2` 是整数除法。

---

### Q4
The expression `(int)(11.0252175 * 100) / 100` evaluates to:

A. `11.02`
B. `11.03`
C. `11`
D. `11.0252175`
E. None of the above

> [!answer]- 答案：C
> 1. `11.0252175 * 100 = 1102.52175`
> 2. `(int) 1102.52175 = 1102`（直接截断，不是四舍五入）
> 3. `1102 / 100 = 11`（整数除法）
>
> 陷阱：`(int)` 是截断 (truncation) 不是四舍五入，且最后一步是整数除法。

---

### Q5
The expression `('a' <= 'b') ? 1 : 0` evaluates to:

A. `true`
B. `false`
C. `1`
D. `0`
E. None of the above

> [!answer]- 答案：C
> - `'a'` 的 Unicode 值为 97，`'b'` 为 98
> - `97 <= 98` → `true`
> - 三元操作符 `? :` 取第一个值 → `1`
> - 注意：`1` 是 `int` 类型，不是 `boolean`

---

### Q6
Given `boolean a = true; boolean b = false; boolean c = true;`, what does the expression `a && !b || c && !(a && b)` evaluate to?

A. `true`
B. `false`
C. `1`
D. `0`
E. None of the above

> [!answer]- 答案：A
> `&&` 优先级高于 `||`，按短路求值：
>
> 1. `!b = !false = true` → `a && true = true && true = true`
> 2. `a && b = true && false = false` → `!(false) = true` → `c && true = true && true = true`
> 3. `true || true = true`

---

### Q7
Given `double A, P = 100.0; int r = 10, n = 10, k = 12;`

Which expression does **NOT** correctly compute `A = P * (1 + r/(100k))^(n*k)`?

A. `A = P * Math.pow(1 + 0.01 * r / k, n * k);`
B. `A = P * Math.pow(1.0 + r * 0.01 / k, (n * k));`
C. `A = P * Math.pow(1 + r / 100 / k, n * k);`
D. `A = P * Math.pow(1.0 + r / 100.0 / k, n * k);`

> [!answer]- 答案：C
> 选项 C 中 `r / 100` 是整数除法：`10 / 100 = 0`，然后 `0 / k = 0.0`，最终 `Math.pow(1.0, 120) = 1.0`，结果错误。
>
> - A: `0.01 * 10 / 12` → 正确
> - B: `10 * 0.01 / 12` → 正确
> - C: `10 / 100` = `0`（整数除法） → 错误！
> - D: `10 / 100.0` = `0.1` → 正确
>
> 陷阱：整数除以整数结果仍是整数，必须至少有一个操作数是浮点数。

---

## 三、循环与控制流（Q8–Q11）

### Q8
What is the value of n at the end of the following code?

```java
int n;
for (n = 0; n <= 2; n--)
    n += 3;
```

A. `2`
B. `3`
C. `4`
D. `5`
E. `6`

> [!answer]- 答案：C
> 追踪每一步：
>
> 1. n=0: 条件 `0 <= 2` → `n--` → n=-1, `n += 3` → n=2
> 2. n=2: 条件 `2 <= 2` → `n--` → n=1, `n += 3` → n=4
> 3. n=4: 条件 `4 <= 2` → false，退出
>
> 注意 for 循环的 action-after-each-iteration (`n--`) 在循环体 `n+=3` 之前执行。

---

### Q9
What is the output of the following code excerpt?

```java
int i = 0;
for (i = 0; i < 10; i++);
System.out.println(i + 4);
```

A. `14`
B. `4`
C. `13`
D. Nothing is printed because the loop runs forever
E. Nothing is printed because there is a compilation error

> [!answer]- 答案：A
> `for (i = 0; i < 10; i++);` 的分号导致空循环体！循环执行 10 次后 `i = 10`。
>
> `System.out.println(10 + 4)` → `14`
>
> 关键陷阱：`;` 分号直接结束了 for 语句，后面的 `println` 不属于循环体。

---

### Q10
How many times will the following code print the string "Java"?

```java
int i = 1;
do {
    i++;
    System.out.println("Java");
} while (i <= 2);
```

A. An infinite number of times
B. The string is never printed
C. 3 times
D. 2 times
E. 1 time

> [!answer]- 答案：D
> `do-while` 至少执行一次：
>
> 1. i=1: `i++` → i=2, print "Java", `2 <= 2` → true，继续
> 2. i=2: `i++` → i=3, print "Java", `3 <= 2` → false，退出
>
> 共打印 **2 次**。

---

### Q11
Consider the code:

```java
Scanner input = new Scanner(System.in);
int n = input.nextInt();
while (n < 30 || n > 90)
    n = input.nextInt();
```

What does this code do?

A. Continue to obtain input until the input is outside [31, 89] inclusive
B. Continue to obtain input until the input is inside [31, 89] inclusive
C. Continue to obtain input until the input is outside [30, 90] inclusive
D. Continue to obtain input until the input is inside [30, 90] inclusive

> [!answer]- 答案：D
> 条件是 `n < 30 || n > 90` 时继续获取输入。循环停止条件是**取反**：`!(n < 30 || n > 90)` = `n >= 30 && n <= 90`。
>
> 所以当输入在 [30, 90] 区间**内**时停止，即持续获取直到输入在 [30, 90] 区间内。

---

## 四、算术、自增与 switch（Q12–Q14）

### Q12
What is printed by the following code?

```java
int i = 2;
int j = i / 10 + i % 10;
int p = 22;
int q = p / 10 + p % 10;
System.out.println(j + " " + q);
```

A. `2 22`
B. `0 2`
C. `0 0`
D. `2 0`
E. `2 4`

> [!answer]- 答案：E
> - `i = 2`: `j = 2/10 + 2%10 = 0 + 2 = 2`
> - `p = 22`: `q = 22/10 + 22%10 = 2 + 2 = 4`
>
> 输出 `"2 4"`。这是 Luhn 算法中拆分数字位的经典模式。

---

### Q13
What is printed by the following code?

```java
int i = 0;
int j = i++;
int k = ++j;
System.out.println(i + " " + j + " " + k);
```

A. `0 0 1`
B. `0 1 1`
C. `1 1 1`
D. `0 1 2`
E. `1 1 2`

> [!answer]- 答案：C
> 逐行追踪：
>
> 1. `i = 0`
> 2. `j = i++` → j=0（先用后加），i=1
> 3. `k = ++j` → j=1（先加后用），k=1
>
> 最终：`i=1, j=1, k=1` → 输出 `"1 1 1"`

---

### Q14
What are the values of k and ch after the following switch statement?

```java
int k = 3;
char ch = 'b';
switch (++ch) {
    case 'a': k = 0; break;
    case 'b': k += 1; break;
    case 'c': k = 2;
    default: k += 4;
}
```

A. `2` and `b`
B. `4` and `b`
C. `6` and `b`
D. `2` and `c`
E. `6` and `c`

> [!answer]- 答案：E
> 1. `++ch` → ch 从 `'b'` 变为 `'c'`
> 2. 匹配 `case 'c'`: `k = 2`，**没有 break！**
> 3. Fall-through 到 `default`: `k += 4` → `k = 6`
>
> 最终：`ch = 'c'`, `k = 6`。
>
> 陷阱：`case 'c'` 缺少 `break`，导致 fall-through。

---

## 五、数组（Q15–Q17）

### Q15
What happens when the following code excerpt is executed?

```java
int[] a = {1, 2, 3, 4, 5};
int value = 0;
for (int i = 0; a[i] != value && i < a.length; i++)
    System.out.print(a[i] + " ");
System.out.println(6);
```

A. The following values are printed: `1 2 3 4 5 6`
B. The following value is printed: `6`
C. Nothing is printed
D. A runtime error occurs
E. A compilation error occurs

> [!answer]- 答案：D
> 循环条件是 `a[i] != value && i < a.length`。注意两个条件的顺序！
>
> i=0~4: `a[i] != 0` 为 true，`i < 5` 为 true → 打印
> i=5: 检查 `a[5] != 0` 时抛出 `ArrayIndexOutOfBoundsException`！
>
> 因为 `a[i] != value` **先**于 `i < a.length` 求值，`&&` 的短路求值无法保护越界访问。
>
> 陷阱：条件顺序至关重要。如果把 `i < a.length` 放前面就不会出错。

---

### Q16
What is printed by the following code excerpt?

```java
int[] x = {1, 2, 3, 4, 5};
System.out.print("[ ");
for (int i = 0; i < 5; i++) {
    if (x[i] > 3)
        break;
    System.out.printf("%d ", x[i]);
}
System.out.println("]");
```

A. `[ 3 4 5 ]`
B. `[ 2 3 4 ]`
C. `[ 4 5 ]`
D. `[ 1 2 ]`
E. `[ 1 2 3 ]`

> [!answer]- 答案：E
> 追踪循环：
>
> - i=0: `x[0]=1`, `1 > 3` = false → 打印 `"1 "`
> - i=1: `x[1]=2`, `2 > 3` = false → 打印 `"2 "`
> - i=2: `x[2]=3`, `3 > 3` = false → 打印 `"3 "`
> - i=3: `x[3]=4`, `4 > 3` = true → break，退出循环
>
> 输出：`[ 1 2 3 ]`

---

### Q17
What is printed by the following code excerpt?

```java
int[] a = {5, 4, 3, 2, 1, 0};
for (int i = a.length - 2; i >= 0; i--)
    a[i + 1] = a[i];
for (int i : a)
    System.out.print(i + " ");
```

A. `5 1 2 3 4 5`
B. `1 1 2 3 4 5`
C. `1 2 3 4 5 5`
D. `5 5 4 3 2 1`
E. `5 4 3 2 1 1`

> [!answer]- 答案：D
> `a.length-2 = 4`，从 i=4 递减到 i=0：`a[i+1] = a[i]`（将每个元素向右移一位）
>
> | i | 操作 | 数组状态 |
> |---|------|----------|
> | 4 | a[5] = a[4] = 1 | {5, 4, 3, 2, 1, **1**} |
> | 3 | a[4] = a[3] = 2 | {5, 4, 3, 2, **2**, 1} |
> | 2 | a[3] = a[2] = 3 | {5, 4, 3, **3**, 2, 1} |
> | 1 | a[2] = a[1] = 4 | {5, 4, **4**, 3, 2, 1} |
> | 0 | a[1] = a[0] = 5 | {5, **5**, 4, 3, 2, 1} |
>
> 最终：`{5, 5, 4, 3, 2, 1}`
>
> 注意最左边的 `a[0]` 保持不变，前面的值被"挤"过来了。

---

## 六、字符串（Q18）

### Q18
What is printed by the following code excerpt?

```java
String s1 = "hello";
String s2 = "";
for (int i = 0; i < s1.length(); i += 2)
    s2 = s1.charAt(i) + s2;
System.out.println(s2);
```

A. `olh`
B. `hlo`
C. `le`
D. `el`
E. None of the above

> [!answer]- 答案：A
> 追踪每次迭代（i 步长为 2）：
>
> - i=0: `s2 = 'h' + "" = "h"`
> - i=2: `s2 = 'l' + "h" = "lh"`
> - i=4: `s2 = 'o' + "lh" = "olh"`
>
> 输出 `"olh"`。注意是 `char + s2`，字符被**前置**到 s2 前面，因此结果反转。

---

## 七、二维数组与引用传递（Q19–Q20）

### Q19
What is printed by the following code excerpt?

```java
int[][] arr1 = new int[3][2];
int[] arr2 = {1, 2, 3};
arr1[1] = arr2;
arr1[1][0] = 10;
arr2[1] = 20;
for (int i : arr2)
    System.out.print(i + " ");
```

A. `1 2 3`
B. `1 20 3`
C. `10 2 3`
D. `10 20 3`
E. None of the above

> [!answer]- 答案：D
> 关键：`arr1[1] = arr2` 使 `arr1[1]` 和 `arr2` **指向同一个数组对象**。
>
> 1. `arr1[1] = arr2` → arr1[1] 指向 {1, 2, 3}
> 2. `arr1[1][0] = 10` → 修改共享数组 → {10, 2, 3}
> 3. `arr2[1] = 20` → 修改共享数组 → {10, 20, 3}
> 4. 遍历 arr2 → 输出 `"10 20 3 "`
>
> 陷阱：共享引用，一个修改影响所有指向该对象的引用。

---

### Q20
What is the output when the following class is executed?

```java
public class Q20 {
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
        i = 222;
        c = new char[4];
        b[0] = true;
        c[0] = 'a';
    }
}
```

A. `11 true false x`
B. `11 true false a`
C. `11 false false a`
D. `222 true false x`
E. `222 false false a`

> [!answer]- 答案：A
> 三个参数的传递与修改分析：
>
> - **i (primitive)**: 传值，`i = 222` 只改形参 → 外部 `i = 11` 不变
> - **b (数组引用)**: 传引用副本，`b[0] = true` 修改的是共享数组对象 → `arr1[0] = true`
> - **c (数组引用)**: `c = new char[4]` 让形参 c 指向**新数组**，跟外部 arr2 断开了！`c[0] = 'a'` 修改的是新数组 → `arr2[0] = 'x'` 不变
>
> 所以 `arr1[1]` 保持默认值 `false`（boolean 默认值），最终输出 `"11 true false x"`。
>
> 陷阱：`c = new char[4]` 改变了形参引用方向，不是修改原数组对象！

---

## 八、OOP 基础 / 类定义（Q21–Q24）

### Q21
Which one of the following correctly defines a Java class named Student with one instance data field named id of type int?

A. `public Student { private int id; }`
B. `public Student() { private int id; }`
C. `public class Student { private int id; }`
D. `public class Student() { private int id; }`
E. None of the above

> [!answer]- 答案：C
> 正确的类定义语法：`public class ClassName { ... }`
>
> - A: 缺少 `class` 关键字
> - B: `Student()` 是构造器语法，不是类定义
> - C: 正确 ✓
> - D: `Student()` 不应有括号

---

### Q22
What is the purpose of the `this` keyword in Java?

A. It refers to the superclass of the class
B. It refers to the calling object
C. It refers to the static members of the class
D. It is used to create new instances of a class
E. None of the above

> [!answer]- 答案：B
> - `this` 指向**当前调用对象**（calling object / current instance）
> - A 描述的是 `super`
> - D 描述的是 `new`

---

### Q23
Which statement should be used in the constructor to initialise the data field `model`?

```java
public class Car {
    private String model;
    public Car(String model) {
        ... // 选择正确语句
    }
}
```

A. `model = model;`
B. `Car.model = model;`
C. `super.model = model;`
D. `this.model = model;`
E. None of the above

> [!answer]- 答案：D
> - A: `model = model` — 形参给自己赋值，实例变量没有被初始化
> - B: `Car.model` — 错误地用类名访问实例变量（且 model 是 private）
> - C: `super.model` — 构造器中 super() 是调用父类构造器，不是访问变量
> - D: `this.model = model` — 正确，区分实例变量和形参 ✓

---

### Q24
What happens when you assign one object reference to another?

```java
Circle c1 = new Circle(1.0);
Circle c2 = new Circle(5.0);
c2 = c1;
```

A. Both c1 and c2 refer to the Circle object with radius 5.0
B. Both c1 and c2 refer to the Circle object with radius 1.0
C. A new Circle object is created
D. A compilation error occurs
E. A runtime error occurs

> [!answer]- 答案：B
> `c2 = c1` 将 c2 指向 c1 所指向的对象（radius=1.0）。
>
> - Circle(5.0) 失去引用，成为垃圾回收的候选
> - 没有新对象创建，只是引用赋值

---

## 九、类型、静态与内聚（Q25–Q27）

### Q25
Which statement about Java primitive and reference types is **NOT** correct?

A. A primitive type stores an actual value directly in memory
B. A reference type stores an actual object directly in memory
C. Classes are reference types in Java
D. Methods cannot be called directly on primitive types
E. Arrays are reference types in Java

> [!answer]- 答案：B
> - B 错误：引用类型变量存储的是对象的**内存地址（引用）**，而不是对象本身
> - A 正确：基本类型直接存储值
> - C 正确：类是引用类型
> - D 正确：不能对基本类型调用方法（如 `5.toString()` 不行）
> - E 正确：数组是引用类型

---

### Q26
Which statement about static variables in Java is correct?

A. A static variable belongs to an instance of a class
B. A static variable cannot be accessed by a constructor
C. A static variable cannot be accessed by an instance method
D. A static variable can only be accessed by static methods
E. A static variable is shared by all instances of a class

> [!answer]- 答案：E
> - E 正确：静态变量属于类，被所有实例共享
> - A 错误：静态变量属于**类**而非实例
> - B、C、D 错误：构造器和实例方法都可以访问静态变量（静态变量可以通过任何方式访问）

---

### Q27
Which statement best describes a **cohesive** class?

A. A class that contains only static methods
B. A class that groups related functionality together
C. A class that contains only constructors and no other methods
D. A class that can be inherited by multiple other classes
E. A class with all private variables and getter/setter methods as appropriate

> [!answer]- 答案：B
> 内聚性 (Cohesion)：类应该将**相关功能**组合在一起，专注于单一职责。
>
> - A 描述的是 utility class
> - D 描述的是可继承性
> - E 描述的是封装 (encapsulation)
> - B 最准确地描述了内聚性 ✓

---

## 十、构造器与 Setter（Q28–Q29）

### Q28
Given a class Account with private `int id`, which correctly implements a **setter** method?

A. `public int getId() { return id; }`
B. `public void setId(int id) { id = id; }`
C. `public int setId() { return id; }`
D. `public void setId() { id = id; }`
E. `public void setId(int id) { this.id = id; }`

> [!answer]- 答案：E
> setter 的要求：
> - 返回 `void`
> - 有参数（要设置的值）
> - 方法内用 `this.id = id` 访问实例变量
>
> - A: 这是 **getter**，不是 setter
> - B: `id = id` 没有用 `this`，形参给自己赋值
> - C: 有返回值无参数，签名错误
> - D: 无参数，且 `id = id` 无意义
> - E: 正确 ✓

---

### Q29
Which correctly defines a constructor for the class?

```java
public class Person {
    private String name;
    private int age;
}
```

A. `public Person(String name, int age) { Person.name = name; Person.age = age; }`
B. `public void Person(String name, int age) { Person.name = name; Person.age = age; }`
C. `public void Person(String name, int age) { this.name = name; this.age = age; }`
D. `public Person(String name, int age) { name = name; age = age; }`
E. `public Person(String name, int age) { this.name = name; this.age = age; }`

> [!answer]- 答案：E
> 构造器的正确写法：
> - 不能有返回类型（不能写 `void`）
> - 必须用 `this.xxx = xxx` 区分形参和实例变量
>
> - A: `Person.name` 用类名访问实例变量 — 错误
> - B: `void` 使其成为普通方法，不是构造器
> - C: 同 B，有 `void`
> - D: `name = name` 没有用 `this`
> - E: 正确 ✓

---

## 十一、继承与多态（Q30–Q32）

### Q30
Given the following classes, which statement is correct?

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

A. A runtime error occurs
B. A compilation error occurs in class Example
C. A compilation error occurs in class TestExample
D. No errors occur and 5 is printed
E. No errors occur and 0 is printed

> [!answer]- 答案：C
> `obj` 的声明类型是 `Object`，`Object` 类没有 `getX()` 方法。编译器根据声明类型检查方法调用 → **编译错误**。
>
> 即使实际对象是 `Example` 类型，编译器只看声明类型 `Object`。需要通过 `((Example) obj).getX()` 向下转型才能编译通过。

---

### Q31
Which statement best describes polymorphism in Java?

A. The ability of a method to change its return type dynamically
B. The ability of an object to take many forms and be treated as a superclass or interface type
C. The ability to store multiple values in a single variable
D. A way to store primitive data types in Java
E. A method of compressing Java code for efficiency

> [!answer]- 答案：B
> 多态 (Polymorphism) 的核心：**一个对象可以以多种形态存在**，被当作其父类或接口类型来对待。
>
> ```java
> Parent p = new Child();  // Child 对象被当作 Parent 类型
> ```
>
> 这就是多态 —— 同一个对象可以有多个类型视角。

---

### Q32
Assume the Circle class has `getRadius()` that returns the radius. Which statement is correct?

```java
Object o = new Circle(5.0);
Circle c = o;
System.out.println(c.getRadius());
```

A. The code compiles and runs and 5.0 is printed
B. A runtime error occurs
C. A compilation error occurs
D. The code compiles and runs but nothing is printed

> [!answer]- 答案：C
> `Circle c = o;` — 试图将 `Object` 类型的变量赋值给 `Circle` 类型，**不允许隐式向下转型**。
>
> 编译器检查：`o` 是 `Object` 类型，不是所有 `Object` 都是 `Circle`，所以需要显式转换：`Circle c = (Circle) o;`

---

## 十二、抽象类与接口（Q33–Q37）

### Q33
Which statement about abstract classes in Java is correct?

A. An abstract class can be instantiated
B. An abstract class cannot be used as a data type
C. Abstract classes cannot have constructors
D. Abstract methods must have a body
E. Abstract classes can contain both abstract and concrete methods

> [!answer]- 答案：E
> - A 错误：抽象类**不能**被实例化（`new`）
> - B 错误：抽象类**可以**作为数据类型
> - C 错误：抽象类**可以有**构造器（子类通过 `super()` 调用）
> - D 错误：抽象方法**不能有**方法体
> - E 正确：抽象类可以同时有抽象方法和具体方法 ✓

---

### Q34
Which correctly declares an abstract method `max` that takes two `int` parameters and returns an `int`?

A. `public abstract max(int x, int y);`
B. `public abstract void max(int x, int y);`
C. `public int max(int x, int y);`
D. `public abstract int max(int x, int y) { }`
E. `public abstract int max(int x, int y);`

> [!answer]- 答案：E
> 抽象方法的要求：
> - `abstract` 修饰符
> - 返回值类型
> - 方法名 + 参数列表
> - **不能有方法体** `{ }`
> - 以分号结尾
>
> - A: 缺少返回值类型
> - B: 返回值类型错误（应为 `int` 不是 `void`）
> - C: 缺少 `abstract` 关键字
> - D: 有方法体 `{ }`，抽象方法不能有实现
> - E: 正确 ✓

---

### Q35
Consider the interface and class:

```java
public interface Test {
    public abstract void p();
    public abstract void q();
}

public abstract class A implements Test {
    @Override
    public void p() {
        System.out.println("here");
    }
}
```

Which statement is correct?

A. Compilation error: abstract class A cannot implement p()
B. Compilation error: abstract class A must implement both p() and q()
C. Runtime error: abstract class A cannot implement p()
D. Runtime error: abstract class A must implement both p() and q()
E. The code compiles

> [!answer]- 答案：E
> **抽象类实现接口时不需要实现所有方法！** 抽象类 `A` 只实现了 `p()`，`q()` 未实现，但 `A` 本身是抽象的，可以留给子类实现。
>
> 如果 `A` 是具体类（非抽象），才必须实现所有接口方法。

---

### Q36
Which statement about final classes in Java is correct?

A. Final classes cannot be instantiated
B. Final classes cannot contain static methods
C. Final classes cannot be extended by other classes
D. Final classes cannot contain final methods
E. Final classes cannot have instance variables

> [!answer]- 答案：C
> `final class` 意味着**不能被继承**（不能被扩展）。
>
> - A 错误：final 类可以被 `new` 实例化
> - B 错误：final 类可以有 static 方法
> - C 正确 ✓
> - D 错误：final 类的方法默认是 final（不能被重写），也可以显式声明 final
> - E 错误：可以有实例变量
>
> 区分：
> - `final class` → 不能被继承
> - `abstract class` → 不能被实例化

---

### Q37
Which statement about interfaces and abstract classes is **NOT** correct?

A. Interfaces cannot define constructors, while abstract classes can
B. Neither interfaces nor abstract classes can be instantiated using `new`
C. Interfaces cannot be used as data types, while abstract classes can
D. Both interfaces and abstract classes can be used as data types

> [!answer]- 答案：C
> - A 正确：接口无构造器，抽象类有（被 super() 调用）
> - B 正确：两者都不能 `new`
> - C **错误**：接口**可以**作为数据类型！例如 `Test t = new ConcreteClass();`
> - D 正确：两者都可以作为类型使用
>
> 接口可以作为引用类型，这是多态的基础之一。

---

## 十三、错误类型与关系（Q38–Q40）

### Q38
Which one is an example of a **logical (semantic) error**?

A. Using an undeclared variable
B. Omitting a semicolon at the end of a statement
C. Assigning a string literal to an int variable
D. Using an incorrect formula that produces incorrect output
E. Accessing an array element that is out of bounds

> [!answer]- 答案：D
> 三种错误类型：
>
> | 类型 | 描述 | 例子 |
> |------|------|------|
> | 语法错误 (Syntax) | 代码违反语言规则 | A (undeclared), B (missing ;), C (type mismatch) |
> | 运行时错误 (Runtime) | 编译通过但运行出错 | E (ArrayIndexOutOfBounds) |
> | 逻辑错误 (Semantic) | 程序能运行但结果错误 | D (错误公式) ✓ |
>
> 逻辑错误最难发现，因为没有编译或运行时错误提示。

---

### Q39
Which statement about classes and interfaces is **NOT** correct?

A. A class can extend only one superclass
B. A class can implement multiple interfaces
C. An interface can extend multiple interfaces
D. An interface can extend only one interface
E. An interface can extend other interfaces but not classes

> [!answer]- 答案：D
> Java 继承规则：
> - 类：**单继承** (extends one class)，**多实现** (implements multiple interfaces)
> - 接口：**可以继承多个接口** (extends multiple interfaces)
>
> D 错误：接口可以继承多个接口。例如：
> ```java
> interface C extends A, B { }
> ```

---

### Q40
Which statement is correct?

A. Aggregation models an is-a relationship
B. Composition models an is-a relationship
C. Inheritance models an is-a relationship
D. Inheritance models a has-a relationship
E. Association models an is-a relationship

> [!answer]- 答案：C
> Java 中三种关系：
>
> | 关系 | 类型 | 关键字 |
> |------|------|--------|
> | Inheritance (继承) | is-a | `extends` |
> | Aggregation (聚合) | has-a | 实例变量引用 |
> | Composition (组合) | has-a (强) | 实例变量引用 + 生命周期绑定 |
>
> - A、B 错误：聚合/组合是 has-a
> - C 正确：继承是 is-a 关系 ✓
> - D 错误：继承是 is-a 不是 has-a
> - E 错误：关联不是 is-a

---

> Sample Paper 共 40 题，覆盖方法签名、表达式、循环、数组、字符串、OOP、继承、多态、抽象类、接口等核心知识点。建议限时 60 分钟完成。错题请重点回顾对应章节的陷阱和易错点。
