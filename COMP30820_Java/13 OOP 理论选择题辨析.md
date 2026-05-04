# OOP 理论选择题辨析

## 对应源码

- [Circle.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_09_examples/Circle.java)
- [Counter.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_10_examples/Counter.java)
- [AggregationDemo.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_10_examples/AggregationDemo.java)
- [PolymorphismAndCastingDemo1.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_11_examples/PolymorphismAndCastingDemo1.java)
- [GeometricObject.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_13_examples_1/GeometricObject.java)
- [Rotatable.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/test_3_code/Rotatable.java)
- [Intern.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/week11/prac_07_code/Intern.java)

## 1. Method Signature

Java 里 method signature 指：

- 方法名
- 参数列表

**不包括返回类型。**

所以：

```java
int foo(int x)
double foo(int x)
```

不能只靠返回类型区分重载。

## 2. Cohesive Class

cohesive class 的最佳理解：

- 把相关的数据和行为组织在一起
- 类的职责集中
- 不应该东拼西凑互不相关的功能

这是“好设计”概念，不是语法规则。

## 3. Primitive vs Reference Type

### primitive

- 直接存值
- 例如 `int`, `double`, `char`, `boolean`

### reference type

- 变量里存的是引用
- 对象本体在别处
- 类和数组都属于 reference type

### 高频误区

“reference type stores the actual object directly in memory”

这类表述通常不对，考试里应判断为错误。

## 4. `this`

`this` 表示：

- 当前调用该方法或构造器的对象

常见用法：

- `this.field = parameter`
- `this(...)` 调另一个构造器

## 5. constructor 规则

### 正确特征

- 名字必须和类名一致
- 没有返回类型
- 可以带参数

### 典型错误

- 写成 `public void Person(...)`
- 用 `name = name` 而不是 `this.name = name`

## 6. static type vs runtime type

这是 sample paper 非常爱考的点。

### 例子

```java
Object obj = new Example();
System.out.println(obj.getX());
```

### 结论

这会编译错误。

原因：

- 编译器看 `obj` 的静态类型是 `Object`
- `Object` 类里没有 `getX()`
- 即使运行时实际对象是 `Example`，也过不了编译

### 对照例子

```java
Object o = new Circle(5.0);
Circle c = (Circle) o;
System.out.println(c.getRadius());
```

这里需要显式强转。

## 7. Polymorphism

多态的核心是：

- 一个对象可以以父类或接口类型出现
- 但运行时表现出自身真实类型的行为

例如：

```java
GeometricObject g = new Circle(5.0);
```

这是 polymorphism，不是类型错误。

## 8. abstract class

### 正确理解

- 不能直接实例化
- 可以当作数据类型
- 可以有构造器
- 可以有抽象方法和具体方法

### 高频误区

- “abstract class cannot have constructors” 是错的
- “abstract class cannot be used as a data type” 也是错的

## 9. interface

### 正确理解

- 接口可以作为数据类型
- 类可以实现多个接口
- 接口可以扩展多个接口

### 高频误区

- “interface cannot be used as a data type” 是错的
- “interface can extend only one interface” 也是错的

## 10. abstract class vs interface

### 相同点

- 都不能直接 `new`
- 都可用于抽象建模
- 都可作为变量类型

### 不同点

- abstract class 可有构造器
- interface 不定义构造器
- 类只能继承一个父类，但能实现多个接口
```java
interface Rotatable {
    void rotate(); // 方法
}

class Wheel implements Rotatable {
    @Override
    public void rotate() {
        System.out.println("Wheel rotates");
    }

    public void repair() {
        System.out.println("Repairing wheel");
    }
}

public class Main {
    public static void main(String[] args) {
        Rotatable r = new Wheel();
        //能不能调用，看引用类型(Rotatable)；具体执行哪个版本，看实际对象类型(Wheel)
        r.rotate(); // 可以,父类定义了方法,并且子类重写
        // r.repair(); // 错,父类没定义方法,引用只能调用接口中声明的方法
    }
}

```
## 11. final class

final class 的核心：

- 不能被继承

但它：

- 可以被实例化
- 可以有实例变量
- 可以有静态方法

## 12. is-a / has-a / aggregation / composition / association/Inheritance
基本关系:
Inheritance: is-a
Association: uses-a / knows-a
Aggregation: weak has-a
Composition: strong has-a

### Inheritance

- `is-a`

例如：

- `Circle is a GeometricObject`
```java
class Animal {
    public void eat() {
        System.out.println("Animal eats");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barks");
    }
}

```
### Aggregation

- 一种 `has-a`
- 整体和部分可以相对独立

来源例子：
- `AggregationDemo.java`

Team you player[]
```java
	// Aggregation：外部传入
class Team {
    private Player player;

    public Team(Player player) { //构造函数参数 -> 通常用于初始化成员变量
        this.player = player; //get instance 外部引用
    }
}

Player p = new Player("Tom");
Team team = new Team(p);

```
### Composition

- 也是 `has-a`
- 但拥有关系更强，生命周期更紧
```java
// Composition：内部创建
class Room {
    public void show() {
    }
}

class House {
    private Room room;

    public House() {
        room = new Room(); // create inside 
    }
}
House h = new House(); //自动创建room


```
### Association

- 更宽泛的“有关联”
- 没有从属关系
	```java
	// Association：
class Student {
    public void askQuestion() {
        //System.out.println("Student asks a question"); 
    }
}

class Teacher {
    public void teach(Student student) { // 方法参数 -> 通常用于方法临时处理
        student.askQuestion(); // association 外部引用 
        System.out.println("Teacher teaches student");
    }
}
	Student s = new Student();
	Teacher t = new Teacher();
	
	t.teach(s);

	```
## 13. equals 规则

来源例子：
- `Intern.java`

重写签名必须是：

```java
public boolean equals(Object obj)
```

而不是：

```java
public boolean equals(Intern other)
```

## 14. 易错点辨析

### 易错 1：返回类型属于 method signature

不对。

### 易错 2：父类 / 接口不能作为变量类型

不对。

恰恰可以，这正是多态基础。

### 易错 3：aggregation 是 is-a

不对。

aggregation 是 has-a。

### 易错 4：final class 不能创建对象

不对。

它只是不能被继承。

