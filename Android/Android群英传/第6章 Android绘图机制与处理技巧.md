# 第6章 Android绘图机制与处理技巧

[TOC]

## 1 屏幕的尺寸信息

### 1.1 屏幕参数

* 屏幕大小 对角线长度
* 分辨路 720x1280指宽有720个像素点，高有1280个像素点
* PPI 每英寸像素（Pixels Per Inch），又称DPI（Dots Per Inch），对角线像素点除以屏幕大小

### 1.2 系统屏幕密度

| 密度    |  ldpi   |  mdpi   |  hdpi   |  xhdpi   | xxhdpi  |
| --------| :----:  | :----:  | :----:  | :----:   | :----:  |
| 密度值  |   120   |   160   |   240   |   320    |   480   | 
| 比例    |   0.75  |   1     |   1.5   |    2     |    3    |
| 分辨率  | 240x320 | 320x480 | 480x800 | 720x1280 |1080x1920|

### 1.3 独立像素密度dp

160的屏幕上1dp == 1px，换算比例见上表。

### 1.4 单位转换

```
public class DisplayUtil {
    /**
     * dp转px
     *
     * @param context 上下文
     * @param dpValue dp值
     * @return px值
     */
    public static int dp2px(Context context, float dpValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dpValue * scale + 0.5f);
    }

    /**
     * px转dp
     *
     * @param context 上下文
     * @param pxValue px值
     * @return dp值
     */
    public static int px2dp(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }

    /**
     * sp转px
     *
     * @param context 上下文
     * @param spValue sp值
     * @return px值
     */
    public static int sp2px(Context context, float spValue) {
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
    }

    /**
     * px转sp
     *
     * @param context 上下文
     * @param pxValue px值
     * @return sp值
     */
    public static int px2sp(Context context, float pxValue) {
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValue / fontScale + 0.5f);
    }
    
     /**
     * dp转px
     * 通过系统提供的函数
     */
    public static int dp2pxByTypedValue(int dp) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dp, getResources().getDisplayMetrics());
    }
    
     /**
     * sp转px
     * 通过系统提供的函数
     */
    public static int sp2pxByTypedValue(int sp) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, sp, getResources().getDisplayMetrics());
    }
}
```

----------


## 2 2D绘图基础
Paint

* setAntiAlias() //设置画笔的锯齿效果
* setColor() //设置画笔的颜色
* setARGB() //设置画笔的ARGB值
* setAlpha() //设置画笔的Alpha值
* setTextSize() //设置字体的尺寸
* setStyle() //设置画笔的风格
* setStrokeWidth() //设置空心边框的宽度

Canvas

* drawPoint(x, y, paint)
* drawLine(startX, startY, endX, endY, paint)
* drawLines(new float[]{startX1, startY1, endX1, endY1, startX2, startY2, endX2, endY2}, paint)
* drawRect(left, top, right, bottom, paint)
* drawRoundRect(left, top, right, bottom, radiusX, radiusY, paint)
* drawCircle(circleX, circleY, radius, paint)
* drawArc(left, top, right, bottom,startAngle, sweepAngle, useCenter, paint) 
Paint.Style.STROKE + useCenter(true) 自己体会形状
Paint.Style.STROKE + useCenter(false) 自己体会形状
Paint.Style.FILL + useCenter(true) 自己体会形状
Paint.Style.FILL + useCenter(false) 自己体会形状
* drawOval(left, top, right, bottom, paint)
* drawText(text, startX, startY, paint)
* drawPosText(text, new float[]{X1, Y1, X2, Y2}, paint)
* drawPath()
Path path = new Path();
path.moveTo(x1, y1);
path.lineTo(x2, y2);
path.lineTo(x3, y3);
path.lineTo(x4, y4);
drawPath(path, paint);


----------


## 3 Android XML绘图
此处不够详细，请参考Android开发艺术探索第6章。传送门- > [第6章 Android的Drawable][1]

### 3.1 Bitmap

### 3.2 Shape

### 3.3 Layer

### 3.4 Selector

----------


## 4 Android绘图技巧
### 4.1 Canvas
* Canvas.save() 保存画布
* Canvas.restore() 合并图层，将save()之后绘制的图像与save()之前的图像进行合并
* Canvas.translate(x, y) 将原点(0,0)移动到(x,y)，之后所有的操作都将以(x,y)为原点执行
* Canvas.rotate() 旋转坐标轴

参考如下一段自定义clock时钟的代码

