---
title: Kotlin 中的集合
category: tech
date: 2026-11-12 11:16:44
tags: Kotlin
description:
---


Kotlin 集合是面试高频模块，核心知识点包括：**只读与可变、集合层次结构、与 Java 集合的关系、常用操作符、Sequence 惰性求值、性能与线程安全**。

---

## 一、集合层次结构

Kotlin 集合分两大类：**只读集合**和**可变集合**。

```text
Iterable
├── Collection
│   ├── List
│   │   └── MutableList
│   └── Set
│       └── MutableSet
└── Map（独立，不继承 Collection）
    └── MutableMap
```

**关键点**：

- `List`、`Set`、`Map` 是**只读接口**，只有读方法。
- `MutableList`、`MutableSet`、`MutableMap` 继承只读接口，增加写方法。
- 只读**不等于不可变**：底层可能被 Java 代码修改，只是 Kotlin 编译器不允许你直接写。

```kotlin
val list: List<Int> = mutableListOf(1, 2, 3)
// list.add(4) // 编译错误
(list as MutableList).add(4) // 运行时可绕过，但不推荐
```

---

## 二、创建集合

| 方式 | 示例 | 说明 |
| ------ | ------ | ------ |
| `listOf` | `listOf(1, 2, 3)` | 只读 List |
| `mutableListOf` | `mutableListOf(1, 2, 3)` | 可变 List |
| `setOf` | `setOf(1, 2, 3)` | 只读 Set |
| `mutableSetOf` | `mutableSetOf(1, 2)` | 可变 Set |
| `mapOf` | `mapOf("a" to 1)` | 只读 Map |
| `mutableMapOf` | `mutableMapOf("a" to 1)` | 可变 Map |
| `arrayListOf` | `arrayListOf(1, 2)` | 可变 ArrayList |
| `hashSetOf` | `hashSetOf(1, 2)` | 可变 HashSet |
| `hashMapOf` | `hashMapOf("a" to 1)` | 可变 HashMap |
| `emptyList()` | `emptyList<Int>()` | 空只读 List |
| `listOfNotNull` | `listOfNotNull(a, b)` | 过滤 null |

**加分点**：

- `listOf()` 返回 `ArraysUtilJVM$ArrayList` 或 `SingletonList`，不可变。
- `mutableListOf()` 返回 `ArrayList`。
- `mapOf()` 返回 `LinkedHashMap`。

---

## 三、只读 vs 可变

| 操作 | `List` | `MutableList` |
| ------ | -------- | --------------- |
| 读取 | `get`、`size`、`forEach` | 同左 |
| 添加 | ❌ | `add`、`addAll` |
| 删除 | ❌ | `remove`、`clear` |
| 修改 | ❌ | `set` |
| 迭代 | ✅ | ✅ |

只读集合真的不可变吗？  

不。Kotlin 只读接口只是编译期限制，底层可能是可变集合，Java 代码可修改。需要真正不可变用 `Collections.unmodifiableList` 或第三方库（如 Guava ImmutableList）。

---

## 四、集合与 Java 的关系

Kotlin 集合编译后映射为 Java 集合：

| Kotlin | Java |
| -------- | ------ |
| `List` | `java.util.List` |
| `MutableList` | `java.util.List` |
| `Set` | `java.util.Set` |
| `Map` | `java.util.Map` |
| `Collection` | `java.util.Collection` |

**关键点**：

- Kotlin 只读集合和可变集合编译后都是 Java 可变接口，区别只在编译期。
- Java 调用 Kotlin 返回的 `List` 时，可以修改，Kotlin 不保证不可变。
- Java 的 `List` 在 Kotlin 中视为平台类型 `(Mutable)List<T!>!`，可读可写。

---

## 五、常用操作符

### 1. 转换

