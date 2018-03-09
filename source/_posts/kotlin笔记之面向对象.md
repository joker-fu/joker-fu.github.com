---
title: kotlin笔记之面向对象
copyright: true
date: 2018-03-09 17:03:25
categories: kotlin
tags: kotlin
---
### 抽象类与接口
- 接口：直观理解是一种约定，是一种能力
接口不能有状态、必须有类对其进行实现后使用
```kotlin
//你可以在接口中定义属性。在接口中声明的属性要么是抽象的，要么提供访问器的实现。
//在接口中声明的属性不能有幕后字段（backing field），因此接口中声明的访问器不能引用它们。
interface MyInterface {
    val prop: Int // 抽象的

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```
- 抽象类
实现了一部分协议的半成品、可以有状态、可以有方法实现、必须由子类继承后使用
```kotlin
//类和其中的某些成员可以声明为 abstract。 抽象成员在本类中可以不用实现。 需要注意的是，我们并不需要用 open 标注一个抽象类或者函数
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

抽象类和接口共性
- 不能直接实例化
- 有需要子类（实现类）实现的方法
- 父类（接口）变量可以接受子类（实现类）的实例赋值

抽象类和接口的区别
- 抽象类有状态，接口没有状态
- 抽象类有方法实现，接口只能有无状态默认实现
- 抽象类只能单继承，接口可以多实现
- 抽象类反应本质，接口体现能力

覆盖方法
Kotlin 力求清晰显式。与 Java 不同，Kotlin 需要（open）显式标注可覆盖的成员（我们称之为开放）和覆盖后的成员：
```kotlin
open class Base {
    //可以覆盖
    open fun v() {}
    //没有open 修饰不可覆盖
    fun nv() {}
}
open class Derived() : Base() {
    // final 修饰禁止再次覆盖
    final override fun v() {}
}
```
Derived.v() 函数上必须加上 override标注。如果没写，编译器将会报错。 如果函数没有标注 open 如 Base.nv()，则子类中不允许定义相同签名的函数， 不论加不加 override。在一个 final 类中（没有用 open 标注的类），开放成员是禁止的。

标记为 override 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 final 关键字。

覆盖属性
覆盖属性和覆盖方法类似；在超类中声明然后在派生类中重新声明的属性必须以 override 开头，并且它们必须具有兼容的类型。每个声明的属性可以由具有初始化器的属性或者具有 getter 方法的属性覆盖。
```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    //var 属性覆盖一个 val 属性，但反之则不行
    override var count: Int = 0
}
```

类委托
委托模式已经证明是实现继承的一个很好的替代方式， 而 Kotlin 可以零样板代码地原生支持它。
```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print() // 输出 10
}
```
Derived 的超类型列表中的 by-子句表示 b 将会在 Derived 中内部存储。 并且编译器将生成转发给 b 的所有 Base 的方法。

请注意，覆盖会以你所期望的方式工作：编译器会使用你的 override 实现取代委托对象中的实现。如果我们为 Derived 添加 override fun print() { print("abc") }，该程序会输出“abc”而不是“10”。

覆盖规则
在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接超类继承相同成员的多个实现， 它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）。 为了表示采用从哪个超类型继承的实现，我们使用由尖括号中超类型名限定的 super，如 super< Base > ：
```kotlin
interface A {
    fun x(): Char = 'A'
}

interface B {
    fun x(): Char = 'B'
}

open class C {
    open fun x(): Char = 'C'
}

class D(private val type: Int) : C(), A, B {
    override fun x(): Char {
        return when {
            type == 0 -> super<C>.x()
            type > 0 -> super<A>.x()
            else -> super<B>.x()
        }
    }
}

fun main(args: Array<String>) {
    val d = D(10)
    println(d.x())
}
```

### 可见性修饰
 Kotlin 中有这四个可见性修饰符：private、 protected、 internal 和 public。 如果没有显式指定修饰符的话，默认可见性是 public。

类、对象、接口、构造函数、方法、属性和它们的 setter 都可以有 可见性修饰符。 （getter 总是与属性有着相同的可见性。）

包
函数、属性和类、对象和接口可以在顶层声明，即直接在包内：

- 如果你不指定任何可见性修饰符，默认为 public，这意味着你的声明将随处可见；
- 如果你声明为 private，它只会在声明它的文件内可见；
- 如果你声明为 internal，它会在相同模块内随处可见；
- protected 不适用于顶层声明。

**注意：要使用另一包中可见的顶层声明，仍需将其导入进来。**

类和接口
对于类内部声明的成员：

- private 意味着只在这个类内部（包含其所有成员）可见；
- protected—— 和 private一样 + 在子类中可见。
- internal —— 能见到类声明的 本模块内 的任何客户端都可见其 internal 成员；
- public —— 能见到类声明的任何客户端都可见其 public 成员。

**注意 对于Java用户：Kotlin 中外部类不能访问内部类的 private 成员。**

如果你覆盖一个 protected 成员并且没有显式指定其可见性，该成员还会是 protected 可见性。

模块
可见性修饰符 internal 意味着该成员只在相同模块内可见。更具体地说， 一个模块是编译在一起的一套 Kotlin 文件：

- 一个 IntelliJ IDEA 模块；
- 一个 Maven 项目；
- 一个 Gradle 源集（例外是 test 源集可以访问 main 的 internal 声明）；
- 一次 ＜kotlinc＞ Ant 任务执行所编译的一套文件。

### object
- 只有一个实例的类
- 不能自定义构造方法
- 可以实现接口、继承父类
- 本质上就是单例模式最基本的实现

### 伴生对象与静态成员
- 每个类可以对应一个伴生对象
- 伴生对象的成员全局独一份儿 相当于java 静态成员
- 静态成员考虑包级函数、变量替代
- JvmField 和 JvmStatic 使Java 调用时更像静态方法 成员
```kotlin
fun main(args: Array<String>) {
    val latitude = Latitude.ofDouble(20.3)

    println(latitude)

    println(Latitude.TAG)
}

