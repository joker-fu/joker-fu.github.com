---
title: Java Android 注解
copyright: true
date: 2018-01-06 11:19:32
categories: [Java,Android]
tags: [Java,Android]
---
#### 一 注解说明
注解(Annotation)，也叫元数据。从 JDK 1.5 开始，java 开始支持注解。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。文档 JSR-269 将它规范化，在 JDK 1.6 写入编译器 javac 中。

#### 二 注解分类

##### 【内置注解】

###### 1. 作用在代码的注解  
```
@Override - 检查该方法是否是重载方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
@Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
@SuppressWarnings - 指示编译器去忽略注解中声明的警告。
```
###### 2. 作用在其他注解的注解(或者说 元注解)
```
@Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。
@Documented - 标记这些注解是否包含在用户文档中。
@Target - 标记这个注解应该是哪种 Java 成员。
@Inherited - 标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)
```
###### 3. 从 Java 7 开始，额外添加了 3 个注解
```
@SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
@FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。
@Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。
```
##### 【自定义注解】

#### 三 注解作用域
###### 1. 通过元注解@Retention标识注解生命周期：
```
源码时注解（RetentionPolicy.SOURCE） //源代码注解 仅在源码有效
编译时注解（RetentionPolicy.CLASS）  //编译是注解 class文件有效
运行时注解（RetentionPolicy.RUNTIME）  //运行时注解 运行时有效 可以反射访问
```
###### 2. 通过元注解@Target 标识注解作用范围：
```
PACKAGE:用于包
TYPE:用于类、接口(包括注解类型) 或enum声明
FIELD:用于属性
CONSTRUCTOR:用于构造器
METHOD:用于方法
PARAMETER:用于参数
LOCAL_VARIABLE:用于局部变量
ANNOTATION_TYPE：用于注解类型
```
#### 四 自定义注解
上面主要描述了关于注解的一些说明，接下来通过一个示例尝试自定义一个注解：
1.自定义注解格式
```
public @interface 注解名 { 定义体 }
```
2.注解参数数据类型
- 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
- String类型
- Class类型
- enum类型
- Annotation类型
- 以及以上所有类型的数组

3.android应用示例
```
//注解类
@Retention(RetentionPolicy.RUNTIME) //运行时注解
@Target(ElementType.FIELD) //作用于属性
public @interface BindView {
    @IdRes
    int value() default -1;
}

//注解解析类
public class Bind {
    public static void inject(Activity activity) {
        Class<?> target = activity.getClass();
        Field[] fields = target.getDeclaredFields();
        try {
            for (Field field : fields) {
                if (field.isAnnotationPresent(BindView.class)) {
                    BindView bind = field.getAnnotation(BindView.class);
                    if (bind.value() != -1) {
                        field.setAccessible(true); //field设置
                        field.set(activity, activity.getWindow().getDecorView().findViewById(bind.value()));
                    }
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}

//测试Activity 布局就一个TextView就不给出了（默认android:text = "cs"）
public class BindActivity extends AppCompatActivity {

    @BindView(R.id.test)
    TextView test;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bind);
        test.setText("测试");
    }
}

```

4.示例运行效果

![运行效果图.png](http://114.67.156.211/bolg/android/Java%20Android%20%E6%B3%A8%E8%A7%A3/2018-01-06-001.png)
