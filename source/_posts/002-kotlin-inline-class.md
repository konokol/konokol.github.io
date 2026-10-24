---
title: Kotlin 内联类（Inline Class / Value Class）
category: Kotlin
date: 2026-10-24 10:32:38
tags: Kotlin
description: 内联类（inline class）是 Kotlin 1.3 引入的实验特性，Kotlin 1.5 后改名为 值类（value class），并推荐使用 @JvmInline 注解。它的核心目标是：用包装类型提供类型安全，同时在多数场景下避免运行时装箱开销。
---

# Kotlin 内联类（Inline Class / Value Class）

内联类（`inline class`）是 Kotlin 1.3 引入的实验特性，Kotlin 1.5 后改名为 **值类（`value class`）**，并推荐使用 `@JvmInline` 注解。它的核心目标是：**用包装类型提供类型安全，同时在多数场景下避免运行时装箱开销**。

---

## 1. 为什么需要内联类？

假设我们要表示“用户 ID”，如果直接用 `Long`，容易把其他 `Long` 参数误传：

```kotlin
fun fetchUser(id: Long) { }
fun deleteOrder(orderId: Long) { }

fetchUser(orderId) // 编译通过，但逻辑错误
```

用 `data class` 包装可以类型安全，但每次创建都有对象头开销：

```kotlin
data class UserId(val id: Long) // 有对象头，堆分配
```

内联类则兼顾两者：

```kotlin
@JvmInline
value class UserId(val id: Long)
```

编译后，多数场景直接用 `long` 表示，无对象分配；但类型系统会阻止误传。

---

## 2. 语法与定义

```kotlin
@JvmInline
value class UserId(val id: Long)

@JvmInline
value class Password(val value: String) {
    init {
        require(value.length >= 6) { "Password too short" }
    }
}

@JvmInline
value class Money(val cents: Long) {
    operator fun plus(other: Money) = Money(cents + other.cents)
    fun toYuan(): Double = cents / 100.0
}
```

**规则**：
- 必须用 `@JvmInline` 注解（Kotlin 1.5+）。
- 主构造器只能有一个 `val` 属性。
- 不能有 `var` 属性，不能有幕后字段（backing field）。
- 可以实现接口。
- 不能继承其他类（只能继承 `Any`）。
- 可以有 `init` 块、方法、伴生对象。
- 不能有内部类？可以有嵌套类。

早期写法 `inline class UserId(val id: Long)` 已废弃，现在编译会提示改用 `value class`。

---

## 3. 编译原理与字节码

### 3.1 多数场景内联

```kotlin
@JvmInline
value class UserId(val id: Long)

fun fetchUser(id: UserId) { }

fun main() {
    val uid = UserId(1001L)
    fetchUser(uid)
}
```

编译后大致等价：

```java
public static final void fetchUser(long id) { }

public static final void main() {
    long uid = UserId.constructor-impl(1001L);
    fetchUser(uid);
}
```

- 方法参数变成底层 `long`。
- 局部变量也是 `long`。
- 无 `UserId` 对象分配。

### 3.2 方法名修饰（Mangling）

如果内联类有方法，编译后方法名会被修饰，避免与底层类型方法冲突：

```kotlin
@JvmInline
value class UserId(val id: Long) {
    fun show(): String = "User $id"
}
```

字节码中方法名类似 `show-impl` 或 `show-xxxxx`，Java 调用时无法直接写 `show()`。

### 3.3 装箱场景

以下情况内联类会**装箱**为包装对象：
- 作为可空类型：`UserId?`
- 作为泛型参数：`List<UserId>`、`Producer<UserId>`
- 作为接口引用：`val any: Any = UserId(1L)`
- 在数组中：`Array<UserId>`，但可用 `LongArray` 替代
- 反射中：`UserId::class`

```kotlin
val a: UserId = UserId(1L)   // 不装箱
val b: UserId? = UserId(1L)  // 装箱
val list = listOf(UserId(1L)) // 装箱
```

---

## 4. 限制与约束

| 限制 | 说明 |
|------|------|
| 只能一个主构造属性 | 且必须是 `val` |
| 不能有 `var` | 不能有可变属性 |
| 不能有幕后字段 | 不能有 `field` |
| 不能继承类 | 只能实现接口 |
| 不能有 `init` 块？ | 可以有 `init` 块，但不能给属性赋值（因为无幕后字段） |
| 不能有内部类？ | 可以有嵌套类，但不能有 `inner` 内部类 |
| 不能有延迟初始化 | `lateinit` 不允许 |
| 不能有自定义 getter/setter | 属性不能有自定义访问器 |

示例：`init` 块可以校验：

```kotlin
@JvmInline
value class Password(val value: String) {
    init {
        require(value.length >= 6) { "Too short" }
    }
}
```

编译后，`init` 逻辑会插入到构造器工厂方法中。

---

## 5. 使用场景

### 5.1 类型安全包装

```kotlin
@JvmInline value class UserId(val id: Long)
@JvmInline value class OrderId(val id: Long)
@JvmInline value class ProductId(val id: Long)

fun fetchUser(id: UserId) { }
fun fetchOrder(id: OrderId) { }
```

