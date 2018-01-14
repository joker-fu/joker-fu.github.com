---
title: Android 绘图之PorterDuffXferMode实例讲解与源码解析
copyright: true
date: 2018-01-05 15:47:47
categories: [Android,自定义控件]
tags: [Android,自定义控件]
---
#### 一 PorterDuffXferMode简介
PorterDuffXferMode使用PorterDuff.Mode规则将所绘制图形和Canvas上图形混合，最终更新Canvas展示新的图形。PorterDuffXferMode的使用也非常简单，在需要使用的时候paint.setXfermode(PorterDuff.Mode mode)设置混合模式，不需要使用了将其设置为null即可。
PorterDuff.Mode共分为16种模式：CLEAR、SRC、DST、SRC_OVER、DST_OVER、SRC_IN、DST_IN、SRC_OUT、DST_OUT、SRC_ATOP、DST_ATOP、XOR、DARKEN、LIGHTEN、MULTIPLY、SCREEN。
官方也给出了各自的效果图，也就是网上搜索PorterDuffXferMode出镜率最高的一张图：

![官方效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-001.png)

#### 二 开胃实例
在网上搜了很多文章，帮助不是很大，还是需要感谢下他们，从中学习到了些套路。看完官方的效果图，接下来通过几个实际的示例撸一撸。
1.示例一
首先自定义一个测试类XfermodeTest，在onDraw中调用实际的绘制方法。接下来的示例，直接给出实际绘制方法，就不再给出整个类。我们不设置PorterDuffXferMode，看看Canvas的绘制效果： 
```
public class XfermodeTest extends View {
    private Paint mPaint = new Paint();

    public XfermodeTest(Context context) {
        this(context, null);
    }

    public XfermodeTest(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public XfermodeTest(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        testNoXfermode(canvas);
    }

    private void testNoXfermode(Canvas canvas) {
        //绘制背景
        canvas.drawColor(Color.GRAY);
        //绘制蓝色矩形
        mPaint.setColor(Color.BLUE);
        canvas.drawRect(200, 200, 600, 600, mPaint);
        //绘制黄色圆形
        mPaint.setColor(Color.YELLOW);
        canvas.drawCircle(200, 200, 200, mPaint);
    }
}
```
![示例一效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-002.png)
我们先讲讲Canvas的绘制原理，我们将Canvas当作我们普通的A4纸就行了。首先我们在纸上绘制了灰色的背景，然后我们将矩形所在位置绘制成了蓝色，圆形区域绘制成了黄色，是不是很好理解。

2.示例二
我们设置PorterDuffXferMode为SRC_IN试试效果，看看Canvas的绘制效果： 
```
    private void testXfermodeSRC_IN(Canvas canvas) {
        canvas.drawColor(Color.GRAY);

        mPaint.setColor(Color.BLUE);
        canvas.drawRect(200, 200, 600, 600, mPaint);

        //设置为SCR_IN模式
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.CLEAR));

        mPaint.setColor(Color.YELLOW);
        canvas.drawCircle(200, 200, 200, mPaint);

        //清空所设置模式
        mPaint.setXfermode(null);
    }
```
![示例二【SRC_IN】效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-003.png)
纳尼！居然不一致，是不是我拿之前的效果来忽悠，换个CLEAR效果试试。
![示例二【CLEAR】效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-004.png)
圆形变成了一块黑色，您可以亲自动手试一试，也可以换个其他效果看一看效果（这里我就不给出其他模式的效果了）。

这里我们又讲讲设置Xfermode后Canvas的绘制原理，其实还是同示例一所讲原理差不多，只是画笔在绘制的时候会判断有没有设置Xfermode，设置了Xfermode会把绘制的地方按照一定的规则进行混合，然后在绘制出来最终效果。

接下来我们来看看官方实例是怎么做的，为什么能绘制出这样的效果：