```
  @Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    init();
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.drawCircle(mCircleX, mCircleY, mCircleRadius, mCirclePaint);
    canvas.drawCircle(mCircleX,mCircleY,mCirclePointRadius,mCirclePointPaint);
    canvas.save();
    for (int i = 0; i < 60; i++) {
        if(i % 5 == 0){
                canvas.drawLine(mCircleX, mCircleY - mCircleRadius, mCircleX, mCircleY - mCircleRadius + mScaleLargeLength, mScaleLargePaint);
            String degree = i == 0 ? String.valueOf(12) : String.valueOf(i/5);
            canvas.drawText(degree,mCircleX-mScaleTextLargePaint.measureText(degree)/2,mCircleY - mCircleRadius + mScaleLargeLength + 50,mScaleTextLargePaint);
        }else{
            canvas.drawLine(mCircleX, mCircleY - mCircleRadius, mCircleX, mCircleY - mCircleRadius + mScaleSmallLength, mScaleSmallPaint);
        }
        canvas.rotate(6,mCircleX,mCircleY);
    }
    canvas.restore();

    canvas.translate(mCircleX,mCircleY);


    mTime.setToNow();
    float secondDegree = mTime.second * 6;
    float minuteDegree = mTime.minute * 6 + secondDegree / 60;
    float hourDegree = mTime.hour * 30 + minuteDegree / 12;
    //hour
    canvas.save();
    canvas.rotate(hourDegree);
    canvas.drawLine(0,0,0,-mHourLength,mHourPaint);
    canvas.restore();
    //minute
    canvas.save();
    canvas.rotate(minuteDegree);
    canvas.drawLine(0,0,0,-mMinuteLength,mMinutePaint);
    canvas.restore();
    //second
    canvas.save();
    canvas.rotate(secondDegree);
    canvas.drawLine(0,0,0,-mSecondLength,mSecondPaint);
    canvas.restore();

    postInvalidateDelayed(1000);
}

private void init() {
    mCirclePaint = new Paint();
    mCirclePaint.setAntiAlias(true);
    mCirclePaint.setStyle(Paint.Style.STROKE);
    mCirclePaint.setStrokeWidth(5);

    mScaleLargePaint = new Paint();
    mScaleLargePaint.setAntiAlias(true);
    mScaleLargePaint.setStrokeCap(Paint.Cap.ROUND);
    mScaleLargePaint.setStrokeWidth(5);

    mScaleSmallPaint = new Paint();
    mScaleSmallPaint.setAntiAlias(true);
    mScaleSmallPaint.setStrokeCap(Paint.Cap.ROUND);
    mScaleSmallPaint.setStrokeWidth(3);

    mScaleTextLargePaint = new Paint();
    mScaleTextLargePaint.setAntiAlias(true);
    mScaleTextLargePaint.setTextSize(45);

    mHourPaint = new Paint();
    mHourPaint.setAntiAlias(true);
    mHourPaint.setStrokeCap(Paint.Cap.ROUND);
    mHourPaint.setStrokeWidth(10);

    mMinutePaint = new Paint();
    mMinutePaint.setAntiAlias(true);
    mMinutePaint.setStrokeCap(Paint.Cap.ROUND);
    mMinutePaint.setStrokeWidth(7);

    mSecondPaint = new Paint();
    mSecondPaint.setAntiAlias(true);
    mSecondPaint.setStrokeCap(Paint.Cap.ROUND);
    mSecondPaint.setStrokeWidth(4);

    mCirclePointPaint = new Paint();
    mCirclePointPaint.setAntiAlias(true);
    mCirclePointPaint.setStyle(Paint.Style.FILL);


    mScreenWidth = getMeasuredWidth();
    mScreenHeight = getMeasuredHeight();

    mCircleX = mScreenWidth / 2;
    mCircleY = mScreenHeight / 2;
    mCircleRadius = (int) (Math.min(mCircleX, mCircleY) * 0.8);
    mCirclePointRadius = mCircleRadius / 2 / 20;

    mScaleLargeLength = mCircleRadius / 15;
    mScaleSmallLength = mCircleRadius / 30;

    mHourLength = mCircleRadius * 2 / 5;
    mMinuteLength = mCircleRadius * 3 / 5;
    mSecondLength = mCircleRadius * 4 / 5;

    mTime = new Time();
}
```

### 4.2 Layer图层
* Layer图层同样是基于栈结构进行管理的。
* 通过调用saveLayer()方法、saveLayerAlpha()方法将一个图层入栈
* 使用restore()方法、restoreToCount()方法将一个图层出栈。
* 入栈的时候，后面所有的操作都发生在这个图层上，而出栈的时候，则会把图像绘制到上层Canvas上。

