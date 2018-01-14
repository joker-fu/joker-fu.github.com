---
title: Android 绘图之Canvas相关API使用
copyright: true
date: 2018-01-05 15:47:32
categories: [Android,自定义控件]
tags: [Android,自定义控件]
---
Android 自定义控件或多或少都会用到Canvas，那么我们就需要熟悉它的API。Canvas给我们提供了大量的的DrawXXX方法，通过这些方法我们就可以绘制出我们想要的效果。接下来看看官方是怎么说的：
>The Canvas class holds the "draw" calls. To draw something, you need 4 basic components: A Bitmap to hold the pixels, a Canvas to host the draw calls (writing into the bitmap), a drawing primitive (e.g. Rect, Path, text, Bitmap), and a paint (to describe the colors and styles for the drawing).

大致意思就是说：
Canvas类持有“draw”调用。 要绘制一些东西，你需要4个基本组件：一个位图来保存像素，一张画布（Canvas）来主持绘图调用（写入位图），一个绘图图元（如Rect，Path，text，Bitmap）和一支画笔（描述绘图的颜色和样式）。

首先介绍下画笔（Paint）的常用API:
```
    private void initPaint() {
        // 设置最基本的属性
        // 设置画笔颜色
        // 可直接引入Color类，如Color.red等
        mPaint.setColor(int color);
        // 设置画笔模式
        mPaint.setStyle(Style style);
        // Style有3种类型：
        // 类型1：Paint.Style.FILLANDSTROKE（描边+填充）
        // 类型2：Paint.Style.FILL（只填充不描边）
        // 类型3：Paint.Style.STROKE（只描边不填充）
        //设置画笔的粗细
        mPaint.setStrokeWidth(float width);
        // 如设置画笔宽度为10px
        mPaint.setStrokeWidth(10f);
        
        // 不常设置的属性
        // 得到画笔的颜色     
        mPaint.getColor();
        // 设置Shader
        // 即着色器，定义了图形的着色、外观
        mPaint.setShader(Shader shader);
        //设置画笔的a,r,p,g值
        mPaint.setARGB(int a, int r, int g, int b);
        //设置透明度
        mPaint.setAlpha(int a);
        //得到画笔的Alpha值
        mPaint.getAlpha();
        
        // 对字体进行设置（大小、颜色）
        //设置字体大小
        mPaint.setTextSize(float textSize)
        // 设置对齐方式   
        mPaint.setTextAlign(Algin algin);
        // LEFT：左对齐
        // CENTER：居中对齐
        // RIGHT：右对齐
        //设置文本的下划线
        mPaint.setUnderlineText(boolean underlineText);
        //设置文本的删除线
        mPaint.setStrikeThruText(boolean strikeThruText);
        //设置文本粗体
        mPaint.setFakeBoldText(boolean fakeBoldText);
        // 设置斜体
        mPaint.setTextSkewX(-0.5f);
        // 设置文字阴影
        mPaint.setShadowLayer(5,5,5,Color.YELLOW);
    }
```
Paint默认的字体大小为12px，在绘制文本时我们往往要考虑密度density设置合适的字体大小。画笔的默认颜色为黑色，默认的style为FILL，默认的cap为BUTT，默认的线宽为0。