#### 三 官方源码解析
文章开头看了官方效果图，但是我们实践出来效果却......，接下来我们看看源码（解析的注释都放在源码里了）：
```
public class Xfermodes extends GraphicsActivity {  
  
    // create a bitmap with a circle, used for the "dst" image  
    //创建一个圆形bitmap,用作dst图片
    static Bitmap makeDst(int w, int h) {  
        //创建宽为w 高为h的bitmap
        Bitmap bm = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);  
        Canvas c = new Canvas(bm);  
        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);  
  
        p.setColor(0xFFFFCC44);  
        //绘制圆 注意绘制圆所占面积比所创建出来的bitmap要小
        c.drawOval(new RectF(0, 0, w*3/4, h*3/4), p);  
        return bm;  
    }  
  
    // create a bitmap with a rect, used for the "src" image  
    //创建一个矩形bitmap，用作src图片
    static Bitmap makeSrc(int w, int h) {  
        //创建宽为w 高为h的bitmap
        Bitmap bm = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);  
        Canvas c = new Canvas(bm);  
        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);  
  
        p.setColor(0xFF66AAFF);  
        //绘制矩形 注意绘制矩形所占面积比所创建出来的bitmap要小
        c.drawRect(w/3, h/3, w*19/20, h*19/20, p);  
        return bm;  
    }  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(new SampleView(this));  
    }  
  
    private static class SampleView extends View {  
        private static final int W = 64;  
        private static final int H = 64;  
        private static final int ROW_MAX = 4;   // number of samples per row  
  
        private Bitmap mSrcB;  
        private Bitmap mDstB;  
        private Shader mBG;     // background checker-board pattern  
  
        private static final Xfermode[] sModes = {  
            new PorterDuffXfermode(PorterDuff.Mode.CLEAR),  
            new PorterDuffXfermode(PorterDuff.Mode.SRC),  
            new PorterDuffXfermode(PorterDuff.Mode.DST),  
            new PorterDuffXfermode(PorterDuff.Mode.SRC_OVER),  
            new PorterDuffXfermode(PorterDuff.Mode.DST_OVER),  
            new PorterDuffXfermode(PorterDuff.Mode.SRC_IN),  
            new PorterDuffXfermode(PorterDuff.Mode.DST_IN),  
            new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT),  
            new PorterDuffXfermode(PorterDuff.Mode.DST_OUT),  
            new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP),  
            new PorterDuffXfermode(PorterDuff.Mode.DST_ATOP),  
            new PorterDuffXfermode(PorterDuff.Mode.XOR),  
            new PorterDuffXfermode(PorterDuff.Mode.DARKEN),  
            new PorterDuffXfermode(PorterDuff.Mode.LIGHTEN),  
            new PorterDuffXfermode(PorterDuff.Mode.MULTIPLY),  
            new PorterDuffXfermode(PorterDuff.Mode.SCREEN)  
        };  
  
        private static final String[] sLabels = {  
            "Clear", "Src", "Dst", "SrcOver",  
            "DstOver", "SrcIn", "DstIn", "SrcOut",  
            "DstOut", "SrcATop", "DstATop", "Xor",  
            "Darken", "Lighten", "Multiply", "Screen"  
        };  
  
        public SampleView(Context context) {  
            super(context);  
            //创建矩形Src的bitmap
            mSrcB = makeSrc(W, H);  
            //创建圆形Dst的bitmap
            mDstB = makeDst(W, H);  
  
            // make a ckeckerboard pattern  
            //制作棋盘布局 这里使用了BitmapShader的Shader.TileMode.REPEAT模式
            Bitmap bm = Bitmap.createBitmap(new int[] { 0xFFFFFFFF, 0xFFCCCCCC,  
                                            0xFFCCCCCC, 0xFFFFFFFF }, 2, 2,  
                                            Bitmap.Config.RGB_565);  
            mBG = new BitmapShader(bm,  
                                   Shader.TileMode.REPEAT,  
                                   Shader.TileMode.REPEAT);  
            Matrix m = new Matrix();  
            //设置缩放
            m.setScale(6, 6);  
            mBG.setLocalMatrix(m);  
        }  
  
        @Override protected void onDraw(Canvas canvas) {  
            canvas.drawColor(Color.WHITE);  
  
            Paint labelP = new Paint(Paint.ANTI_ALIAS_FLAG);  
            labelP.setTextAlign(Paint.Align.CENTER);  
  
            Paint paint = new Paint();  
            paint.setFilterBitmap(false);  
  
            canvas.translate(15, 35);  
  
            int x = 0;  
            int y = 0;  
            for (int i = 0; i < sModes.length; i++) {  
                // draw the border  
                //绘制边框
                paint.setStyle(Paint.Style.STROKE);  
                paint.setShader(null);  
                canvas.drawRect(x - 0.5f, y - 0.5f,  
                                x + W + 0.5f, y + H + 0.5f, paint);  
  
                // draw the checker-board pattern  
                //绘制棋盘图案
                paint.setStyle(Paint.Style.FILL);  
                paint.setShader(mBG);  
                //绘制棋盘图案矩形大小 注意这个矩形的宽为W 高为H和Src、Dst的Bitmap大小一致
                canvas.drawRect(x, y, x + W, y + H, paint);  
  
                // draw the src/dst example into our offscreen bitmap  
                //将src / dst示例绘制到我们的屏幕外位图中（其实就是保存了当前Canvas，创建了新的Canvas，这之后的操作在新的Canvas执行）
                int sc = canvas.saveLayer(x, y, x + W, y + H, null,  
                                          Canvas.MATRIX_SAVE_FLAG |  
                                          Canvas.CLIP_SAVE_FLAG |  
                                          Canvas.HAS_ALPHA_LAYER_SAVE_FLAG |  
                                          Canvas.FULL_COLOR_LAYER_SAVE_FLAG |  
                                          Canvas.CLIP_TO_LAYER_SAVE_FLAG);  
                canvas.translate(x, y);  
                //绘制圆形Dst的bitmap
                canvas.drawBitmap(mDstB, 0, 0, paint);  
                //设置画笔的Xfermode
                paint.setXfermode(sModes[i]);  
                //绘制圆形的Src的bitmap
                canvas.drawBitmap(mSrcB, 0, 0, paint);  
                //清空画笔的Xfermode
                paint.setXfermode(null);  
                //恢复之前保存的图层，并将新图层和保存图层合并
                canvas.restoreToCount(sc);  
  
                // draw the label  
                //绘制标签
                canvas.drawText(sLabels[i],  
                                x + W/2, y - labelP.getTextSize()/2, labelP);  
  
                x += W + 10;  
  
                // wrap around when we've drawn enough for one row  
                if ((i % ROW_MAX) == ROW_MAX - 1) {  
                    x = 0;  
                    y += H + 30;  
                }  
            }  
        }  
    }  
}
```
在源码中添加了一些中文注释，看起来也好理解多了。
首先SampleView 构造器一进来就创建了分别创建了作为Dst、Src和背景的Bitmap，然后在onDraw里面做了绘制操作。
我们仔细观察三个矩形的大小发现，Dst和Src背景矩形和作为背景图矩形大小一直，但是绘制圆和矩形实际所占面积要比这个矩形小。__（通俗点讲就是，实际绘制的黄色圆和蓝色矩形并不只有我们所看到的那么大，它们实际大小和棋盘背景大小一致，他们分别位于各自bm的左上角和右下角。)__
还有不明白的可以改变makeSrc里面的Canvas背景颜色看一看。

