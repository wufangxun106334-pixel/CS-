# Thinking in Objects

对应课件：

- `10 - Thinking in Objects`

## 对应源码

- [Counter.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_10_examples/Counter.java)
- [AggregationDemo.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_10_examples/AggregationDemo.java)
- [Person.java](/Users/alex/Documents/COMP30820_Java/comp30820/src/test/java/lec_10_examples/Person.java)

## 1. String 是 immutable

课件重点：

- `String` 对象创建后内容不能改
- 变量重新赋值，不是修改原字符串，而是指向新对象

### 例子

```java
String s = "Java";
s = "HTML";
```

这不是修改 `"Java"`，而是让 `s` 指向新对象 `"HTML"`。

## 2. StringBuilder 是 mutable

如果你需要频繁修改字符串，更适合 `StringBuilder`。

## 3. 类设计思维

这讲的重点不是语法，而是：

- 什么该是类
- 什么该是字段
- 什么该是方法
- 哪些数据应该封装

## 4. static / final / identity

来源示例：
- `lec_10_examples/Counter.java`

### Demo

```java
private static int total = 0;
private final int id;

public Counter() {
    id = ++total;
}
```

### 解释

- `static total`：所有对象共享总计数
- `final id`：每个对象自己的 id，一旦赋值就不能改

## 5. Aggregation 聚合

来源示例：
- `lec_10_examples/AggregationDemo.java`

### Demo

```java
class Department {
    private final String deptName;
    private final Student[] students;

    public Department(String deptName, Student[] students) {
        this.deptName = deptName;
        this.students = students;
    }
}
```

### 核心理解

- `Department` 不继承 `Student`
- 它是“拥有一组 Student”
- 这是 has-a 关系，不是 is-a

## 6. 类方法 vs 实例方法

同样来自 `AggregationDemo.java`：

```java
Student s2 = Student.createClassmate("Bob"); // 类方法
Student s3 = s1.createFriend("Charlie");     // 实例方法
```

### 区分

- 类方法：通过类名调用
- 实例方法：通过对象调用

## 7. 易错点辨析

### 易错 1：重新给 String 变量赋值就是修改原对象

不对。

是让变量指向了新的字符串对象。

### 易错 2：聚合和继承差不多

不对。

- 聚合：has-a
- 继承：is-a

### 易错 3：`final` 变量等于全局常量

不对。

`final` 只表示“赋值后不能再改”，不等于一定是 `static`。
