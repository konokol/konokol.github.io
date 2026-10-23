---
title: Kotlin中的数字类型的相等性
tags: Kotlin
date: 2019-02-27 14:44:10
description: Kotlin中有2种相等性，结构型相等和引用型相等。可以理解为结构型相等是值比较，和Java中的equals类似，引用型相等是通过比较引用指向的对象是否为同一个，和Java中的==类似。
---



## 相等性

### 相等性分类

Kotlin中有2种相等性  

- 结构型相等(==)
- 引用型相等(===)

可以理解为结构型相等是值比较，和Java中的equals类似，引用型相等是通过比较引用指向的对象是否为同一个，和Java中的==类似。

Kotlin中，结构性相等使用双等号==来表示（否定形式是!=），引用型相等使用恒等号===表示（否定形式是!==）。

``` kotlin
open operator fun equals(other: Any?): Boolean
```
kotlin中，Any类重载了==操作符， 但是a==b却不完全等价于a.equlas(b)。

```kotlin
fun equals() {
    val n : Int = 123
    val m : Int = 3

    n.equals(m)
    m == n

    val p: Int? = null
    val q: Int? = 3
    val s = "123"
    
    p == q
    p == n
    p == s   //Operator '==' cannot be applied to 'Int?' and 'String'
}
```

```java
public static final void equals() {
      int n = 123;
      int m = 3;
      Integer.valueOf(n).equals(Integer.valueOf(m));
      Integer p = (Integer)null;
      Integer q = 3;
      Intrinsics.areEqual(p, q);
   }
```

根据变量类型不同，==会被编译成不同类型

- 变量为非空类型，a==b被编译成a.equals(b)
- 变量未可空类型，a==b被编译成Intrinsics.areEqual(p, q)
- ==两侧类型不同，编译失败

其中Intrinscs.areEquals(Object, Object)中对参数都进行了判空，所以在kotlin中，只要能编译通过，请放心大胆的使用==，不用担心空指针的问题。

```kotlin
 public static boolean areEqual(Object first, Object second) {
        return first == null ? second == null : first.equals(second);
 }
```

### 数字类型的相等性

在Kotlin中，数字“基本类型”一共有6种，分别是Byte、Short、Int、Long、Float、Double，这6种类型都继承自Number类。

从形式上看，这6种类型和Java中的包装数字类型相同，但是他们和Java中的数字类型不完全等价。

kotlin中，被声明成可空类型(Int?等)的数字变量在JVM中会被表示成包装类型数字，被声明成非空类型的数字在JVM中可以被表示基本类型数字。

#### 整数的相等性

```Kotlin
fun primitive() {
    val a: Int = 128
    println(a == a) // 1
    val primitiveA: Int = a
    println(a === primitiveA) // 2
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    println(boxedA === primitiveA)// 3
    println(boxedA === anotherBoxedA)// 4
    println(boxedA == primitiveA)// 5
    println(boxedA == anotherBoxedA)//6
}
```

在这段代码中，注释1～5中的打印结果分别对应：

**1、数字类型始终和自身相等**

a为非空变量，所以a为基本类型，无论是a==a，还是a===a都是true.

**2、基本类型整数，值相等即满足结构型相等和引用型相等**

a和primitiveA都是非空变量，都被存储为基本类型，只要它们的值相等，无论是==还是===，比较的结果都是true.

**3、装箱类型，即便是值相等，只要类型不同，则不满足引用型相等**

由于被声明成了可空变量，boxedA被存储为装箱类型，primitiveA是非空变量，被存储为基本类型，装箱类型和基本类型相比，类型不同，所以不满足引用型相等，boxedA === primitiveA为false.

**4、装箱类型，即使值相等，只要引用不指向同一个对象，不满足引用型相等**

boxedA和anotherBoxedA都是可空变量，都被存储为装箱类型，值都是128，赋值操作会创建出不同的Int对象，所以不满足引用型相等，boxedA === anotherBoxedA值为false.

Kotlin在处理整数的时候，和Java中的Integer一样，0到127都有缓存池，数值小于128时，取到的是同一个对象，大于127则会创建新的对象。所以在上述代码块中，a的值是128，创建了不同的对象，使用===比较，值为false. 将上述代码中a的值改成126，取到的都是缓存池中的对象，因此，即使使用===比较，值也是true.

**5、值相等的数字，满足结构型相等**

在注释5和6中，primitiveA，boxedA，anotherBoxedA值都是128，primitiveA是基本类型，boxedA和anotherBoxedA是装箱类型，使用==比较，只比较它们的值，都相等，都满足结构型相等。

#### 浮点数的相等性

对于浮点数的比较

- 相等性检测：a == b 与 a != b
- 比较操作符：a < b、 a > b、 a <= b、 a >= b
- 区间实例以及区间检测：a..b、 x in a..b、 x !in a..b

当其中的操作数 a 与 b 都是明确的（可空或非空的）Float 或 Double 类型时，该检测遵循 IEEE 754 浮点数运算标准。

否则会使用不符合该标准的结构相等性检测，这会导致 NaN 等于其自身，而 -0.0 不等于 0.0。
然而，为了支持泛型场景并提供全序支持，当这些操作数并非静态类型为浮点数（例如是 Any、 Comparable<……>、 类型参数）时，这些操作使用为 Float 与 Double 实现的不符合标准的 equals 与 compareTo，这会出现：

 - 认为 NaN 与其自身相等
 - 认为 NaN 比包括正无穷大（POSITIVE_INFINITY）在内的任何其他元素都大
 - 认为 -0.0 小于 0.0

IEEE 754中对于浮点数比较的定义

>  **浮点数的比较**
> 
> 浮点数基本上可以按照符号位、指数域、尾数域的顺序作字典比较。显然，所有正数大于负数；正负号相同时，指数的二进制表示法更大的其浮点数值更大。
> 
>  **标准运算**
> 
>  下述函数必须提供:
> 
> ......
> 
> - 比较运算. -Inf <负的规约浮点数数<负的非规约浮点数< -0.0 = 0.0 <正的非规约浮点数<正的规约浮点数< Inf；
> - 特殊比较： -Inf = -Inf, Inf = Inf, NaN与任何浮点数（包括自身）的比较结果都为假，即 (NaN ≠ x) = false.


```kotlin
fun stdFloatNumbers() {
    // standard
    println(Float.NaN == Float.NaN) //false

    println(Float.NaN > Float.POSITIVE_INFINITY) //false
    println(Float.NaN < Float.POSITIVE_INFINITY) //false

    println(Float.NaN < Float.NEGATIVE_INFINITY) //false
    println(Float.NaN > Float.NEGATIVE_INFINITY) //false

    println(-0.0 == 0.0) //true
    println(-0.0 < 0.0)  //false
    val f0 : Float = Float.NaN
    println(f0 == f0) //false
}
```

```kotlin

fun floatNumbers() {

    val f1 : Comparable<Float> = Float.NaN
    val f2 : Comparable<Float> = Float.NaN
    val f3 : Any = Float.NaN
    val f4 : Any? = Float.NaN
    val f5 : Comparable<Double> = -0.0
    val f6 : Comparable<Double> = 0.0
    val pInf = Float.POSITIVE_INFINITY

    // NaN not equals NaN
    println(f1 == f1) //true
    println(f1 == f2) //true
    println(f1 == f3) //true
    println(f3 == f4) //true

    // -0.0 not equals 0.0
    println(f5 == f6) //fasle
}

```
