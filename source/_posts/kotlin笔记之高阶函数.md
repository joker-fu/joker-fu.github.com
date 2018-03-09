---
title: kotlin笔记之高阶函数
copyright: true
date: 2018-03-09 17:03:35
categories: kotlin
tags: kotlin
---
### 高阶函数基本概念
- 传入或者返回函数的函数
- 函数引用 ::println
- 带有Receiver的引用 printer::Println
```kotlin
fun main(args: Array<String>) {
    val arr = arrayOf("1", "22", "", "4444", "55555")
    arr.forEach(::println)

    println("------------------------")

    arr.filter(String::isNotEmpty).forEach(::println)

    println("------------------------")

    val printer = Printer()
    arr.forEach(printer::println)
}


class Printer {
    fun println(any: Any) {
        kotlin.io.println(any)
    }
}
```

### 常见高阶函数
- forEach

- map

- flatMap

- reduce

- flod

- joinToString

- filter

- takeWhile

- let

- apply

- with

- use


### 尾递归优化
- 递归得一种特殊形式
- 调用自身后无其他操作
- tailrec 关键字提示编译器尾递归优化
- 尾递归于迭代的关系
```kotlin
data class NodeList(var value: Int, var next: NodeList? = null)

tailrec fun findNode(head: NodeList?, value: Int): NodeList? {
    head ?: return null
    if (head.value == value) return head
    return findNode(head.next, value)
}

fun main(args: Array<String>) {
    val head = NodeList(0)
    var p = head
    for (i in 1..1000000) {
        p.next = NodeList(i)
        p = p.next!!
    }

    println(findNode(head, 8)?.value)
}
```

### 闭包
- 函数运行的环境
- 持有函数运行状态
- 函数内部可以定义函数
- 函数内部也可以定义类
```kotlin
fun justCount(): () -> Unit {

    var count = 0
    return {
        println(++count)
    }
}

fun main(args: Array<String>) {

    val count = justCount()
    count()
    count()
//
//    println({ x: Int, y: Int ->
//        println("${x + y}")
//    }(2,3))

}
```

### 函数复合
- f(g(x))
```kotlin
val mulBy2 = { value: Int -> value * 2 }
val add5 = { value: Int -> value + 5 }

//fun a5(i: Int) = i + 5
//fun m2(i: Int) = i * 2

fun main(args: Array<String>) {
    println(mulBy2(add5(8)))

    val add5ThenMulBy2 = add5 andThen mulBy2
    val add5ComposeMulBy2 = add5 compose mulBy2
    println(add5ThenMulBy2(8)) //(5+8)*2
    println(add5ComposeMulBy2(8)) //8*2+5
}

/**
 * 扩展Function1 输入P1得到R
 */
infix fun <P1, P2, R> Function1<P1, P2>.andThen(function: Function1<P2, R>): Function1<P1, R> {
    //输入P1得到R
    return fun(p1): R {
        //this(p1) ==> P2
        //function(this(p1)) == function(P2) ==> R
        return function(this(p1))
    }
}

infix fun <P1, P2, R> ((P2) -> R).compose(f: (P1) -> P2): (P1) -> R {
    return fun(p1): R {
        return this(f(p1))
    }
}
```

### 科理化（Currying）
- 多元函数变成一元函数调用链
```kotlin
fun log(tag: String, os: OutputStream, message: String) {
    os.write("[$tag]: $message \n".toByteArray())
}

//fun log(tag: String) = fun(os: OutputStream) = fun(message: String) = os.write("[$tag]: $message \n".toByteArray())

fun main(args: Array<String>) {
    log("Debug", System.out, "this is debug log!")
//    log("Debug")(System.out)("this is debug log!!")
    ::log.curry()("Debug")(System.out)("this is debug log!!")
}

fun <P1, P2, P3, R> Function3<P1, P2, P3, R>.curry() = fun(p1: P1) = fun(p2: P2) = fun(p3: P3) = this(p1, p2, p3)
```

### 偏函数
- 传入部分参数得到新函数
```kotlin
fun log(tag: String, os: OutputStream, message: String) {
    os.write("[$tag]: $message \n".toByteArray())
}


fun main(args: Array<String>) {
    //偏函数
    val makeString = ::log.curry()("Debug")(System.out)
    makeString("偏函数使用测试")

    val charSetString = { byteArray: ByteArray, charSet: Charset -> String(byteArray, charSet) }
    println(charSetString("charSetString".toByteArray(), charset("GBK")))
    //偏函数
    val partial2 = charSetString.partial2(charset("GBK"))
    println(partial2("partial2 GBK测试".toByteArray()))
}

fun <P1, P2, P3, R> Function3<P1, P2, P3, R>.curry() = fun(p1: P1) = fun(p2: P2) = fun(p3: P3) = this(p1, p2, p3)

fun <P1, P2, R> Function2<P1, P2, R>.partial2(p2: P2) = fun(p1: P1) = this(p1, p2)
```