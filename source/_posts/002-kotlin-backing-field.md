---
title: Kotlin 中的幕后字段
category: Kotlin
date: 2026-10-26 17:15:32
tags: Kotlin
description: 幕后字段是 Kotlin 属性在需要存储实际值时，编译器自动生成的私有字段。它用 `field` 关键字在自定义 getter/setter 中访问。
---

# Kotlin 幕后字段（Backing Field）详解

**幕后字段**是 Kotlin 属性在**需要存储实际值**时，编译器自动生成的私有字段。它用 `field` 关键字在自定义 getter/setter 中访问。

核心结论：
- **有自定义 getter/setter 且用到 `field`** → 有幕后字段。
- **计算属性（只依赖其他属性）** → 没有幕后字段。
- **扩展属性** → 永远没有幕后字段。
- **抽象属性** → 没有幕后字段。

---

## 一、什么是幕后字段

Kotlin 属性本质上是一组 getter/setter，但很多属性需要**真正存一个值**。这个实际存值的字段就是幕后字段。

```kotlin
class User {
    var name: String = "Alice"
}
```

编译后大致等价：

```java
public final class User {
    private String name = "Alice";   // ← 幕后字段

    public String getName() { return name; }
    public void setName(String value) { name = value; }
}
```

`name` 这个属性有幕后字段，因为需要保存值。

---

## 二、什么时候有幕后字段

满足以下条件之一，属性**自动生成幕后字段**：

1. 属性有初始化器：`var name = "Alice"`。
2. 属性在 `init` 块中被赋值。
3. 自定义 getter/setter 中使用了 `field`。

```kotlin
class Foo {
    var a = 1                     // 有幕后字段
    var b: Int = 0                // 有幕后字段
        get() = field + 1         // 使用 field
        set(value) { field = value }

    var c: Int                    // 没有幕后字段
        get() = a + b
        set(value) { a = value }
}
```

`c` 是计算属性，值来自 `a` 和 `b`，不需要单独存储，所以没有幕后字段。

---

## 三、`field` 关键字

只能在自定义 getter/setter 中使用 `field`，它指向幕后字段。

```kotlin
class User {
    var name: String = ""
        get() = field.uppercase()
        set(value) {
            require(value.isNotBlank())
            field = value.trim()
        }
}
```

- `get()` 中 `field` 是当前存储值。
- `set()` 中 `field = value` 是写入幕后字段。
- 不能在 getter/setter 外使用 `field`。

**注意**：如果在 getter 中直接写 `name`，会递归调用 getter，导致栈溢出。必须用 `field`。

```kotlin
var name: String = ""
    get() = name  // 错误：无限递归
```

---

## 四、没有幕后字段的情况

### 4.1 计算属性

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height   // 没有幕后字段
}
```

`area` 每次访问都计算，不存储。

### 4.2 扩展属性

```kotlin
val String.lastChar: Char
    get() = this[length - 1]
```

扩展属性不能有幕后字段，因为不属于任何类实例。

### 4.3 抽象属性

```kotlin
interface Foo {
    val name: String   // 没有幕后字段，由实现类提供
}
```

### 4.4 委托属性

```kotlin
val name: String by lazy { "Alice" }
```

值存在委托对象里，属性本身没有幕后字段。

### 4.5 只读属性且无初始化器

```kotlin
val x: Int
    get() = 42   // 没有幕后字段
```

---

## 五、幕后字段与 `lateinit`、`const`

| 特性 | 幕后字段 | 说明 |
|------|---------|------|
| 普通 `var` 有初始化器 | 有 | 存值 |
| 普通 `val` 有初始化器 | 有 | 存值 |
| 计算属性 | 无 | 每次计算 |
| 扩展属性 | 无 | 不属于实例 |
| `lateinit var` | 有 | 延迟初始化，底层仍有字段 |
| `const val` | 无（编译期常量） | 内联到调用处，无字段 |
| 委托属性 | 无 | 值在委托对象中 |

---

## 六、幕后字段与 Java 对比

| Kotlin | Java |
|--------|------|
| 属性 + 幕后字段 + getter/setter | 字段 + 手写 getter/setter |
| `field` 关键字 | `this.field` |
| 计算属性无字段 | 方法无字段 |
| `const val` 编译期常量 | `static final` 常量 |

Kotlin 的幕后字段相当于 Java 中私有字段，但由编译器自动管理。

---

## 七、常见面试题与答案

**Q1：什么是幕后字段？**  
A：Kotlin 属性在需要存储实际值时，编译器自动生成的私有字段，用 `field` 在 getter/setter 中访问。

**Q2：什么时候有幕后字段？**  
A：属性有初始化器、在 `init` 中赋值、或 getter/setter 中使用 `field` 时。

**Q3：什么时候没有幕后字段？**  
A：计算属性、扩展属性、抽象属性、委托属性、`const val`。

**Q4：`field` 能用在 getter/setter 外吗？**  
A：不能，只能在自定义访问器中使用。

**Q5：为什么 getter 中不能直接写属性名？**  
A：会递归调用 getter，导致栈溢出，必须用 `field`。

**Q6：扩展属性能有幕后字段吗？**  
A：不能，扩展属性不属于实例，无法存储状态。

**Q7：`lateinit` 有幕后字段吗？**  
A：有，底层仍需要字段存储值，只是延迟初始化。

**Q8：`const val` 有幕后字段吗？**  
A：没有，编译期常量内联到调用处。

**Q9：委托属性有幕后字段吗？**  
A：没有，值存在委托对象中。

**Q10：幕后字段和 Java 字段区别？**  
A：Kotlin 幕后字段由编译器生成，用 `field` 访问；Java 字段直接声明。

---

## 八、完整示例

```kotlin
class User {
    // 有幕后字段
    var name: String = "Alice"
        get() = field.uppercase()
        set(value) {
            require(value.isNotBlank())
            field = value.trim()
        }

    // 没有幕后字段（计算属性）
    val nameLength: Int
        get() = name.length

    // 有幕后字段（lateinit）
    lateinit var email: String
}

// 扩展属性：没有幕后字段
val String.lastChar: Char
    get() = this[length - 1]

fun main() {
    val user = User()
    println(user.name)        // ALICE
    println(user.nameLength)  // 5
}
```

---

## 九、总结

- 幕后字段是属性实际存储值的私有字段，用 `field` 访问。
- 有初始化器、`init` 赋值、使用 `field` 时生成幕后字段。
- 计算属性、扩展属性、抽象属性、委托属性、`const val` 没有幕后字段。
- `field` 只能在自定义 getter/setter 中使用，避免递归。
- 面试高频：何时有/无幕后字段、`field` 用法、递归问题、扩展属性。