#### 四 正餐实例
我们已经就官方源码做了解析，我们可以看出onDraw里面对Canvas进行了保存和恢复。
1.示例三
```
private void testXfermodeSRC_IN(Canvas canvas) {
        canvas.drawColor(Color.GRAY);

        //保存图层
        int id = canvas.saveLayer(0, 0, canvas.getWidth(), canvas.getHeight(), null,
                Canvas.MATRIX_SAVE_FLAG |
                        Canvas.CLIP_SAVE_FLAG |
                        Canvas.HAS_ALPHA_LAYER_SAVE_FLAG |
                        Canvas.FULL_COLOR_LAYER_SAVE_FLAG |
                        Canvas.CLIP_TO_LAYER_SAVE_FLAG);
        
        mPaint.setColor(Color.BLUE);
        canvas.drawRect(200, 200, 600, 600, mPaint);

        //设置为Canvas Xfermode
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));

        mPaint.setColor(Color.YELLOW);
        canvas.drawCircle(200, 200, 200, mPaint);

        //清空所设置模式
        mPaint.setXfermode(null);
        //恢复Canvas
        canvas.restoreToCount(id);
    }
```
![示例三【SRC_IN】效果](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-005.png)
似乎跟效果图的SRC_IN还是差一大截呢？源码里面创建了2个和背景大小相等的bitmap，我们修改黄色圆所占矩形大小覆盖着蓝色矩形呢？

