---
title: PorterDuffXferMode实战之WaveProgressBar 圆形水波纹进度
copyright: true
date: 2018-01-05 15:48:38
categories: [Android,自定义控件]
tags: [Android,自定义控件]
---
#### 一 写在前面的话
前几天写了一篇[Android 绘图之PorterDuffXferMode实例讲解与源码解析](http://www.abolg.com/2018/01/05/Android-绘图之PorterDuffXferMode实例讲解与源码解析)，没看过的可以先进去看看。今天我们就来使用PorterDuffXferMode的DST_IN模式，本来想写个圆形头像来看看效果就行了，但是恰巧群里有朋友问到水波纹的进度效果。于是乎去看了下实现方式正好也可以用到PorterDuffXferMode，立马决定还是做个个效果更炫。先看看下面效果图，就这么一张动图，懒癌犯了懒得做中间效果动图。

![效果图.gif](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-001.gif)

#### 二 效果分析
看了上面效果图，闲来分析下怎么做出这个效果。
1. 静态波浪效果（通过Canvas的drawPath方法画二阶贝塞尔曲线实现）
2. 动态波浪效果（改变波浪高度和X方向绘制位置后定时invalidate）
3. 圆形效果（通过PorterDuffXferMode的DST_IN和动态波浪效果组合）
4. 进度文字（最终效果上绘制出文字）

#### 三 波浪效果
- __静态波浪效果实现__
首先我们通过Canvas的drawPath方法画二阶贝塞尔曲线的方法，来绘制静态波浪效果。我们先看效果，然后看代码。代码一般都注视得比较详细。避免单独再解释代码。来上静态波浪效果图：
![静态波浪效果1.png](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-002.png)

  先上一段xml，然后是代码。原则上不改xml，就不再上了：
```
    <com.joker.widget.test.WaveProgressBar
        android:id="@+id/wave"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:layout_gravity="center_horizontal"
        android:padding="10dp"/>
```
```
//静态波浪效果1
public class WaveProgressBar extends View {
    private Paint mWavePaint = new Paint(); //水波画笔
    private Path mPath = new Path();  //路径
    private int mWidth;  //控件宽
    private int mHeight; //控件高
    private int mWaveWidth = 200; //水波宽
    private int mWaveHeight = 100; //水波高
    int currentY = 200; //当前Y值

    public WaveProgressBar(Context context) {
        this(context, null);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mWavePaint.setAntiAlias(true);
        mWavePaint.setColor(Color.GREEN);
    }

    //获取控件的宽和高
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0) {
            mWidth = w;
        }
        if (h > 0) {
            mHeight = h;
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mWidth <= 0 || mHeight <= 0) {
            return;
        }
        //绘制一个灰色背景 方便看效果
        canvas.drawColor(Color.GRAY);
        mPath.reset();
        //路径起点
        mPath.moveTo(0, 200);
        //i+2: 一上一下2个半波为一组 count+2: 保证波浪占满控件
        for (int i = 0, count = mWidth / mWaveWidth + 2; i < count; i += 2) {
            //上半波
            mPath.quadTo(mWaveWidth * (i + 0.5f), currentY - mWaveHeight, mWaveWidth * (i + 1), currentY);
            //下半波
            mPath.quadTo(mWaveWidth * (i + 1.5f), currentY + mWaveHeight, mWaveWidth * (i + 2), currentY);
        }
        //绘制路径
        canvas.drawPath(mPath, mWavePaint);
    }
}
```
接下来我们让波浪封闭起来，并且让它水位升高，有总水杯里的水的感觉。还是和前面一样，看图看代码：

![封闭起来的静态波浪效果.png](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-003.png)

![水位升高的的静态波浪效果.png](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-004.png)

```
//封闭起来的静态波浪效果
public class WaveProgressBar extends View {
    private Paint mWavePaint = new Paint();
    private Path mPath = new Path();
    private int mWidth;
    private int mHeight;
    private int mWaveWidth = 200; //水波宽
    private int mWaveHeight = 100; //水波高
    int currentY = 200; //当前Y值

    public WaveProgressBar(Context context) {
        this(context, null);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mWavePaint.setAntiAlias(true);
        mWavePaint.setColor(Color.GREEN);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0) {
            mWidth = w;
        }
        if (h > 0) {
            mHeight = h;
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mWidth <= 0 || mHeight <= 0) {
            return;
        }
        canvas.drawColor(Color.GRAY);
        mPath.reset();
        //路径起点
        mPath.moveTo(0, 200);
        //i+2: 一上一下2个半波为一组 count+2: 保证波浪占满控件
        for (int i = 0, count = mWidth / mWaveWidth + 2; i < count; i += 2) {
            //上半波
            mPath.quadTo(mWaveWidth * (i + 0.5f), currentY - mWaveHeight, mWaveWidth * (i + 1), currentY);
            //下半波
            mPath.quadTo(mWaveWidth * (i + 1.5f), currentY + mWaveHeight, mWaveWidth * (i + 2), currentY);
        }
        //一下两句代码就完成了水波的封闭效果
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
        mPath.close();
        canvas.drawPath(mPath, mWavePaint);
    }
}
```
还是不得不啰嗦一下，我们这里添加了2行代码实现了水波的封闭：
```
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
```
接下来就是水位的上升了，但是这个地方就上代码就有点故意拉长篇幅的嫌疑了。其实很简单我们只需要改变currentY的值，就可以实现水位的上升了。
这里关于Canvas路径绘制，或者是坐标系搞不明白的可以看我的：[Android 绘图之Canvas相关API使用](http://www.abolg.com/2018/01/05/Android-绘图之Canvas相关API使用)一文，在里面有比较详细的说明。
2. 动态波浪效果实现
上面我们已经实现了静态的波浪效果了，代码还是比较简单的，接下来我们让波浪动次打次动次打次动起来。
![动态波浪效果.gif](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-005.gif)

```
  //动态波浪效果
  public class WaveProgressBar extends View {
    private Paint mWavePaint = new Paint();
    private Path mPath = new Path();
    private int mWidth;
    private int mHeight;
    private int mWaveWidth = 200; //水波宽
    private int mWaveHeight = 100; //水波高
    int currentY = 200; //当前Y值
    private double mRate = 0.1; //流速
    private int distance;   //距离
    private Handler mHandler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            distance += mWaveWidth * mRate; //计算流动总距离
            distance = distance % (mWaveWidth << 1);  //根据总距离算出当前移动距离
            invalidate(); //刷新重绘
            mHandler.sendEmptyMessageDelayed(0, 20); //20ms发送一个消息
            return false;
        }
    });

    public WaveProgressBar(Context context) {
        this(context, null);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mWavePaint.setAntiAlias(true);
        mWavePaint.setColor(Color.GREEN);
        //20ms发送一个消息
        mHandler.sendEmptyMessageDelayed(0, 20);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0) {
            mWidth = w;
        }
        if (h > 0) {
            mHeight = h;
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mWidth <= 0 || mHeight <= 0) {
            return;
        }
        canvas.drawColor(Color.GRAY);
        mPath.reset();
        //路径起点
        mPath.moveTo(0, 200);
        //i+2: 一上一下2个半波为一组 count+2: 保证波浪占满控件
        for (int i = 0, count = mWidth / mWaveWidth + 2; i < count; i += 2) {
            //上半波  减去distance 造成左移效果 
            mPath.quadTo(mWaveWidth * (i + 0.5f) - distance, currentY - mWaveHeight, mWaveWidth * (i + 1) - distance, currentY);
            //下半波
            mPath.quadTo(mWaveWidth * (i + 1.5f) - distance, currentY + mWaveHeight, mWaveWidth * (i + 2) - distance, currentY);
        }
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
        mPath.close();
        canvas.drawPath(mPath, mWavePaint);
    }
}
```
看效果图的水流还是很急的，减小速率就可以让它流慢点，我们先不关心这个，说说实现思路。主要是利用Handler定时发送消息通知重绘的方式，每20ms计算出移动距离并刷新重绘一次。在绘制的时候减去移动距离就有了左移效果，其他的说明在代码中注释很详尽了。

#### 四 圆形效果
水波效果我们就做到这里，接下来就是利用PorterDuffXferMode的DST_IN模式来实现圆形效果，我们先绘制的水波后绘制的圆，所以我们需要使用DST_IN。其实绘制圆形头像效果原理也是一样的，至于用什么模式就跟你绘制的先后顺序，还有需求有关。不了解PorterDuffXferMode可以去看看我之前的文章，本文后我会给出相关链接地址。
![圆形效果.gif](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-006.gif)

```
  //圆形波浪效果
  public class WaveProgressBar extends View {
    private Paint mWavePaint = new Paint();
    private Path mPath = new Path();
    private int mWidth;
    private int mHeight;
    private int mWaveWidth = 200; //水波宽
    private int mWaveHeight = 100; //水波高
    int currentY = 200; //当前Y值
    private double mRate = 0.01; //流速
    private int distance;   //距离
    private PorterDuffXfermode mXfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
    private Bitmap mCircleBitmap;   //圆形bitmap
    private RectF mBorderRectF; //边框矩形
    private int mBorderRadius; //边框半径
    private Handler mHandler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            distance += mWaveWidth * mRate; //计算流动总距离
            distance = distance % (mWaveWidth << 1);
            invalidate();
            mHandler.sendEmptyMessageDelayed(0, 20);
            return false;
        }
    });

    public WaveProgressBar(Context context) {
        this(context, null);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mWavePaint.setAntiAlias(true);
        mWavePaint.setColor(Color.GREEN);
        mHandler.sendEmptyMessageDelayed(0, 20);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0) {
            mWidth = w;
        }
        if (h > 0) {
            mHeight = h;
        }
        mWidth = mHeight = Math.min(mWidth, mHeight);
        //创建边框矩形
        mBorderRectF = new RectF(0, 0, mWidth, mHeight);
        //边框半径
        mBorderRadius = (mWidth >> 1);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mWidth <= 0 || mHeight <= 0) {
            return;
        }
        canvas.drawBitmap(createWaveBitmap(), 0, 0, mWavePaint);
    }

    //创建水波bitmap
    private Bitmap createWaveBitmap() {
        Bitmap bitmap = Bitmap.createBitmap(mWidth, mHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawColor(Color.GRAY);
        mPath.reset();
        //路径起点
        mPath.moveTo(0, 200);
        //i+2: 一上一下2个半波为一组 count+2: 保证波浪占满控件
        for (int i = 0, count = mWidth / mWaveWidth + 2; i < count; i += 2) {
            //上半波  减去distance 造成左移效果
            mPath.quadTo(mWaveWidth * (i + 0.5f) - distance, currentY - mWaveHeight, mWaveWidth * (i + 1) - distance, currentY);
            //下半波
            mPath.quadTo(mWaveWidth * (i + 1.5f) - distance, currentY + mWaveHeight, mWaveWidth * (i + 2) - distance, currentY);
        }
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
        mPath.close();
        canvas.drawPath(mPath, mWavePaint);
        if (mCircleBitmap == null) {
            //创建圆形的bitmap
            mCircleBitmap = createShapeBitmap();
        }
        //设置mXfermode模式
        mWavePaint.setXfermode(mXfermode);
        canvas.drawBitmap(mCircleBitmap, 0, 0, mWavePaint);
        mWavePaint.setXfermode(null);
        return bitmap;
    }

    //创建形状的bitmap
    private Bitmap createShapeBitmap() {
        Bitmap bitmap = Bitmap.createBitmap(mWidth, mHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawCircle(mBorderRectF.centerX(), mBorderRectF.centerY(), mBorderRadius, mWavePaint);
        return bitmap;
    }
}
```
这里把onDraw里面的代码抽取出来了，把onDraw的背景绘制放在了createWaveBitmap中便于观看效果，利用PorterDuffXferMode的DST_IN模式来实现圆形效果。

#### 五 绘制文字
我们已经完成了圆形的水波纹效果，接下来就是将文字绘制在圆形的水波里面。绘制文字就比较简单了，调用Canvas的drawText方法在圆形水波纹上将文字绘制出来就O啦。
![添加文字后的水波纹效果.gif](http://114.67.156.211/bolg/android/PorterDuffXferMode%E5%AE%9E%E6%88%98%E4%B9%8BWaveProgressBar%20%E5%9C%86%E5%BD%A2%E6%B0%B4%E6%B3%A2%E7%BA%B9%E8%BF%9B%E5%BA%A6/2018-01-05-007.gif)
```
public class WaveProgressBar extends View {
    private Paint mWavePaint = new Paint();
    private Paint mTextPaint = new Paint();
    private Path mPath = new Path();
    private int mWidth;
    private int mHeight;
    private int mWaveWidth = 200; //水波宽
    private int mWaveHeight = 100; //水波高
    int currentY = 200; //当前Y值
    private double mRate = 0.1; //流速
    private int distance;   //距离
    private PorterDuffXfermode mXfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
    private Bitmap mCircleBitmap;   //圆形bitmap
    private RectF mBorderRectF; //边框矩形
    private int mBorderRadius; //边框半径
    private Handler mHandler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            distance += mWaveWidth * mRate; //计算流动总距离
            distance = distance % (mWaveWidth << 1);
            invalidate();
            mHandler.sendEmptyMessageDelayed(0, 20);
            return false;
        }
    });

    public WaveProgressBar(Context context) {
        this(context, null);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mWavePaint.setAntiAlias(true);
        mWavePaint.setColor(Color.GREEN);

        mTextPaint.setAntiAlias(true);
        mTextPaint.setColor(Color.WHITE);
        mTextPaint.setTextSize(32);
        mHandler.sendEmptyMessageDelayed(0, 20);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0) {
            mWidth = w;
        }
        if (h > 0) {
            mHeight = h;
        }
        mWidth = mHeight = Math.min(mWidth, mHeight);
        //创建边框矩形
        mBorderRectF = new RectF(0, 0, mWidth, mHeight);
        //边框半径
        mBorderRadius = (mWidth >> 1);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mWidth <= 0 || mHeight <= 0) {
            return;
        }
        canvas.drawBitmap(createWaveBitmap(), 0, 0, mWavePaint);
        canvas.drawText("80%", mBorderRectF.centerX(), mBorderRectF.centerY(), mTextPaint);
    }

    //创建水波bitmap
    private Bitmap createWaveBitmap() {
        Bitmap bitmap = Bitmap.createBitmap(mWidth, mHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawColor(Color.GRAY);
        mPath.reset();
        //路径起点
        mPath.moveTo(0, 200);
        //i+2: 一上一下2个半波为一组 count+2: 保证波浪占满控件
        for (int i = 0, count = mWidth / mWaveWidth + 2; i < count; i += 2) {
            //上半波  减去distance 造成左移效果
            mPath.quadTo(mWaveWidth * (i + 0.5f) - distance, currentY - mWaveHeight, mWaveWidth * (i + 1) - distance, currentY);
            //下半波
            mPath.quadTo(mWaveWidth * (i + 1.5f) - distance, currentY + mWaveHeight, mWaveWidth * (i + 2) - distance, currentY);
        }
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
        mPath.close();
        canvas.drawPath(mPath, mWavePaint);
        if (mCircleBitmap == null) {
            //创建圆形的bitmap
            mCircleBitmap = createShapeBitmap();
        }
        //设置mXfermode模式
        mWavePaint.setXfermode(mXfermode);
        canvas.drawBitmap(mCircleBitmap, 0, 0, mWavePaint);
        mWavePaint.setXfermode(null);
        return bitmap;
    }

    //创建形状的bitmap
    private Bitmap createShapeBitmap() {
        Bitmap bitmap = Bitmap.createBitmap(mWidth, mHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawCircle(mBorderRectF.centerX(), mBorderRectF.centerY(), mBorderRadius, mWavePaint);
        return bitmap;
    }
}
```
这一步没什么可说的，新创建了mTextPaint 画笔，设置了字体颜色大小等属性，最后在原本画好的效果上再绘制上文字就成了。可我们文章开头的效果图上，文字是随着水位而上升的，其实这个很简单，改变文本绘制的Y轴位置就能达到上升效果，我们的水波纹效果到这里就完成了它的雏形。

上面的演示代码还是比较杂乱，创建Bitmap都还是在onDraw中，没有考虑控件的测量，也不支持自定义属性等。大家可以根据上面的步骤，自己整理实现定义成自己想要的效果。

最后我将源码整理了下，优化后的源码后续会补上地址，就不在文中给出了。

__附：__
[Android 绘图之Canvas相关API使用](http://www.abolg.com/2018/01/05/Android-绘图之Canvas相关API使用)
[Android 绘图之PorterDuffXferMode实例讲解与源码解析](http://www.abolg.com/2018/01/05/Android-绘图之PorterDuffXferMode实例讲解与源码解析)
