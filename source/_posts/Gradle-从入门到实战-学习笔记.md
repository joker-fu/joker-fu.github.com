---
title: Gradle 从入门到实战 学习笔记
copyright: true
date: 2018-01-02 22:35:40
categories: [Android, Gradle]
tags: [Android, Gradle]
---
Gradle是目前为止Android的主流构建工具，不管用命令行还是Android Studio来build项目，Gradle都不可或缺。Gradle不单单是一个配置脚本，更是**Groovy Language、Gradle DSL、Android DSL** 3门语言的组合( DSL的全称是Domain Specific Language，即领域特定语言 )。学习来源[Gradle从入门到实战 - Groovy基础](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649492338&idx=1&sn=49cb619fb057720db505b7c3b8f894e8&chksm=8eec808db99b099b6b0bc5e983fc10df48a085a78ca935593737ec9d76b373188e20cf1042d9#rd)，学习主要内容如下：

* Groovy基础
* 全面理解Gradle
* 分析Android的build tools插件
* 从0到1完成一款Gradle插件

#### 一 Groovy基础
通过def关键字声明变量和方法：
```
def a = 1;
def b = "hello";
def int c = 2;

def hello() {
    println ("hello world!");
    return 1;
}
```
在Groovy中语法中，很多地方可以省略：
- 语句后面的分号可以省略
- 变量类型可以省略
- 方法的参数类型和返回值可以省略
- 方法调用括号可以省略
- 方法中return关键字也可以省略

所以上面代码这样写也是正确的：
```
def a = 1 //省略分号
def b = "hello"
def c = 2

def hello() {
    println "hello world!" //省略调用时括号
    1 //省略return
}

def hello(String msg){
    println msg
}

int hello(msg){ //省略参数类型
    println (msg)
    return 1
}

int hello(msg){
    println msg
    return 1 // 这个return不能省略
    println "done"
}
```
**总结**
- 在Groovy中，类型是弱化的，所有的类型都可以动态推断，但是Groovy仍然是强类型的语言，类型不匹配仍然会报错
- 在Groovy中很多东西都可以省略，所以寻找一种自己喜欢的写法
- Groovy中的注释和Java中相同

#### 二 Groovy的数据类型
在Groovy中，数据类型有：
- Java中的基本数据类型
- Java中的对象
- Closure（闭包）
- 加强的List、Map等集合类型
- 加强的File、Stream等IO类型

类型可以显示声明，也可以用 def 来声明，用 def 声明的类型Groovy将会进行类型推断

1. String

```
def a = 1
def b = "hello"
def c = "a = ${a}, b = ${b}"

println c
```
    outputs:
    a = 1, b = hello

2. 闭包(Closure)
闭包在很多语言都存在，类似于C语言的指针。闭包作为一种特殊的数据类型而存在，闭包可以作为方法的参数和返回值，也可以作为一个变量

声明闭包：
```
{
  parameters ->
  code
}
```
闭包可以有返回和参数，也可以没有。看看以下例子：
```
def closure = { int a, String b ->
  println "a = ${a}, b = ${b}, This is a Closure!"
}

def test = { a, b ->
  println "a = ${a}, b = ${b}, This is a Closure!"
}

def sum = { a, b ->
  a + b
}

// 这里省略了闭包的参数类型
def testIt = {
  println "find ${it}, This is a Closure!"
}

closure(100, 200)
test.call(100, "test")
println sum(100, 200)
testIt(100)
```
闭包不指定参数，那么会隐含一个参数it,闭包可以当做函数一样使用，上例将得到以下输出：
```
a = 100, b = 200, This is a Closure!
a = 100, b = test, This is a Closure!
300
find 100, This is a Closure!
```
闭包的一个难题是如何确定闭包的参数，尤其当我们调用Groovy的API时，这个时候没有其他办法，只有查询Groovy的文档：
http://www.groovy-lang.org/api.html
http://docs.groovy-lang.org/latest/html/groovy-jdk/index-all.html

3. List和Map

```
def emptyList = []
def test = [100, "hello", true]
test[1] = "world"
test << 200
println test.size
println test[0]
println test[1]
println test[2]
println test[3]
```
    outputs:
	  100
	  world
	  true
	  200

上例中操作符<<，表示向List中新增加元素
```
def emptyMap = [:]
def test = ["id":1, "name":"test", "isMale":true]
test["id"] = 2
test.id = 900
println test.id
println test.isMale
```
    outputs:
      900
      true
 
通过闭包对Map进行遍历，如果闭包传递2个参数就是遍历key，value；如果不传参就是遍历entry：
```
def emptyMap = [:]
def test = ["id":1, "name":"test", "isMale":true]
test.each{ key, value ->
  println "two params, key = ${key}, value = ${value}"
}
test.each{
  println "one param, key = ${it.key}, value = ${it.value}"
}
```

4. 文件I/O
Groovy在使用I / O时提供了许多辅助方法，让文件操作比java中简单很多。下面举例，具体使用请查阅API：

```
def file = new File("E:/Example.txt")
println "-------------------------------"
file.eachLine{ line ->  //一行一行读取文件
  println "read line: $line"
}
println "-------------------------------"
file.eachLine{ line, lineNum -> 
  println "read  line ${lineNum}: $line"
}
println "-------------------------------"
println file.text //将文本以字符串读取
```
    outputs:
      -------------------------------
      read line: hello
      read line: 你好
      -------------------------------
      read line 1: hello
      read line 2: 你好
      -------------------------------
      hello
      你好

接下来看看xml文件访问，也是比Java中简单多了
Groovy访问xml有两个类：XmlParser和XmlSlurper，二者几乎一样，在性能上有细微的差别，具体的也请查看具体API。接下来看看示例，首先假设我们有attrs.xml文件：
```
<resources>
<declare-styleable name="CircleView">
    <attr name="circle_color" format="color">#98ff02</attr>
    <attr name="circle_size" format="integer">100</attr>
    <attr name="circle_title" format="string">xml</attr>
</declare-styleable>
</resources>
```
对xml文件操作
```
def xml = new XmlParser().parse(new File("attrs.xml"))
// 访问declare-styleable节点的name属性
println xml['declare-styleable'].@name[0]
// 访问declare-styleable的第三个子节点的内容
println xml['declare-styleable'].attr[2].text()
```
```
outputs：
CircleView
xml
```

#### 三 Groovy的其他特性

1. 在Groovy中，如在任何其他面向对象语言中一样，存在类和对象的概念以表示编程语言的对象定向性质。
```
class Student {
  int id
  String Name
	
  static void main(String[] args) {
    Student st = new Student();
    st.id= 1
    st.Name= "Joe"    
    fun(Student .class) //参数为class类型，可省略.class后缀 
    fun(Student)  //省略.class后缀 
 } 

  def func(Class clazz) {
  }
}
```

2. 在java中使用private关键字隐藏实例成员，而是提供getter和setter方法来相应地设置和获取实例变量的值。Groovy也是一样，不一样的是Groovy中只要有属性就有getter&setter，有getter&setter就有隐含的属性。所以以下两个类是一样的
```
class Student {
   private int id
	
   void setId(int id) {
       this.id = id
   }

   int getId() {
      return this.id
   }
}

class Student {
   private int id
}
```

3. 在Groovy中，当对同一个对象进行操作时，可以使用with操作符，比如：
```
class Student {
  int id
  String name
  static void main(String[] args) {
   Student st = new Student();
   st.id= 1
   st.name= "Joe"    
  //以上操作可以简化为
  st.with{
    id = 1
    name = "Joe"
  }
 } 
}
```

4. 在Groovy中，判断是否为真可以更简洁：
```
if (name != null && name.length > 0) {}
//Groovy中简写
if (name) {}
```

5. 在Groovy中，更加简洁的三目运算：
```
def name = "name"
def result = name != null ? name : "test"
//Groovy中简写
def result = name ? : "test"
```

6. 非空判断
```
if (order != null) {
    if (order.getCustomer() != null) {
        if (order.getCustomer().getAddress() != null) {
        System.out.println(order.getCustomer().getAddress());
        }
    }
}

//Groovy中简写
println order?.customer?.address
```

7. equals和==，Groovy中==与Java中equals一致，如果需要判断是否为同一对象需要使用.is()
```
Object a = new Object()
Object b = a.clone()

assert a == b //判断相等
assert !a.is(b)  //判断是否是同一对象
```

8. 断言关键字assert的使用
```
def check(String name) {
    assert name
    assert name?.size() > 3 //如果断言结果为false将抛出异常
}
```

9. 在Groovy中，switch方法更加灵活，可以同时支持更多的参数类型
```
在Groovy中，switch方法变得更加灵活，可以同时支持更多的参数类型：
def x = 1.23
def result = ""
switch (x) {
    case "foo": result = "found foo"
    // lets fall through
    case "bar": result += "bar"
    case [4, 5, 6, 'inList']: result = "list"
    break
    case 12..30: result = "range"
    break
    case Integer: result = "integer"
    break
    case Number: result = "number"
    break
    case { it > 3 }: result = "number > 3"
    break
    default: result = "default"
}

assert result == "number"
```

#### 四 编译并运行Groovy
我们可以通过下载和安装Groovy sdk来编译和运行，但是我们是为了学习Gradle，所以这里不用搞得过于复杂。推荐如下操作来编译和运行Groovy：
1. 首先在任意文件夹创建build.gradle
2. 在build.gradle中创建一个task
3. 使用gradle test命令编译和运行

拿上面的一个例子（可能需要配置环境变量，请自行百度，其实和Java环境变量配置方法一致），来做一个完整示例：

```
task(test).doLast {
    println "start execute test"
    doTest()
}
def doTest() {
	def emptyList = []
	def test = [100, "hello", true]
	test[1] = "world"
	test << 200
	println "以下是输出内容"
	println test.size
	println test[0]
	println test[1]
	println test[2]
	println test[3]
	println "--------------"
}
```

直接附上cmd命令行编译运行结果截图：

![具体操作.png](http://upload-images.jianshu.io/upload_images/6865060-efe5189ac88f9b7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再次感谢：任玉刚 - [Gradle从入门到实战](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649492338&idx=1&sn=49cb619fb057720db505b7c3b8f894e8&chksm=8eec808db99b099b6b0bc5e983fc10df48a085a78ca935593737ec9d76b373188e20cf1042d9#rd)