---
title: Gradle 打包配置 常用技巧
copyright: true
date: 2018-02-12 00:38:16
categories: [Android, Gradle]
tags: [Android, Gradle]
---
前面写了一篇文章关于使用[Gradle 多渠道打包 动态修改应用标题 / 应用图标 / 版本名 / 版本号](http://www.abolg.com/2018/02/10/Gradle-%E5%A4%9A%E6%B8%A0%E9%81%93%E6%89%93%E5%8C%85-%E5%8A%A8%E6%80%81%E4%BF%AE%E6%94%B9%E5%BA%94%E7%94%A8%E6%A0%87%E9%A2%98-%E5%BA%94%E7%94%A8%E5%9B%BE%E6%A0%87-%E7%89%88%E6%9C%AC%E5%90%8D-%E7%89%88%E6%9C%AC%E5%8F%B7/#more)，发现我们经常会在Gradle里面做一些常用的处理。所以有了这篇文章来作为记录使用，方便以后查阅。

### 一、 构建类型
```java
android {

  ......

    buildTypes {
        release {
            //混淆开关
            minifyEnabled true
            //是否zip对齐
            zipAlignEnabled true
            //是否打开debuggable开关
            debuggable false
            //是否打开jniDebuggable开关
            jniDebuggable false
            //是否移除无用资源
            shrinkResources true
            //签名配置
            signingConfig signingConfigs.ks
            //applicationId后缀
            applicationIdSuffix ".release"
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled false
            applicationIdSuffix ".debug"
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        jnidebug {
            // 拷贝指定的构建类型属性
            initWith release
            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }

  ......

}
```

### 二、 多渠道/多环境不同配置
```java
android {

  ......

    // 维度：用于组建多个Flavor，这里指定了2个维度 isPay 、 isFree
    flavorDimensions "isPay", "isFree"

    // 多Flavor（渠道）配置
    productFlavors {

        main1 {
            dimension "isPay"
            //当前渠道包名
            applicationId "com.****.****"
            //当前渠道版本号
            versionCode 1
            //当前渠道版本名
            versionName "1.0"
            //当前渠道版本名后缀
            versionNameSuffix "-main1"
            //applicationId后缀  这里的指定的后缀会在构建类型的前面（com.****.****".main1.release）
            applicationIdSuffix ".main1"
            //当前渠道资源常量 不能与资源文件已有名字相同 否则会报错
            //R.string.app_name
            resValue "string", "app_name", "app_name"
            // 当前渠道动态修改常量
            //BuildConfig.API_HOST
            buildConfigField "String", "API_HOST", '"www.main1.com"'
            //当前渠道 AndroidManifest.xml 占位变量
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }

        main2 {
            dimension "isPay"
            applicationId "com.****.****"
            versionCode 2
            versionName "2.0"
            versionNameSuffix "-main2"
            applicationIdSuffix ".main2"
            resValue "string", "app_name", "app_name"
            buildConfigField "String", "API_HOST", '"www.main2.com"'
            manifestPlaceholders = [app_name_desktop: "app_name_desktop"]
        }

        pay {
            dimension 'isFree'
        }

        free {
            dimension 'isFree'
        }
    }

  ......

}
```
productFlavors 支持与 defaultConfig 相同的属性，这是因为 defaultConfig 实际上属于 ProductFlavor 类。这里构建了2个维度、2个Flavor，实际打包时会根据 flavorDimensions 、 productFlavors、buildTypes 生成不同的包，假设这里打relase包，则是生成main1PayRelease、main1FreeRelease。**这里需要说明的是 productFlavors 构建出的 Flavor 设置会覆盖 defaultConfig 里面的设置。**

对于项目而言，有时候需要配置某些敏感信息。比如API_HOST、appKey等,我们不希望别人看到。而这些信息需要被很多类共同使用，所以必须有一个全局的配置，那么我们可以将这些信息设置在gradle.properties中。

```
    //当前渠道资源常量 不能与资源文件已有名字相同 否则会报错
    resValue "string", "app_key", app_key
    // 当前渠道动态修改常量
    buildConfigField "String", "API_HOST", API_HOST
```

gradle.properties文件中进行变量初始化
```
  ......
    app_key = "1234567890"
    API_HOST = "http://120.79.189.85:3000"
```

### 三、 过滤变体
```java
android {

  ......

  variantFilter { variant ->
      def names = variant.flavors*.name
      // To check for a certain build type, use variant.buildType.name == "<buildType>"
      if (names.contains("main1 ") && names.contains("free")) {
          // Gradle ignores any variants that satisfy the conditions above.
          setIgnore(true)
      }
  }

  ......

}
```

### 四、 创建源集与更改默认源集配置

1. 创建原集
默认情况下，Android Studio 会创建 main/ 源集和目录，用于存储您要在所有构建变体之间共享的一切资源。然而，您可以创建新的源集来控制 Gradle 要为特定的构建类型、productFlavors （以及使用维度时的 productFlavors  组合）和构建变体编译和打包的确切文件。创建源集方式在[Gradle 多渠道打包 动态修改应用标题 / 应用图标 / 版本名 / 版本号](http://www.abolg.com/2018/02/10/Gradle-%E5%A4%9A%E6%B8%A0%E9%81%93%E6%89%93%E5%8C%85-%E5%8A%A8%E6%80%81%E4%BF%AE%E6%94%B9%E5%BA%94%E7%94%A8%E6%A0%87%E9%A2%98-%E5%BA%94%E7%94%A8%E5%9B%BE%E6%A0%87-%E7%89%88%E6%9C%AC%E5%90%8D-%E7%89%88%E6%9C%AC%E5%8F%B7/#more)第二点已经说明了。**注意：Gradle 要求您按照与 main/ 源集类似的特定方式组织源集文件和目录。**

2. 更改默认源集配置
如果您的源未组织到 Gradle 期望的默认源集文件结构中（如上面的创建源集部分中所述），您可以使用 sourceSets {} 代码块更改 Gradle 希望为源集的每个组件收集文件的位置。您不需要重新定位文件；只需要为 Gradle 提供相对于模块级 build.gradle 文件的路径，Gradle 应当可以在此路径下为每个源集组件找到文件。
```java
 android {

  ......

     sourceSets {
        main1 {
            //main1 Flavor 使用main2 Flavor 的res 这里只是举例 据实际情况编写
            res.srcDir('src/main2/res')
        }

        main2 {
            //main2 Flavor 使用main1 Flavor 的res
            res.srcDir('src/main1/res')
        }
    }

  ......

}
```

### 五、批量打包重命名
有时候发给测试或者产品的包需要根据一些指定规则来命名，每次都手动修改显得很麻烦，打一个包也还好，但是一次打release、debug都要改就更麻烦了。
```java
 android {

  ......

	applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def flavors = variant.productFlavors[0]
                def fileName
                if(variant.buildType.name == 'release'){
                    //将release版本的包名重命名
                    fileName = "${flavors.applicationId}_V${flavors.versionName}_${releaseTime()}.apk"
                }else{
                    //将debug版本的包名重命名
                    fileName = "${flavors.applicationId}_V${flavors.versionName}_debug_${releaseTime()}.apk"
                }
                output.outputFile = new File(outputFile.parent, fileName)
            }
       

  ......

}

static def releaseTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}
```

### 六、 移除lint检测的error
```java
 android {

  ......
  
    lintOptions {
        //lint 遇到 error 时继续 构建
        abortOnError false
        //build release 版本 时 开启lint 检测
        checkReleaseBuilds false
        // 防止在发布的时候出现因MissingTranslation导致Build Failed!
        disable 'MissingTranslation'
    }

  ......

}
```

### 七、签名设置
除非您为发布构建显式定义签名配置，否则，Gradle 不会签名发布构建的APK。

要使用 Gradle 构建配置为您的发布构建类型手动配置签署配置：
1. 创建密钥库。密钥库是一个二进制文件，它包含一组私钥。您必须将密钥库存放在安全可靠的地方。
2. 创建私钥。私钥代表将通过应用识别的实体，如某个人或某家公司。
3. 将签署配置添加到模块级 build.gradle 文件中：
```java
 android {

  ......

    defaultConfig {...}
    signingConfigs {
        release {
            storeFile file("myreleasekey.jks")
            storePassword "password"
            keyAlias "MyReleaseKey"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }

  ......

}
```
**注：将发布密钥和密钥库的密码放在构建文件中并不安全。作为替代方案，您可以将此构建文件配置在local.properties。**

使用 local.properties 存放私密配置：
在项目跟目录下，有个 local.properties 文件，我们可以使用它来存放一些私密的属性，然后在 gradle 中读取，而 local.properties 文件不需要上传。

local.properties 文件里设置如下：
```java
#对应自己实际的证书路径和名字，这里签名文件是放在app目录下，没有写绝对路径。
keystroe_storeFile = myreleasekey.jks
keystroe_storePassword = password
keystroe_keyAlias = MyReleaseKey
keystroe_keyPassword = password
```

在 build.gradle 读取 local.properties 字段信息
```java

//获取local.properties的内容
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
 
android {

  ......

    //第二种：为了保护签名文件，把它放在local.properties中并在版本库中排除
    // ，不把这些信息写入到版本库中（注意，此种方式签名文件中不能有中文）
    signingConfigs {
        config {
            storeFile file(properties.getProperty("keystroe_storeFile"))
            storePassword properties.getProperty("keystroe_storePassword")
            keyAlias properties.getProperty("keystroe_keyAlias")
            keyPassword properties.getProperty("keystroe_keyPassword")
        }
    }

  ......

}
```
这样就可以将自己想要隐藏的一些数据隐藏起来。

### 八、 dependencies 依赖配置
不同 buildTypes 和 productFlavors 依赖不同的jar的配置情况
```java
 dependencies {
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
    main1Compile files("libs****.jar")
    main1Compile files("libs/****.jar")
    main2Compile files("libs/****.jar")
    main2Compile files("libs/****.jar")
}
```

感谢阅读！有错之处还望指正......
