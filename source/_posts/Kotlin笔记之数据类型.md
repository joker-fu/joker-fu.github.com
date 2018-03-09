---
title: Kotlin笔记之数据类型
copyright: true
date: 2018-02-27 10:52:02
categories: Kotlin
tags: Kotlin
---
### Boolean 类型
```kotlin
val isTrue: Boolean = true
var isFalse: Boolean = false
val isSure = true
val isCancel = false
```

### Number 类型
| 分类| 类型 | 位宽 |
|-----|-----|------|
| 浮点型| Double | 64 |
| 浮点型| Float| 32 |
| 整型| Long| 64 |
| 整型| Int| 32 |
| 整型| Short| 16 |
| 字节型| Byte| 8 |

```kotlin
val anInt: Int = 8
val anotherInt: Int = 0xff
val moreInt: Int = 0b0011
val minInt: Int = Int.MIN_VALUE
val maxInt: Int = Int.MAX_VALUE

val minLong :Long = Long.MIN_VALUE
val maxLong :Long = Long.MAX_VALUE

val aFloat : Float = 1.0F
val another : Float = 1e3F
val minFloat : Float = -Float.MAX_VALUE
val maxFloat : Float = Float.MAX_VALUE

val aDouble : Double = 3.0
val minDouble : Double = -Double.MAX_VALUE
val maxDouble : Double = Double.MAX_VALUE

val minByte : Byte = Byte.MIN_VALUE
val maxByte : Byte = Byte.MAX_VALUE

val minShort : Short = Short.MIN_VALUE
val maxShort : Short = Short.MAX_VALUE

//数字字面值中的下划线（自 1.1 起）
//你可以使用下划线使数字常量更易读：
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### Char 类型
```kotlin
val aChar: Char = '中'
val anotherChar: Char = 'a'
val moreChar: Char = '\u000f'
```
> 支持这几个转义序列：\t、 \b、\n、\r、\'、\"、\\ 和 \$。 编码其他字符要用 Unicode 转义序列语法：'\uFF00'

### 数组
数组在 Kotlin 中使用 Array 类来表示，它定义了 get 和 set 函数（按照运算符重载约定这会转变为 []）和 size 属性，以及一些其他有用的成员函数
```kotlin
//创建了Array[1,2,3]
arrayOf(1, 2, 3)

arrayOf(1, 2, 3).iterator().forEach { println(it) }
println("-------")
println(arrayOfNulls<Int>(10).size)

val asc: Array<Int> = Array(5, { i -> i * 2 })
asc[2] = 10
asc.iterator().forEach { println("it: $it") }
```
>注意: 与 Java 不同的是，Kotlin 中数组是不型变的（invariant）。这意味着 Kotlin 不让我们把 Array< String > 赋值给 Array< Any >

Kotlin 也有无装箱开销的专门的类来表示原生类型数组: ByteArray、 ShortArray、IntArray 等等。这些类和 Array 并没有继承关系，但是它们有同样的方法属性集。它们也都有相应的工厂方法

### String 类型
```kotlin
val string: String = "hello world"
val fromChars: String = String(charArrayOf('h', 'e', 'l', 'l', 'o'))

fun main(args: Array<String>) {
    println(string)
    println(fromChars)

    //相当于java equals 比较内容
    println(string == fromChars)
    //相当于java == 比较对象是否相同
    println(string === fromChars)

    val arg1 = 0
    val arg2 = 1

    println("" + arg1 + " + " + arg2 + " = " + (arg1 + arg2))
    //字符串模板
    println("$arg1 + $arg2 = ${arg1 + arg2}")
}
```
>原生字符串 使用三个引号（"""）分界符括起来，内部没有转义并且可以包含换行和任何其他字符:
val text = """
    for (c in "foo")
        print(c)
"""
你可以通过 trimMargin() 函数去除前导空格：
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
**默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")**。

### 基本类型转换
**显式转换**
```kotlin
    val anInt: Int = 5
    val aLong: Long = anInt.toLong()