参考如下一段代码

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    canvas.drawCircle(200, 200, 100, paint);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        canvas.saveLayerAlpha(0, 0, getMeasuredWidth(), getMeasuredHeight(), 127);
        paint.setColor(Color.GREEN);
        canvas.drawCircle(300, 200, 100, paint);
        canvas.saveLayerAlpha(0, 0, getMeasuredWidth(), getMeasuredHeight(), 127);
        paint.setColor(Color.BLUE);
        canvas.drawCircle(250, 300, 100, paint);
    }
}
```

----------


## 5 Android图像处理之色彩特效处理

* 色调——物体传播的颜色
* 饱和度——颜色的纯度，从0（灰）到100%（饱和）来描述
* 亮度——颜色的相对明暗程度

在Android中，系统使用一个**4x5的颜色矩阵——ColorMatrix**（令其为矩阵A），来处理图像的这些色彩效果。
而对于每个像素点，都有一个**5x1的颜色分量矩阵**（令其为矩阵B）用来保存颜色的RGBA值。
```
R = AC
```
* 第一行决定新的颜色值中的R——红色
* 第二行决定新的颜色值中的G——绿色
* 第三行决定新的颜色值中的B——蓝色
* 第四行决定新的颜色值中的A——透明度
* 第五列的ejot值分别决定每个分量中的offset——偏移量

### 5.1 使用系统提供的API来实现颜色效果的处理
```
@Override
public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
    switch (seekBar.getId()){
        case R.id.sb_sedu:
            mHue = (progress - 50) * 1.0f / 50 * 180;//范围-180f到180f度，0f是正常值
            break;
        case R.id.sb_baohedu:
            mSaturation = progress * 1.0f / 50;//范围0f-2f，1f是正常值
            break;
        case R.id.sb_liangdu:
            mLum = progress * 1.0f / 50;//范围0f-2f，1f是正常值
            break;
        }
    mImageView.setImageBitmap(handleImageEffect(mBitmap,mHue,mSaturation,mLum));
}

