---
title: Kotlin 中的嵌套类和内部类
category: Android
date: 2026-10-24 09:42:53
tags: Kotlin
description: Kotlin 中的嵌套类和内部类默认行为正好和 Java 中相反。写在内部的类默认不持有外部类的引用，仅当加了 inner 关键字之后才持有外部类的引用。
---

# Kotlin 中的嵌套类（Nested Class）与内部类（Inner Class）

在 Kotlin 中，定义在另一个类内部的类分为两种：

- **嵌套类（Nested Class）**：默认，不持有外部类引用，相当于 Java 的 `static` 嵌套类。
- **内部类（Inner Class）**：用 `inner` 修饰，持有外部类引用，相当于 Java 的非静态内部类。

这是 Kotlin 与 Java 的一个重要区别，也是面试高频考点。

---

## 1. 嵌套类（Nested Class）

**定义**：直接写在类内部的类，没有 `inner` 关键字。

**特点**：
- 默认是静态的，不持有外部类实例引用。
- 不能访问外部类的成员（属性、方法）。
- 可以访问外部类的伴生对象（`companion object`）成员。
- 实例化：`Outer.Nested()`。

**代码示例**：
```kotlin
class Outer {
    val outerProp = "outer"
    companion object { val COMPANION = "companion" }

    class Nested {
        fun foo() {
            // println(outerProp) // 编译错误：不能访问外部类实例成员
            println(COMPANION)    // 可以访问伴生对象
        }
    }
}

fun main() {
    val nested = Outer.Nested()  // 直接通过外部类名实例化
    nested.foo()
}
```

**对应 Java 写法**：
```java
public class Outer {
    static class Nested {
        void foo() { ... }
    }
}
```
Kotlin 的嵌套类默认就是 Java 的 `static` 嵌套类。

---

## 2. 内部类（Inner Class）

**定义**：用 `inner` 关键字修饰的嵌套类。

**特点**：
- 持有外部类实例的引用。
- 可以直接访问外部类的所有成员（包括私有成员）。
- 实例化必须依赖外部类实例：`outer.Inner()`。
- 内部类中可用 `this@Outer` 显式引用外部类实例。
- 可能引起内存泄漏（如 Android 中 Handler、Listener）。

**代码示例**：
```kotlin
class Outer {
    private val outerProp = "outer"

    inner class Inner {
        fun foo() {
            println(outerProp)          // 直接访问外部类成员
            println(this@Outer.outerProp) // 显式引用
        }
    }
}

fun main() {
    val outer = Outer()
    val inner = outer.Inner()  // 必须通过外部类实例
    inner.foo()
}
```

**对应 Java 写法**：
```java
public class Outer {
    private String outerProp = "outer";
    class Inner {
        void foo() {
            System.out.println(outerProp);
        }
    }
}
```

---

## 3. 与 Java 的对比

| 特性 | Kotlin 嵌套类 | Kotlin 内部类 | Java 静态嵌套类 | Java 非静态内部类 |
|------|--------------|--------------|----------------|------------------|
| 关键字 | 无 | `inner` | `static class` | 无（默认） |
| 持有外部引用 | 否 | 是 | 否 | 是 |
| 访问外部实例成员 | 不能 | 能 | 不能 | 能 |
| 实例化方式 | `Outer.Nested()` | `outer.Inner()` | `new Outer.Nested()` | `outer.new Inner()` |
| 内存泄漏风险 | 低 | 高 | 低 | 高 |

**记忆口诀**：  
Kotlin 默认嵌套类就是 Java 的静态嵌套类；需要访问外部类实例时，加 `inner`。

---

## 4. 匿名内部类

Kotlin 中使用 `object` 表达式创建匿名内部类：
```kotlin
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        // 如果是内部类，会持有外部类引用
    }
})
```

如果不需要持有外部引用，可以用 lambda：
```kotlin
button.setOnClickListener { v -> ... }
```

在 Android 中，匿名内部类若在 Activity 中，可能持有 Activity 引用导致泄漏，需注意。

---

## 5. 使用场景

- **嵌套类**：组织辅助类、工具类，不需要访问外部类状态。例如 `Map.Entry`、`Builder`。
- **内部类**：需要访问外部类状态，如 Adapter 的 ViewHolder、事件监听器、迭代器。
- **伴生对象**：`companion object` 也是嵌套的一种，用于静态成员。

---

## 6. 常见面试题与答案

**Q1：Kotlin 中嵌套类和内部类的区别？**  
A：嵌套类默认静态，不持有外部引用，不能访问外部实例成员；内部类用 `inner` 修饰，持有外部引用，可访问外部所有成员。

**Q2：为什么 Kotlin 默认嵌套类是静态的？**  
A：为了减少内存泄漏风险，默认不持有外部引用，更安全。需要时显式加 `inner`。

**Q3：如何在内部类中访问外部类实例？**  
A：直接访问即可；若命名冲突，用 `this@Outer`。

**Q4：Kotlin 嵌套类能访问外部类的伴生对象吗？**  
A：能，因为伴生对象是静态的。

**Q5：Java 如何调用 Kotlin 的嵌套类和内部类？**  
- 嵌套类：`new Outer.Nested()`  
- 内部类：`outer.new Inner()`

**Q6：内部类为什么可能内存泄漏？**  
A：内部类持有外部类引用，若内部类实例生命周期长于外部类，外部类无法回收。

**Q7：Kotlin 中 `object` 表达式和内部类的关系？**  
A：`object : Interface {}` 创建匿名内部类，若在外部类中会持有外部引用；可用 lambda 避免。

---

## 7. 代码示例：完整对比

```kotlin
class Outer {
    private val message = "Hello"
    companion object { const val TAG = "Outer" }

    // 嵌套类：静态，不持有外部引用
    class Nested {
        fun printTag() = println(TAG)          // 可访问伴生对象
        // fun printMessage() = println(message) // 编译错误
    }

    // 内部类：持有外部引用
    inner class Inner {
        fun printMessage() = println(message)  // 可访问外部实例成员
        fun printOuter() = println(this@Outer.message)
    }
}

fun main() {
    // 嵌套类实例化
    val nested = Outer.Nested()
    nested.printTag()

    // 内部类实例化
    val outer = Outer()
    val inner = outer.Inner()
    inner.printMessage()
}
```

---

## 8. 总结

- Kotlin 嵌套类默认静态，不持有外部引用；内部类用 `inner`，持有外部引用。
- 嵌套类不能访问外部实例成员，内部类可以。
- 实例化：`Outer.Nested()` vs `outer.Inner()`。
- 内部类有内存泄漏风险，Android 中需注意。
- 与 Java 对比：Kotlin 默认嵌套类 = Java 静态嵌套类；Kotlin 内部类 = Java 非静态内部类。
- 面试常考区别、内存泄漏、`this@Outer`、Java 互操作。