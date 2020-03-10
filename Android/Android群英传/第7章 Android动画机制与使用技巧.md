# 第7章 Android动画机制与使用技巧

[TOC]

----------


## 1、View动画

> * 效率比较高，使用方便
> * 不具备交互性，只能做普通动画效果

### 1.1 代码中使用View动画

#### 透明度动画

```
//构造函数
AlphaAnimation(Context context, AttributeSet attrs)
AlphaAnimation(float fromAlpha, float toAlpha)

//讲解 略

//示例
AlphaAnimation aa = new AlphaAnimation(0,1);
aa.setDuration(1000);
view.startAnimation(aa);
```

#### 旋转动画

```
//构造函数
RotateAnimation(Context context, AttributeSet attrs)
RotateAnimation(float fromDegrees, float toDegrees)
RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY)
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)

//讲解
pivotXType有如下三种：
ABSOLUTE
RELATIVE_TO_PARENT
RELATIVE_TO_SELF

//示例
RotateAnimation ra = new RotateAnimation(0,360,100,100);
ra.setDuration(1000);
view.startAnimation(ra);
    
RotateAnimation ra = new RotateAnimation(0,360,Animation.RELATIVE_TO_SELF,0.5F,
                                                Animation.RELATIVE_TO_SELF,0.5F);
ra.setDuration(1000);
view.startAnimation(ra);
```

#### 位移动画

```
//构造函数
TranslateAnimation(Context context, AttributeSet attrs)
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue)

//讲解
pivotXType有如下三种：
ABSOLUTE
RELATIVE_TO_PARENT
RELATIVE_TO_SELF

//示例
TranslateAnimation ta = new TranslateAnimation(0f,200f,0f,300f);
ta.setDuration(1000);
view.startAnimation(ta);

TranslateAnimation ta = new TranslateAnimation(RELATIVE_TO_SELF, 0f, RELATIVE_TO_SELF, 1f,
                                                    RELATIVE_TO_SELF, 0f, RELATIVE_TO_SELF, 1f);
ta.setDuration(1000);
view.startAnimation(ta);
```

#### 缩放动画

```
//构造函数
ScaleAnimation(Context context, AttributeSet attrs)
ScaleAnimation(float fromX, float toX, float fromY, float toY)
ScaleAnimation(float fromX, float toX, float fromY, float toY, float pivotX, float pivotY)
ScaleAnimation(float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)

//讲解
pivotXType有如下三种：
ABSOLUTE
RELATIVE_TO_PARENT
RELATIVE_TO_SELF

//示例
ScaleAnimation sa = new ScaleAnimation(0,2,0,2);
sa.setDuration(1000);
view.startAnimation(sa);

ScaleAnimation sa = new ScaleAnimation(0,2,0,2,Animation.RELATIVE_TO_SELF,0.5F,
                                                Animation.RELATIVE_TO_SELF,0.5F);
sa.setDuration(1000);
view.startAnimation(sa);
```

#### 动画合集

```
//构造函数
AnimationSet(Context context, AttributeSet attrs)
AnimationSet(boolean shareInterpolator)

//讲解
shareInterpolator指：各个Animation是否共享一个Interpolator

//示例
AnimationSet as = new AnimationSet(true);
AlphaAnimation aa = new AlphaAnimation(0,1);
TranslateAnimation ta = new TranslateAnimation(0,100,0,200);
as.addAnimation(aa);
as.addAnimation(ta);
as.setDuration(1000);
view.startAnimation(as);
```

#### 监听回调

```
animation.setAnimationListener(new Animation.AnimationListener(){
    @Override
    public void onAnimationStart(Animation animation){
    }
    @Override
    public void onAnimationEnd(Animation animation){
    }
    @Override
    public void onAnimationRepeat(Animation animation){
    }
});
```

#### Animation类常用API

```
android:duration	    setDuration(long)	
android:fillAfter	    setFillAfter(boolean)
android:fillBefore	    setFillBefore(boolean)	
android:interpolator	setInterpolator(Interpolator)
android:repeatCount	    setRepeatCount(int)	 INFINITE
android:repeatMode	    setRepeatMode(int)	 RESTART REVERSE
android:startOffset	    setStartOffset(long)
```

### 1.2 xml中使用View动画

* anim文件夹下存放 tween animation 和 frame animation ；
* xml文件里只有 scale、rotate、translate、alpha、set 五个标签；
* 在代码中使用AnimationUtils.loadAnimation()方法加载；
* 使用mView.setAnimation(mAnimation)为mView加载动画；
* 使用mView.startAnimation()开启动画；

