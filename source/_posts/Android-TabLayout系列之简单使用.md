---
title: Android TabLayout系列之简单使用
copyright: true
date: 2018-01-02 22:43:00
categories: [Android, TabLayout]
tags: [Android, TabLayout]
---
#### 1 前言
在上一篇 [Android TabLayout系列之属性](http://www.abolg.com/2018/01/02/Android-TabLayout系列之属性) 中我们介绍了TabLayout的属性，同时也给出了一些简单的效果图。但是没有具体到它的使用，今天就来看看TabLayout的简单使用。不知道大家留意到我们仿网易的效果布局中，我们明明是写的小写字母，字母就变成大写了，还有字体大小能改变否？我们一步一步来解决这些问题......
#### 2 使用
介绍一种在实际开发中TabLayout较为常用的方式，那就是和ViewPager配合使用，实现联动。首先看看xml布局：
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.design.widget.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabBackground="@android:color/white"
        app:tabIndicatorColor="@android:color/holo_red_light"
        app:tabIndicatorHeight="2dp"
        app:tabMode="scrollable"
        app:tabSelectedTextColor="@android:color/holo_red_light"
        app:tabTextColor="@android:color/darker_gray"/>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
是不是很简单的布局，就只有一个TabLayout和ViewPager垂直排列，其实这个还不是官方的布局样式，官方的是这样的：
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

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
            app:tabTextColor="@android:color/darker_gray"/>

    <android.support.v4.view.ViewPager/>
</LinearLayout>
```
第一种是我们常见到的，第二种是官方的，两种的区别是官方这种写法不用调用setupWithViewPager方法。接下来我们看看Activity的代码怎么实现的：
```
public class MainActivity extends BaseActivity {

    private TabLayout mTabLayout;
    private ViewPager mViewPager;
    private MainPagerAdapter mAdapter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        AppStatusTracker.init(getApplication());
        super.onCreate(savedInstanceState);
    }

    @Override
    protected void initContentView() {
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void initView() {
        mTabLayout = findView(tabLayout);
        mViewPager = findView(R.id.viewPager);
        for (int i = 0; i < 11; i++) {
            //为TabLayout添加10个tab并设置上文本
            mTabLayout.addTab(mTabLayout.newTab().setText("Tab " + i));
        }

        mAdapter = new MainPagerAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(mAdapter);
        //官方推荐的绑定ViewPager方式
        mTabLayout.setupWithViewPager(mViewPager);
    }

    @Override
    protected void initData(@Nullable Bundle savedInstanceState) {

    }

    //ViewPager适配器  10个Fragment
    private class MainPagerAdapter extends FragmentPagerAdapter {
        public MainPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            return BlankFragment.newInstance(position);
        }

        @Override
        public int getCount() {
            return 10;
        }
    }
}
```
接下来是BlankFragment的实现：
```
public class BlankFragment extends Fragment {
    private int index;

    public BlankFragment() {
        // Required empty public constructor
    }

    public static Fragment newInstance(int position) {
        BlankFragment fragment = new BlankFragment();
        fragment.index = position;
        return fragment;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_blank, container, false);
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        ((TextView) view.findViewById(R.id.textView)).setText("this Tab " + index);
    }
}
```
这里需要先解释newTab，这个newTab是使用TabLayout中默认的Tab实现。从它的实现可以看到它支持设置图标，标题和内容描述（图标描述）。
```
    public static final class Tab {

        /**
         * An invalid position for a tab.
         *
         * @see #getPosition()
         */
        public static final int INVALID_POSITION = -1;

        private Object mTag;
        private Drawable mIcon;
        private CharSequence mText;
        private CharSequence mContentDesc;
        private int mPosition = INVALID_POSITION;
        private View mCustomView;

        TabLayout mParent;
        TabView mView;
  .....
    }