| 操作符 | 作用 | 示例 |
| -------- | ------ | ------ |
| `map` | 元素转换 | `list.map { it * 2 }` |
| `flatMap` | 转换并展平 | `list.flatMap { listOf(it, it) }` |
| `filter` | 过滤 | `list.filter { it > 0 }` |
| `filterNot` | 反向过滤 | `list.filterNot { it > 0 }` |
| `mapNotNull` | 转换并过滤 null | `list.mapNotNull { it.toIntOrNull() }` |
| `flatten` | 展平嵌套集合 | `listOf(listOf(1), listOf(2)).flatten()` |
| `zip` | 合并两个集合 | `listA.zip(listB) { a, b -> a + b }` |
| `unzip` | 拆分 Pair 集合 | `pairs.unzip()` |

### 2. 聚合

| 操作符 | 作用 | 说明 |
| -------- | ------ | ------ |
| `fold` | 带初始值累积 | `list.fold(0) { acc, i -> acc + i }` |
| `reduce` | 无初始值累积 | 空集合抛异常 |
| `sum` | 求和 | 数字集合 |
| `average` | 平均值 | 数字集合 |
| `count` | 计数 | 可带条件 |
| `maxOrNull` / `minOrNull` | 最大/最小 | 空集合返回 null |
| `maxByOrNull` / `minByOrNull` | 按条件最大/最小 | 空集合返回 null |

### 3. 排序

| 操作符 | 作用 |
| -------- | ------ |
| `sorted` | 自然排序 |
| `sortedBy` | 按选择器排序 |
| `sortedDescending` | 降序 |
| `sortedByDescending` | 按选择器降序 |
| `sortedWith` | 自定义 Comparator |

### 4. 分组与关联

| 操作符 | 作用 | 返回 |
| -------- | ------ | ------ |
| `groupBy` | 按 key 分组 | `Map<K, List<T>>` |
| `associate` | 自定义键值对 | `Map<K, V>` |
| `associateBy` | 指定 key，value 是元素 | `Map<K, T>` |
| `associateWith` | 指定 value，key 是元素 | `Map<T, V>` |
| `partition` | 按条件分两组 | `Pair<List<T>, List<T>>` |

### 5. 查找与判断

| 操作符 | 作用 |
| -------- | ------ |
| `find` / `firstOrNull` | 找第一个匹配 |
| `any` | 是否有匹配 |
| `all` | 是否全部匹配 |
| `none` | 是否无匹配 |
| `contains` | 是否包含 |
| `isEmpty` / `isNotEmpty` | 是否为空 |

### 6. 取子集

| 操作符 | 作用 |
| -------- | ------ |
| `take(n)` | 取前 n 个 |
| `takeLast(n)` | 取后 n 个 |
| `drop(n)` | 去掉前 n 个 |
| `dropLast(n)` | 去掉后 n 个 |
| `slice(range)` | 取指定范围 |
| `distinct` | 去重 |
| `distinctBy` | 按选择器去重 |

---

## 六、Sequence 惰性求值

### 1. 区别

| 特性 | `Iterable` | `Sequence` |
| ------ | ----------- | ----------- |
| 求值 | 立即求值 | 惰性求值 |
| 中间操作 | 生成新集合 | 返回新 Sequence |
| 内存 | 每步生成中间集合 | 无中间集合 |
| 性能 | 小数据量快 | 大数据量、链式多时快 |
| 终端操作 | 无 | `toList`、`first`、`count` 触发执行 |

### 2. 示例

```kotlin
// Iterable：每步生成新集合
val result = (1..1_000_000)
    .map { it * 2 }       // 生成 100 万列表
    .filter { it % 3 == 0 } // 再生成列表
    .take(10)             // 再生成列表

// Sequence：惰性，只计算前 10 个
val result = (1..1_000_000).asSequence()
    .map { it * 2 }
    .filter { it % 3 == 0 }
    .take(10)
    .toList()             // 终端操作，触发执行
```

### 3. 什么时候用 Sequence

- 大数据量（十万以上）。
- 链式操作多。
- 只需部分结果（`take`、`first`）。
- 避免中间集合内存开销。

**加分点**：Sequence 也有开销，小数据量可能比 Iterable 慢；`sequence { }` 构建器用 `yield` 发射元素。

---

## 七、集合与数组