public static Bitmap handleImageEffect(Bitmap bitmap,float hue, float saturation, float lum){
    Bitmap out = Bitmap.createBitmap(
                            bitmap.getWidth(),bitmap.getHeight(),Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(out);

    ColorMatrix hueMatrix = new ColorMatrix();
    hueMatrix.setRotate(0,hue);
    hueMatrix.setRotate(1,hue);
    hueMatrix.setRotate(2,hue);

    ColorMatrix saturationMatrix = new ColorMatrix();
    saturationMatrix.setSaturation(saturation);

    ColorMatrix lumMatrix = new ColorMatrix();
    lumMatrix.setScale(lum,lum,lum,1);

    ColorMatrix matrix = new ColorMatrix();
    matrix.postConcat(hueMatrix);
    matrix.postConcat(saturationMatrix);
    matrix.postConcat(lumMatrix);

    Paint paint = new Paint();
    paint.setColorFilter(new ColorMatrixColorFilter(matrix));

    canvas.drawBitmap(bitmap,0,0,paint);

    return out;
}
```

### 5.2 修改颜色矩阵的值来实现颜色效果的处理

```
public class Color2Activity extends AppCompatActivity {
    private ImageView mImageView;
    private GridLayout mGridLayout;
    private Button mButtonChange;
    private Button mButtonReset;
    private Bitmap mBitmap;
    private EditText[] mEditTextArray = new EditText[20];
    private float[] mColorArray = new float[20];

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_color2);

        mImageView = (ImageView) findViewById(R.id.iv_photo);
        mGridLayout = (GridLayout) findViewById(R.id.gl_group);
        mButtonChange = (Button) findViewById(R.id.btn_change);
        mButtonReset = (Button) findViewById(R.id.btn_reset);
        
        mButtonChange.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix();
                setMatrix();
            }
        });
        mButtonReset.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                initMatrix();
                getMatrix();
                setMatrix();
            }
        });

        mBitmap = fitBitmap(144, 144);
        mImageView.setImageBitmap(mBitmap);
        
        mGridLayout.post(new Runnable() {
            @Override
            public void run() {
                addEditText();
                initMatrix();
            }
        });
    }

    public Bitmap fitBitmap(int destWidth, int destHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(getResources(), R.drawable.color, options);

        float srcWidth = options.outWidth;
        float srcHeight = options.outHeight;

        int inSampleSize = 1;

        if (srcWidth > destWidth || srcHeight > destHeight) {
            if (srcWidth > srcHeight) {
                inSampleSize = Math.round(srcHeight / destHeight);
            } else {
                inSampleSize = Math.round(srcWidth / destWidth);
            }
        }

        options = new BitmapFactory.Options();
        options.inSampleSize = inSampleSize;
        return BitmapFactory.decodeResource(getResources(), R.drawable.color, options);
    }

    private void addEditText() {
        int width = mGridLayout.getMeasuredWidth() / 5;
        int height = mGridLayout.getMeasuredHeight() / 4;
        EditText editText = null;
        for (int i = 0; i < 20; i++) {
            editText = new EditText(Color2Activity.this);
            mGridLayout.addView(editText, width, height);
            mEditTextArray[i] = editText;
        }
    }

    private void initMatrix() {
        for (int i = 0; i < 20; i++) {
            if (i % 6 == 0) {
                mEditTextArray[i].setText(String.valueOf(1));
            } else {
                mEditTextArray[i].setText(String.valueOf(0));
            }
        }
    }

    private void getMatrix() {
        for (int i = 0; i < 20; i++) {
            mColorArray[i] = Float.valueOf(mEditTextArray[i].getText().toString());
        }
    }

    private void setMatrix() {
        Bitmap out = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        ColorMatrix matrix = new ColorMatrix();
        matrix.set(mColorArray);

        Paint paint = new Paint();
        paint.setColorFilter(new ColorMatrixColorFilter(matrix));
        Canvas canvas = new Canvas(out);
        canvas.drawBitmap(mBitmap, 0, 0, paint);
        mImageView.setImageBitmap(out);
    }
}
```

### 5.3 常用图像颜色矩阵处理效果

```
public class Color3Activity extends AppCompatActivity {
    private Button mButton1;
    private Button mButton2;
    private Button mButton3;
    private Button mButton4;
    private Button mButton5;
    private Button mButton6;
    private ImageView mImageView;
    private float[] mColorArray = new float[20];
    private float[] mColorArray1;
    private float[] mColorArray2;
    private float[] mColorArray3;
    private float[] mColorArray4;
    private float[] mColorArray5;
    private float[] mColorArrayReset;
    private Bitmap mBitmap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_color3);
        
        mImageView = (ImageView) findViewById(R.id.iv_photo);
        mBitmap = fitBitmap(144,144);
        mImageView.setImageBitmap(mBitmap);

        mButton1 = (Button) findViewById(R.id.btn_1);
        mButton2 = (Button) findViewById(R.id.btn_2);
        mButton3 = (Button) findViewById(R.id.btn_3);
        mButton4 = (Button) findViewById(R.id.btn_4);
        mButton5 = (Button) findViewById(R.id.btn_5);
        mButton6 = (Button) findViewById(R.id.btn_6);

        mButton1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(1);
                setMatrix();
            }
        });
        mButton2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(2);
                setMatrix();
            }
        });
        mButton3.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(3);
                setMatrix();
            }
        });
        mButton4.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(4);
                setMatrix();
            }
        });
        mButton5.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(5);
                setMatrix();
            }
        });
        mButton6.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getMatrix(6);
                setMatrix();
            }
        });

        initMatrix();
    }

    public Bitmap fitBitmap(int destWidth, int destHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(getResources(), R.drawable.color, options);

        float srcWidth = options.outWidth;
        float srcHeight = options.outHeight;

        int inSampleSize = 1;

        if (srcWidth > destWidth || srcHeight > destHeight) {
            if (srcWidth > srcHeight) {
                inSampleSize = Math.round(srcHeight / destHeight);
            } else {
                inSampleSize = Math.round(srcWidth / destWidth);
            }
        }

        options = new BitmapFactory.Options();
        options.inSampleSize = inSampleSize;
        return BitmapFactory.decodeResource(getResources(), R.drawable.color, options);
    }

    private void initMatrix() {
        mColorArray1 = new float[]{
                0.33f, 0.59f, 0.11f, 0, 0,
                0.33f, 0.59f, 0.11f, 0, 0,
                0.33f, 0.59f, 0.11f, 0, 0,
                0, 0, 0, 1, 0};//灰度效果
        mColorArray2 = new float[]{
                -1, 0, 0, 1, 1,
                0, -1, 0, 1, 1,
                0, 0, -1, 1, 1,
                0, 0, 0, 1, 0};//颜色反转
        mColorArray3 = new float[]{
                0.393f, 0.769f, 0.189f, 0, 0,
                0.349f, 0.686f, 0.168f, 0, 0,
                0.272f, 0.534f, 0.131f, 0, 0,
                0, 0, 0, 1, 0};//怀旧
        mColorArray4 = new float[]{
                1.5f, 1.5f, 1.5f, 0, -1,
                1.5f, 1.5f, 1.5f, 0, -1,
                1.5f, 1.5f, 1.5f, 0, -1,
                0, 0, 0, 1, 0};//去色
        mColorArray5 = new float[]{
                1.438f, -0.122f, -0.016f, 0, -0.03f,
                -0.062f, 1.378f, -0.016f, 0, 0.05f,
                -0.062f, -0.122f, 1.483f, 0, -0.02f,
                0, 0, 0, 1, 0};//高饱和度
        mColorArrayReset = new float[]{
                1, 0, 0, 0, 0,
                0, 1, 0, 0, 0,
                0, 0, 1, 0, 0,
                0, 0, 0, 1, 0};//原图
    }

    private void getMatrix(int flag) {
        switch (flag) {
            case 1:
                mColorArray = mColorArray1;
                break;
            case 2:
                mColorArray = mColorArray2;
                break;
            case 3:
                mColorArray = mColorArray3;
                break;
            case 4:
                mColorArray = mColorArray4;
                break;
            case 5:
                mColorArray = mColorArray5;
                break;
            case 6:
                mColorArray = mColorArrayReset;
                break;
        }
    }

    private void setMatrix() {
        Bitmap out = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        ColorMatrix matrix = new ColorMatrix();
        matrix.set(mColorArray);

        Paint paint = new Paint();
        paint.setColorFilter(new ColorMatrixColorFilter(matrix));
        Canvas canvas = new Canvas(out);
        canvas.drawBitmap(mBitmap, 0, 0, paint);
        mImageView.setImageBitmap(out);
    }
}
```


### 5.4 像素点分析

mBitmap.getPixels(pixels, offset, stride, x, y, width, height);

* pixels——接收位图颜色的数组
* offset——写入到pixels[]中的第一个像素索引值
* stride——pixels[]的行间距
* x——从位图中读取的第一个像素的x坐标值
* y——从位图中读取的第一个像素的y坐标值
* width——从一行读取的像素宽度
* height——读取的行数

使用步骤
```
mBitmap.getPixels(mOldPx, 0, mWidth, 0, 0, mWidth, mHeight);

