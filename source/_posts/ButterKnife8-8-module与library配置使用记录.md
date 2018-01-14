---
title: ButterKnife8.8 module与library配置使用记录
copyright: true
date: 2018-01-02 22:44:48
categories: [Android, ButterKnife]
tags: [Android, ButterKnife]
---
本文使用前提是在Android Studio2.3.3 Stable版本使用记录，其他版本未测试。

#### 一、module中配置
 如果在module中配置，那就GitHub中配置一致三步曲操作，然后就可以正常使用了。

1 在module的build中添加依赖
```
dependencies {
  compile 'com.jakewharton:butterknife:8.8.1'
  annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
}
```
如果你是使用Kotlin, 将annotationProcessor替换为kapt.

2 project的build中添加
```
buildscript {
  repositories {
    mavenCentral()
   }
  dependencies {
    classpath 'com.jakewharton:butterknife-gradle-plugin:8.8.1'
  }
}
```

3 在module中apply:
```
apply plugin: 'com.android.library'
apply plugin: 'com.jakewharton.butterknife'
```

现在确定Butter Knife注解可以使用R2 替换 R。

```
class ExampleActivity extends Activity {
  @BindView(R2.id.user) EditText username;
  @BindView(R2.id.pass) EditText password;
...
}
```


#### 二、library中配置
一般我们会在BaseActivity中使用ButterKnife.bind(this)，但是我的base类都在library中。一开始我直接在library中配置，module依赖library，让module依赖library。但是直接报R2找不到，然后我就使用R替换，不过具体使用的地方会报空指针。看github的issue中有很多人也遇到这个问题，但是我是在没找到一个准确的解决办法，然后自己摸索了一个办法，还是可以使用的，如果有比较官方的办法，可以告诉我。

1 我们还是直接在中library按配置module方式进行配置

2 既然我们不能找到R2，说明我们module没有apply plugin。所以需要在对应的module的build添加如下代码，然后点击sync now并rebuild，R2已经不报错了。
```
//不使用下面这句，因为我们的library项目已经有了
//apply plugin: 'com.android.library'

apply plugin: 'com.jakewharton.butterknife'
```
3 R2不报错了，我们运行后，依旧会有空指针。我觉得是这个module没有解析到R2，导致BindView失败，所以将注解解析器依赖到对应的module
```
  annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
```
最后点击sync now并rebuild运行完美通过。


#### 三、配置小总结：
1 需要使用ButterKnife的module一定要配置如下两句：
```
 apply plugin: 'com.jakewharton.butterknife'

 annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
```