```
Tab默认图标在标题的上方，还有xml里面设置TabItem的话，最终也是设置到Tab上。关于Tab就说明这些，具体的大家可以自行测试。
可以看到关于TabLayout的实现也很简单，就是给布局里面的TextView设置了文本显示当前是第几个Tab。布局就更简单了，FrameLayout中嵌套了一个TextView，就不单独给出了。接下来我们就可以来瞅一瞅实际的使用效果了。
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/2018-01-02-001.gif)

#### 3 “填坑”与源码解析
- 源码解析
看到上面的效果图是不是觉得很怪异？没错，我们所设置的Tab上的文字没了，只有下面的指示条。相信大家一通搜索后，相信大家都知道了，最常见的就是分析源码后得出结论，被remove了！！！接下来我们也来装下文化人，看看怎么回事？
首先是TabLayout的setupWithViewPager方法，一顿单击后会到达如下源码位置：
```
private void setupWithViewPager(@Nullable final ViewPager viewPager, boolean autoRefresh,
            boolean implicitSetup) {
        if (mViewPager != null) {
            // If we've already been setup with a ViewPager, remove us from it
            if (mPageChangeListener != null) {
                mViewPager.removeOnPageChangeListener(mPageChangeListener);
            }
            if (mAdapterChangeListener != null) {
                mViewPager.removeOnAdapterChangeListener(mAdapterChangeListener);
            }
        }
......
......
            if (adapter != null) {
                // Now we'll populate ourselves from the pager adapter, adding an observer if
                // autoRefresh is enabled
                setPagerAdapter(adapter, autoRefresh);
            }
......
......

            // Now update the scroll position to match the ViewPager's current item
            setScrollPosition(viewPager.getCurrentItem(), 0f, true);
        } else {
            // We've been given a null ViewPager so we need to clear out the internal state,
            // listeners and observers
            mViewPager = null;
            setPagerAdapter(null, false);
        }

        mSetupViewPagerImplicitly = implicitSetup;
    }
```
这里没什么说的就是设置判断移除、设置监听等操作，我们最主要去找到网上最多叙述问题所在的源码位置，接下来我们看setPagerAdapter：
```
    void setPagerAdapter(@Nullable final PagerAdapter adapter, final boolean addObserver) {
        if (mPagerAdapter != null && mPagerAdapterObserver != null) {
            // If we already have a PagerAdapter, unregister our observer
            mPagerAdapter.unregisterDataSetObserver(mPagerAdapterObserver);
        }

        mPagerAdapter = adapter;

        if (addObserver && adapter != null) {
            // Register our observer on the new adapter
            if (mPagerAdapterObserver == null) {
                mPagerAdapterObserver = new PagerAdapterObserver();
            }
            adapter.registerDataSetObserver(mPagerAdapterObserver);
        }

        // Finally make sure we reflect the new adapter
        populateFromPagerAdapter();
    }
```
这段代码主要是判断之前的mPagerAdapter是否为空，不为空就移除DataSetObserver监听，将新的adapter设置进来并注册上DataSetObserver监听。这里我们捕获方法populateFromPagerAdapter一枚，我们再看看它的实现：
```
    void populateFromPagerAdapter() {
        removeAllTabs();

        if (mPagerAdapter != null) {
            final int adapterCount = mPagerAdapter.getCount();
            for (int i = 0; i < adapterCount; i++) {
                addTab(newTab().setText(mPagerAdapter.getPageTitle(i)), false);
            }

            // Make sure we reflect the currently set ViewPager item
            ......
        }
    }
```
哇塞！removeAllTabs，大家说得最多的一个方法，观看方法名就很吓人了，remove all tabs....看看它到底干了什么：
```
    public void removeAllTabs() {
        // Remove all the views
        for (int i = mTabStrip.getChildCount() - 1; i >= 0; i--) {
            removeTabViewAt(i);
        }

        for (final Iterator<Tab> i = mTabs.iterator(); i.hasNext();) {
            final Tab tab = i.next();
            i.remove();
            tab.reset();
            sTabPool.release(tab);
        }

        mSelectedTab = null;
    }