#### 一 坐标系
- Canvas坐标系
Canvas坐标系即Canvas本身的坐标系。Canvas坐标原点（0,0）在View的左上角，向右为X轴正方向，向下为Y轴正方向。Canvas坐标系唯一且不会改变。
- 绘图坐标系
绘图坐标系区别于Canvas坐标系，以我们绘图起始点为坐标原点（0,0），绘图坐标系默认与Canvas坐标系重合。绘图坐标系原点位置可以更改，Canvas的 translate、rotate、scale 操作都是基于绘图坐标系变化。
```
    //绘制坐标系
    private void drawAxis(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int translate = width / 5;

        //设置画笔宽度
        mPaint.setStrokeWidth(10);
        //设置笔触样式
        mPaint.setStrokeCap(Paint.Cap.ROUND);

        //绘制默认绘图坐标系（此时与Canvas坐标系重合）
        //绘制红色X轴
        mPaint.setColor(Color.RED);
        canvas.drawLine(0, 0, width, 0, mPaint);
        //绘制蓝色Y轴
        mPaint.setColor(Color.BLUE);
        canvas.drawLine(0, 0, 0, height, mPaint);

        //平移
        canvas.translate(translate, translate);
        //绘制平移后绘图坐标系
        //绘制红色X轴
        mPaint.setColor(Color.RED);
        canvas.drawLine(0, 0, width, 0, mPaint);
        //绘制蓝色Y轴
        mPaint.setColor(Color.BLUE);
        canvas.drawLine(0, 0, 0, height, mPaint);

        //平移 旋转30度
        canvas.translate(translate, translate);
        canvas.rotate(30);

        //绘制平移旋转后绘图坐标系
        //绘制红色X轴
        mPaint.setColor(Color.RED);
        canvas.drawLine(0, 0, width, 0, mPaint);
        //绘制蓝色Y轴
        mPaint.setColor(Color.BLUE);
        canvas.drawLine(0, 0, 0, height, mPaint);
    }
```

#### 二 Canvas绘制颜色
单一颜色填充Canvas画布
- drawColor
- drawRGB
- drawARGB
```
    //绘制画布颜色
    private void drawCanvasColor(Canvas canvas) {
        //drawARGB
        canvas.drawARGB(255, 47, 140, 150);
    }
```

#### 三 绘制点
- drawPoint 绘制点
- drawPoints 绘制一组点
```
    //绘制点
    private void drawPoint(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int translateY = height/5;

        //设置画笔颜色
        mPaint.setColor(Color.RED);
        //设置画笔宽度
        mPaint.setStrokeWidth(200);

        //设置笔触样式
        mPaint.setStrokeCap(Paint.Cap.BUTT);
        //保存当前画布状态（坐标系）
        canvas.save();
        canvas.drawPoint(width / 3, height / 3, mPaint); 
        //恢复上次画布保存状态（坐标系）
        canvas.restore();

        mPaint.setStrokeCap(Paint.Cap.ROUND);
        mPaint.setColor(Color.GREEN);
        canvas.save();
        canvas.translate(0, translateY);
        canvas.drawPoint(width / 3, height / 3, mPaint);
        canvas.restore();

        mPaint.setStrokeCap(Paint.Cap.SQUARE);
        mPaint.setColor(Color.BLUE);
        canvas.save();
        canvas.translate(translateY, translateY);
        canvas.drawPoint(width / 3, height / 3, mPaint);
        canvas.restore();
    }
```