避免 `Long` 误传。

### 5.2 单位与度量

```kotlin
@JvmInline value class Dp(val value: Float)
@JvmInline value class Sp(val value: Float)
@JvmInline value class Px(val value: Float)
```

Compose 中大量使用 `Dp`、`Sp` 等。

### 5.3 密码、邮箱等

```kotlin
@JvmInline value class Email(val value: String) {
    init { require(value.contains("@")) }
}
```

### 5.4 协程中的 `CoroutineContext.Key`

Kotlin 协程内部用 `value class` 包装 `CoroutineContext.Key`，减少开销。

---

## 6. 与 `typealias`、`data class` 对比

| 特性 | `value class` | `typealias` | `data class` |
|------|--------------|-------------|--------------|
| 类型安全 | 是 | 否（只是别名） | 是 |
| 运行时开销 | 多数内联 | 无 | 有对象头 |
| 可定义方法 | 是 | 否 | 是 |
| 可实现接口 | 是 | 否 | 是 |
| 可空/泛型装箱 | 会装箱 | 无 | 始终对象 |
| 反射可见 | 是 | 否 | 是 |

**`typealias` 只是编译期别名**，不提供类型安全：

```kotlin
typealias UserId = Long
fun fetchUser(id: UserId) { }
fetchUser(1001L) // 编译通过，无法区分
```

---

## 7. Java 互操作

Java 调用 Kotlin 的 `value class`：

```kotlin
@JvmInline
value class UserId(val id: Long)

fun fetchUser(id: UserId) { }
```

Java 侧看到的是：

```java
public static final void fetchUser(long id) { }
```

但如果你在 Java 中想构造 `UserId`，需要调用生成的工厂方法：

```java
long uid = UserId.constructor-impl(1001L);
```

方法名被修饰，Java 调用不便。因此，**给 Java 用的 API 尽量避免暴露 `value class`**，或提供普通 `Long` 重载。

可以用 `@JvmName` 修饰方法名：

```kotlin
@JvmInline
value class UserId(val id: Long) {
    @JvmName("show")
    fun show(): String = "User $id"
}
```

---

## 8. 常见面试题与答案

**Q1：Kotlin 内联类是什么？和 `value class` 什么关系？**  
A：内联类是 Kotlin 1.3 的实验特性，1.5 后改名为 `value class`，推荐用 `@JvmInline value class`。核心是包装类型在多数场景内联为底层类型，避免装箱。

**Q2：内联类和 `data class` 区别？**  
A：`data class` 是普通类，有对象头、`copy/equals/hashCode`；`value class` 多数场景内联为底层类型，无对象分配，但限制多。

**Q3：内联类和 `typealias` 区别？**  
A：`typealias` 只是别名，无类型安全；`value class` 是真实类型，编译器会阻止误传。

**Q4：什么时候会装箱？**  
A：作为可空类型、泛型参数、接口引用、数组元素、反射时。

**Q5：内联类能实现接口吗？**  
A：能。但作为接口引用时会装箱。

**Q6：内联类能有多个属性吗？**  
A：不能，只能有一个 `val` 主构造属性。

**Q7：内联类能继承吗？**  
A：不能继承类，只能实现接口。

**Q8：内联类在 Java 中怎么用？**  
A：Java 看到的是底层类型方法，方法名被修饰，调用不便。给 Java 的 API 尽量避免暴露。

**Q9：内联类能做 `lateinit` 吗？**  
A：不能，因为无幕后字段。

**Q10：内联类能用于协程吗？**  
A：能，协程内部大量使用，如 `CoroutineContext.Key`。

**Q11：`@JvmInline` 注解作用？**  
A：告诉编译器生成内联类。Kotlin 1.5+ 必须加。

**Q12：内联类的 `init` 块什么时候执行？**  
A：在构造器工厂方法中执行，每次创建时校验。

---

## 9. 完整示例

```kotlin
@JvmInline
value class UserId(val id: Long) {
    init {
        require(id > 0) { "id must be positive" }
    }

    fun display(): String = "User#$id"
}

@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains("@")) { "Invalid email" }
    }
}

fun sendEmail(email: Email) {
    println("Sending to ${email.value}")
}

fun main() {
    val uid = UserId(1001L)
    println(uid.display())

    val email = Email("test@example.com")
    sendEmail(email)

    // 编译错误：类型不匹配
    // sendEmail(UserId(1L))
}
```

---

## 10. 总结

- 内联类（`value class`）用 `@JvmInline` 修饰，包装单个 `val` 属性。
- 编译后多数场景内联为底层类型，避免装箱，提供类型安全。
- 可空、泛型、接口、数组、反射时会装箱。
- 限制：单属性、无 `var`、无幕后字段、不能继承类。
- 使用场景：ID、单位、密码、邮箱等类型安全包装。
- 与 `typealias` 不同，提供真实类型安全；与 `data class` 不同，无对象头。
- Java 互操作不便，给 Java 的 API 尽量不暴露。
- 面试高频：装箱场景、与 data class 区别、与 typealias 区别、限制。