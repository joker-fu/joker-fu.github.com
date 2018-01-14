---
title: Android TabLayout系列之属性
copyright: true
date: 2018-01-02 22:41:50
categories: [Android, TabLayout]
tags: [Android, TabLayout]
---
#### 1 前言
以前没有写博客的习惯，需要什么东西都是直接搜索，然后再来看怎么使用，有什么需要注意的，时间久了也就忘了是怎么回事了，又得重复这个过程。比如这个今天要介绍的TabLyout，很久之前用过，最近又需要用到，但是已经忘了怎么使用了，这也是为什么最近开始在简书上开始记录的原因。废话不多说...
其实Android在5.1（22.2.0）的时候给我们提供了一个水平布局来显示标签TabLayout，实际应用中也有很多需要横向标签的应用场景。直接先来看个效果图（偷的网易的）：
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-001.gif)

#### 2 TabLayout简单使用
今天这篇文章是奔着TabLayout的属性来的，就用最简单的使用方式来介绍它的属性。我们在xml文件中直接使用TabLayout：
```
    <android.support.design.widget.TabLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="tab1"/>

        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="tab2"/>

        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="tab3"/>

    </android.support.design.widget.TabLayout>
```
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-002.png)
看上去还是有那么点标签的意思...接下来就正式介绍它的属性了。
#### 3 属性介绍
1. 背景颜色
```
app:tabBackground="@android:color/white"
```
2. 选中tab字体颜色
```
app:tabSelectedTextColor="@android:color/holo_red_light"
```
3. 未选中tab字体颜色
```   
app:tabTextColor="@android:color/holo_blue_dark"
```
4. 指示器颜色
```
app:tabIndicatorColor="@android:color/holo_green_dark"
```
设置完演的后的TabLayout:

![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-003.png)
5. 指示器高度
```
app:tabIndicatorHeight="5dp"
```
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-004.png)
6. tabY轴偏移量（没看出效果）
```
app:tabContentStart="100dp"
```
7. tab显示方式
```
app:tabGravity="center"   //center：居中 fill：充满
```

![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-006.png)
8. tab最大最小宽度
```
app:tabMaxWidth="100dp"
app:tabMinWidth="100dp"
```
9. tab布局模式
```
app:tabMode="scrollable"  //可取fixed 固定,scrollable 滚动，默认fixed：标签很多时候会被挤压，不能滑动。
```
10. Tab里内容的内边距
```
app:tabPadding="10dp"
app:tabPaddingStart="10dp"
app:tabPaddingEnd="10dp"
app:tabPaddingTop="10dp"
app:tabPaddingBottom="10dp"
```
11. tab文字大小设置
```
app:tabTextAppearance="@style/Base.TextAppearance.AppCompat.Large"
```

![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-007.png)
以上就是TabLayout的基本属性，接下来我们用这些基本属性实现一个类似文章开头的效果:
```
    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.TabLayout
            android:id="@+id/tabLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabBackground="@android:color/white"
            app:tabIndicatorColor="@android:color/holo_red_light"
            app:tabIndicatorHeight="2dp"
            app:tabMode="scrollable"
            app:tabSelectedTextColor="@android:color/holo_red_light"
            app:tabTextColor="@android:color/darker_gray">

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab1"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab2"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab3"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab4"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab5"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab6"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab7"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab8"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab9"/>

            <android.support.design.widget.TabItem
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="tab10"/>

        </android.support.design.widget.TabLayout>

    </android.support.v4.view.ViewPager>
```
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E5%B1%9E%E6%80%A7/2018-01-02-008.gif)
TabLayout属性就介绍到这里，接下来的文章会具体介绍它的使用。

**附：**
1. [Android TabLayout系列之属性](http://www.abolg.com/2018/01/02/Android-TabLayout系列之属性)
2. [Android TabLayout系列之简单使用](http://www.abolg.com/2018/01/02/Android-TabLayout系列之简单使用)
3. [Android TabLayout系列之进阶使用](http://www.abolg.com/2018/01/02/Android-TabLayout系列之进阶使用)