| 特性 | 数组 | 集合 |
| ------ | ------ | ------ |
| 长度 | 固定 | 可变（Mutable） |
| 元素类型 | 基本类型有专用数组（`IntArray`） | 泛型 |
| 协变 | 数组协变（有风险） | 只读集合协变 |
| 性能 | 更高 | 略低 |
| 使用场景 | 底层、性能敏感 | 业务逻辑 |

```kotlin
val array = intArrayOf(1, 2, 3)
val list = listOf(1, 2, 3)
array[0] = 10  // 可变
// list[0] = 10 // 编译错误
```

---

## 八、集合与线程安全

- Kotlin 标准集合**非线程安全**。
- 多线程场景用：
  - `java.util.concurrent.ConcurrentHashMap`
  - `CopyOnWriteArrayList`
  - `Collections.synchronizedList`
- 协程中用 `Mutex` 保护共享集合，或用 `Channel`、`Flow` 传递数据。

---

## 九、常见面试题与答案

**Q1：Kotlin 只读集合真的不可变吗？**  
A：不。只读只是编译期接口限制，底层可能是可变集合，Java 可修改。真正不可变用 `Collections.unmodifiableList` 或 Guava。

**Q2：`List` 和 `MutableList` 编译后有什么区别？**  
A：都映射为 Java `List`，区别只在编译期。Java 调用 Kotlin 返回的 `List` 仍可修改。

**Q3：`listOf` 和 `mutableListOf` 区别？**  
A：`listOf` 返回只读接口，`mutableListOf` 返回可变 `ArrayList`。

**Q4：`Sequence` 和 `Iterable` 区别？**  
A：`Iterable` 立即求值生成中间集合；`Sequence` 惰性逐元素，无中间集合，适合大数据量。

**Q5：`fold` 和 `reduce` 区别？**  
A：`fold` 有初始值；`reduce` 无初始值，空集合抛异常。

**Q6：`associate` 和 `associateBy` 区别？**  
A：`associate` 自定义键值对；`associateBy` 指定 key，value 是元素本身。

**Q7：`map` 和 `flatMap` 区别？**  
A：`map` 一对一转换；`flatMap` 转换后展平。

**Q8：`groupBy` 返回什么？**  
A：`Map<K, List<T>>`，按 key 分组。

**Q9：Kotlin 集合和 Java 集合如何互操作？**  
A：Kotlin 只读/可变集合编译后都是 Java 集合接口；Java 的 `List` 在 Kotlin 中是平台类型，可读可写。

**Q10：如何选择 `List`、`Set`、`Map`？**  
A：有序可重复用 `List`；去重用 `Set`；键值对用 `Map`。只读优先，需要修改用 `Mutable`。

**Q11：`take` 和 `drop` 区别？**  
A：`take(n)` 取前 n 个；`drop(n)` 去掉前 n 个。

**Q12：`distinct` 和 `distinctBy` 区别？**  
A：`distinct` 按元素去重；`distinctBy` 按选择器结果去重。

**Q13：Kotlin 集合为什么分只读和可变？**  
A：编译期表达意图，减少误修改；与 Java 互操作时只读接口更安全。

**Q14：`Array` 和 `List` 区别？**  
A：数组固定长度、基本类型有专用数组、性能高；List 可变、泛型、业务常用。

**Q15：`zip` 和 `unzip` 作用？**  
A：`zip` 合并两个集合为 Pair 列表；`unzip` 拆分 Pair 列表为两个列表。

---

## 十、速记口诀

| 主题 | 口诀 |
| ------ | ------ |
| 集合层次 | List/Set/Map 只读，Mutable 可变 |
| 创建 | listOf 只读，mutableListOf 可变 |
| 与 Java | 编译后都是 Java 接口，只读仅编译期 |
| Sequence | 惰性求值，无中间集合，大数据量 |
| fold/reduce | fold 有初始，reduce 无初始 |
| associate | associate 自定义，associateBy 指定 key |
| groupBy | 分组返回 Map<K, List<T>> |
| 线程安全 | 标准集合非线程安全，用并发集合 |