```
它果真和它的命名一样，遍历remove了所有的tab，并且将当前选中也置为null。怎么办？难道Google故意留下这么个坑？我们继续看上面的populateFromPagerAdapter，会发现removeAllTabs后，它会判断adapter是否为空，不为空就调用了addTab(newTab().setText(mPagerAdapter.getPageTitle(i)), false)添加上了新的tab。

- "填坑"思路
1. 从上面的分析，我们发现如果要有tab还得去重写adapter的getPageTitle方法。再看一段官方文档中的说明：
>If you're using a [ViewPager]() together with this layout, you can call [setupWithViewPager(ViewPager)]()
 to link the two together. This layout will be automatically populated from the [PagerAdapter]()
's page titles.

就是说官方推荐我们使用setupWithViewPager(ViewPager)来关联Tablayout和Viewpager，且TabLayout会自动填充PagerAdapter的Title。也就是说它会自动创建tab，并绑定为adapter的page
 title。这下我们来改造下Activity：
```
public class MainActivity extends BaseActivity {

    private TabLayout mTabLayout;
    private ViewPager mViewPager;
    private MainPagerAdapter mAdapter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        AppStatusTracker.init(getApplication());
        super.onCreate(savedInstanceState);
    }

    @Override
    protected void initContentView() {
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void initView() {
        mTabLayout = findView(tabLayout);
        mViewPager = findView(R.id.viewPager);
        mAdapter = new MainPagerAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(mAdapter);
        //官方推荐的绑定ViewPager方式
        mTabLayout.setupWithViewPager(mViewPager);
    }

    @Override
    protected void initData(@Nullable Bundle savedInstanceState) {

    }

    //ViewPager适配器  10个Fragment
    private class MainPagerAdapter extends FragmentPagerAdapter {
        public MainPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            return BlankFragment.newInstance(position);
        }

        //TabLayout会根据当前page的title自动绑定tab
        @Override
        public CharSequence getPageTitle(int position) {
            return "Tab " + position;
        }

        @Override
        public int getCount() {
            return 10;
        }
    }
}
```
这就是新的Activity实现，改动很小。我们首先删除了initView中关于tab的初始化操作，然后重写了FragmentPagerAdapter的getPageTitle方法。
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/2018-01-02-002.gif)

2. 我们既然知道他会自动为我们绑定tab，那么我们可以利用它自动帮我绑定tab，而不去重写getPageTitle方法，在绑定后去设置关于tab的显示。
```
public class MainActivity extends BaseActivity {

    private TabLayout mTabLayout;
    private ViewPager mViewPager;
    private MainPagerAdapter mAdapter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        AppStatusTracker.init(getApplication());
        super.onCreate(savedInstanceState);
    }

    @Override
    protected void initContentView() {
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void initView() {
        mTabLayout = findView(tabLayout);
        mViewPager = findView(R.id.viewPager);
        mAdapter = new MainPagerAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(mAdapter);
        //官方推荐的绑定ViewPager方式
        mTabLayout.setupWithViewPager(mViewPager);
        int tabCount = mTabLayout.getTabCount();
        for (int i = 0; i < tabCount; i++) {
            //这里tab可能为null 根据实际情况处理吧
            mTabLayout.getTabAt(i).setText("Tab" + i);
        }
    }

    @Override
    protected void initData(@Nullable Bundle savedInstanceState) {

    }

    //ViewPager适配器  10个Fragment
    private class MainPagerAdapter extends FragmentPagerAdapter {
        public MainPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            return BlankFragment.newInstance(position);
        }

        @Override
        public int getCount() {
            return 10;
        }
    }
}
```
可以看到只是将initView中初始化Tab的位置调整到setupWithViewPager的后面，设置上我们需要的标题。这里效果和上面一样就不给出来占篇幅了。

3. 上面我们依旧利用了setupWithViewPager自动为我们绑定tab的实现，在分析源码的时候发现它会调用ViewPager的addOnPageChangeListener和TabLayout的addOnTabSelectedListener，实现TabLayout与ViewPager的关联。那么我们就自己来实现关联，不去使用官方推荐的setupWithViewPager方法绑定。
```
public class MainActivity extends BaseActivity {