```
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android">  
    <translate  
        android:fromXDelta="0"      // 设置动画开始时 x 坐标的位置  
        android:toXDelta="-100%p"   // 设置动画结束时 x 坐标的位置  
        android:duration="300" />   // 设置动画持续的时间 300 毫秒  
  
    <alpha  
        android:fromAlpha="1.0"     // 设置动画开始时的透明度   1.0 代表不透明  
        android:toAlpha="0.0"       // 设置动画开始时的透明度   0.0 表示完全透明  
        android:duration="300" />   // 设置动画持续的时间 300 毫秒  
  
    <scale  
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"//设置动画出入器  
        android:fromXScale="0.0"        // 设置动画开始时 x 坐标上的伸缩长度  
        android:toXScale="1.4"          // 设置动画结束时 x 坐标上的伸缩长度  
        android:fromYScale="0.0"        // 设置动画开始时 y 坐标上的伸缩长度  
        android:toYScale="1.4"          // 设置动画开始时 y 坐标上的伸缩长度  
        android:pivotX="50%"            // 设置动画相对于控件的 x 坐标的位置  
        android:pivotY="50%"            // 设置动画相对于控件的 y 坐标的位置  
        android:fillAfter="false"       // 该动画转化在动画结束前开始应用  
        android:duration="700" />       // 设置动画持续的时间  
    
    <rotate  
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"//设置动画出入器
        android:fromDegrees="0"         // 设置动画开始时的角度  
        android:toDegrees="+350"        // 设置动画结束时的旋转角度  
        android:pivotX="50%"            // 设置动画相对于控件的 x 坐标的位置  
        android:pivotY="50%"            // 设置动画相对于控件的 y 坐标的位置  
        android:duration="3000" />      // 设置动画持续的时间  
      
</set>  

Animation animation = AnimationUtils.loadAnimation(mContext,R.anim.tween_animation);
mView.startAnimation(animation);
```

----------


## 2 、属性动画

> * 功能强大，具有交互性
> * 比View动画复杂一点

### 2.1 代码中使用属性动画

#### ObjectAnimator

```
//常用的构造函数，注意float... values
public static ObjectAnimator ofFloat (Object target, String propertyName, float... values)
```

* translationX、translationY

```
ObjectAnimator.ofFloat(mButton,"translationX",300f).setDuration(3000).start();//移动到300
ObjectAnimator.ofFloat(mButton,"translationY",100f,300f).setDuration(3000).start();//从100移动到300
```

* rotation、ratationX、ratationY

```
ObjectAnimator.ofFloat(mButton,"rotation",180f).setDuration(3000).start();//绕Z轴旋转到180
ObjectAnimator.ofFloat(mButton,"rotationX",90f,180f).setDuration(3000).start();//绕X轴从90到180
```

* scaleX、scaleY

```
ObjectAnimator.ofFloat(mButton,"scaleX",1f,0.5f).setDuration(3000).start();//X方向从1缩小到0.5
```
    
* pivotX、pivotY
    
```
有这个属性吗？，没找到呀。想要指定中心点执行动画怎么办？
```

* x、y

```
ObjectAnimator.ofFloat(mButton, "x", 100f).setDuration(3000).start();
ObjectAnimator.ofFloat(mButton, "y", 0f, 100f).setDuration(3000).start();
```

* alpha

```
ObjectAnimator.ofFloat(mButton, "alpha", 0f, 1f).setDuration(1000).start();//0完全透明
```

定义getter、setter

```
private static class WrapperView{
    private View mTarget;
    public WrapperView(View target){
        mTarget = target;
    }
    public int getWidth(){
        return mTarget.getLayoutParams().width;
    }
    public void setWidth(int width){
        mTarget.getLayoutParams().width = width;
        mTarget.requestLayout();
    }
}
```

使用自定义属性

```
WrapperView wrapper = new WrapperView(mButton);
ObjectAnimator.ofInt(wrapper,"width",500).setDuration(5000).start();
```

#### ValueAnimator

基本使用

```
ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
animation.setDuration(1000);
animation.start();
```

自定义估值器使用

```
ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(), 
                                                    startPropertyValue, endPropertyValue);
animation.setDuration(1000);
animation.start();
```

#### PropertyValuesHolder

