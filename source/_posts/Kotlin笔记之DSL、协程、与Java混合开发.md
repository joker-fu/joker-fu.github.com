---
title: Kotlin笔记之DSL、协程、与Java混合开发
copyright: true
date: 2018-03-18 16:13:07
categories: kotlin
tags: kotlin
---

### DSL
- 只在特定领域使用的语言(例如HTML、Gradle、SQL)
- 计算机编程语言
- 具有语言表达能力
- 有限的表达能力
- 关注某个特定领域

### 协程
1. 什么是协程
协作程序，解决异步问题
应用层完成调度
支持协程的语言（Lua C#）

2. 协程解决什么问题
异步代码像同步代码一样直观
简化异步代码异常处理
轻量级并发方案

3. kotlin如何支持协程
编译器suspend函数的编译支持
标准库的基本API支持

******

### 属性读写
- Kotlin 自动识别 Java Getter/Setter
- Java 操作Kotlin 属性通过 Getter/Setter

### 空安全类型
- Kotlin 空安全类型的理解
- 平台类型 Platform Type
- @Nullable 和 @NotNull

### 函数调用
- (Kotlin)包级函数 -> (Java) 静态方法
- (Kotlin)扩展方法 -> (Java) 带 Receiver 的静态方法
- (Kotlin)运算符重载 -> (Java) 带 Receiver 的对应名称的静态方法

### 常用注解
- @JvmField: 将属性编译为 Java 变量
- @JvmStatic: 将对象编译成 Java 静态方法
- @JvmOverloads: 默认参数生成重载方法
- @file:Jvmname: 指定 Koltlin 文件编译后的类名

### NoArg 与 AllOpen
- NoArg 为被标注的类生成无参构造方法（支持 Jpa 注解，如 @Entity）
- AllOpen 为被标注的类去掉 final， 允许被继承（支持 Spring 注解， 如 @Component）
- 支持自定义注解类型
注： Data Class 会用到

### 泛型
- 通配符 Kotlin 的 * 对应 Java 中的 ?
- 协变和逆变 out/in （ArrayList<out String>）
- 没有 Raw 类型 (Java 的List -> Kotlin 的 List<*>)

### SAM 转化
- Single Abstract Method
- SAM 转换的条件（java 接口，单一接口方法）
- 注意转换后的实例变化
- Kotlin 要使用类似效果 需要类型别名 （typealias）

### 正则表达式
- 用 Raw 字符串定义正则表达式
- Java 的 Pattern
- Kotlin 的 Regex
```kotlin
    val source = "Hello, my phone number is: 010-12345678"
    val pattern = ".*(\\d{3}-\\d{8}).*"
    val matcher = Pattern.compile(pattern).matcher(source)

    while (matcher.find()) {
        println(matcher.groupCount())
        println(matcher.group())
        println(matcher.group(1))
    }

    val source = "Hello, my phone number is: 010-12345678"
    val pattern = """.*(\d{3}-\d{8}).*"""

    Regex(pattern).findAll(source).toList().flatMap(MatchResult::groupValues).forEach(::println)
```

### 集合框架
- kotlin 到 Java 的类型映射
- Kotlin 的 List Map Set 是不可变集合接口
- Kotlin 的 MutableList MutableMap MutableSet 是不可变集合接口
- 部分接口优化 remove removeAt


### IO操作
- File Stream Reader Writer 的扩展方法
- 使用 use 扩展自动关闭资源
- 小文件一次性读写操作 readLines 等

### 拆箱与装箱

- 了解 Kotlin 基本类型到 Java 的映射关系
- 注意规避基本类型的相关的问题

### 使用Kotlin关键字
- Kotlin关键字in、object、is等等使用它们作为属性或方法调用时需要通过 `` 包含
```kotlin
    val `is` = 5
    println(`is`)
```