    private TabLayout mTabLayout;
    private ViewPager mViewPager;
    private MainPagerAdapter mAdapter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        AppStatusTracker.init(getApplication());
        super.onCreate(savedInstanceState);
    }

    @Override
    protected void initContentView() {
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void initView() {
        mTabLayout = findView(tabLayout);
        mViewPager = findView(viewPager);
        mAdapter = new MainPagerAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(mAdapter);
        //初始化tab
        for (int i = 0; i < 10; i++) {
            mTabLayout.addTab(mTabLayout.newTab().setText("item" + i));
        }
        //自己实现关联
        mViewPager.addOnPageChangeListener(new TabLayout.TabLayoutOnPageChangeListener(mTabLayout));
        mTabLayout.addOnTabSelectedListener(new TabLayout.ViewPagerOnTabSelectedListener(mViewPager));
    }

    @Override
    protected void initData(@Nullable Bundle savedInstanceState) {

    }

    //ViewPager适配器  10个Fragment
    private class MainPagerAdapter extends FragmentPagerAdapter {
        public MainPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            return BlankFragment.newInstance(position);
        }

        @Override
        public int getCount() {
            return 10;
        }
    }
}
```
这段代码，在initView中移除了官方推荐的设置相关代码，自己初始化了tab并实现了关联。注意这里我们改变了Tab的标题作为区分。效果也和上面一样，只是改变了Tab的标题。

#### 4 其他问题
这样TabLayout的简单使用就介绍完了，这里还有两个问题一个是TabLayout的标题怎么就是大写了，还有它的文字大小怎么改呢？还记得我怎么在上一篇[Android TabLayout系列之属性](http://www.abolg.com/2018/01/02/Android-TabLayout系列之属性)中介绍的tabTextAppearance属性么？我们就利用它来改变标题字母大小写和文字大小问题。先开看看TabLayout源码中的默认使用：
```
      mTabTextAppearance = a.getResourceId(R.styleable.TabLayout_tabTextAppearance,
      R.style.TextAppearance_Design_Tab);


      <style name="TextAppearance.Design.Tab" parent="TextAppearance.AppCompat.Button">
          <item name="android:textSize">@dimen/design_tab_text_size</item>
          <item name="android:textColor">?android:textColorSecondary</item>
          <item name="textAllCaps">true</item>
      </style>
```

我们为Tablayout添加上tabTextAppearance属性：
```
      app:tabTextAppearance="@style/TabLayoutStyle"
```
这里需要把具体的属性定义到stytle中，这里有两种方式：
```
    <style name="TabLayoutStyle" parent="TextAppearance.Design.Tab">
        <item name="android:textSize">16sp</item>
        <item name="textAllCaps">false</item>
    </style>
```
```
      <style name="TabLayoutStyle">
          <item name="android:textSize">16sp</item>
         //<item name="textAllCaps">false</item>
          <item name="android:textAllCaps">false</item>
      </style>
```
需要注意 parent="TextAppearance.Design.Tab"时，textAllCaps没有android，当不继承的时候有没有都可以。其实这里的字母大小写，不能算个坑。我们可以看material设计中所有tab的字母都是大写，估计歪果仁标题都习惯大写吧，或者是为了符合material设计，所以默认的Tab被设置成了大写。最后运行效果如下：
![](http://114.67.156.211/bolg/android/Android%20TabLayout%E7%B3%BB%E5%88%97%E4%B9%8B%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/2018-01-02-003.gif)

这里附上我的build.gradle，有可能不同版本会有不一样的效果，所以大家实际使用的时候需要注意一下：
```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"

    defaultConfig {
        applicationId "com.joker.demo"
        minSdkVersion 19
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```
TabLayou的简单使用就介绍这么多，其实真的是简单使用，主要是中间插入了一段源码分析，所以看起来多了点。
**附：**
1. [Android TabLayout系列之属性](http://www.abolg.com/2018/01/02/Android-TabLayout系列之属性)
2. [Android TabLayout系列之简单使用](http://www.abolg.com/2018/01/02/Android-TabLayout系列之简单使用)
3. [Android TabLayout系列之进阶使用](http://www.abolg.com/2018/01/02/Android-TabLayout系列之进阶使用)