2.示例四
```
    private void testXfermodeSRC_IN(Canvas canvas) {
        canvas.drawColor(Color.GRAY);

        //保存图层
        int id = canvas.saveLayer(0, 0, canvas.getWidth(), canvas.getHeight(), null,
                Canvas.MATRIX_SAVE_FLAG |
                        Canvas.CLIP_SAVE_FLAG |
                        Canvas.HAS_ALPHA_LAYER_SAVE_FLAG |
                        Canvas.FULL_COLOR_LAYER_SAVE_FLAG |
                        Canvas.CLIP_TO_LAYER_SAVE_FLAG);

        mPaint.setColor(Color.BLUE);
        canvas.drawRect(200, 200, 600, 600, mPaint);
        
        mPaint.setColor(Color.YELLOW);
        //改变圆形的大小背景矩形大小
        Bitmap bitmap = Bitmap.createBitmap(600, 600, Bitmap.Config.ARGB_8888);
        Canvas bmCanvas = new Canvas(bitmap);
        bmCanvas.drawCircle(200, 200, 200, mPaint);
        //设置为Canvas Xfermode
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
        canvas.drawBitmap(bitmap, 0, 0, mPaint);

        //清空所设置模式
        mPaint.setXfermode(null);
        //恢复Canvas
        canvas.restoreToCount(id);
    }
```
![示例四【SRC_IN】修改黄色圆所占矩形大小效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-006.png)
看着跟官方效果图差不多了，但是怎么好像跟DST_IN是一致的呢？那我们将Xfermode模式设置为DST_IN呢？只是修改模式，其他部分跟示例四一致：
![修改示例四【SRC_IN】为【DST_IN】效果图](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-007.png)
效果跟官方效果图DST_IN一致。真是奇怪了...再仔细看看源码......源码先绘制的圆形，而我们先绘制的矩形，是不是这个原因呢？我们尝试修改绘制顺序：
3.示例五
```
    private void testXfermodeSRC_IN(Canvas canvas) {
        canvas.drawColor(Color.GRAY);

        //保存图层
        int id = canvas.saveLayer(0, 0, canvas.getWidth(), canvas.getHeight(), null,
                Canvas.MATRIX_SAVE_FLAG |
                        Canvas.CLIP_SAVE_FLAG |
                        Canvas.HAS_ALPHA_LAYER_SAVE_FLAG |
                        Canvas.FULL_COLOR_LAYER_SAVE_FLAG |
                        Canvas.CLIP_TO_LAYER_SAVE_FLAG);

        mPaint.setColor(Color.YELLOW);
        canvas.drawCircle(200, 200, 200, mPaint);

        mPaint.setColor(Color.BLUE);
        //改变圆形的大小背景矩形大小
        Bitmap bitmap = Bitmap.createBitmap(600, 600, Bitmap.Config.ARGB_8888);
        Canvas bmCanvas = new Canvas(bitmap);
        bmCanvas.drawRect(200, 200, 600, 600, mPaint);
        //设置为Canvas Xfermode
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
        canvas.drawBitmap(bitmap, 0, 0, mPaint);

        //清空所设置模式
        mPaint.setXfermode(null);
        //恢复Canvas
        canvas.restoreToCount(id);
    }
```
![示例五【SRC_IN】修改绘制顺序](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-008.png)

