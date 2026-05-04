# 18 综合案例分析

三个大型案例覆盖：Stepwise Refinement、OOP 设计关系、跨层级多态。

---

## 案例一：信用卡 Luhn 校验 (Stepwise Refinement)

来源：`lec_06_examples/ValidateCCStubs.java`

### 完整算法

```
1. hasValidLength(number)      → 13-16 位？
2. hasValidPrefix(number)      → 4/5/37/6 开头？
3. satisfiesMod10Check(number) → Luhn 算法
   ├── sumOfDoubleEvenPlace(number)   → 从右往左，双倍偶数位，≥10 则拆分
   │     └── getSumDigits(n)          → 两位数拆为个位+十位
   └── sumOfOddPlace(number)          → 从右往左，累加奇数位
   → (sumOfDoubleEvenPlace + sumOfOddPlace) % 10 == 0 ?
```

### 完整代码

```java
public class ValidateCC {

    // 主入口：分步骤检查
    public static boolean isValidNumber(String number) {
        return hasValidLength(number) &&
               hasValidPrefix(number) &&
               satisfiesMod10Check(number);
    }

    // Step 1: 长度 13-16
    public static boolean hasValidLength(String number) {
        int len = number.length();
        return len >= 13 && len <= 16;
    }

    // Step 2: 前缀检查
    public static boolean hasValidPrefix(String number) {
        if (number.startsWith("4")) return true;      // Visa
        if (number.startsWith("5")) return true;      // MasterCard
        if (number.startsWith("37")) return true;     // AmEx
        if (number.startsWith("6")) return true;      // Discover
        return false;
    }

    // Step 3: Luhn 算法
    public static boolean satisfiesMod10Check(String number) {
        int sum = sumOfDoubleEvenPlace(number) + sumOfOddPlace(number);
        return sum % 10 == 0;
    }

    // 双倍偶数位（从右往左第 2/4/6...位）
    public static int sumOfDoubleEvenPlace(String number) {
        int sum = 0;
        for (int i = number.length() - 2; i >= 0; i -= 2) {
            sum += getSumDigits((number.charAt(i) - '0') * 2
            );
        }
        return sum;
    }

    // 奇数位求和
    public static int sumOfOddPlace(String number) {
        int sum = 0;
        for (int i = number.length() - 1; i >= 0; i -= 2) {
            sum += number.charAt(i) - '0';
        }
        return sum;
    }

    // 两位数拆分（≥10 时十位 + 个位）
    public static int getSumDigits(int num) {
        if (num < 10) return num;
        return num / 10 + num % 10;
    }

    // 测试
    public static void main(String[] args) {
        System.out.println(isValidNumber("4388576018410707")); // true (Visa)
        System.out.println(isValidNumber("1234567812345678")); // false
    }
}
```

### 考点

- 方法抽象：每个子方法只做一件事
- 自上而下：先写 Stub 占位，再逐步实现
- `charAt(i) - '0'` 将字符数字转为整数

---

## 案例二：Aggregation vs Composition（聚合 vs 组合）

### Aggregation（聚合）: has-a，松散关系

```java
class Student {
    private String name;
    public Student(String name) { this.name = name; }
    public String getName() { return name; }
}

class Department {
    private String deptName;
    private Student[] students;  // Department "有" Students

    public Department(String deptName, Student[] students) {
        this.deptName = deptName;
        this.students = students;
    }

    public void display() {
        System.out.println("Department: " + deptName);
        for (Student s : students) {
            System.out.println("  - " + s.getName());
        }
    }
}

// 使用
Student[] students = {new Student("Alice"), new Student("Bob")};
Department cs = new Department("CS", students);
cs.display();
// Alice 和 Bob 可以独立于 Department 存在 → Aggregation
```

### Composition（组合）: 强 has-a，生命周期绑定

```java
class Engine {
    public void start() { System.out.println("Engine started"); }
}

class Car {
    private Engine engine;  // Car 拥有 Engine

    public Car() {
        this.engine = new Engine();  // Engine 随 Car 创建
    }

    public void start() {
        engine.start();
        System.out.println("Car is running");
    }
    // Car 销毁时，Engine 也销毁 → Composition
}

// 使用
new Car().start();
```

### 对比

| | Aggregation | Composition |
|---|---|---|
| 关系 | 较弱的 has-a | 较强的 has-a |
| 生命周期 | 部分可独立存在 | 部分随整体共存亡 |
| 共享 | 一个部分可属于多个整体 | 通常独占 |
| 例子 | Department ↔ Student | Car ↔ Engine |
| UML | 空心菱形 | 实心菱形 |

---

## 案例三：Edible 跨层级多态

来源：`lec_13_examples_4/TestEdible.java`

### 完整代码

```java
// 接口：定义"可食用"行为
interface Edible {
    String howToEat();
}

// 抽象类：Animal 层次
abstract class Animal {
    public abstract String sound();
}

class Chicken extends Animal implements Edible {
    public String sound() { return "cock-a-doodle-doo"; }
    public String howToEat() { return "Fry it"; }
}

class Tiger extends Animal {  // Tiger 不实现 Edible
    public String sound() { return "RROOAARR"; }
}

// 抽象类：Fruit 层次（另一棵继承树）
abstract class Fruit implements Edible { }

class Apple extends Fruit {
    public String howToEat() { return "Apple: Make pie"; }
}

class Orange extends Fruit {
    public String howToEat() { return "Orange: Make juice"; }
}

// 测试
public class TestEdible {
    public static void main(String[] args) {
        Object[] objects = {new Tiger(), new Chicken(), new Apple(), new Orange()};

        for (Object o : objects) {
            if (o instanceof Edible) {
                System.out.println(((Edible) o).howToEat());
            }
            if (o instanceof Animal) {
                System.out.println(((Animal) o).sound());
            }
        }
    }
}
```

### 输出

```
RROOAARR
Fry it
cock-a-doodle-doo
Apple: Make pie
Orange: Make juice
```

### 关键理解

- `Edible` 打破了 Animal 和 Fruit 的继承边界
- Chicken 和 Apple **没有任何继承关系**，但都可以用 `Edible` 类型引用
- 使用 `instanceof` 判断对象是否实现了某个接口
- **接口的真正威力**：让不同继承树的类拥有共同的能力

### 考点

- 接口可以跨继承树工作
- `instanceof` 判断支持接口
- 向下转型 `(Edible) o` 需要先 instanceof 检查
- 一个类可以同时继承父类并实现接口
