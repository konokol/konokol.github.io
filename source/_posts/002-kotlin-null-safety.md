---
title: Kotlin 空安全
category: Kotlin
date: 2026-10-27 15:38:52
tags: Kotlin
description: Kotlin 空安全的核心目标是：把空指针异常从运行时尽可能提前到编译期。它通过类型系统区分「可空类型」和「非空类型」，强制开发者在编译期处理可能的 null。
---

# Kotlin 空安全（Null Safety）详解

Kotlin 空安全的核心目标是：**把空指针异常（NPE）从运行时尽可能提前到编译期**。它通过类型系统区分「可空类型」和「非空类型」，强制开发者在编译期处理可能的 `null`。

---

## 一、核心概念：可空类型与非空类型

Kotlin 中所有类型默认**非空**，不能赋值为 `null`：

```kotlin
var name: String = "Alice"
// name = null // 编译错误
```

需要允许 `null` 时，在类型后加 `?`：

```kotlin
var name: String? = "Alice"
name = null // 合法
```

**关键区别**：
- `String`：非空，永远不会是 `null`，可直接调用成员。
- `String?`：可空，可能是 `null`，调用成员前必须处理。

---

## 二、安全调用操作符 `?.`

如果接收者非空，调用方法/属性；否则返回 `null`，不抛异常。

```kotlin
val name: String? = "Alice"
println(name?.length) // 5

val nullName: String? = null
println(nullName?.length) // null
```

链式调用：

```kotlin
val length = user?.address?.city?.length
```

任何一环为 `null`，整个表达式返回 `null`。

---

## 三、Elvis 操作符 `?:`

如果左侧为 `null`，返回右侧默认值。

```kotlin
val name: String? = null
val display = name ?: "Unknown" // "Unknown"

val length = name?.length ?: 0 // 0
```

可与 `return`、`throw` 结合：

```kotlin
fun process(name: String?) {
    val n = name ?: return
    val m = name ?: throw IllegalArgumentException("name is null")
}
```

---

## 四、非空断言 `!!`

强制将可空类型转为非空，如果为 `null` 则抛 `NullPointerException`（准确说是 `KotlinNullPointerException`）。

```kotlin
val name: String? = null
val length = name!!.length // 运行时抛 NPE
```

**使用建议**：
- 仅在确定非空时使用，并加注释。
- 尽量避免，优先用 `?.`、`?:`、`requireNotNull`。

---

## 五、安全转换 `as?`

类型转换失败返回 `null`，而不是抛 `ClassCastException`。

```kotlin
val obj: Any = "hello"
val str: String? = obj as? String // "hello"
val num: Int? = obj as? Int       // null
```

---

## 六、可空接收者扩展

可以为可空类型定义扩展函数，内部处理 `null`：

```kotlin
fun String?.orEmpty(): String = this ?: ""

val s: String? = null
println(s.orEmpty()) // ""
```

标准库常用：
- `String?.orEmpty()`
- `Collection<T>?.orEmpty()`
- `T?.let { }`

---

## 七、`let`、`run`、`also`、`apply` 处理可空

```kotlin
val name: String? = "Alice"

// let：非空时执行
name?.let { println(it.length) }

// run：非空时执行并返回结果
val length = name?.run { this.length } ?: 0

// also：非空时执行副作用
name?.also { println("name: $it") }
```

`?.let` 是最常用的可空处理模式。

---

## 八、集合与可空元素

```kotlin
val list: List<String?> = listOf("a", null, "b")
```

- `List<String?>`：集合非空，元素可空。
- `List<String>?`：集合可空，元素非空。
- `List<String?>?`：集合和元素都可空。

处理：

```kotlin
list.filterNotNull() // List<String>
```

---

## 九、平台类型 `T!` 与 Java 互操作

Java 类型在 Kotlin 中默认是**平台类型** `T!`，可空性未知：

```java
// Java
public String getName() { 
  return null; 
  }
```

```kotlin
val name = javaObj.name // String!
val length = name.length // 编译通过，运行时可能 NPE
```

**安全做法**：
- 对无注解的 Java API，用可空类型接收：`String?`。
- 或用 `?.`、`?:`、`requireNotNull` 兜底。

```kotlin
val safeName: String? = javaObj.name
val length = safeName?.length ?: 0
```

---

## 十、`lateinit` 与 `by lazy`

### `lateinit`

用于非空 `var`，延迟初始化，不能用于基本类型和可空类型。

```kotlin
lateinit var name: String

fun init() {
    name = "Alice"
}

// 访问未初始化抛 UninitializedPropertyAccessException
```

判断是否初始化：

```kotlin
if (::name.isInitialized) { ... }
```

### `by lazy`

用于 `val`，首次访问时初始化，线程安全可配置。

```kotlin
val name: String by lazy { "Alice" }
```

**对比**：

| 特性 | `lateinit` | `by lazy` |
|------|-----------|-----------|
| 适用类型 | 非空引用类型 | 任意 |
| 可变性 | `var` | `val` |
| 初始化 | 手动 | 首次访问 |
| 线程安全 | 否 | 可配置 |
| 可重置 | 否 | 否 |

---

## 十一、`requireNotNull`、`checkNotNull`

```kotlin
fun process(name: String?) {
    val n = requireNotNull(name) { "name is null" } // 抛 IllegalArgumentException
    val m = checkNotNull(name) { "name is null" }   // 抛 IllegalStateException
}
```

