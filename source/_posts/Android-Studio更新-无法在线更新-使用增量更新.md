---
title: Android Studio更新 无法在线更新 使用增量更新
copyright: true
date: 2017-12-27 16:00:49
categories: Android Studio
tags: Android Studio
---

最近Android Studio更新到了3.0版本，而我还是处于2.2版本，无法使用Android Studio Help下的check for updates...在线更新到最新版。由于本人比较懒，不想卸载后重装去配置环境等繁琐操作，记得偶然一次看视频时别人提到不能直接解决办法。只记得需要一步步升级，少量更新到最新版，具体操作在一顿Google下发现【**增量更新**】。很多博客都有写怎么操作，经过实践好多都不起作用，经过多次尝试最终成功。记录下操作，避免日后忘记....
***
### 一 、检查当前Android Studio版本
- 1  Android Studio Help => about 查看版本号，我的是145.3276617
- 2 Android Studio 安装根目录下build.txt查看版本号，同样是145.3276617
<small>**注**：此处没有图截到旧版图参考，可依照第五步截图</small>

### 二、获取最新的版本信息
  历史版本以及最新的版本信息可以在 **[https://dl.google.com/android/studio/patches/updates.xml](https://dl.google.com/android/studio/patches/updates.xml)**
![image.png](http://upload-images.jianshu.io/upload_images/6865060-75cb86b502bc9f80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 三、增量更新包下载
我们通过如下的方式拼接适合自己的增量包的URL：

https://dl.google.com/android/studio/patches/AI-[当前安装版本号]-[要更新的目标版本]-patch-win.jar
比如，如果按我们刚刚截图的版本号来获取离线包，则 URL 为：

https://dl.google.com/android/studio/patches/AI-145.3276617-162.3508619-patch-win.jar

### 四、增量更新包安装
- 1 将更新包放在除Android Studio 安装根目录之外的任意目录，可以放在其他盘
- 2 首先关闭所有的 Android Studio任务，把得到的 jar 更新包放到除Android Studio安装目录的任意目录下。在Android Studio根目录按住Shift+右键打开命令窗口或者运行打开CMD切换到Android Studio根目录，输入如下命令：

      java -classpath D:\AI-145.3276617-162.3508619-patch-win.jar com.intellij.updater.Runner install .

注意最后的 **.**，意味着将更新包解压到 Android Studio 的安装目录。然后回车执行命令。这时候会弹出对话框，展示安装具体进度：
![image.png](http://upload-images.jianshu.io/upload_images/6865060-88a104488d966e8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 3 正常更新结束，重启Android Studio导入以前配置即完成更新

![image.png](http://upload-images.jianshu.io/upload_images/6865060-3f93a1909032d647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 五、更新异常处理
如果你遇到安装错误，一般是如下：

> Some conflicts were found in the installation area.

>Please select desired solutions from the Solution column and press Proceed…..
 
一般可能的原因有如下一些：
- 你还有Android Studio任务没有关闭，检查一下，如果有，请关闭
- 你把离线jar放到了Android Studio的安装目录了，导致文件占用或其他一些错误
- 如果是Linux或者Mac系统，有可能需要sudo权限来执行安装命令

如此一步步增量，即可更新到能直接使用check for updates...在线更新，或者直接增量到最新版...
