---
title: Android 轮播图之BannerPager
copyright: true
date: 2018-02-06 14:31:44
categories: [Android,Banner]
tags: [Android,Banner]
---
### BannerPager简介

[BannerPager](https://github.com/joker-fu/BannerPager)是一个Android Banner库，它支持无限轮播、预显示前后页、指示器位置、颜色、大小修改、点击/选中监听、滚动速度控制、设置自动翻页和时间、手指触碰暂停翻页，离开自动翻页、默认提供缩放/卡片/手风琴/立方体翻页效果 ,支持recyclerview列表嵌套 自动控制轮播

### BannerPager效果图

![更多效果查看demo](http://114.67.156.211/bolg/android/Androir-%E8%BD%AE%E6%92%AD%E5%9B%BE%E4%B9%8BBannerPager/gif.gif)

### BannerPager使用流程

**1. 布局使用BannerPager**
```xml
    <com.joker.pager.BannerPager
        android:id="@+id/banner_pager0"
        android:layout_width="match_parent"
        android:layout_height="180dp"
        android:layout_marginBottom="20dp"
        android:layout_marginTop="20dp" />
```

**2. 代码配置BannerPager**
```java
    bannerPager0 = findViewById(R.id.banner_pager0);

    final List<String> data = new ArrayList<>();

    data.add("http://7xi8d6.com1.z0.glb.clouddn.com/20180109085038_4A7atU_rakukoo_9_1_2018_8_50_25_276.jpeg");
    data.add("http://7xi8d6.com1.z0.glb.clouddn.com/20180102083655_3t4ytm_Screenshot.jpeg");
    data.add("http://7xi8d6.com1.z0.glb.clouddn.com/20171228085004_5yEHju_Screenshot.jpeg");

    //设置PagerOptions
    final PagerOptions pagerOptions0 = new PagerOptions.Builder(this)
            .setTurnDuration(2000)
            .setIndicatorSize(20)
            .setIndicatorVisibility(View.GONE)
            .setLoopEnable(false)
            .setPageTransformer(new ScaleTransformer())
            .setIndicatorMarginBottom(40)
            .build();

    //设置BannerPager
    bannerPager0
            .setPagerOptions(pagerOptions0)
            .setPages(data, new ViewHolderCreator<BannerPagerHolder>() {
                @Override
                public BannerPagerHolder createViewHolder() {
                    final View view = LayoutInflater.from(ScaleBannerActivity.this).inflate(R.layout.item_image_banner, null);
                    return new BannerPagerHolder(view);
                }
            });
```

**3. 适合地方开启停止/轮播**
```
    @Override
    protected void onResume() {
        super.onResume();
        bannerPager0.startTurning();
    }

    @Override
    protected void onStop() {
        super.onStop();
        bannerPager0.stopTurning();
    }
```

### BannerPager 常用API

- **[BannerPager](https://jitpack.io/com/github/joker-fu/BannerPager/0.0.8/javadoc/com/joker/pager/BannerPager.html)**
    - startTurning 开始轮播
    - stopTurning() 停止轮播
    - setPages(java.util.List<T> data, ViewHolderCreator creator) 设置 page data
    - setPagerOptions(PagerOptions options) 设置 PagerOptions
    - setOnPageChangeListener(OnPageChangeListener listener) 设置 page 改变监听
    - setOnItemClickListener(OnItemClickListener listener) 设置 Item点击监听

- **[PagerOptions.Builder](https://jitpack.io/com/github/joker-fu/BannerPager/0.0.8/javadoc/com/joker/pager/PagerOptions.Builder.html)**
    - 	openDebug(boolean debug) 开启 debug
    - 	setIndicatorAlign(int align) 设置指示器位置
    -   setIndicatorColor(int unSelected, int selected) 设置指示器
    -   setIndicatorDrawable(int unSelected, int selected) 设置指示器
    -   setIndicatorDistance(int distance) 设置指示器间距
    -   setIndicatorMarginBottom(int marginBottom) 设置指示器距离底部间距
    -   setIndicatorSize(int size) 设置指示器
    -   setIndicatorVisibility(int visibility) 设置Indicator 是否可见
    -   setLoopEnable(boolean loop) 设置可否循环
    -   setPagePadding(int px) 设置每个 page 之间间隔
    -   setPageTransformer(PageTransformer transformer) 设置轮播切换效果
    -   setPrePagerWidth(int px) 左右两侧预显示宽度
    -   setScrollDuration(int duration) 设置ViewPager的滚动速度
    -   setTurnDuration(int duration) 设置切换时间

### BannerPager 切换效果
- AccordionTransformer 手风琴效果
- CubeOutTransformer 立方体效果
- DepthCardTransformer 卡片效果
- ScaleTransformer 缩放效果