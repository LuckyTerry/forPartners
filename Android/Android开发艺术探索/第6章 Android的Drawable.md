# 第6章 Android的Drawable

[TOC]

这一节容可以参考 guide/topics/resources/drawable-resource的 相关内容

## 1 Drawable简介

Drawable表示一种图像的概念，常被用来作为View的背景使用
Drawable可以没有内部宽高，比如颜色形成的Drawable
Drawable若有内部宽高，可以通过getIntrinsicWidth和getIntrinsicHeight获得
Drawable内部宽高不等同于它的大小
Drawable没有大小概念，当用作View背景时，会被拉伸至View的同等大小

## 2 Drawable的分类

### 2.1 BitmapDrawable

```
android:src 图片的资源id
android:antialias 是否开启图片的抗锯齿功能，默认false，应该开启
android:dither 是否开启抖动效果，默认true，应该开启
android:filter 是否开启过滤效果，默认true，应该开启
android:gravity 对图片定位，默认fill，其他常用left,right,top,bottom,center_vertical,center_horizontal,center,fill_vertical,fill_horizontal,fill，可使用'|'组合
android:mipMap 是否开启纹理，默认false，不常用
android:tileMode 平铺模式，默认disabled，其他常用clamp,repeat,mirror
```

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher"
    android:antialias="true"
    android:dither="true"
    android:filter="true"
    android:gravity="fill"
    android:mipMap="false"
    android:tileMode="disabled"/>
```
### 2.2 NinePatchDrawable

属性值同BitmapDrawable，实际使用中发现，BitmapDrawable也可以代表.9格式的图片

```
<?xml version="1.0" encoding="utf-8"?>
<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher"
    android:antialias="true"
    android:dither="true"
    android:filter="true"
    android:gravity="fill"
    android:mipMap="false"
    android:tileMode="disabled"/>
```

### 2.3 GradientDrawable

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```

```
<corners> 只适用于android:shape="rectangle"

<gradient> 与 <solid> 相对立，前者表示渐变填充，后者表示纯色填充
 
<padding> 表示的不是shape的空白，而是包含它的View的空白
 
<size> 固有宽高，没什么意义
 
<stroke> shape的描边


当android:shape="line"或android:shape="ring"时，必须指定 <stroke>

当android:shape="ring"时，可使用以下属性
android:innerRadius
android:innerRadiusRatio
android:thickness
android:thicknessRatio
android:useLevel
```

### 2.4 ShapeDrawable

还是没搞明白ShapeDrawable怎么用？

```
If no Shape is given, then the ShapeDrawable will default to a RectShape.

XML Attributes
android:bottom		Bottom padding. 
android:color		Defines the color of the shape. 
android:height		Defines the height of the shape. 
android:left		Left padding. 
android:right		Right padding. 
android:top		    Top padding. 
android:width	    Defines the width of the shape. 
```


### 2.5 LayerDrawable

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:id="@[+][package:]id/resource_name"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension"
        android:left="dimension" />
</layer-list>
```
每一个item表示一个Drawable，通过数个Drawable叠加，能产生很不错的效果，微信中的文本对话框实现见原书

### 2.6 StateListDrawable

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>
```

定义res/drawable/button.xml:
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>
```

使用
```
<Button
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:background="@drawable/button" />
```

### 2.7 LevelListDrawable

电量指示图标就是用的这个哦！

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<level-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/drawable_resource"
        android:maxLevel="integer"
        android:minLevel="integer" />
</level-list>
```

定义xml，显示规则是：按照从上往下定义的顺序，显示第一个满足的item
```
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/status_off"
        android:maxLevel="0" />
    <item
        android:drawable="@drawable/status_on"
        android:maxLevel="1" />
</level-list>
```

使用
```
ImageButton button = (ImageButton) findViewById(R.id.button);
LevelListDrawable drawable = (LevelListDrawable) button.getDrawable();
drawable.setLevel(0);//显示第一个item，参考上面的显示规则
drawable.setLevel(1);//显示第二个item
```

