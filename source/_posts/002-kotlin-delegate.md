---
title: Kotlin 委托（Delegation）详解
date: 2026-10-23 21:46:31
tags: Kotlin
description: Kotlin 的委托是一种将职责转交给其他对象的机制，分为属性委托和类委托，委托是 Kotlin 替代继承、实现组合复用的重要手段，也是面试高频考点。
---


Kotlin 的委托是一种**将职责转交给其他对象**的机制，分为两大类：

- **属性委托（Property Delegation）**：把属性的 `get` / `set` 委托给另一个对象。
- **类委托（Class Delegation）**：把接口的实现委托给另一个对象。

委托是 Kotlin 替代继承、实现组合复用的重要手段，也是面试高频考点。

---

## 一、属性委托

### 1.1 基本语法

```kotlin
val/var <属性名>: <类型> by <委托对象>
```

委托对象必须提供：
- `getValue`：用于 `val` 和 `var`
- `setValue`：仅用于 `var`

这两个方法可以是成员函数，也可以是扩展函数。

### 1.2 标准委托

Kotlin 标准库提供三种常用属性委托：

#### `by lazy`

延迟初始化，首次访问时计算，之后缓存。

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

println(lazyValue) // computed! Hello
println(lazyValue) // Hello（不重复计算）
```

线程安全模式：
```kotlin
val value by lazy(LazyThreadSafetyMode.SYNCHRONIZED) { ... } // 默认，双重检查锁
val value by lazy(LazyThreadSafetyMode.PUBLICATION) { ... }  // 多线程可多次初始化，保留一个
val value by lazy(LazyThreadSafetyMode.NONE) { ... }         // 无锁，单线程
```

**适用场景**：单例、昂贵对象初始化、Android 中初始化 Adapter、ViewModel。

#### `by observable`

属性变化时回调旧值和新值。

```kotlin
var name: String by Delegates.observable("init") { property, old, new ->
    println("${property.name}: $old -> $new")
}

name = "Alice" // name: init -> Alice
```

#### `by vetoable`

属性变化时回调，返回 `false` 可阻止赋值。

```kotlin
var age: Int by Delegates.vetoable(0) { _, old, new ->
    new >= 0 // 负数拒绝赋值
}
```

#### `by notNull`

用于替代 `lateinit`，支持非空类型，但可检查是否初始化。

```kotlin
var name: String by Delegates.notNull()
// println(name) // 抛 IllegalStateException
```

**适用场景**：Android 中 `lateinit` 无法用于基本类型时。

### 1.3 自定义属性委托

实现 `ReadWriteProperty` 或 `ReadOnlyProperty` 接口：

```kotlin
class LoggingProperty<T>(
    private var value: T
) : ReadWriteProperty<Any?, T> {

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        println("get ${property.name}: $value")
        return value
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        println("set ${property.name}: $value -> $newValue")
        value = newValue
    }
}

var name: String by LoggingProperty("init")
```

也可以直接写 `getValue` / `setValue` 的 `operator` 方法（不实现接口）：

```kotlin
class SimpleDelegate<T>(private var value: T) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T = value
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        value = newValue
    }
}
```

### 1.4 编译原理

```kotlin
var name: String by LoggingProperty("init")
```

编译后大致等价：

```java
private final LoggingProperty name$delegate = new LoggingProperty("init");

public final String getName() {
    return name$delegate.getValue(this, $$delegatedProperties[0]);
}

public final void setName(String value) {
    name$delegate.setValue(this, $$delegatedProperties[0], value);
}
```

- 生成一个委托对象字段。
- 属性访问转发到 `getValue` / `setValue`。
- `$$delegatedProperties` 是 `KProperty` 数组，缓存属性元信息。

### 1.5 委托的 `provideDelegate`

在属性声明时执行额外逻辑：

```kotlin
class ResourceDelegate {
    operator fun provideDelegate(thisRef: Any?, property: KProperty<*>): ResourceDelegate {
        println("creating delegate for ${property.name}")
        return this
    }

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String = "value"
}

val x: String by ResourceDelegate()
```

**执行顺序**：`provideDelegate` → `getValue` / `setValue`。

**适用场景**：属性名校验、依赖注入、资源绑定。

---

## 二、类委托

### 2.1 基本语法

```kotlin
class Derived(private val base: Base) : Base by base
```

编译器自动生成所有接口方法的转发实现。

```kotlin
interface Printer {
    fun print(msg: String)
}

class ConsolePrinter : Printer {
    override fun print(msg: String) = println(msg)
}

class LoggingPrinter(private val inner: Printer) : Printer by inner {
    override fun print(msg: String) {
        println("Before")
        inner.print(msg)
        println("After")
    }
}
```

### 2.2 原理

类委托编译后生成：

```java
public final class LoggingPrinter implements Printer {
    private final Printer inner;