int r, g, b, a;
int color;
for (int i = 0; i < mWidth * mHeight; i++) {
    color = mOldPx[i];
    r = Color.red(color);
    g = Color.green(color);
    b = Color.blue(color);
    a = Color.alpha(color);

    算法转换

    mNewPx[i] = Color.argb(a, r, g, b);
 }

Bitmap out = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), mBitmap.Config.ARGB_8888);
out.setPixels(mNewPx, 0, mWidth, 0, 0, mWidth, mHeight);
```
### 5.5 常用图像像素点处理效果

使用示例
```
private void initNewAndOldPx() {
    mWidth = mBitmap.getWidth();
    mHeight = mBitmap.getHeight();
    mOldPx = new int[mWidth * mHeight];
    mNewPx = new int[mWidth * mHeight];
}

@Override
public void onClick(View v) {
    mBitmap.getPixels(mOldPx, 0, mWidth, 0, 0, mWidth, mHeight);
    int r, g, b, a;
    int color;

    switch (v.getId()) {
        case R.id.btn_1://底片效果
            for (int i = 0; i < mWidth * mHeight; i++) {
                color = mOldPx[i];
                r = Color.red(color);
                g = Color.green(color);
                b = Color.blue(color);
                a = Color.alpha(color);
                r = 255 - r;
                g = 255 - g;
                b = 255 - b;
                if (r > 255) {
                    r = 255;
                } else if (r < 0) {
                    r = 0;
                }
                if (g > 255) {
                    g = 255;
                } else if (g < 0) {
                    g = 0;
                }
                if (b > 255) {
                    b = 255;
                } else if (b < 0) {
                    b = 0;
                }
                mNewPx[i] = Color.argb(a, r, g, b);
            }
            break;
        case R.id.btn_2://老照片效果
            for (int i = 0; i < mWidth * mHeight; i++) {
                color = mOldPx[i];
                r = Color.red(color);
                    g = Color.green(color);
                b = Color.blue(color);
                a = Color.alpha(color);
                r = (int) (0.393 * r + 0.769 * g + 0.189 * b);
                g = (int) (0.349 * r + 0.686 * g + 0.168 * b);
                b = (int) (0.272 * r + 0.534 * g + 0.131 * b);
                if (r > 255) {
                    r = 255;
                } else if (r < 0) {
                    r = 0;
                }
                    if (g > 255) {
                    g = 255;
                } else if (g < 0) {
                    g = 0;
                }
                if (b > 255) {
                    b = 255;
                } else if (b < 0) {
                    b = 0;
                }
                mNewPx[i] = Color.argb(a, r, g, b);
                }
            break;
        case R.id.btn_3:
            for (int i = 0; i < mWidth * mHeight; i++) {
                mNewPx[i] = mOldPx[i];
            }
            break;
    }
    Bitmap out = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);
    out.setPixels(mNewPx,0,mWidth,0,0,mWidth,mHeight);
    mImageView.setImageBitmap(out);
}
```

----------


## 6 Android图像处理之图形特效处理

### 6.1 Android变形矩阵——Matrix

3x3的图像变形矩阵A
```
a b c
d e f
g h i
```
3x1的像素分量矩阵C
```
X
Y
1
```
结果矩阵
```
R = AC
```

> 矩阵的四种变换 

* Translate——平移变换

```
1 0 dx
0 1 dy
0 0 1
```

* Rotate——旋转变换

```
cos -sin  0
sin  cos  0
0    0    1
```

* Scale——缩放变换

```
k1 0  0
0  k2 0
0  0  1
```

* Skew——错切变换

```
1  k1 0
k2 1  0
0  0  1
```

> 系统API实现矩阵的四种变换 

* matrix.setRotate()——旋转变换
* matrix.setTranslate()——平移变换
* matrix.setScale()——缩放变换
* matrix.setSkew()——错切变换
* pre()——提供矩阵的前乘
* post()——提供矩阵的后乘

Matrix类的set方法会充值矩阵的所有值，而post和pre方法不会，这两个方法常用来实现矩阵的混合作用

关于pre()、post()的补充说明如下：

旋转45度，再平移到(200, 200)
使用后乘运算
matrix.setRotate(45);
matrix.postTranslate(200, 200);//当前矩阵matrix乘上参数(200, 200)代表的矩阵
使用前乘运算
matrix.setTranslate(200, 200);//参数(200, 200)代表的矩阵乘上当前矩阵matrixx
matrix.preRotate(45);

### 6.2 像素块分析

```
drawBitmapMesh(Bitmap bitmap, int meshWidth, int meshHeight, float[] verts, int vertOffset, int[] colors, int colorOffset, Paint paint)
```

* bitmap：将要扭曲的图像
* meshWidth：需要的横向网格数目
* meshHeight：需要的纵向网格数目
* verts：网格交叉点坐标数组
* vertOffset：verts数组中开始跳过的(x, y)坐标对的数目

在图像上横纵各画N-1条线，将图像分成N块，而这横纵各N块就交织成了N x N各点
每个点的坐标以x,y的形式保存在verts数组中，也就是说verts数组的每两位用来保存一个交织点

参考实现旗帜飘扬的这段代码

```
public class Graphic2View extends View {
    private static final int WIDTH = 200;
    private static final int HEIGHT = 100;
    private float[] orig = new float[(WIDTH + 1) * (HEIGHT + 1) * 2];
    private float[] verts = new float[(WIDTH + 1) * (HEIGHT + 1) * 2];
    private Bitmap mBitmap;
    private float mK;

