---
title: Kotlin笔记之程序结构
copyright: true
date: 2018-02-27 10:52:14
categories: Kotlin
tags: Kotlin
---
### 常量与变量
- 常量
-- val = value, 值类型
-- 类似Java的final
-- 不可重复赋值
>运行时常量：val x = 5
编译期常量：const val x = 5

- 变量
-- var = variable
> var x = "x"
x = "y" //再次赋值

- 类型推导
> val s = "hello" //String
val x = 5 //Int
var z = "z" + 5 //String

### 函数
函数定义：
-- fun [函数名]([参数列表]):[返回值类型]{[函数体]}
-- fun [函数名]([参数列表])=[表达式]
匿名函数：
fun(参数列表)...
```kotlin
//匿名函数
val result = fun(x: Int) = 2 * x

//函数
fun double1(x: Int): Int {
    return 2 * x
}

fun double2(x: Int): Int = 2 * x

fun out1(): Unit {
    println("out")
}

fun out2() {
    println("out")
}

fun inner() {
    val s = "String"

    println(s)
    //局部函数
    fun inInner() {
        println("in inner $s")
    }

    inInner()
}

fun sum(a: Int, b: Int = 2, c: Int = 3): Int {
    return a + b + c
}
```

###Lambda 表达式
Lambda表达式是匿名函数：
{[参数列表] -> [函数体， 最后一行是返回值]}
```kotlin
var mul = { arg1: Int, arg2: Int ->
    println("-------------")
    arg1 * arg2
}

newSum(3, 2, { s, y -> (s * y).toLong() })

fun newSum(a: Int, b: Int, body: (Int, Int) -> Long) {
    println(body(a, b))
}
```
Lambda 的类型示例：
- ()->Unit 无参 返回Unit
- (Int)->Int 传入Int 返回Int
- (String,(String)->String)->Boolean 传入String Lambda表达式 返回Boolean
Lambda表达式调用：
用()调用 等价于 invoke()
> var mul = { arg1: Int, arg2: Int ->
    println("-------------")
    arg1 * arg2
}
sum(2,3)
sum.invoke(2,3)

Lambda 表达式简化：
函数参数调用时最后一个Lambda可以移出去
函数参数只有一个Lambda，调用时小括号可以省略
Lambda 只有一个参数可以认为it
入参、返回值与形参一致的函数可以用函数引用的方式作为实参传入

### 类成员

1. 属性 或者说成员变量，类范围内的变量
2. 方法 或者说成员函数，类范围内的函数

函数和方法的区别：
- 函数强调功能本身，不考虑从属
- 方法的称呼通常是从类的角度出发

定义方法： 写法与普通函数完全一致，只是它时写在类里面的

定义属性： 
- 构造方法参数中 val/var 修饰的都是属性 没有修饰的仅是一个参数
- 类内部也是可以定义属性

属性访问控制：
属性可以定义 getter / setter (val 只能定义getter)

属性初始化：
- 属性的初始化尽量放在构造方法中完成
- 无法再构造方法中初始化， 尝试降级为局部变量
- 若无法在工造函数中初始化且无法降级 var 用lateinit 延迟初始化 val 用lazy
- 可空类型谨慎用 null 初始化，否则使用时需要做 null 处理

```kotlin
class A {
    //常量 const 修饰则是编译器常量
    val FINAL_STRING = "hello world"
    var filedA = 0
    //常量只能有getter
    val fieldB: Int = 1
        get() {
            return field + 1
        }
    //修改getter setter
    var fieldC = "sss"
        get() {
            return field + ": bye bye"
        }
        set(value) {
            field = "hello" + value
        }
    var filedD = 5
        get() = 10
    //var 延迟初始化
    lateinit var filedE: String
    //val 延迟初始化
    val filedF: String by lazy {
        println("init x")
        "sssssss"
    }
}

fun main(args: Array<String>) {
    val a = A()
    println(a.FINAL_STRING)
    println(a.filedA)
    println(a.fieldB)
    a.fieldC = "zhang"
    println(a.fieldC)
    println(a.filedD)
    a.filedE = "filedE"
    println(a.filedE)
    println(a.filedF)
}
```

### 基本运算符
运算符重载：
- 任何类可以定义或者重载父类的基本运算符
- 通过运算符对应的具名函数来定义
- 对于参数个数作要求，对参数和返回值类型不作要求
- 不能像Scala一样定义任意运算符
```kotlin
class Complex(val x: Double, val y: Double) {
    operator fun plus(other: Complex): Complex {
        return Complex(x + other.x, y + other.y)
    }

    operator fun plus(other: Int): Complex {
        return Complex(x + other, y)
    }

    operator fun plus(other: Double): Int {
        return (x + y + other).toInt()
    }

    override fun toString(): String {
        return "$x + ${y}i"
    }
}


fun main(args: Array<String>) {
    val c1 = Complex(3.0, 4.0)
    val c2 = Complex(2.0, 7.5)
    println(c1 + c2)
    println(c1 + 5 + 4.0)

    println(c1)
}
```

### 表达式
中缀表达式：
只有一个参数 且用infix修饰的函数
```kotlin
class Book {
    infix fun on(any: Any): Boolean {
        return any is Desk
    }
}

class Desk {

}

fun main(args: Array<String>) {
    println(Book() on Desk())
}
```

if 表达式：
- if ... else
- 表达式与完备性
```kotlin
const val NAME_ADMIN = "admin"
const val PASSWORD_ADMIN = "admin"

const val NAME_USER = "user"
const val PASSWORD_USER = "user"

const val ADMIN_MODE = 0
const val USER_MODE = 1

fun main(args: Array<String>) {
    //表达式
    val mode = if (args.isNotEmpty() && args[0] == "0")
        ADMIN_MODE//最后的表达式作为该块的值
    else
        USER_MODE

    println("请输入用户名：")
    val name = readLine()
    println("请输入用户密码：")
    val password = readLine()

    if (mode == ADMIN_MODE && name == NAME_ADMIN && password == PASSWORD_ADMIN)
        println("管理员登录成功！")
    else if (name == NAME_USER && password == PASSWORD_USER)
        println("用户登录成功！")
    else
        println("登录失败")
}
```

when 表达式：
- 加强版switch,支持任意类型
- 支持纯表达式条件分支（类似if）
- 表达式与完备性
```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}

//表达式
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

### 循环语句

for 循环

while 循环

do...while 循环

跳过当前循环 continue 终止循环 break

多层循环嵌套的终止结合标签使用
```kotlin
Outter@for(...){
    Inner@while(i < 0){
        if(...) break@Outter
    }
}
```

### 异常捕获
>try {
    // 一些代码
}
catch (e: SomeException) {
    // 处理程序
}
finally {
    // 可选的 finally 块
}

可以有零到多个 catch 块。finally 块可以省略。 但是 catch 和 finally 块至少应该存在一个。 
try...catch 也是表达式 类似 if...else / when
 
### 具名参数
给函数的实参附上形参
```KOTLIN
fun sum(x: Int, y: Int) = x + y

//具名参数 交换参数顺序也可以执行
sum(x = 1, y = 2)
```

### 变长参数
关键字 vararg
```kotlin
fun hello (vararg ints: int, string: String){
    ...
}

hello(1,2,3,4,string = "123456")

var array = intArrayOf(1,2,3,4)
//*展开Array 仅支持array 仅支持边长参数
hello(*array,string = "1,2,3,4")
```

### 默认参数
```kotlin
//默认参数如果不是最后一个参数 则它后面参数需要使用具名参数
fun hello (vararg ints: int, string: String = "123456"){
    ...
}
```