```
PropertyValuesHolder pvh1 = PropertyValuesHolder.ofFloat("translationX", 300f);
PropertyValuesHolder pvh2 = PropertyValuesHolder.ofFloat("scaleX", 1f, 0f, 1f);
PropertyValuesHolder pvh3 = PropertyValuesHolder.ofFloat("scaleY", 1f, 0f, 1f);
ObjectAnimator.ofPropertyValuesHolder(mButton, pvh1, pvh2, pvh3).setDuration(3000).start();
```

#### AnimatorSet

* playTogether() 、play() 、with() 、before() 、after()

```
ObjectAnimator animator1 = ObjectAnimator.ofFloat(view, "translationX", 300f);
ObjectAnimator animator2 = ObjectAnimator.ofFloat(view, "scaleX", 1f, 0f, 1f);
ObjectAnimator animator3 = ObjectAnimator.ofFloat(view, "scaleY", 1f, 0f, 1f);
AnimatorSet set = new AnimatorSet();
set.playTogether(animator1, animator2, animator3);
set.start();
```

#### 动画事件的监听

```
//AnimatorListener是abstract的Class，需全部Override
animator.addListener(new AnimatorListener(){
    @Override
    public void onAnimationStart(Animator animation){
    }
    @Override
    public void onAnimationRepeat(Animator animation){
    }
    @Override
    public void onAnimationEnd(Animator animation){
    }
    @Override
    public void onAnimationCancel(Animator animation){
    }
});

//AnimatorListenerAdapter不是abstract的Class，可选择Override
animator.addListener(new AnimatorListenerAdapter(){
    @Override
    public void onAnimationStart(Animator animation){
    }
    @Override
    public void onAnimationRepeat(Animator animation){
    }
    @Override
    public void onAnimationEnd(Animator animation){
    }
    @Override
    public void onAnimationCancel(Animator animation){
    }
});

//ValueAnimator独有,每一帧触发一次，10ms一帧
valueAnimator.addListener(new AnimatorUpdateListener(){
    @Override
    public void onAnimationUpdate(ValueAnimator animation){
    }
});
```

### 2.2 在XML中使用属性动画

* animator文件夹下存放property animation，即属性动画，xml文件里有animator、objectAnimator和set三个标签；
* 在代码中使用AnimatorInflater.loadAnimator()方法加载动画；
* 使用mAnimation.setTarget(mView)为mView加载动画。
* 使用mAnimation.start()开启动画

示例如下

```
<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
    <animator
        android:duration="500"
        android:valueFrom="0f"
        android:valueTo="100f"/>
</set>


AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(mContext,R.anim.property_animator);
set.setTarget(myObject);
set.start();
```

### 2.3 View的animate方法，即 ViewPropertyAnimator 

```
mButton.animate().x(50f).y(100f).alpha(0).setDuration(3000)
                    .withStartAction(new Runnable(){
                        @Override
                        public void run(){
                        }
                    })
                    .withEndAction(new Runnable(){
                        @Override
                        public void run(){
                        }
                    }).start();
```

------

## 3、布局动画

基本方法，使用android:animateLayoutChanges="true"来打开布局动画

```
<LinearLayout
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:id="@+id/verticalContainer"
    android:animateLayoutChanges="true" />
```

自定义布局动画

* ORDER_NORMAL---顺序
* ORDER_RANDOM---随机
* ORDER_REVERSE---返序

```
ScaleAnimation sa = new ScaleAnimation(0, 1, 0, 1);
sa.setDuration(3000);
LayoutAnimationController lac = new LayoutAnimationController(sa,0.5f);
lac.setOrder(LayoutAnimationController.ORDER_NORMAL);
mViewGroup.setLayoutAnimation(lac);
```

------

## 4、插值器Interpolators

* AccelerateDecelerateInterpolator
* AccelerateInterpolator
* AnticipateInterpolator
* AnticipateOvershootInterpolator
* BaseInterpolator
* BounceInterpolator
* CycleInterpolator
* DecelerateInterpolator
* FastOutLinearInInterpolator
* FastOutSlowInInterpolator
* LinearInterpolator
* LinearOutSlowInInterpolator
* OvershootInterpolator
* PathInterpolator

```
animator.setInterpolator(new AccelerateDecelerateInterpolator());
animation.setInterpolator(new AccelerateDecelerateInterpolator());//View动画也有插值器
```

------

## 5、自定义动画
==

模板代码如下：initialize做一些初始化工作，applyTransformation实现动画的具体逻辑