    public Graphic2View(Context context) {
        super(context);
        init();
    }

    public Graphic2View(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public Graphic2View(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        flagWave();
        canvas.drawBitmapMesh(mBitmap, WIDTH, HEIGHT, verts, 0, null, 0, null);
        mK += 0.1f;
//        postInvalidateDelayed(16);
        invalidate();
    }

    private void init() {
        mBitmap = fitBitmap(144, 144);
        float width = mBitmap.getWidth();
        float height = mBitmap.getHeight();
        for (int i = 0; i <= HEIGHT; i++) {
            float dy = i * height / HEIGHT;
            for (int j = 0; j <= WIDTH; j++) {
                float dx = j * width / WIDTH;
                orig[i * (WIDTH + 1) * 2 + j * 2 + 0] = verts[i * (WIDTH + 1) * 2 + j * 2 + 0] = dx;
                orig[i * (WIDTH + 1) * 2 + j * 2 + 1] = verts[i * (WIDTH + 1) * 2 + j * 2 + 1] = dy + 100;
            }
        }
    }

    private void flagWave() {
        for (int i = 0; i <= HEIGHT; i++) {
            for (int j = 0; j <= WIDTH; j++) {
                verts[i * (WIDTH + 1) * 2 + j * 2 + 0] += 0;
                float offset = (float) Math.sin((float)j / WIDTH * 2 * Math.PI + mK * Math.PI);
                verts[i * (WIDTH + 1) * 2 + j * 2 + 1] = orig[i * (WIDTH + 1) * 2 + j * 2 + 1] + offset * 30;
            }
        }
    }

    public Bitmap fitBitmap(int destWidth, int destHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(getResources(), R.drawable.color, options);

        float srcWidth = options.outWidth;
        float srcHeight = options.outHeight;

        int inSampleSize = 1;

        if (srcWidth > destWidth || srcHeight > destHeight) {
            if (srcWidth > srcHeight) {
                inSampleSize = Math.round(srcHeight / destHeight);
            } else {
                inSampleSize = Math.round(srcWidth / destWidth);
            }
        }

        options = new BitmapFactory.Options();
        options.inSampleSize = inSampleSize;
        return BitmapFactory.decodeResource(getResources(), R.drawable.color, options);
    }

}
```
----------


## 7 Android图像处理之画笔特效处理
### 7.1 PorterDuffXfermode

控制两个图像间的混合显示模式
PorterDuffXfermode设置的是两个图层交集区域的显示方式
dst是先画的图形，src是后画的图形
使用一张图片作为另一张图片的遮罩层，通过控制遮罩层的图形，来控制下面被遮罩图形的显示效果

共有16种PorterDuffXfermode，如下所示
CLear、Src、Dst
SrcOver、DstOver
SrcIn、DstIn
SrcOut、DstOut
SrcATop、DstATop
Xor、Darken、Lighten、Multiply、Screen

参考如下代码，实现圆角图片
```
mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.color);//一般来说应该加载缩放后的bitmap
Bitmap out = mBitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);

