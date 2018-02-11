---
title: Gradle 多渠道打包 动态修改应用标题 / 应用图标 / 版本名 / 版本号
copyright: true
date: 2018-02-10 22:54:15
categories: [Android, Gradle]
tags: [Android, Gradle]
---
最近有个需求需要一次打4个包，且它们具有不同的标题、不同的图标、不同的版本号、版本名。虽然之前项目也需要打release和debug包，但是一次也只打包一个，仅仅更改下API_HOST，且没有更换图标这些需求。恰逢这次，需求方一直要求我打包，一次4个每次都要去改标题、包名、版本号、版本名，好不容易打包完。然后告诉我错了要修改......一万只草泥马奔腾啊！
根据之前的了解，Gradle 是可以解决这个问题的。所以就决定配置下 Gradle 实现需求。

### 一、使用productFlavors构建Flavor
```java
    productFlavors {

        main1 {
            applicationId "com.****.****"
            versionCode 5
            versionName "1.0.4"
            resValue "string", "app_name", "app_name"
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }

        main2 {
            applicationId "com.****.****"
            versionCode 43
            versionName "1.3.7"
            resValue "string", "app_name", "app_name"
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }

        main3 {
            applicationId "com.****.****"
            versionCode 2017053026
            versionName "6.8.2"
            resValue "string", "app_name", "app_name"
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }

        main4 {
            applicationId "com.****.****"
            versionCode 100006
            versionName "2.0.6"
            resValue "string", "app_name", "app_name"
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }
    }

```
如上我分别设置了4个包的applicationId、versionCode 、versionName ，且设置了resValue、manifestPlaceholders 。这里主要是需求方要求每个应用里面显示名字和桌面显示名字不一样，所以我增加了app_name_desktop用于桌面显示的应用名，而app_name就作为应用中显示使用。

这样就做到了4个包的的包名、版本号、版本名以及应用名的配置，**这里需要说明的是 productFlavors 构建出的 Flavor 设置会覆盖 defaultConfig 里面的设置。**

### 二、为对应Flavor创建文件夹
接下来需要根据上面 productFlavors 中的 Flavor 创建文件夹，创建位置如下：

![对应Flavor文件夹](http://114.67.156.211/bolg/android/gradle_20180210/1.png)

这里**创建的 Flavor 文件夹中的子文件夹需要和main目录层级中结构一致，**内容根据自己的需求酌情增加或删除。例如我这个项目中只保存了drawable-xhdpi、drawable-xxhdpi、values三个文件夹，里面的内容也是一样的。可以看到drawable-xhdpi、drawable-xxhdpi只有ic_launcher.png，而values中的strings.xml中内容如下图。

![strings内容](http://114.67.156.211/bolg/android/gradle_20180210/2.png)

我这里注释掉了，是因为我后来把他定义在了 productFlavors 中，所以这里面不需要了。打包时Flavor中的文件会替换掉main中的，比如main1中的ic_launcher.png会替换main中的ic_launcher.png。如果strings.xml中的app_name没有注释掉，它也会替换掉main中的app_name。

这就完成了应用图标和应用里面app_name的替换。

### 三、manifest修改应用名

![manifest修改](http://114.67.156.211/bolg/android/gradle_20180210/3.png)

这里额外说下：
- applicantion的label ：安装和卸载应用时显示的名称 。
- launcher activity的label：App标题和桌面上App的显示名称。


修改应用图标也可以使用resValue、manifestPlaceholders方式如下：

**resValue:**
```java
    //Flavor添加
    resValue "drawable", "app_icon", "@drawable/ic_launcher"
    
    <application
        android:name=".app.RedApp"
        android:allowBackup="true"
        android:icon="@drawable/app_icon"
        android:label="${app_name_desktop}"
        android:theme="@style/Theme.AppCompat.AppTheme">
        .....
    </application>
```

**manifestPlaceholders:**
```java
    //Flavor添加
    manifestPlaceholders = [app_icon : "@drawable/icon_prod"]
    
    <application
        android:name=".app.RedApp"
        android:allowBackup="true"
        android:icon="${app_name_desktop}"
        android:label="${app_name_desktop}"
        android:theme="@style/Theme.AppCompat.AppTheme">
        .....
    </application>
```