```
class MyAnimation extends Animation{  
    int mCenterX; 
    int mCenterY;  
    Camera camera = new Camera();  
    public ViewAnimation(){  
    }  
    @Override 
    public void initialize(int width, int height, int parentWidth, int parentHeight){  
        super.initialize(width, height, parentWidth, parentHeight);  
        mCenterX = width / 2;   
        mCenterY = height / 2;  
        setDuration(2500);  
       // setRepeatCount(100);
        setInterpolator(new LinearInterpolator());  
    }  
    @Override 
    protected void applyTransformation(float interpolatedTime, Transformation t){
        final Matrix matrix = t.getMatrix();  
        camera.save();  
        camera.translate(0.0f, 0.0f, (1300 - 1300.0f * interpolatedTime));  
        camera.rotateY(360 * interpolatedTime);  
        camera.getMatrix(matrix);  
        camera.restore();  
        matrix.preTranslate(-mCenterX, -mCenterY);  
        matrix.postTranslate(mCenterX, mCenterY);  
    }
}
```

------

## 6、SVG矢量动画

指令

* M X,Y                               将画笔移动到指定的坐标位置
* L X,Y                               画直线到指定的坐标位置
* H X                                 画水平线到指定的X坐标位置
* V Y                                 画垂直线到指定的Y坐标位置
* C X1,Y1,X2,Y2,ENDX,ENDY             三次贝赛曲线
* S X2,Y2,ENDX,ENDY                   三次贝赛曲线 smooth
* Q X,Y,ENDX,ENDY                     二次贝赛曲线
* T ENDX,ENDY                         映射前面路径后的终点
* A RX,RY,XROTATION,FLAG1,FLAG2,X,Y   弧线
RX RY 指所在椭圆的半轴大小
XROTATION 指椭圆的X轴与水平方向顺时针夹角
FLAG1有两个值，1表示大角度弧线，0表示小角度弧线  
FLAG2有两个值，确定从起点到终点的方向，1为顺时针，0为逆时针
X Y 为终点坐标
* Z                                   关闭路径
* 坐标轴以(0,0)为中心，X轴水平向右，Y轴竖直向下
大写绝对定位，参考全局坐标系
小写相对定位，参考父容器坐标系


实现原理

* VectorDrawable 对应vector标签。创建一个静态SVG图形
* ObjectAnimator 对应objectAnimator标签。创建静态SVG图形需要的动画
* AnimatedVectorDrawable 对应animated-vector标签。谷歌工程师把AnimatedVectorDrawable比喻为一个胶水，通过AnimatedVectorDrawable来连接静态的VectorDrawable和动态的objectAnimator
    
标准模板如下所示，animated-vector和vector放drawable目录下，objectAnimator放animator目录下。

```
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="">
    <target
        android:animation=""
        android:name=""/>
    <target
        android:animation=""
        android:name=""/>
</animated-vector>
        
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width=""            //SVG图形的具体宽
    android:height=""           //SVG图形的具体高
    android:viewportWidth=""    //SVG图形划分的比例宽
    android:viewportHeight="" > //SVG图形划分的比例高
    <group
        android:name=""
        android:pivotX=""
        android:pivotY=""
        android:rotation="" >
        <path
            android:name=""
            android:fillColor=""
            android:strokeColor=""=""
            android:strokeWidth=""
            android:strokeLineCap="round"
            android:strokeAlpha=""
            android:pathData="" />
    </group>
</vector>

<objectAnimator
    android:duration=""
    android:propertyName="pathData|translationX|translationY|"
    android:valueType="pathType|intType|floatType"
    android:valueFrom=""
    android:valueTo=""
    android:interpolator="" />
```

### 6.1 SVG动画实例之线图动画

#### 6.3.1. 定义动画    
    
line_anim_vector.xml    
    
```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/line_vector">

    <target
        android:animation="@animator/line_anim_1"
        android:name="path1"/>

    <target
        android:animation="@animator/line_anim_2"
        android:name="path2"/>

</animated-vector>
```

line_vector.xml

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">
    
   <group>
        <path
            android:name="path1"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:strokeColor="@android:color/holo_green_dark"
            android:pathData="
            M 20,20
            L 50,20 80,20"/>
        <path
            android:name="path2"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:strokeColor="@android:color/holo_green_dark"
            android:pathData="
            M 20,80
            L 50,80 80,80"/>

    </group>