Canvas canvas = new Canvas(out);
Paint paint = new Paint();
paint.setAntiAlias(true);
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    canvas.drawRoundRect(0, 0, out.getWidth(), out.getHeight(), 100, 100, paint);
    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
    canvas.drawBitmap(mBitmap,0,0,paint);
}
mImageViewPhoto.setImageBitmap(out);
```

参考如下代码，实现刮刮卡效果
```
private void init(){
    mPaint = new Paint();
    mPaint.setAlpha(0);
    mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));//此处SRC_IN也可以？
    mPaint.setAntiAlias(true);
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeJoin(Paint.Join.ROUND);
    mPaint.setStrokeWidth(50);
    mPaint.setStrokeCap(Paint.Cap.ROUND);

    mPath = new Path();

    mBgBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.color);
    mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(),mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
    mCanvas = new Canvas(mFgBitmap);
    mCanvas.drawColor(Color.GRAY);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch(event.getAction()){
        case MotionEvent.ACTION_DOWN:
            mPath.reset();
            mPath.moveTo(event.getX(),event.getY());
            break;
        case MotionEvent.ACTION_MOVE:
            mPath.lineTo(event.getX(),event.getY());
            break;
        }
    mCanvas.drawPath(mPath,mPaint);
    invalidate();
    return true;
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.drawBitmap(mBgBitmap,0,0,null);
    canvas.drawBitmap(mFgBitmap,0,0,null);
}

public void reset(){
    mCanvas = new Canvas(mFgBitmap);
    mCanvas.drawColor(Color.GRAY);
    invalidate();
}
```

### 7.2 Shader

Shader又被称为着色器、渲染器，它用来实现一系列的渐变、渲染效果。

* BitmapShader——位图Shader
CLAMP拉伸——图片最后的那一个像素，不断重复
REPEAT重复——横向、纵向不断重复
MIRROR镜像——横向不断翻转重复，纵向不断翻转重复

参考如下代码，实现圆形图片
```
mPaint = new Paint();
mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.color);
BitmapShader shader = new BitmapShader(mBitmap, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT);
mPaint.setShader(shader);
canvas.drawCircle(500,250,250,mPaint);
```

参考如下代码，实现图片倒影效果
```
private Bitmap mSrcBitmap;
private Bitmap mRefBitmap;
private Paint mPaint;
private PorterDuffXfermode mXfermode;

private void initRes(Context context) {
    mSrcBitmap = fitBitmap(144,144);

    Matrix matrix = new Matrix();
    matrix.setScale(1f, -1f);
    mRefBitmap = Bitmap.createBitmap(mSrcBitmap,0,0,mSrcBitmap.getWidth(),mSrcBitmap.getHeight(),matrix,true);

    mPaint = new Paint();
    mPaint.setShader(new LinearGradient(0, mSrcBitmap.getHeight(),
                0, mSrcBitmap.getHeight()+mSrcBitmap.getHeight()/4,
                0xDD000000, 0x10000000, Shader.TileMode.CLAMP));

    mXfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.drawColor(Color.BLACK);
    canvas.drawBitmap(mSrcBitmap,0,0,null);
    canvas.drawBitmap(mRefBitmap,0,mSrcBitmap.getHeight(),null);
    mPaint.setXfermode(mXfermode);
    canvas.drawRect(0,mSrcBitmap.getHeight(),mSrcBitmap.getWidth(),mSrcBitmap.getHeight() * 2,mPaint);
    mPaint.setXfermode(null);
}
```

* LinearGradient——线性Shader
* RadialGradient——光束Shader
* SweepGradient——梯度Shader
* ComposeShader——混合Shader

参考如下代码
```
mPaint = new Paint();
LinearGradient shader = new LinearGradient(250, 1100, 750, 1500,  Color.BLUE,Color.YELLOW, Shader.TileMode.CLAMP);
mPaint.setShader(shader);
canvas.drawRect(250,1100,750,1500,mPaint);
```

### 7.3 PathEffect

用各种笔触效果来绘制一个路径。

* CornerPathEffect：将拐角处变得圆滑
* DiscretePathEffect：线段上会产生许多杂点
* DashPathEffect：
可以用来绘制虚线用一个数组来设置各个点之间的间隔，此后绘制虚线就重复这样的间隔进行绘制；
另一个参数phase用来控制数组的一个偏移量，通常可以用来设置值来实现路径的动态效果。
* PathDashPathEffect：与DashPathEffect类似，可以设置显示点的图形，即方形点的虚线、圆形点的虚线
* ComposePathEffect：组合任意两种PathEffect

参考如下一段代码，实现各种效果的路径
```
private PathEffect[] mEffects;
private Path mPath;
private Paint mPaint;