### 2.8 TransitionDrawable

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<transition
xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:id="@[+][package:]id/resource_name"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension"
        android:left="dimension" />
</transition>
```

定义res/drawable/transition.xml
```
<?xml version="1.0" encoding="utf-8"?>
<transition xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/on" />
    <item android:drawable="@drawable/off" />
</transition>
```

使用
```
<ImageButton
    android:id="@+id/button"
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:src="@drawable/transition" />
    
ImageButton button = (ImageButton) findViewById(R.id.button);
TransitionDrawable drawable = (TransitionDrawable) button.getDrawable();
drawable.startTransition(500);
```

### 2.9 InsetDrawable

当一个View希望自己的背景比自己的实际区域小的时候，可以采用InsetDrawable来实现

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<inset
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:insetTop="dimension"
    android:insetRight="dimension"
    android:insetBottom="dimension"
    android:insetLeft="dimension" />
```

使用1
```
<?xml version="1.0" encoding="utf-8"?>
<inset xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@mipmap/ic_launcher"
    android:insetTop="10dp"
    android:insetLeft="10dp" />
```

使用2
```
<?xml version="1.0" encoding="utf-8"?>
<inset xmlns:android="http://schemas.android.com/apk/res/android"
    android:insetTop="15dp"
    android:insetRight="15dp"
    android:insetBottom="15dp"
    android:insetLeft="15dp" >
    <shape android:shape="rectangle">
        <solid android:color="#ff0000">
    </shape>
</inset>
```

### 2.10 ScaleDrawable

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<scale
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:scaleGravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                          "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                          "center" | "fill" | "clip_vertical" | "clip_horizontal"]
    android:scaleHeight="percentage"
    android:scaleWidth="percentage" />
```

定义res/drawable/scale_drawable.xml
```
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/logo"
    android:scaleGravity="center"
    android:scaleHeight="70%"
    android:scaleWidth="70%" />
```

使用
```
View testScale = findViewById(R.id.test_scale);
ScaleDrawable testScaleDrawable = (ScaleDrawable) testScale.getBackground();
testScaleDrawable.setLevel(1);//level取值[1,10000]

//以上操作可将View缩小为原图的30%.
//这样理解，拿出70%进行缩放，level1表示完全缩放（结果30%），level10000表示完全不缩放（结果100%）///，level5000表示缩放一半（结果65%）
```

### 2.11 ClipDrawable

句法规则
```
<?xml version="1.0" encoding="utf-8"?>
<clip
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:clipOrientation=["horizontal" | "vertical"]
    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                     "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                     "center" | "fill" | "clip_vertical" | "clip_horizontal"] />
```

定义res/drawable/clip.xml:（裁剪右边）
```
<?xml version="1.0" encoding="utf-8"?>
<clip xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/android"
    android:clipOrientation="horizontal"
    android:gravity="left" />
```

使用
```
<ImageView
    android:id="@+id/image"
    android:background="@drawable/clip"
    android:layout_height="wrap_content"
    android:layout_width="wrap_content" />
    
ImageView imageview = (ImageView) findViewById(R.id.image);
ClipDrawable drawable = (ClipDrawable) imageview.getDrawable();
drawable.setLevel(drawable.getLevel() + 1000);
//0表示完全裁剪，10000表示不裁剪，故 8000表示裁剪20%
```
## 3 自定义Drawable

```
public class CustomDrawable extends Drawable {
    private Paint mPaint;

    public CustomDrawable(int color) {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(color);
    }

    @Override
    public void draw(Canvas canvas) {
        final Rect rect = getBounds();
        float cx = rect.exactCenterX();
        float cy = rect.exactCenterY();
        canvas.drawCircle(cx, cy, Math.min(cx, cy), mPaint);
    }

    @Override
    public void setAlpha(int i) {
        mPaint.setAlpha(i);
        invalidateSelf();
    }

    @Override
    public void setColorFilter(ColorFilter filter) {
        mPaint.setColorFilter(filter);
        invalidateSelf();
    }

    @Override
    public int getOpacity() {
        return PixelFormat.TRANSLUCENT;
    }
}
```




