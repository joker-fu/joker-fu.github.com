---
title: Fragment中使用setOnItemClickListener监听无效的解决办法
copyright: true
date: 2017-12-27 16:24:48
categories: Android
tags: [Android, Fragment]
---

Activity中用fragment 实现了一个ListView并对每一个Item设置监听，不是对Item里的组件设置监听。测试点击的时候发现点击不起作用，后来找到解决办法如下：
 需要在listview的item选项中配置** Android:focusable="false"**
``` 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="38dp"
              android:focusable="true"
              android:gravity="center_vertical"
              android:orientation="horizontal">

    <ImageView
        android:id="@+id/music_menu_image"
        android:layout_width="50dp"
        android:layout_height="match_parent"
        android:src="@drawable/img_icn_local"/>

    <TextView
        android:id="@+id/music_menu_text"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:gravity="center_vertical"
        android:textColor="@android:color/black"
        android:textSize="16sp"/>

    <TextView
        android:id="@+id/music_menu_num"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:gravity="center_vertical"
        android:paddingLeft="2dp"
        android:textSize="12sp"/>

</LinearLayout>
```