@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    mEffects = new PathEffect[6];

    mEffects[0] = null;
    mEffects[1] = new CornerPathEffect(5f);
    mEffects[2] = new DiscretePathEffect(3.0f,5.0f);
    mEffects[3] = new DashPathEffect(new float[]{20,10,5,10},0);
    Path path = new Path();
    path.addRect(0,0,8,8, Path.Direction.CCW);
    mEffects[4] = new PathDashPathEffect(path,12,0, PathDashPathEffect.Style.ROTATE);
    mEffects[5] = new ComposePathEffect(mEffects[3],mEffects[1]);

    mPath = new Path();
    mPath.moveTo(0,0);
    for (int i = 0; i < 30; i++) {
        mPath.lineTo(i * 35, (float)(Math.random() * 100));
    }

    mPaint = new Paint();
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(5);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.translate(0,200);
    for (int i = 0; i < mEffects.length; i++) {
        mPaint.setPathEffect(mEffects[i]);
        canvas.drawPath(mPath,mPaint);
        canvas.translate(0,200);
    }

}
```

----------


## 8 View之孪生兄弟——SurfaceView
### 8.1 SurfaceView与View的区别

* View 适用于主动更新，SurfaceView 适用于被动更新，例如频繁地刷新
* View 在主线程对画面进行刷新，SurfaceView 通常会通过一个子线程来进行页面的刷新
* View 在绘图时没有使用双缓冲机制，SurfaceView 在底层实现机智钟就已经实现了双缓冲机制

一句话总结：自定义View需要频繁刷新，或者刷新时数据处理量比较大，就考虑使用SurfaceView 。

### 8.2 SurfaceView的使用

SurfaceView 在使用时，有一套使用的模板代码，大部分都可以套用该模板进行编写。

* 创建 SurfaceView
创建自定义的SurfaceView继承自SurfaceView，并实现两个接口——SurfaceHolder.Callback和Runnable
```
public class SurfaceViewTemplate extends SurfaceView implements SurfaceHolder.Callback, Runnable{
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }
    
    @Override
    public void run() {
        while (mIsDrawing) {

        }
    }

}
```

* 初始化 SurfaceView
初始化一个SurfaceHolder对象，并注册SurfaceHolder的回调方法
```
//SurfaceHolder
private SurfaceHolder mSurfaceHolder;
// 用于绘图的Canvas
private Canvas mCanvas;
// 子线程标志位
private boolean mIsDrawing;

mSurfaceHolder = getHolder();
mSurfaceHolder.addCallback(this);
```

* 使用 SurfaceView
通过SurfaceHolder对象的lockCanvas()方法，可以获得当前的Canvas绘图对象
drawColor()方法来进行清屏操作
unlockCanvasAndPost(mCanvas)方法对画布内容进行提交

模板代码如下，可以满足大部分的 SurfaceView 绘图需求
```
public class SurfaceViewTemplate extends SurfaceView implements SurfaceHolder.Callback, Runnable {
    private SurfaceHolder mSurfaceHolder;
    private Canvas mCanvas;
    private boolean mIsDrawing;
    
    //extra define

    public SurfaceViewTemplate(Context context) {
        super(context);
        initView();
    }

    public SurfaceViewTemplate(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public SurfaceViewTemplate(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mSurfaceHolder = getHolder();
        mSurfaceHolder.addCallback(this);
        setFocusable(true);
        setFovusableInTouchMode(true);
        this.setKeepScreenOn(true);
        //mHolder.setFormat(PixelFormat.OPAQUE);
        
        //extra init
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
            // extra logic 
        }
    }

    private void draw() {
        try {
            mCanvas = mSurfaceHolder.lockCanvas();
            // draw something
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (mCanvas != null) {
                mSurfaceHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}
```

### 8.3 SurfaceView实例

#### 8.3.1 正弦曲线

以下代码基于模板代码
```
private Path mPath;
private Paint mPaint;

private int mX;
private int mY;
    
@Override
public void run() {
    while (mIsDrawing) {
        draw();
        mX += 1;
        mY = 400 + (int) (Math.sin(mX * 2 * Math.PI / 180) * 200);
        mPath.lineTo(mX, mY);
    }
}

private void initView() {
    mSurfaceHolder = getHolder();
    mSurfaceHolder.addCallback(this);

    mIsDrawing = false;
    mPath = new Path();
    mPath.moveTo(0, 0);

    mPaint = new Paint();
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(5);
}

private void draw() {
    try {
        mCanvas = mSurfaceHolder.lockCanvas();
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath, mPaint);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (mCanvas != null) {
            mSurfaceHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

#### 8.3.2 绘图板

以下代码基于模板代码
```
private Path mPath;
private Paint mPaint;

 private void initView() {
    mSurfaceHolder = getHolder();
    mSurfaceHolder.addCallback(this);

    mIsDrawing = false;

    mPath = new Path();

    mPaint = new Paint();
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(20);
}

private void draw() {
    try {
        mCanvas = mSurfaceHolder.lockCanvas();
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath, mPaint);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (mCanvas != null) {
            mSurfaceHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()){
        case MotionEvent.ACTION_DOWN:
            mPath.moveTo(event.getX(),event.getY());
            break;
        case MotionEvent.ACTION_MOVE:
            mPath.lineTo(event.getX(),event.getY());
            break;
        }
    return true;
}

public void reset(){
    mPath.reset();
}
```


  [1]: https://github.com/LuckyTerry/ReadingNotes/blob/master/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2/%E7%AC%AC6%E7%AB%A0%20Android%E7%9A%84Drawable.md