---
title: Kotlin 对象声明与对象表达式
category: Kotlin
date: 2026-10-24 10:38:19
tags: Kotlin
description: Kotlin 可以用 objects 来创建对象，分别是对象声明和对象表达式，语法相似语义不同。一句话记忆，对象声明是单例，对象表达式是匿名内部类。
---

# Kotlin 对象声明与对象表达式

Kotlin 用 `object` 关键字同时支持两种截然不同的东西：**对象声明（Object Declaration）** 和 **对象表达式（Object Expression）**。它们语法相似，但语义和用途完全不同，是面试高频考点。

---

## 1. 核心区别速览

| 特性 | 对象声明 | 对象表达式 |
|------|---------|-----------|
| 关键字 | `object Foo` | `object : Bar { }` 或 `object { }` |
| 本质 | 单例类，编译后有 `INSTANCE` 静态字段 | 匿名内部类，每次执行创建新实例 |
| 实例数量 | 全局唯一 | 每次求值都新创建一个 |
| 是否有名字 | 有，通过名字访问 | 通常匿名，可赋值给变量 |
| 能否继承/实现 | 能 | 能 |
| 能否捕获外部变量 | 不能 | 能（闭包） |
| 初始化时机 | 首次访问时惰性初始化，线程安全 | 每次执行到时创建 |
| 典型用途 | 单例、工具类、伴生对象 | 匿名监听器、临时实现、Adapter |
| 与 Java 对应 | 静态单例 | 匿名内部类 |

**一句话记忆**：  
**对象声明是单例，对象表达式是匿名内部类。**

---

## 2. 对象声明（Object Declaration）

### 2.1 语法

```kotlin
object Singleton {
    val name = "Singleton"
    fun doSomething() { println("Doing") }
}

// 使用
Singleton.doSomething()
println(Singleton.name)
```

对象声明可以继承类或实现接口：

```kotlin
interface Logger {
    fun log(msg: String)
}

object ConsoleLogger : Logger {
    override fun log(msg: String) = println(msg)
}
```

### 2.2 编译原理

```kotlin
object Singleton {
    val name = "Singleton"
}
```

编译后大致等价：

```java
public final class Singleton {
    public static final Singleton INSTANCE = new Singleton();
    private final String name = "Singleton";

    private Singleton() { }

    public final String getName() { return name; }
}
```

- 生成一个 `final` 类。
- 私有构造器，防止外部实例化。
- 静态字段 `INSTANCE` 持有唯一实例。
- 首次访问 `Singleton.INSTANCE` 时初始化，**线程安全**（JVM 类加载机制保证）。

### 2.3 对象声明的种类

1. **普通对象声明**：`object Foo { }`
2. **伴生对象**：`companion object { }`，与外部类关联，成员可通过外部类名访问。
3. **数据对象**（Kotlin 1.9+）：`data object Foo { }`，自动生成 `toString/equals/hashCode`，常用于密封类。

```kotlin
sealed interface State {
    data object Loading : State
    data class Success(val data: String) : State
}
```

### 2.4 使用场景

- 单例模式（最常用）。
- 工具类、常量集合。
- 伴生对象。
- 接口的默认实现对象。
- 密封类的无状态子类（`data object`）。

### 2.5 注意点

- 对象声明不能有构造器（因为只有一个实例）。
- 对象声明可以嵌套在类中，但它是**静态单例**，不持有外部类实例引用。
- 对象声明可以访问外部类的静态成员（如伴生对象），但不能访问实例成员。
- 对象声明可以定义 `init` 块，在首次访问时执行。

---

## 3. 对象表达式（Object Expression）

### 3.1 语法

```kotlin
// 实现接口
val listener = object : View.OnClickListener {
    override fun onClick(v: View?) {
        println("Clicked")
    }
}

// 继承类
val runnable = object : Thread() {
    override fun run() {
        println("Running")
    }
}

// 不继承任何东西，纯匿名对象
val obj = object {
    val x = 1
    fun foo() = "foo"
}
```

### 3.2 编译原理

对象表达式编译为**匿名内部类**。每次执行到该表达式时，创建一个新实例。

```kotlin
fun createListener(): View.OnClickListener {
    return object : View.OnClickListener {
        override fun onClick(v: View?) { }
    }
}
```

每次调用 `createListener()` 都会返回不同的对象。

### 3.3 捕获外部变量（闭包）

对象表达式可以访问并捕获外部作用域的变量：

```kotlin
fun createCounter(): () -> Int {
    var count = 0
    return object : () -> Int {
        override fun invoke(): Int {
            count++          // 捕获外部变量
            return count
        }
    }
}
```

对象声明不能这样做，因为它没有外部作用域。

### 3.4 使用场景

- 匿名内部类（监听器、回调、Runnable）。
- 临时实现接口或抽象类。
- 需要访问外部变量的场合。
- 作为局部变量或参数。

### 3.5 注意点