class Latitude private constructor(private val value: Double) {

    companion object Factory {
        //静态方法 JvmStatic修饰 使用Java调用时 更符合Java静态方法调用样式
        @JvmStatic
        fun ofDouble(value: Double): Latitude {
            return Latitude(value)
        }

        //静态属性 const 或 JvmField修饰 使用Java调用时 更符合Java静态属性调用样式
        const val TAG = "Latitude"
    }

    override fun toString(): String {
        return value.toString()
    }
}
```

### 方法重载和默认参数
方法重载(Overloads)
- 名称相同 参数不同的方法
- Jvm函数签名的概念：函数名、参数列表
- 函数签名跟返回值没关系

默认参数
- 为函数参数提供一个默认值
- 可以为任意位置参数设置默认值
- 函数调用产生混淆时使用具名参数

方法重载与默认参数关系
- 二者可以相互转换
- @JvmOverloads Java 调kotlin重载方法有默认参数方法时使用
- 避免定义关系不大的重载（不能用默认参数替换重载）
不好的设计：
```
//当List中存放的时int值 假设remove(2)，则会产生意想不到的问题
List.remove(int)
List.remove(Object)
```

### 扩展成员
Kotlin 同 C# 和 Gosu 类似，能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。 这通过叫做 扩展 的特殊声明完成。Kotlin 支持 扩展函数 和 扩展属性。

- 为现有的类添加方法、属性
> fun X.y()：Z{...}
val X.m 注意扩展属性不能有初始化器，类似接口属性
- Java 调用阔咱成员类似调用静态方法

```kotlin
// 使用扩展属性(extension property)
var View.padLeft: Int
    set(value) =
        setPadding(value, paddingTop, paddingRight, paddingBottom)
    get() {
        return paddingLeft
    }
```

### 委托属性
语法是： val/var <属性名>: <类型> by <表达式>。在 by 后面的表达式是该 委托， 因为属性对应的 get()（和 set()）会被委托给它的 getValue() 和 setValue() 方法。 属性的委托不必实现任何的接口，但是需要提供一个 getValue() 函数（和 setValue()——对于 var 属性）。
```kotlin
class Example {
    var p: String by Delegate()
}

class Delegate {
    var value: String? = null
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        println("$thisRef, thank you for delegating '${property.name}' to me!")
        return value ?: ""
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
        this.value = value
    }
}
```
标准委托，Kotlin 标准库为几种有用的委托提供了工厂方法。
- 延迟属性 Lazy
- 可观察属性 Observable
- 把属性储存在映射中

### 数据类
- 替换JavaBean
- 默认实现copy、toString、equals等方法
- componentN方法 按声明顺序对应于所有属性
- allOpen（去掉final关键字） 和 noArg（生成无参构造方法）插件

在 JVM 中，如果生成的类需要含有一个无参的构造函数，则所有的属性必须指定默认值。

### 嵌套类
内部类
- 定义在类内部的类
- 与类成员有相似的访问控制
- 默认是静态内部类，非静态内部类使用 inner 修饰
- this@Outter，this@Inner的用法

匿名内部类
- 没有定义名字的内部类
- 类名编译时生成，类似Outter$1.class
- 可继承父类、实现多个接口，与Java注意区别

### 枚举
- 实例可数的类，注意枚举类也是类
- 可以修改构造，添加成员
- 可以提升代码表现力也有一定性能开销

### 密封类 
声明一个密封类，需要在类名前面添加 sealed 修饰符。虽然密封类也可以有子类，但是所有子类都必须在与密封类自身相同的文件中声明。
```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```
（上文示例使用了 Kotlin 1.1 的一个额外的新功能：数据类扩展包括密封类在内的其他类的可能性。 ）

一个密封类是自身抽象的，它不能直接实例化并可以有抽象（abstract）成员。

密封类不允许有非-private 构造函数（其构造函数默认为 private）。

请注意，扩展密封类子类的类（间接继承者）可以放在任何位置，而无需在同一个文件中。

使用密封类的关键好处在于使用 when 表达式 的时候，如果能够验证语句覆盖了所有情况，就不需要为该语句再添加一个 else 子句了。当然，这只有当你用 when 作为表达式（使用结果）而不是作为语句时才有用。
```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 不再需要 `else` 子句，因为我们已经覆盖了所有的情况
}
```
