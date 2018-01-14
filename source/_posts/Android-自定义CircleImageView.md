---
title: Android 自定义CircleImageView
copyright: true
date: 2018-01-05 15:43:03
categories: [Android,自定义控件]
tags: [Android,自定义控件]
---
android开发中时常用到圆形、圆角的ImageView，网上也有很多现成的轮子，本不应该重复造轮子。但是本着不断学习和进步的决心，今天就来手撸一个圆形、圆角ImageView，且可设置边框颜色、大小。无图无真相，首先我们来看下效果图:

![我是效果图](http://114.67.156.211/bolg/android/Android%20%E8%87%AA%E5%AE%9A%E4%B9%89CircleImageView/2018-01-05-001.png)

看完效果图，不知道各位看官觉得如何，觉得好那就 just do it ! (就是干！)。

接下来 嗯 嗯 啊 啊 啊......

嗯 得先来个说明 

本文标题说的是自定义CircleImageView，但是我们实际干的却是圆形 还有圆角矩形。所以我们就不叫CircleImageView，且叫它ShapeImageView，虽然只有两种形状。觉得不合适的可随意修改嘛...

然后我们得知道自定义控件的四步曲：
1、自定义View的属性
2、在View的构造方法中获得我们自定义的属性
3、重写onMeasure
4、重写onDraw

接下来就正式开干......
#### 一 自定义属性
为了感觉高大上，我们的自定义控件还是配上几个自定义属性才像话嘛！！！总共定义了6个自定义属性，分别是边框宽、边框颜色、边框重叠、填充颜色、圆角半径、形状类型。
```
    <declare-styleable name="ShapeImageView">
        <attr name="siv_border_width" format="dimension"/>
        <attr name="siv_border_color" format="color"/>
        <attr name="siv_border_overlay" format="boolean"/>
        <attr name="siv_fill_color" format="color"/>
        <attr name="siv_round_radius" format="dimension"/>
        <attr name="siv_shape_type">
            <enum name="circle" value="0"/>
            <enum name="round" value="1"/>
        </attr>
    </declare-styleable>
```
#### 二 获取自定义属性
这里得分两步，这第一步就得是继承自我们的ImageView，并重写他的构造方法：
```
//这里我们让一个、两个参数的构造方法都去调用三个参数的构造方法
public class ShapeImageView extends ImageView {
    public ShapeImageView(Context context) {
        this(context, null); 
    }

    public ShapeImageView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ShapeImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```
没错，这第二步就是获取自定义属性了，因为我们都统一调用的三个参数的构造方法，所以直接在三个参数的构造方法中获取属性：
```
    public ShapeImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.ShapeImageView, defStyleAttr, 0);
        mBorderWidth = a.getDimensionPixelSize(R.styleable.ShapeImageView_siv_border_width, DEFAULT_BORDER_WIDTH);
        mBorderColor = a.getColor(R.styleable.ShapeImageView_siv_border_color, DEFAULT_BORDER_COLOR);
        mFillColor = a.getColor(R.styleable.ShapeImageView_siv_fill_color, DEFAULT_FILL_COLOR);
        mBorderOverlay = a.getBoolean(R.styleable.ShapeImageView_siv_border_overlay, DEFAULT_BORDER_OVERLAY);
        mShapeType = a.getInt(R.styleable.ShapeImageView_siv_shape_type, SHAPE_CIRCLE);
        mRoundRadius = a.getDimensionPixelSize(R.styleable.ShapeImageView_siv_round_radius, DEFAULT_ROUND_RADIUS);
        a.recycle();
    }
```
这里没什么好说的，就是把我们刚才的6个自定义属性值获取出来了，没有值的都赋上默认值。记得最后调用下recycle()，就行了。

#### 三 ShapeImageView之onMeasure
我们这里是继承自ImageView，所以测量的事情就让ImageView去祸祸，这一步我们就这么愉快的完成了。

#### 四 ShapeImageView之onDraw
这一步是最关键的一步了，也是我们的最后一步，完成它就可以看到最终的效果图了。接下来我们就需要把这一步细分成几小步来完成：
- 获取要绘制的Bitmap
```
    //从Drawable中取出Bitmap 
    private Bitmap getBitmapFormDrawable() {
        Drawable drawable = getDrawable();
        if (drawable == null) {
            return null;
        }
        //是BitmapDrawable直接取出Bitmap 
        if (drawable instanceof BitmapDrawable) {
            return ((BitmapDrawable) drawable).getBitmap();
        }
        try {
            //这里需要对DEFAULT_DRAWABLE_DIMENSION说明：在ColorDrawable中getIntrinsicWidth为-1，所以给了个默认值2
            int dWidth = drawable.getIntrinsicWidth() <= 0 ? DEFAULT_DRAWABLE_DIMENSION : drawable.getIntrinsicWidth();
            int dHeight = drawable.getIntrinsicHeight() <= 0 ? DEFAULT_DRAWABLE_DIMENSION : drawable.getIntrinsicHeight(); 
            //创建Bitmap画布，并绘制出来
            Bitmap bitmap = Bitmap.createBitmap(dWidth, dHeight, BITMAP_CONFIG);
            Canvas canvas = new Canvas(bitmap);
            drawable.setBounds(0, 0, canvas.getWidth(), canvas.getHeight());
            drawable.draw(canvas);
            return bitmap;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
```
- 更新设置
这里需要简单介绍下BitmapShader:
>BitmapShader，顾名思义，就是用Bitmap对绘制的图形进行渲染着色，其实就是用图片对图形进行贴图。 
BitmapShader构造函数如下所示：
BitmapShader(Bitmap bitmap, Shader.TileMode tileX, Shader.TileMode tileY)
第一个参数是Bitmap对象，该Bitmap决定了用什么图片对绘制的图形进行贴图。
第二个参数和第三个参数都是Shader.TileMode类型的枚举值，有以下三个取值：CLAMP(拉伸) 、REPEAT(重复) 和 MIRROR(镜像)。

```
    private void updateSetup() {
        if (mWidth == 0 && mHeight == 0) {
            return;
        }
        if (mBitmap == null) {
            super.invalidate();
            return;
        }
        //设置BitmapShader  使用拉伸模式
        BitmapShader bitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
        //设置BitmapPaint
        mBitmapPaint.setAntiAlias(true);
        mBitmapPaint.setShader(bitmapShader);
        //设置BorderPaint
        mBorderPaint.setStyle(Paint.Style.STROKE);
        mBorderPaint.setAntiAlias(true);
        mBorderPaint.setColor(mBorderColor);
        mBorderPaint.setStrokeWidth(mBorderWidth);
        //设置FillPaint
        mFillPaint.setStyle(Paint.Style.FILL);
        mFillPaint.setAntiAlias(true);
        mFillPaint.setColor(mFillColor);
        //将坐标从src复制到这个mContentRect矩形
        mContentRect.set(calculateRect());
        if (!mBorderOverlay && mBorderWidth > 0) {
            //允许边框重叠操作
            mContentRect.inset(mBorderWidth - 0.0f, mBorderWidth - 0.0f);
        }
        if (mShapeType == SHAPE_CIRCLE) {
            //边框半径
            mBorderRadius = Math.min((mContentRect.width() - mBorderWidth) / 2.0f, (mContentRect.height() - mBorderWidth) / 2.0f);
            //内容半径
            mContentRadius = Math.min(mContentRect.width() / 2.0f, mContentRect.height() / 2.0f);
        } else if (mShapeType == SHAPE_ROUND) {
            // TODO: 2017/8/21
        }
        updateMatrix(bitmapShader);
    }

    //计算矩形大小
    private RectF calculateRect() {
        int width = mWidth - getPaddingLeft() - getPaddingRight();
        int height = mHeight - getPaddingTop() - getPaddingBottom();
        int left = getPaddingLeft();
        int top = getPaddingTop();
        return new RectF(left, top, left + width, top + height);
    }

    //更新矩阵（这里使用ImageView CENTER_CROP类似计算 所以我们这个控件也仅支持CENTER_CROP 其他的都不起作用）
    private void updateMatrix(BitmapShader bitmapShader) {
        float scale;
        float dx = 0, dy = 0;
        final int bHeight = mBitmap.getHeight();
        final int bWidth = mBitmap.getWidth();
        final float cWidth = mContentRect.width();
        final float cHeight = mContentRect.height();
        //计算缩放比例 平移距离
        if (bWidth * cHeight > cWidth * bHeight) {
            //宽度比 > 高度比 取高度比缩放
            scale = cHeight / (float) bHeight;
            //计算横向移动距离
            dx = (cWidth - bWidth * scale) * 0.5f;
        } else {
            scale = cWidth / (float) bWidth;
            dy = (cHeight - bHeight * scale) * 0.5f;
        }
        Matrix mMatrix = new Matrix();
        //缩放 平移
        mMatrix.setScale(scale, scale);
        mMatrix.postTranslate(Math.round(dx) + mContentRect.left, Math.round(dy) + mContentRect.left);
        // 设置变换矩阵
        bitmapShader.setLocalMatrix(mMatrix);
    }
```

- 接下来需要在onDraw中进行绘制操作：
```
    @Override
    protected void onDraw(Canvas canvas) {
        if (mBitmap == null) {
            return;
        }
        if (mShapeType == SHAPE_CIRCLE) {
            drawCircle(canvas);
        } else if (mShapeType == SHAPE_ROUND) {
            drawRoundRect(canvas);
        }
    }

    //圆角矩形
    private void drawRoundRect(Canvas canvas) {
        if (mFillColor != Color.TRANSPARENT) {
            canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mFillPaint);
        }
        canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mBitmapPaint);
        if (mBorderWidth > 0) {
            canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mBorderPaint);
        }
    }

    //圆形
    private void drawCircle(Canvas canvas) {
        if (mFillColor != Color.TRANSPARENT) {
            canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mContentRadius, mFillPaint);
        }
        canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mContentRadius, mBitmapPaint);
        if (mBorderWidth > 0) {
            canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mBorderRadius, mBorderPaint);
        }
    }
```

- 到这一步，就是万事具备，只欠东风。我们需要在合适的地方来调用getBitmapFormDrawable和updateSteup方法，到现在还没有调用过这两个方法。不卖关子直接贴出代码：
```
    @Override
    public void invalidate() {
        mBitmap = getBitmapFormDrawable();
        updateSetup();
        super.invalidate();
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
        invalidate();
    }
```

最开始想在setImageDrawable调用这两个方法，在跟踪源码的是否发现，ImageView的大部分设置图片的最终都调用了setImageDrawable方法，但是setImageURI并没有。

可是它们都调用了invalidate方法，而且当参数改变的时候也会调用这个方法。所以在这里获取getBitmapFormDrawable和updateSteup。

最后我们需要在onSizeChanged获取到mWidth 、mHeight 并invalidate，ShapIamgeView就大功告成了。

#### 五 代码设置自定义属性
我们发现6个自定义属性只能在xml配置，但是我们想要通过代码改变怎么办呢？其实也很简单，拿自定义边框宽度属性举个例子：
```
   public void setBorderWidth(int borderWidth) {
        if (mBorderWidth == borderWidth) {
            return;
        }
        mBorderPaint.setColor(mBorderWidth = borderWidth);
        invalidate();
    }
```

就是这样，我们就可以得到开头效果图一样的运行效果了，上面四部曲注释也很详细了。

最后福利，贴出完整代码(没有通过代码设置自定义属性的方法)：

```
public class ShapeImageView extends ImageView {
    public static final int SHAPE_CIRCLE = 0;  //圆形
    public static final int SHAPE_ROUND = 1;   //圆角
    private final int DEFAULT_BORDER_WIDTH = 0;
    private final int DEFAULT_ROUND_RADIUS = 0;
    private final int DEFAULT_BORDER_COLOR = Color.BLACK;
    private final int DEFAULT_FILL_COLOR = Color.TRANSPARENT;
    private final boolean DEFAULT_BORDER_OVERLAY = false;
    private final Bitmap.Config BITMAP_CONFIG = Bitmap.Config.ARGB_8888;
    private final int DEFAULT_DRAWABLE_DIMENSION = 2;

    private int mRoundRadius; //圆角
    private int mBorderWidth; //边框宽
    private int mBorderColor; //边框颜色
    private int mFillColor;   //填充颜色
    private boolean mBorderOverlay;  //边框叠加
    private int mShapeType; //形状

    private RectF mContentRect = new RectF();
    private Paint mBorderPaint = new Paint();
    private Paint mBitmapPaint = new Paint();
    private Paint mFillPaint = new Paint();

    private float mContentRadius;
    private float mBorderRadius;
    private Bitmap mBitmap;
    private int mWidth;
    private int mHeight;

    public ShapeImageView(Context context) {
        this(context, null);
    }

    public ShapeImageView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ShapeImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.ShapeImageView, defStyleAttr, 0);
        mBorderWidth = a.getDimensionPixelSize(R.styleable.ShapeImageView_siv_border_width, DEFAULT_BORDER_WIDTH);
        mBorderColor = a.getColor(R.styleable.ShapeImageView_siv_border_color, DEFAULT_BORDER_COLOR);
        mFillColor = a.getColor(R.styleable.ShapeImageView_siv_fill_color, DEFAULT_FILL_COLOR);
        mBorderOverlay = a.getBoolean(R.styleable.ShapeImageView_siv_border_overlay, DEFAULT_BORDER_OVERLAY);
        mShapeType = a.getInt(R.styleable.ShapeImageView_siv_shape_type, SHAPE_CIRCLE);
        mRoundRadius = a.getDimensionPixelSize(R.styleable.ShapeImageView_siv_round_radius, DEFAULT_ROUND_RADIUS);
        a.recycle();
    }

    @Override
    public void invalidate() {
        mBitmap = getBitmapFormDrawable();
        updateSetup();
        super.invalidate();
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
        invalidate();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mBitmap == null) {
            return;
        }
        if (mShapeType == SHAPE_CIRCLE) {
            drawCircle(canvas);
        } else if (mShapeType == SHAPE_ROUND) {
            drawRoundRect(canvas);
        }
    }

    //圆角矩形
    private void drawRoundRect(Canvas canvas) {
        if (mFillColor != Color.TRANSPARENT) {
            canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mFillPaint);
        }
        canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mBitmapPaint);
        if (mBorderWidth > 0) {
            canvas.drawRoundRect(mContentRect, mRoundRadius, mRoundRadius, mBorderPaint);
        }
    }

    //圆形
    private void drawCircle(Canvas canvas) {
        if (mFillColor != Color.TRANSPARENT) {
            canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mContentRadius, mFillPaint);
        }
        canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mContentRadius, mBitmapPaint);
        if (mBorderWidth > 0) {
            canvas.drawCircle(mContentRect.centerX(), mContentRect.centerY(), mBorderRadius, mBorderPaint);
        }
    }

    private void updateSetup() {
        if (mWidth == 0 && mHeight == 0) {
            return;
        }
        if (mBitmap == null) {
            super.invalidate();
            return;
        }
        BitmapShader bitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
        mBitmapPaint.setAntiAlias(true);
        mBitmapPaint.setShader(bitmapShader);

        mBorderPaint.setStyle(Paint.Style.STROKE);
        mBorderPaint.setAntiAlias(true);
        mBorderPaint.setColor(mBorderColor);
        mBorderPaint.setStrokeWidth(mBorderWidth);

        mFillPaint.setStyle(Paint.Style.FILL);
        mFillPaint.setAntiAlias(true);
        mFillPaint.setColor(mFillColor);
        //边框
        mContentRect.set(calculateRect());
        if (!mBorderOverlay && mBorderWidth > 0) {
            mContentRect.inset(mBorderWidth - 0.0f, mBorderWidth - 0.0f);
        }
        if (mShapeType == SHAPE_CIRCLE) {
            //边框半径
            mBorderRadius = Math.min((mContentRect.width() - mBorderWidth) / 2.0f, (mContentRect.height() - mBorderWidth) / 2.0f);
            //内容半径
            mContentRadius = Math.min(mContentRect.width() / 2.0f, mContentRect.height() / 2.0f);
        } else if (mShapeType == SHAPE_ROUND) {
            // TODO: 2017/8/21
        }
        updateMatrix(bitmapShader);
    }

    private void updateMatrix(BitmapShader bitmapShader) {
        float scale;
        float dx = 0, dy = 0;
        final int bHeight = mBitmap.getHeight();
        final int bWidth = mBitmap.getWidth();
        final float cWidth = mContentRect.width();
        final float cHeight = mContentRect.height();
        //计算缩放比例 平移距离
        if (bWidth * cHeight > cWidth * bHeight) {
            //宽度比 > 高度比 取高度比缩放
            scale = cHeight / (float) bHeight;
            //计算横向移动距离
            dx = (cWidth - bWidth * scale) * 0.5f;
        } else {
            scale = cWidth / (float) bWidth;
            dy = (cHeight - bHeight * scale) * 0.5f;
        }
        Matrix mMatrix = new Matrix();
        mMatrix.setScale(scale, scale);
        mMatrix.postTranslate(Math.round(dx) + mContentRect.left, Math.round(dy) + mContentRect.left);
        bitmapShader.setLocalMatrix(mMatrix);
    }

    private RectF calculateRect() {
        int width = mWidth - getPaddingLeft() - getPaddingRight();
        int height = mHeight - getPaddingTop() - getPaddingBottom();
        int left = getPaddingLeft();
        int top = getPaddingTop();
        return new RectF(left, top, left + width, top + height);
    }

    private Bitmap getBitmapFormDrawable() {
        Drawable drawable = getDrawable();
        if (drawable == null) {
            return null;
        }
        if (drawable instanceof BitmapDrawable) {
            return ((BitmapDrawable) drawable).getBitmap();
        }
        try {
            int dWidth = drawable.getIntrinsicWidth() <= 0 ? DEFAULT_DRAWABLE_DIMENSION : drawable.getIntrinsicWidth();
            int dHeight = drawable.getIntrinsicHeight() <= 0 ? DEFAULT_DRAWABLE_DIMENSION : drawable.getIntrinsicHeight();
            Bitmap bitmap = Bitmap.createBitmap(dWidth, dHeight, BITMAP_CONFIG);
            Canvas canvas = new Canvas(bitmap);
            drawable.setBounds(0, 0, canvas.getWidth(), canvas.getHeight());
            drawable.draw(canvas);
            return bitmap;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

到此......

全剧终！！！