</vector>
```

line_anim_1.xml

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
                android:propertyName="pathData"
                android:valueType="pathType"
                android:duration="500"
                android:interpolator="@android:interpolator/accelerate_decelerate"
                android:valueFrom="
                M 20,20
                L 50,20 80,20"
                android:valueTo="
                M 20,20
                L 50,50 80,20">

</objectAnimator>
```

line_anim_2.xml

``` 
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
                android:propertyName="pathData"
                android:valueType="pathType"
                android:duration="500"
                android:interpolator="@android:interpolator/accelerate_decelerate"
                android:valueFrom="
                M 20,80
                L 50,80 80,80"
                android:valueTo="
                M 20,80
                L 50,50 80,80">

</objectAnimator>
```

#### 6.3.2. 使用动画

```
<ImageView
    android:src="@drawable/line_anim_vector"
    android:id="@+id/iv_photo1"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"/>
    
Drawable drawable= mImageView1.getDrawable();
if(drawable instanceof Animatable){
    ((Animatable)drawable).start();
}
```

### 6.2 SVG动画实例之模拟三球仪

#### 6.3.1. 定义动画

planet_anim_vector.xml

```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/planet_vector">

    <target
        android:name="sun"
        android:animation="@animator/planet_anim"/>
    <target
        android:name="earth"
        android:animation="@animator/planet_anim"/>
    <target
        android:name="moon"
        android:animation="@animator/planet_anim"/>
</animated-vector>
```

planet_vector.xml
    
```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="200dp"
        android:height="200dp"
        android:viewportHeight="100"
        android:viewportWidth="100">
    <group
        android:name="sun"
        android:pivotX="50"
        android:pivotY="50"
        android:rotation="0">
        <path
            android:fillColor="@android:color/holo_blue_light"
            android:pathData="
            M 40,50
            a 10,10 0 1,0 20,0
            a 10,10 0 1,0 -20,0" />
        <group
            android:name="earth"
            android:pivotX="65"
            android:pivotY="50"
            android:rotation="0">
            <path
                android:fillColor="@android:color/holo_orange_dark"
                android:pathData="
                M 60,50
                a 5,5 0 1,0 10,0
                a 5,5 0 1,0 -10,0"
                />
                <group
                    android:name="moon"
                    android:pivotX="75"
                    android:pivotY="50"
                    android:rotation="0">
                    <path
                        android:fillColor="@android:color/holo_green_dark"
                        android:pathData="
                        M 72,50
                        a 3,3 0 1,0 6,0
                        a 3,3 0 1,0 -6,0" />
            </group>
        </group>
    </group>
</vector>
```

planet_anim.xml    

```    
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360"
    android:duration="4000">
</objectAnimator>
```

#### 6.3.2. 使用动画    
    
```
<ImageView
    android:src="@drawable/planet_anim_vector"
    android:id="@+id/iv_photo2"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"/>

Drawable drawable= mImageView2.getDrawable();
if(drawable instanceof Animatable){
    ((Animatable)drawable).start();
}
```

### 6.3 SVG动画实例之轨迹动画

#### 6.3.1. 定义动画

search_bar_anim_vector.xml

```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"                          android:drawable="@drawable/search_bar_vector">
    <target
        android:animation="@animator/search_bar_anim"
        android:name="search"/>
</animated-vector>
```

search_bar_vector.xml

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="40dp"
    android:viewportWidth="100"
    android:viewportHeight="100">
    <path
        android:name="bar"
        android:strokeAlpha="0.8"
        android:strokeWidth="2"
        android:strokeLineCap="round"
        android:strokeColor="#FF3570BE"
        android:pathData="
        M 10,80
        L 90,80" />
    <path
        android:name="search"
        android:strokeAlpha="0.8"
        android:strokeWidth="2"
        android:strokeLineCap="round"
        android:strokeColor="#FF3570BE"
        android:pathData="
        M 85,55
        a 5,25 0 1,1 0.1,-0.5
        L 90,80" />
</vector>
```

search_bar_anim.xml

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:propertyName="trimPathStart"
    android:valueType="floatType"
    android:valueFrom="0"
    android:valueTo="1"
    android:interpolator="@android:interpolator/accelerate_decelerate">

</objectAnimator>
```

#### 6.3.2. 使用动画

```
<ImageView
    android:src="@drawable/search_bar_anim_vector"
    android:id="@+id/iv_photo3"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"/>

Drawable drawable= mImageView3.getDrawable();
if(drawable instanceof Animatable){
    ((Animatable)drawable).start();
}
```

------

## 7、动画特效
==

暂时略
