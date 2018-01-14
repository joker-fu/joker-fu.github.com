---
title: Android ListView ScrollView回弹效果
copyright: true
date: 2018-01-02 22:33:40
categories: Android
tags: Android
---

转自：http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0312/1590.html

ios中对可以滚动的视图都在系统层面上实现了触碰到边缘的阻尼回弹效果，用户一看便知自己的操作已经到了边界。android中也有类似的方案，不过当到达边界的时候不是用阻尼的方式，而是逐渐显示一个渐变颜色。ios的那种体验无疑会友好很多，也许是当初ios吵着要把这个设计申请专利的缘故吧，android不得不放弃橡皮筋效果，至少在系统层面。但这不意味着安卓中无法实现和ios一样的效果，这里介绍两种实现的方法。

第一种简单，但是效果不如意，而且必须要在api level大于9的情况下使用：

比如我是要给listview加上阻尼回弹效果，那么只需如下重写listview：
```
    package com.thinkfeed.bouncelistview;
    import android.content.Context;
    import android.util.AttributeSet;
    import android.util.DisplayMetrics;
    import android.widget.ListView;
    public class BounceListView extends ListView{
        private static final int MAX_Y_OVERSCROLL_DISTANCE = 200;
        private static final float SCROLL_RATIO = 0.5f;// 阻尼系数  
        private Context mContext;
        private int mMaxYOverscrollDistance;
                                                                                                                                                                                                                                              
        public BounceListView(Context context){
            super(context);
            mContext = context;
            initBounceListView();
        }
                                                                                                                                                                                                                                              
        public BounceListView(Context context, AttributeSet attrs){
            super(context, attrs);
            mContext = context;
            initBounceListView();
        }
                                                                                                                                                                                                                                              
        public BounceListView(Context context, AttributeSet attrs, int defStyle){
            super(context, attrs, defStyle);
            mContext = context;
            initBounceListView();
        }
                                                                                                                                                                                                                                              
        private void initBounceListView(){
            //get the density of the screen and do some maths with it on the max overscroll distance
            //variable so that you get similar behaviors no matter what the screen size
                                                                                                                                                                                                                                                  
            final DisplayMetrics metrics = mContext.getResources().getDisplayMetrics();
                final float density = metrics.density;
                                                                                                                                                                                                                                                  
            mMaxYOverscrollDistance = (int) (density * MAX_Y_OVERSCROLL_DISTANCE);
        }
                                                                                                                                                                                                                                              
        @Override
        protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent){
            //This is where the magic happens, we have replaced the incoming maxOverScrollY with our own custom variable mMaxYOverscrollDistance;
                                                                                                                                                                                                                                                  
            int newDeltaY = deltaY;
            int delta = (int) (deltaY * SCROLL_RATIO);
            if (delta != 0) newDeltaY = delta;
            return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, maxOverScrollX, mMaxYOverscrollDistance, isTouchEvent);
        }
                                                                                                                                                                                                                                              
    }
```
其中deltaX,deltaY为本次滑动偏移，scrollX，scrollY为当前总偏移，为了达到阻尼效果，我们增加了阻尼系数SCROLL_RATIO。
但是这个方法受限于api 而且效果并不是很好。
在github上看到有人完全重写了ScrollView将橡皮筋效果完美的移植在其中。
项目地址：https://github.com/EverythingMe/OverScrollView (**注**： 此项目已不再维护 作者维护新项目地址：https://github.com/EverythingMe/overscroll-decor)
用法和ScrollView一模一样，因为其实就是将系统的ScrollView修改而来。
```
<me.everything.android.widget.OverScrollView
 android:id="@+id/OverScroller"
 android:layout_width="match_parent"
 android:layout_height="match_parent" >
 <.../>
</me.everything.android.widget.OverScrollView>
```
这个OverScrollView需要稍作修改才能用，不然在第一次加载的时候显示不出来。
将原来的onLayout方法替换成下面的代码：
```
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b)
{
 super.onLayout(changed, l, t, r, b);
 mIsLayoutDirty = false;
 // Give a child focus if it needs it
 if (mChildToScrollTo != null && isViewDescendantOf(mChildToScrollTo, this))
 {
 scrollToChild(mChildToScrollTo);
 }
 mChildToScrollTo = null;
 // Calling this with the present values causes it to re-clam them
 scrollTo(getScrollX(), getScrollY());
 post(new Runnable()
 {
 public void run()
 {
 scrollTo(0, child.getPaddingTop());
 }
 }); 
}
```