    public void print(String msg) {
        // 如果覆写了，则用覆写；否则自动转发
        inner.print(msg);
    }
}
```

**特点**：
- 自动生成所有接口方法。
- 只覆写需要增强的方法。
- 委托对象可以是构造参数。
- 支持多个接口委托。

### 2.3 适用场景

- 装饰器模式（日志、缓存、监控）。
- 组合优于继承。
- 包装第三方库。

```kotlin
class CachedRepository(
    private val delegate: Repository
) : Repository by delegate {
    private val cache = mutableMapOf<Long, User>()

    override suspend fun getUser(id: Long): User? {
        return cache[id] ?: delegate.getUser(id)?.also { cache[id] = it }
    }
}
```

---

## 三、属性委托 vs 类委托

| 特性 | 属性委托 | 类委托 |
|------|---------|--------|
| 作用对象 | 单个属性 | 整个接口实现 |
| 关键字 | `by` 在属性声明处 | `by` 在类声明处 |
| 实现方式 | `getValue` / `setValue` | 自动转发接口方法 |
| 典型用途 | `lazy`、`observable`、`vetoable` | 装饰器、组合复用 |
| 编译产物 | 委托字段 + 转发访问器 | 转发方法 |

---

## 四、委托与 `lateinit`、`lazy` 对比

| 特性 | `lateinit` | `by lazy` | 自定义委托 |
|------|-----------|-----------|-----------|
| 适用类型 | 非空引用类型 | 任意 | 任意 |
| 可变性 | `var` | `val` | `var`/`val` |
| 初始化时机 | 手动赋值 | 首次访问 | 自定义 |
| 线程安全 | 否 | 可配置 | 自定义 |
| 能否重置 | 否 | 否 | 自定义 |
| 能否用于基本类型 | 否 | 是 | 是 |

---

## 五、委托在 Android 中的典型应用

### 5.1 `viewModels`、`activityViewModels`

```kotlin
val viewModel: MyViewModel by viewModels()
```

`viewModels` 是属性委托，延迟获取 ViewModel。

### 5.2 `SharedPreferences` 委托

```kotlin
fun SharedPreferences.stringPref(key: String, default: String = "") =
    object : ReadWriteProperty<Any, String> {
        override fun getValue(thisRef: Any, property: KProperty<*>) = getString(key, default) ?: default
        override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {
            edit().putString(key, value).apply()
        }
    }
```

### 5.3 `arguments` 委托

```kotlin
val userId: String by arguments.getString("userId")
```

### 5.4 Compose 中的 `by remember`

```kotlin
var count by remember { mutableStateOf(0) }
```

`by` 是委托，`MutableState` 提供 `getValue/setValue`。

---

## 六、常见面试题与答案

**Q1：Kotlin 委托是什么？**  
A：把职责转交给其他对象，分属性委托和类委托。属性委托把 `get/set` 委托给 `getValue/setValue`；类委托把接口实现转发给另一个对象。

**Q2：属性委托需要实现什么？**  
A：`val` 需要 `getValue`；`var` 需要 `getValue` 和 `setValue`。可实现 `ReadOnlyProperty` / `ReadWriteProperty`。

**Q3：`by lazy` 是线程安全的吗？**  
A：默认 `SYNCHRONIZED` 线程安全；可选 `PUBLICATION`、`NONE`。

**Q4：`by lazy` 和 `lateinit` 区别？**  
A：`lazy` 用于 `val`，首次访问初始化，可线程安全；`lateinit` 用于 `var`，手动赋值，不能用于基本类型。

**Q5：`observable` 和 `vetoable` 区别？**  
A：`vetoable` 的 `setValue` 返回 `Boolean`，`false` 阻止赋值；`observable` 只通知。

**Q6：类委托底层如何实现？**  
A：编译器生成所有接口方法的转发方法，调用委托对象。

**Q7：类委托和继承区别？**  
A：委托是组合，避免脆弱基类，只暴露接口；继承耦合父类实现。

**Q8：`provideDelegate` 作用？**  
A：在属性声明时执行额外逻辑，如校验、注入。执行在 `getValue/setValue` 之前。

**Q9：委托属性编译后长什么样？**  
A：生成委托字段，属性访问转发到 `getValue/setValue`。

**Q10：Android 中哪些地方用了委托？**  
A：`by viewModels()`、`by arguments`、`by remember`、`SharedPreferences` 委托。

**Q11：属性委托能用于局部变量吗？**  
A：能，Kotlin 1.1+ 支持局部委托变量。

**Q12：`Delegates.notNull` 和 `lateinit` 区别？**  
A：`notNull` 支持基本类型，`lateinit` 不支持；`notNull` 是委托，`lateinit` 是语言特性。

---

## 七、完整示例

```kotlin
class User {
    var name: String by Delegates.observable("init") { _, old, new ->
        println("name: $old -> $new")
    }

    var age: Int by Delegates.vetoable(0) { _, _, new -> new >= 0 }

    val id: Long by lazy { System.currentTimeMillis() }
}

fun main() {
    val user = User()
    user.name = "Alice"
    user.age = 30
    user.age = -1 // 拒绝
    println(user.id)
}
```

---

## 八、总结

- 委托是 Kotlin 替代继承、实现组合复用的核心机制。
- 属性委托：`lazy`、`observable`、`vetoable`、`notNull`、自定义。
- 类委托：`class Derived(base: Base) : Base by base`，自动转发接口。
- 编译原理：属性委托生成委托字段和转发访问器；类委托生成转发方法。
- Android 中大量使用：`viewModels`、`arguments`、`remember`、SharedPreferences。
- 面试高频：`lazy` 线程安全、`observable` vs `vetoable`、`provideDelegate`、类委托与继承区别。