#### 四 绘制线
-  drawLine 绘制线
- drawLines 绘制一组线
```
    //绘制线
    private void drawLine(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int startX = width / 5;
        int startY = height / 5;

        //设置画笔颜色
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(20);
        //设置笔触样式
        mPaint.setStrokeCap(Paint.Cap.BUTT);
        //画笔触为BUTT的线
        canvas.drawLine(startX, startY, width - startX, startY, mPaint);

        //向下平移20
        canvas.translate(0, 60);

        mPaint.setColor(Color.GREEN);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        //画笔触为ROUND的线
        canvas.drawLine(startX, startY, width - startX, startY, mPaint);

        canvas.translate(0, 60);

        mPaint.setColor(Color.BLUE);
        mPaint.setStrokeCap(Paint.Cap.SQUARE);
        //画笔触为SQUARE的线
        canvas.drawLine(startX, startY, width - startX, startY, mPaint);

        canvas.translate(0, 80);

        mPaint.setColor(Color.BLACK);
        float[] pts = new float[]{startX, startY, width - startX, startY, startX, startY + 60, width - startX, startY + 60};
        canvas.drawLines(pts, mPaint);
    }
```
#### 五 绘制矩形
- drawRect 绘制矩形
- drawRoundRect 绘制圆角矩形
```
    //绘制矩形
    private void drawRect(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();

        //设置画笔颜色
        mPaint.setColor(Color.RED);
        //设置画笔宽度
        mPaint.setStrokeWidth(16);
        //设置画笔样式
        mPaint.setStyle(Paint.Style.FILL);

        canvas.drawRect(20, 20, 400, 200, mPaint);
        //绘制矩形 API21
        //canvas.drawRoundRect(220, 220, 400, 400, 10, 10, mPaint);
        RectF rect = new RectF(20, 420, 800, 600);
        canvas.drawRoundRect(rect, 10, 10, mPaint);

        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);

        canvas.drawRect(20, 800, 400, 1000, mPaint);
        RectF rect1 = new RectF(20, 1200, 800, 1400);
        canvas.drawRoundRect(rect1, 10, 10, mPaint);
    }
```
#### 六 绘制圆
- drawCircle 绘制圆
```
    //绘制圆形
    private void drawCircle(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int halfWidth = width / 2;
        int d = height / 3;
        int r = d / 2 - 10;

        //设置画笔颜色
        mPaint.setColor(Color.RED);
        //设置画笔宽度
        mPaint.setStrokeWidth(2);
        //设置画笔样式
        mPaint.setStyle(Paint.Style.FILL);

        //绘制圆
        canvas.drawCircle(halfWidth, (float) (d * 0.5), r, mPaint);

        //绘制两个圆实现圆环效果
        mPaint.setColor(Color.GREEN);
        canvas.drawCircle(halfWidth, (float) (d * 1.5), r, mPaint);
        mPaint.setColor(Color.WHITE);
        canvas.drawCircle(halfWidth, (float) (d * 1.5), r - 40, mPaint);

        //绘制一个圆实现圆环效果
        mPaint.setStrokeWidth(40);
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);
        //半径减去画笔画笔宽度的一半 圆的大小和前面一致
        canvas.drawCircle(halfWidth, (float) (d * 2.5), r - 20, mPaint);
    }
```
#### 七 绘制椭圆
- drawOval 绘制椭圆
跟绘制圆差不多，唯一不同就是绘制圆是指定圆心和半径，绘制椭圆是指定左上右下4个距离。
```
    //绘制椭圆 
    private void drawOval(Canvas canvas) {
        //设置画笔颜色
        mPaint.setColor(Color.RED);
        //设置画笔宽度
        mPaint.setStrokeWidth(2);
        //设置画笔样式
        mPaint.setStyle(Paint.Style.FILL);
        RectF overRect = new RectF(20, 20, 400, 200);
        canvas.drawOval(overRect, mPaint);
        
        mPaint.setColor(Color.GREEN);
        mPaint.setStrokeWidth(20);
        mPaint.setStyle(Paint.Style.STROKE);
        overRect = new RectF(20, 220, 200, 600);
        canvas.drawOval(overRect, mPaint);
    }
```
#### 八 绘制圆弧
- drawArc 绘制弧
绘制圆弧，可以绘制弧面和弧线。弧面即用弧围成的填充面，弧线即为弧面的轮廓线。
```
    //绘制弧
    private void drawAcr(Canvas canvas) {
        int width = canvas.getWidth();
        
        //设置画笔颜色
        mPaint.setColor(Color.RED);
        //设置画笔宽度
        mPaint.setStrokeWidth(2);
        //设置画笔样式
        mPaint.setStyle(Paint.Style.FILL);
        //和前面绘制椭圆的矩形一致（正方形也可）
        RectF arcRect = new RectF(20, 20, 400, 200);
        //根据矩形绘制弧形 开始角度0 扫过角度360 通过中心点首尾连接：true
        // 和前面绘制的椭圆效果一致
        canvas.drawArc(arcRect, 0, 360, true, mPaint);

        canvas.translate(0, width / 5);
        canvas.drawArc(arcRect, 0, 90, true, mPaint);

        canvas.translate(0, width / 5);
        canvas.drawArc(arcRect, 0, 90, false, mPaint);

        //正方形
        arcRect = new RectF(20, 20, 220, 220);
        mPaint.setStrokeWidth(4);
        mPaint.setStyle(Paint.Style.STROKE);
        canvas.translate(0, width / 5);
        //根据矩形绘制弧形 角度从0-1800 首尾连接
        canvas.drawArc(arcRect, 0, 180, true, mPaint);

        canvas.translate(0, width / 5);
        //根据矩形绘制弧形 角度从0-1800 首尾不连接
        canvas.drawArc(arcRect, 0, 180, false, mPaint);

        mPaint.setStrokeWidth(10);
        canvas.translate(width / 2, width / 5);
        //drawArc绘制出圆环
        canvas.drawArc(arcRect, 0, 270, false, mPaint);
        mPaint.setColor(Color.GREEN);
        canvas.drawArc(arcRect, 270, 90, false, mPaint);
    }
```
#### 九 绘制文字
- drawText 绘制文本
- drawPosText 根据位置绘制文本
- drawTextOnPath 根据路径绘制文本
```
    //绘制文字
    private void drawText(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int textHeight = 32;
        int halfWidth = width / 2;

        //设置画笔颜色
        mPaint.setColor(Color.GRAY);
        //设置画笔宽度
        mPaint.setStrokeWidth(2);
        canvas.drawLine(halfWidth, 0, halfWidth, height, mPaint);

        mPaint.setColor(Color.BLACK);
        //设置文本大小
        mPaint.setTextSize(textHeight);
        //绘制普通文本
        canvas.drawText("绘制普通文本", 0, textHeight, mPaint);

        mPaint.setColor(Color.RED);
        mPaint.setTextAlign(Paint.Align.LEFT);
        //存储画布状态 坐标系
        canvas.save();
        //平移
        canvas.translate(halfWidth, textHeight);
        //绘制左对齐文本
        canvas.drawText("绘制左对齐文本", 0, textHeight, mPaint);
        //恢复上次存储状态
        canvas.restore();

        mPaint.setColor(Color.GREEN);
        mPaint.setTextAlign(Paint.Align.CENTER);
        //存储画布状态 坐标系
        canvas.save();
        //平移
        canvas.translate(halfWidth, textHeight * 2);
        //绘制居中对齐文本(对齐是根据绘图坐标系而言)
        canvas.drawText("绘制居中对齐文本", 0, textHeight, mPaint);
        //恢复上次存储状态
        canvas.restore();

        mPaint.setColor(Color.BLUE);
        mPaint.setTextAlign(Paint.Align.RIGHT);
        //存储画布状态 坐标系
        canvas.save();
        //平移
        canvas.translate(halfWidth, textHeight * 3);
        //绘制右对齐文本
        canvas.drawText("绘制右对齐文本", 0, textHeight, mPaint);
        //恢复上次存储状态
        canvas.restore();


        mPaint.setColor(Color.BLACK);
        //恢复默认对齐方式
        mPaint.setTextAlign(Paint.Align.LEFT);
        //设置下划线
        mPaint.setUnderlineText(true);
        //设置加粗
        mPaint.setFakeBoldText(true);
        //设置删除线
        mPaint.setStrikeThruText(true);
        //存储画布状态 坐标系
        canvas.save();
        //平移
        canvas.translate(halfWidth, textHeight * 4);
        //绘制下划线加粗文本
        canvas.drawText("绘制下划线加粗文本", 0, textHeight, mPaint);
        //恢复上次存储状态
        canvas.restore();

        mPaint.reset();
        mPaint.setColor(Color.BLACK);
        //设置文本大小
        mPaint.setTextSize(textHeight);
        //存储画布状态 坐标系
        canvas.save();
        canvas.translate(width / 3, height / 3);
        //旋转
        canvas.rotate(45);
        //绘制倾斜文本
        canvas.drawText("绘制倾斜文本", 0, textHeight, mPaint);
        //恢复上次存储状态
        canvas.restore();

                float[] pos = new float[]{40, 40, 80, 80, 120, 120, 160, 160, 200, 200, 240, 240};
        //存储画布状态 坐标系
        canvas.save();
        canvas.translate(width / 3, height / 2);
        canvas.drawPosText("绘制位置文本", pos, mPaint);
        //恢复上次存储状态
        canvas.restore();

        Path path = new Path();
        path.cubicTo(550, 750, 350, 250, 850, 800);
        //画路径
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        canvas.drawPath(path,mPaint);
        canvas.drawTextOnPath("绘制文本OnPath绘制文本OnPath绘制文本OnPath绘制文本OnPath绘制文本OnPath", path, 50, 0, mPaint);
    }
```
#### 十 绘制Path
- drawPath 绘制路径
```
    //path拐点集合
    List<Point> points = new ArrayList<>();

    //绘制Path
    private void drawPath(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int deltaX = width / 4;
        int deltaY = (int) (deltaX * 0.75);

        //设置画笔颜色
        mPaint.setColor(Color.GRAY);
        //设置画笔宽度
        mPaint.setStrokeWidth(4);

        //Fill模式画面
        Path path1 = new Path();
        RectF arcRect = new RectF(0, 0, deltaX, deltaY);
        path1.addArc(arcRect, 0, 360);
        path1.addRect(deltaX, 0, deltaX << 1, deltaY, Path.Direction.CW);
        canvas.drawPath(path1, mPaint);

        //STROKE模式画线
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        canvas.translate(0, deltaY << 1);
        canvas.drawPath(path1, mPaint);

        mPaint.setColor(Color.BLUE);
//        mPaint.setStyle(Paint.Style.FILL);
        canvas.translate(0, deltaY << 1);

        Path path2 = new Path();
        path2.lineTo(100, 0);   //画线
        points.add(new Point(100, 0));  //终点 X: 100 Y: 0

        RectF overRect = new RectF(100, -50, 300, 50);
        path2.arcTo(overRect, 180, -90); //画椭圆 左下部分
        points.add(new Point(200, 50)); //终点 X: 250 Y: 50

        overRect = new RectF(200, 0, 400, 100);
        path2.arcTo(overRect, 180, 90); //画椭圆右上部分
        points.add(new Point(300, 0));  //终点 X: 300 Y: 0

        path2.lineTo(350, 50);  //画线
        points.add(new Point(350, 50)); //终点 X: 350 Y: 50

        //画二阶贝塞尔曲线 第一个坐标为控制点 最后为终点
        path2.quadTo(450, -50, 500, 150);
        points.add(new Point(500, 150)); //终点 X: 500 Y: 150

        //画三阶阶贝塞尔曲线 前两个坐标为控制点 最后为终点
        path2.cubicTo(600, 150, 700, 0, 380, -200);
        points.add(new Point(380, -200)); //终点 X: 480 Y: -200

        path2.rLineTo(0, 100);
        points.add(new Point(380, -200 + 100)); //终点 X: 480 Y: -100
        canvas.drawPath(path2, mPaint);

        mPaint.setStrokeWidth(10);
        mPaint.setColor(Color.RED);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        for (Point point : points) {
            canvas.drawPoint(point.x, point.y, mPaint);
        }
    }
```
#### 十一 绘制Bitmap
- drawBitmap 绘制Bitmap
```
    //绘制Bitmap
    private void drawBitmap(Canvas canvas) {
        BitmapDrawable drawable = (BitmapDrawable) getResources().getDrawable(R.mipmap.ic_launcher);
        Bitmap bitmap = drawable.getBitmap();

        //如果bitmap不存在，那么就不执行下面的绘制代码
        if (bitmap == null) {
            return;
        }
        //直接完全绘制Bitmap
        canvas.drawBitmap(bitmap, 0, 0, mPaint);

        //绘制Bitmap的一部分，并对其拉伸
        //srcRect定义了要绘制Bitmap的哪一部分
        Rect srcRect = new Rect();
        srcRect.left = 0;
        srcRect.right = bitmap.getWidth();
        srcRect.top = 0;
        srcRect.bottom = (int) (0.33 * bitmap.getHeight());
        float radio = (float) (srcRect.bottom - srcRect.top) / bitmap.getWidth();
        //dstRecF定义了要将绘制的Bitmap拉伸到哪里
        RectF dstRecF = new RectF();
        dstRecF.left = 0;
        dstRecF.right = canvas.getWidth();
        dstRecF.top = bitmap.getHeight();
        float dstHeight = (dstRecF.right - dstRecF.left) * radio;
        dstRecF.bottom = dstRecF.top + dstHeight;
        canvas.drawBitmap(bitmap, srcRect, dstRecF, mPaint);
    }
```
以上就是大部分的绘制操作，当然还有一些没介绍到，还有一些需要在API21（5.0）以上才能使用，这个使用到的时候可以弄个小demo看下效果，接下来介绍下针对画布变换和画布裁剪做一些介绍。
#### 十二 画布变换
- translate 平移
- scale 缩放
- rotate 旋转
- skew 错切