- `requireNotNull`：参数校验，抛 `IllegalArgumentException`。
- `checkNotNull`：状态校验，抛 `IllegalStateException`。

对比 `!!`：可自定义异常消息，更易排查。

---

## 十二、空安全与泛型

```kotlin
fun <T> List<T>.firstOrNull(): T? = if (isEmpty()) null else this[0]
```

泛型 `T` 默认非空，`T?` 可空。注意：

```kotlin
fun <T> process(item: T) { }
process<String?>(null) // 合法，T 可以是可空类型
```

---

## 十三、空安全与 `when`

```kotlin
fun describe(name: String?): String = when (name) {
    null -> "no name"
    else -> "name: $name"
}
```

`when` 对 `null` 分支可智能转换。

---

## 十四、智能转换（Smart Cast）

Kotlin 在确定非空后自动转换类型：

```kotlin
fun process(name: String?) {
    if (name != null) {
        println(name.length) // 智能转换为 String
    }
}
```

支持：
- `if (x != null)`
- `x?.let { }`
- `when (x) { is Type -> }`
- `requireNotNull(x)`

不支持：
- `var` 属性（可能被其他线程修改）。
- 自定义 getter 属性。

```kotlin
class Foo {
    var name: String? = null

    fun process() {
        if (name != null) {
            // println(name.length) // 编译错误，var 属性不能智能转换
            println(name?.length)
        }
    }
}
```

---

## 十五、空安全与反射

```kotlin
val kClass = String::class
println(kClass.isMarkedNullable) // false
val nullableClass = String::class.createType(nullable = true)
println(nullableClass.isMarkedNullable) // true
```

---

## 十六、常见面试题与答案

**Q1：Kotlin 空安全是什么？**  
A：通过类型系统区分可空和非空类型，强制编译期处理 `null`，减少运行时 NPE。

**Q2：`String` 和 `String?` 区别？**  
A：`String` 非空，不能赋 `null`；`String?` 可空，需安全处理。

**Q3：`?.`、`?:`、`!!` 区别？**  
A：`?.` 安全调用，`null` 返回 `null`；`?:` 提供默认值；`!!` 强制非空，`null` 抛 NPE。

**Q4：`as?` 和 `as` 区别？**  
A：`as?` 失败返回 `null`；`as` 失败抛 `ClassCastException`。

**Q5：平台类型 `T!` 是什么？**  
A：Java 类型在 Kotlin 中的表示，可空性未知，调用可能 NPE。

**Q6：`lateinit` 和 `by lazy` 区别？**  
A：`lateinit` 用于 `var`，手动初始化；`by lazy` 用于 `val`，首次访问初始化。

**Q7：`!!` 和 `requireNotNull` 区别？**  
A：`!!` 抛 NPE；`requireNotNull` 抛 `IllegalArgumentException`，可自定义消息。

**Q8：Kotlin 能完全避免 NPE 吗？**  
A：不能。平台类型、`!!`、Java 互操作、反射仍可能 NPE。

**Q9：智能转换什么时候失效？**  
A：`var` 属性、自定义 getter、跨线程修改时。

**Q10：可空接收者扩展是什么？**  
A：为可空类型定义扩展函数，如 `String?.orEmpty()`。

**Q11：如何处理集合中的 null？**  
A：用 `filterNotNull()`、`mapNotNull()`。

**Q12：`let` 如何配合空安全？**  
A：`name?.let { }`，非空时执行。

**Q13：`when` 如何处理 null？**  
A：`when (name) { null -> ...; else -> ... }`。

**Q14：Kotlin 空安全和 Java 对比？**  
A：Java 无编译期空检查，NPE 运行时才暴露；Kotlin 编译期强制处理。

**Q15：`checkNotNull` 和 `requireNotNull` 区别？**  
A：`requireNotNull` 抛 `IllegalArgumentException`（参数校验）；`checkNotNull` 抛 `IllegalStateException`（状态校验）。

---

## 十七、完整示例

```kotlin
data class User(val name: String?, val address: Address?)
data class Address(val city: String?)

fun main() {
    val user = User("Alice", null)

    // 安全调用链
    val city = user.address?.city
    println(city) // null

    // Elvis 默认值
    val cityName = user.address?.city ?: "Unknown"
    println(cityName) // Unknown

    // let 处理非空
    user.name?.let { println("name: $it") }

    // 安全转换
    val obj: Any = "hello"
    val str: String? = obj as? String

    // requireNotNull
    val name = requireNotNull(user.name) { "name required" }
}
```

---

## 十八、总结

- Kotlin 通过 `?` 区分可空/非空类型，编译期强制处理 `null`。
- `?.` 安全调用、`?:` 默认值、`!!` 强制非空、`as?` 安全转换。
- 平台类型 `T!` 是 Java 互操作的 NPE 风险点。
- `lateinit` 用于 `var` 延迟初始化，`by lazy` 用于 `val` 首次访问初始化。
- 智能转换在 `var` 属性、自定义 getter 时失效。
- 可空接收者扩展、`let`、`filterNotNull` 是常用处理手段。
- 面试高频：`?.`/`?:`/`!!` 区别、平台类型、智能转换、`lateinit` vs `lazy`。
