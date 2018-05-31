---
title: Android全屏设置、全屏切换到非全屏状态栏问题修复
copyright: true
date: 2018-05-11 22:20:04
categories: [Android]
tags: [Android]
---
# Android全屏设置、全屏切换到非全屏状态栏问题修复

### 全屏设置

- 方式一: setContentView前调用如下代码
```kotlin
        requestWindowFeature(Window.FEATURE_NO_TITLE)
        window.setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN)
```
 
- 方式二：manifest对应activity设置theme
```kotlin
    <!--全屏隐藏 标题栏-->
    <style name="AppTheme.FullScreen">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>
```

### 问题修复

- 方式一: 全屏跳转到非全屏之前，清除全屏
```kotlin
    private fun toNext() {
        //清除全屏标记 推荐
        window.clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN)
        //设置为非全屏
        //window.setFlags(WindowManager.LayoutParams.FLAG_FORCE_NOT_FULLSCREEN, WindowManager.LayoutParams.FLAG_FORCE_NOT_FULLSCREEN)
        startActivity(Intent(this, LoginActivity::class.java))
        finish()
    }
```
- 方式二 ：在非全屏页面setContentView前调用
```kotlin
    private fun smoothSwitchScreen() {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            val rootView = this.findViewById(android.R.id.content) as ViewGroup
            val resourceId = resources.getIdentifier("status_bar_height", "dimen", "android")
            val statusBarHeight = resources.getDimensionPixelSize(resourceId)
            rootView.setPadding(0, statusBarHeight, 0, 0)
            window.addFlags(WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN)
            window.addFlags(WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS)
        }
    }
```