在学习绘制操作的时候很多地方都使用到了translate 和rotate 这里就不再对它们进行操作

1.画布scale
```
    //画布缩放
    private void canvasScale(Canvas canvas) {
        int width = canvas.getWidth();
        int height = canvas.getHeight();
        int hWidth = width / 2;
        int hHeight = height / 2;

        mPaint.setColor(Color.GRAY);
        mPaint.setStrokeWidth(2);
        mPaint.setStyle(Paint.Style.STROKE);
        //使用path 画布中心的X Y 轴
        Path path = new Path();
        path.moveTo(hWidth, 0);
        path.lineTo(hWidth, height);
        path.moveTo(0, hHeight);
        path.lineTo(width, hHeight);
        canvas.drawPath(path, mPaint);

        //画布平移
        canvas.translate(hWidth, hHeight);

        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(5);
        RectF rectF = new RectF(0, 0, 200, 100);
        canvas.drawRect(rectF, mPaint);

        //默认0,0进行缩放
        canvas.scale(1.5f, 1.5f);
        mPaint.setColor(Color.BLUE);
        canvas.drawRect(rectF, mPaint);

        //100,0进行缩放
        canvas.scale(1.5f, 1.5f, 100, 0);
        mPaint.setColor(Color.GREEN);
        canvas.drawRect(rectF, mPaint);

        /*
        当缩放倍数为负数时，会先进行缩放，然后根据不同情况进行图形翻转：
        （设缩放倍数为（a,b），旋转中心为（px，py））：
        a<0，b>0：以px为轴翻转
        a>0，b<0：以py为轴翻转
        a<0，b<0：以旋转中心翻转
         */
        //100,0进行缩放 以Y轴翻转
        canvas.scale(1f, -1f, 100, 0);
        mPaint.setColor(Color.BLACK);
        canvas.drawRect(rectF, mPaint);
    }
```
2.画布skew
```
    //画布错切
    private void canvasSkew(Canvas canvas) {
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(5);
        mPaint.setStyle(Paint.Style.STROKE);
        RectF rectF = new RectF(0, 0, 400, 200);
        canvas.translate(100,100);
        canvas.drawRect(rectF, mPaint);

        canvas.save();
        canvas.translate(0,300);
        //画布X正方向错切45°
        canvas.skew(1f, 0);
        canvas.drawRect(rectF, mPaint);
        canvas.restore();

        canvas.save();
        canvas.translate(0,600);
        //画布Y负方向错切45°
        canvas.skew(0, -1f);
        canvas.drawRect(rectF, mPaint);
        canvas.restore();
        
        mPaint.setColor(Color.GREEN);
        canvas.drawLine(0,400,600,400,mPaint);
    }
```
#### 十三 画布裁剪
- clipPath
- clipRect 
- clipRegion(过时)
- getClipBounds
```
    //画布裁剪
    private void canvasClip(Canvas canvas) {
        canvas.drawColor(Color.RED);
        canvas.save();
        //矩形裁剪
        canvas.clipRect(0, 0, 400, 200);
        canvas.drawColor(Color.BLUE);
        canvas.restore();

        canvas.save();
        canvas.translate(20, 400);
        Path path = new Path();
        path.addRect(0, 0, 400, 200, Path.Direction.CCW);
        canvas.drawPath(path, mPaint);
        //路径裁剪
        canvas.clipPath(path);
        canvas.drawColor(Color.GREEN);
        //获得裁剪边界
        Rect bounds = canvas.getClipBounds();
        mPaint.setColor(Color.BLACK);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10);
        canvas.drawRect(bounds, mPaint);
        canvas.restore();
    }
```

这里针对Canvas的API进行了一些说明，希望对大家有所帮助。没有添加完整工程上来，但是每一个方法都是实际跑过的，只需要在onDraw中调用就可以看到效果了。