```

>由于不同的表示方式，较小类型并不是较大类型的子类型。 如果它们是的话，就会出现下述问题：
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(a == b) // 惊！这将输出“false”鉴于 Long 的 equals() 检测其他部分也是 Long
所以同一性还有相等性都会在所有地方悄无声息地失去。
**因此较小的类型不能隐式转换为较大的类型。** 这意味着在不进行显式转换的情况下我们不能把 Byte 型值赋给一个 Int 变量。
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
我们可以显式转换来拓宽数字
val i: Int = b.toInt() // OK: 显式拓宽
**每个数字类型支持如下的转换:**
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
**缺乏隐式类型转换并不显著，因为类型会从上下文推断出来，而算术运算会有重载做适当转换**，例如：
val l = 1L + 3 // Long + Int => Long

### 类类型
```kotlin
//open 才能被继承 或重写
open class People(val name: String, val age: String) {
    init {
        println("this is ${this.javaClass.simpleName}, name is $name, age is $age")
    }

    //不能重写 相当于加了final
    fun sayHello() {
        println("hello, my name is $name, age is $age")
    }
}

class Woman(name: String, age: String) : People(name, age)

class Man(name: String, age: String) : People(name, age)

fun main(args: Array<String>) {
    val zhang: Man = Man("zhang", "20")
    zhang.sayHello()
    val li: Woman = Woman("li", "18")
    li.sayHello()
    //相当于Java instanceOf
    println("li is people: ${li is People}")
}
```

### 空安全
许多编程语言（包括 Java）中最常见的陷阱之一，就是访问空引用的成员会导致空引用异常。在 Java 中，这等同于 NullPointerException 或简称 NPE。

Kotlin 的类型系统旨在从我们的代码中消除 NullPointerException。
- 非空引用(不能容纳空)
```kotlin
var s: String = "hello"
s = null //报错 s不能为空
```
- 可空引用(可容纳空)
```kotlin
var s: String? = "hello"
s = null //ok s可为空
```
NULL检查：
1. 在条件中检查null，只适用于对象不可变的情况，如下：
```kotlin
//此处s属性 不可变 var为可变
val s: String? = "hello"

val l = if (s != null) s.length else -1

if (s != null && s.length > 0) {
    print("String of length ${s.length}")
} else {
    print("Empty string")
}
```
>- val 局部变量——总是可以，局部委托属性除外；
- val 属性——如果属性是 private 或 internal，或者该检查在声明属性的同一模块中执行。智能转换不适用于 open 的属性或者具有自定义 getter 的属性；
- var 局部变量——如果变量在检查和使用之间没有修改、没有在会修改它的 lambda 中捕获、并且不是局部委托属性；
- var 属性——决不可能（因为该变量可以随时被其他代码修改）。

2. 安全调用操作符（?.）
```kotlin
//s 非空，就返回 s.length，否则返回 null，这个表达式的类型是 Int?
s?.length
```
3. 非空断言运算符（!!）
```kotlin
//非空断言运算符（!!）将任何值转换为非空类型，若该值为空则抛出异常。
//使用这个运算符在你确定对象不为空的时候 否侧将会得到NPE
s!!.length
```
Elvis 操作符（?:）：
当我们有一个可空的引用 r 时，我们可以说“如果 r 非空，我使用它；否则使用某个非空的值 x”：
```kotlin
val l: Int = if (b != null) b.length else -1
```
除了完整的 if-表达式，这还可以通过 Elvis 操作符表达，写作 ?:：
```kotlin
val l = b?.length ?: -1
```
**如果 ?: 左侧表达式非空，elvis 操作符就返回其左侧表达式，否则返回右侧表达式。 请注意，当且仅当左侧为空时，才会对右侧表达式求值。**

安全的类型转换：
如果对象不是目标类型，那么常规类型转换可能会导致 ClassCastException。 另一个选择是使用安全的类型转换，如果尝试转换不成功则返回 null：
```kotlin
val aInt: Int? = a as? Int
```
可空类型的集合：
如果你有一个可空类型元素的集合，并且想要过滤非空元素，你可以使用 filterNotNull 来实现：
```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```