- 对象表达式每次执行都会创建新实例。
- 如果在 Activity/Fragment 中创建并持有外部引用，可能引起内存泄漏。
- 对象表达式可以继承类或实现接口，但只能继承一个类，可实现多个接口。
- 对象表达式没有名字，但可以赋值给变量，通过变量访问。
- 如果对象表达式作为局部变量，其类型是匿名的，只能通过变量访问成员；如果作为参数，类型是声明的父类型。

---

## 4. 对象声明 vs 对象表达式：代码对比

```kotlin
// 对象声明：单例
object Singleton {
    var count = 0
    fun increment() { count++ }
}

fun main() {
    Singleton.increment()
    Singleton.increment()
    println(Singleton.count) // 2

    // 对象表达式：每次新实例
    fun makeCounter() = object {
        var count = 0
        fun increment() { count++ }
    }

    val c1 = makeCounter()
    val c2 = makeCounter()
    c1.increment()
    c1.increment()
    println(c1.count) // 2
    println(c2.count) // 0
}
```

---

## 5. 与 Java 的对比

| Kotlin | Java 对应 |
|--------|----------|
| `object Singleton { }` | `static` 单例（`INSTANCE` 字段） |
| `object : Interface { }` | `new Interface() { }` 匿名内部类 |
| `companion object { }` | `static` 成员 + 单例 |
| `data object` | 无直接对应 |

Java 调用 Kotlin 对象声明：
```java
Singleton.INSTANCE.doSomething();
```

---

## 6. 常见面试题与答案

**Q1：对象声明和对象表达式的区别？**  
A：对象声明是单例，编译后有 `INSTANCE` 静态字段，全局唯一；对象表达式是匿名内部类，每次执行创建新实例，可捕获外部变量。

**Q2：对象声明是线程安全的吗？**  
A：是。JVM 类加载机制保证静态初始化线程安全，首次访问时初始化一次。

**Q3：对象表达式能捕获外部变量吗？对象声明呢？**  
A：对象表达式能捕获外部作用域变量（闭包）；对象声明不能，因为它没有外部作用域。

**Q4：对象声明能继承类吗？**  
A：能，可以继承一个类并实现多个接口。

**Q5：对象表达式每次都会创建新对象吗？**  
A：是，每次执行到对象表达式都会创建新实例。

**Q6：对象声明能用在什么地方？**  
A：单例、工具类、伴生对象、数据对象、密封类子类。

**Q7：`data object` 和 `object` 区别？**  
A：`data object` 自动生成 `toString/equals/hashCode`，适合密封类无状态子类；`object` 不生成。

**Q8：对象表达式会导致内存泄漏吗？**  
A：在 Android 中，若匿名内部类持有 Activity 引用且生命周期长于 Activity，可能泄漏。可用静态内部类 + WeakReference 或 lambda 避免。

**Q9：对象声明和伴生对象区别？**  
A：伴生对象是对象声明的特殊形式，用 `companion` 修饰，与外部类关联，成员可通过外部类名访问；普通对象声明独立。

**Q10：对象表达式能访问外部类的私有成员吗？**  
A：能，如果它是在外部类内部创建的，会持有外部类引用，可以访问私有成员。

**Q11：对象声明初始化时机？**  
A：首次访问时惰性初始化，线程安全，只一次。

**Q12：对象表达式能作为函数返回值吗？**  
A：能，但返回类型通常是父类型或 `Any`，匿名类型只能在局部作用域使用。

**Q13：Kotlin 中如何实现单例？**  
A：用 `object` 声明最简单，线程安全，懒加载。也可用 `companion object` + `by lazy` 等。

**Q14：对象声明能定义扩展函数吗？**  
A：能，可以为对象声明定义扩展函数，但需通过对象名调用。

---

## 7. 完整示例：单例与匿名监听器

```kotlin
// 对象声明：单例
object Config {
    var debug = false
    fun enableDebug() { debug = true }
}

// 对象表达式：匿名监听器
class Button {
    fun setOnClickListener(listener: OnClickListener) { /* ... */ }
}

interface OnClickListener {
    fun onClick()
}

fun main() {
    Config.enableDebug()
    println(Config.debug)

    val button = Button()
    val count = intArrayOf(0)
    button.setOnClickListener(object : OnClickListener {
        override fun onClick() {
            count[0]++
            println("Clicked ${count[0]} times")
        }
    })
}
```

---

## 8. 总结

- **对象声明**：`object Foo`，单例，编译后 `INSTANCE` 静态字段，线程安全，懒加载。用于单例、工具类、伴生对象、`data object`。
- **对象表达式**：`object : Bar { }`，匿名内部类，每次创建新实例，可捕获外部变量。用于监听器、回调、临时实现。
- 对象声明不能捕获外部变量；对象表达式可以。
- 对象声明全局唯一；对象表达式每次新实例。
- 与 Java 对比：对象声明 ≈ 静态单例；对象表达式 ≈ 匿名内部类。
- 面试高频：区别、线程安全、初始化时机、内存泄漏、与伴生对象对比、单例实现。
