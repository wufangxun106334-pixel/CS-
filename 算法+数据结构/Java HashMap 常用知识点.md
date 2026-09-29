
#
---

## Java HashMap 常用知识点

### 1. 创建 HashMap

```java
HashMap<String, String> map = new HashMap<>();
```

### 2. put(key, value) — 添加/更新键值对

```java
map.put("1", "apple");    // 添加键 "1"，值 "apple"
map.put("2", "banana");   // 添加键 "2"，值 "banana"
```

### 3. get(key) — 根据键获取值

```java
map.get("1");  // 返回 "apple"
map.get("5");  // 返回 null（键不存在）
```

### 4. containsKey(key) — 判断键是否存在

```java
if (map.containsKey("1")) {
    System.out.println("Key 1 exists in the map: " + map.get("1"));
}
```

### 5. getOrDefault(key, defaultValue) — 获取值（不存在时返回默认值）

```java
map.getOrDefault("3", "not found");  // 返回 "not found"
map.getOrDefault("1", "not found");  // 返回 "apple"
```

### 6. 频率统计模式（核心！）

```java
map1.put(num, map1.getOrDefault(num, 0) + 1);
```

**拆解**：
1. `getOrDefault(num, 0)` → 获取键 `num` 的当前频率（不存在则返回 0）
2. `+ 1` → 频率加 1
3. `put(num, ...)` → 把新频率存回键 `num`

**本质**：初始化 + 递增，一行搞定频率统计，避免 if-else。

```java
// 等价写法
if (map1.containsKey(num)) {
    map1.put(num, map1.get(num) + 1);
} else {
    map1.put(num, 1);
}
```

### 7. 遍历数组统计频率

```java
int[] nums = {1, 2, 3, 2, 1, 3, 1, 1};
HashMap<Integer, Integer> freq = new HashMap<>();

for (int num : nums) {
    freq.put(num, freq.getOrDefault(num, 0) + 1);
}

// 结果：{1=4, 2=2, 3=2}
```

### 8. 字符串拼接输出

```java
System.out.println("Key 1: " + map.get("1"));
// 输出：Key 1: apple
// 不会自动加空格，注意手动加 " " 或 ": "
```

---

## HashMap 小结

| 方法 | 作用 | 示例 |
|------|------|------|
| `put(K, V)` | 添加/更新键值对 | `map.put("1", "apple")` |
| `get(K)` | 获取值 | `map.get("1")` → `"apple"` |
| `getOrDefault(K, V)` | 获取值，不存在返回默认值 | `map.getOrDefault("3", "none")` |
| `containsKey(K)` | 判断键是否存在 | `map.containsKey("1")` → `true` |
| `size()` | 获取键值对数量 | `map.size()` |
| `keySet()` | 获取所有键 | `map.keySet()` |
| `values()` | 获取所有值 | `map.values()` |
| `entrySet()` | 获取所有键值对 | `map.entrySet()` |

---

## HashMap<Integer, Integer> — 为什么写两个 Integer？

### 泛型语法

```java
HashMap<K, V> map = new HashMap<>();
//      ↑ ↑
//      │ └── V (Value)：值的类型
//      └──── K (Key)：键的类型
```

### 拆解示例

```java
HashMap<Integer, Integer> map = new HashMap<>();
//         ↑       ↑
//         │       └── 值：频率（整数）
//         └────────── 键：数字本身（整数）
```

**两个 Integer 各有职责**：
- 第一个 `Integer` = 键的类型（Key）
- 第二个 `Integer` = 值的类型（Value）

### 类比：储物柜

```java
HashMap<Integer, Integer> map = new HashMap<>();
// 柜子编号：整数  |  柜子里放的：也是整数

HashMap<String, String> map = new HashMap<>();
// 柜子编号：字符串  |  柜子里放的：也是字符串
```

### 不同场景的声明

| 场景 | 声明 | 含义 |
|------|------|------|
| 频率统计 | `HashMap<Integer, Integer>` | 数字 → 出现次数 |
| 字典映射 | `HashMap<String, String>` | 单词 → 翻译 |
| 缓存 | `HashMap<Integer, String>` | ID → 名字 |
| 嵌套 | `HashMap<String, List<Integer>>` | 单词 → 数字列表 |

---

## HashSet vs HashMap 对比

### 语法对比

```java
// HashSet —— 只需要一个泛型参数（元素类型）
HashSet<Integer> set = new HashSet<>();

// HashMap —— 需要两个泛型参数（键类型 + 值类型）
HashMap<Integer, Integer> map = new HashMap<>();
```

| 对比项 | HashSet | HashMap |
|--------|---------|---------|
| 泛型参数 | **1个**：`<E>` | **2个**：`<K, V>` |
| 存储结构 | 只存**元素**（Key） | 存**键值对**（Key + Value） |
| 底层实现 | 内部用一个空的 Object 作为 Value 的 HashMap | 直接用 HashMap |

### 核心方法对比

| 操作 | HashSet | HashMap |
|------|---------|---------|
| 添加 | `set.add(1)` | `map.put(1, 100)` |
| 查找 | `set.contains(1)` | `map.containsKey(1)` |
| 删除 | `set.remove(1)` | `map.remove(1)` |
| 获取值 | ❌ 没有（只存 Key） | `map.get(1)` → 返回 Value |
| 大小 | `set.size()` | `map.size()` |
| 判空 | `set.isEmpty()` | `map.isEmpty()` |

### 功能对比

**HashSet**：只关心 **"有没有"**
```java
Set<Integer> set = new HashSet<>();
set.add(1);  // 只知道 1 存在
set.add(1);  // add 重复元素无效，还是只有一个 1
set.size();  // 返回 1，不是 2！
```

**HashMap**：关心 **"有多少"**
```java
Map<Integer, Integer> map = new HashMap<>();
map.put(1, map.getOrDefault(1, 0) + 1);  // {1=1}
map.put(1, map.getOrDefault(1, 0) + 1);  // {1=2}
map.get(1);  // 返回 2
```

### 适用场景

| 场景 | 用谁 | 示例 |
|------|------|------|
| 判断元素是否存在 | `HashSet` | 快速去重 |
| 统计频率 | `HashMap` | `getOrDefault + 1` |
| 映射关系 | `HashMap` | 单词 → 翻译 |
| 两个数组是否有交集 | `HashSet` | 元素查找 |

### 常用声明方式

```java
// HashSet
HashSet<String> set = new HashSet<>();

// HashMap
HashMap<String, Integer> map = new HashMap<>();

// 都可以用接口类型声明（推荐）
Set<String> set = new HashSet<>();
Map<String, Integer> map = new HashMap<>();
```

> 💡 **记住**：`HashSet` 是"单元素"容器，`HashMap` 是"双元素"容器（键+值）。频率统计首选 `HashMap`，去重首选 `HashSet`。