#### 五：PorterDuffXfermode总结
先写到到这里，我们就可以稍作休息做个总结，以上先介绍了Canvas绘图的基本原理，然后以SRC_IN和DST_IN做对比介绍了PorterDuffXfermode使用，其他的规则大同小异。

__总结：__
a.关于DST和SRC，简单理解先绘制的就是DST，后绘制的就是SRC。

b.需要使用PorterDuffXfermode的地方，开始绘制的时候saveLayer，绘制完成后restoreToCount。

c.需要注意DST和SRC所占矩形的大小，很多人说需要两个大小一致的bitmap，其实是正确的，我们这里只创建了一个bitmap，也无所谓大小一致一说。我这里的示例只需要SRC所占面积大于DST所占面积就OK，所有大家需要根据实际需求来做。

d.不是官方的图才是正确的，其实示例三、示例四、示例五都是正确的。示例三的黄色圆形所占矩形没有覆盖完蓝色矩形，效果图左上角那一点扇形黄色正式SRC_IN的效果。示例四和五则是因为DST是蓝色矩形，而官方的DST是黄色圆形。

e.如果你觉得混合模式没有正确使用，可以让调用setLayerType(View.LAYER_TYPE_SOFTWARE, null)方法切换到软件渲染模式，把我们的View禁用掉GPU硬件加速，这样所有的混合模式都能正常使用

PS:关于文中黄色圆形和蓝色矩形的【所占矩形】或者【所占面积】可能说得不太明白，附上图一张再叙述弥补一下：
![所占矩形和所占面积](http://114.67.156.211/bolg/android/Android-%E7%BB%98%E5%9B%BE%E4%B9%8BPorterDuffXferMode%E5%AE%9E%E4%BE%8B%E8%AE%B2%E8%A7%A3%E4%B8%8E%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/2018-01-05-009.png)
```
    private void testXfermode(Canvas canvas) {
        canvas.drawColor(Color.GRAY);
        
        mPaint.setColor(Color.YELLOW);
        canvas.drawCircle(200, 200, 200, mPaint);

        mPaint.setColor(Color.BLUE);
        //改变圆形的大小所占矩形大小
        Bitmap bitmap = Bitmap.createBitmap(600, 600, Bitmap.Config.ARGB_8888);
        Canvas bmCanvas = new Canvas(bitmap);
        bmCanvas.drawColor(Color.RED);
        bmCanvas.drawRect(200, 200, 600, 600, mPaint);
        canvas.drawBitmap(bitmap, 0, 0, mPaint);
    }
```
可以看到代码里，我们绘制的顺序是：绘制灰色背景->绘制黄色圆形->绘制蓝色矩形，绘制蓝色矩形前，我们改变了蓝色矩形所在canvas的颜色为红色。红色就是蓝色矩形文中所说的所占矩形和所占面积，而我们实际看到的只有蓝色矩形。而红色又是不透明的根据绘制原理，也就覆盖了之前的黄色圆形。因为new Canvas()默认背景是透明的。也就是我在官方文档中所注释的比所创建出来的bitmap要小是一样的：
>  //绘制圆 注意绘制圆所占面积比所创建出来的bitmap要小
//绘制矩形 注意绘制矩形所占面积比所创建出来的bitmap要小

关于PorterDuffXfermode的讲解就这么多，如有不对可以指出，共同学习进步。

谢谢阅览！
