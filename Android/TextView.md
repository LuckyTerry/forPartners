# TextView

[TOC]

---

## 一、TextView基础使用

### 1.字符串资源里变量替换
```
<string name="welcome">你好A，欢迎使用我们的App。</string>
String welcome = getString(R.string.welcome);

<string name="welcome">你好%1$s，欢迎使用我们的App。</string>
String welcome = getString(R.string.welcome, "小丸子");
```
### 2.TextView xml文件可配置的属性
```
candroid:autoLink    设置是否当文本为URL链接/email/电话号码/map时，文本显示为可点击的链接。可选值(none/web/email/phone/map/all)  

android:autoText    如果设置，将自动执行输入值的拼写纠正。此处无效果，在显示输入法并输入的时候起作用。  

android:bufferType  指定getText()方式取得的文本类别。选项editable 类似于StringBuilder可追加字符，也就是说getText后可调用append方法设置文本内容。spannable 则可在给定的字符区域使用样式，参见这里1、这里2。  

android:capitalize  设置英文字母大写类型。此处无效果，需要弹出输入法才能看得到，参见EditView此属性说明。  

android:cursorVisible   设定光标为显示/隐藏，默认显示。  

android:digits  设置允许输入哪些字符。如“1234567890.+-*/%\n()”  

android:drawableBottom  在text的下方输出一个drawable，如图片。如果指定一个颜色的话会把text的背景设为该颜色，并且同时和background使用时覆盖后者。  

android:drawableLeft    在text的左边输出一个drawable，如图片。  

android:drawableRight   在text的右边输出一个drawable，如图片。  

android:drawableTop 在text的正上方输出一个drawable，如图片。  

android:drawablePadding 设置text与drawable(图片)的间隔，与drawableLeft、drawableRight、drawableTop、drawableBottom一起使用，可设置为负数，单独使用没有效果。  

android:editable    设置是否可编辑。这里无效果，参见EditView。  

android:editorExtras    设置文本的额外的输入数据。在EditView再讨论。  

android:ellipsize   设置当文字过长时,该控件该如何显示。有如下值设置：”start”—–省略号显示在开头；”end”——省略号显示在结尾；”middle”—-省略号显示在中间；”marquee” ——以跑马灯的方式显示(动画横向移动)  

android:freezesText 设置保存文本的内容以及光标的位置。参见：这里。  

android:gravity 设置文本位置，如设置成“center”，文本将居中显示。  

android:hint    Text为空时显示的文字提示信息，可通过textColorHint设置提示信息的颜色。此属性在EditView中使用，但是这里也可以用。  

android:imeOptions  附加功能，设置右下角IME动作与编辑框相关的动作，如actionDone右下角将显示一个“完成”，而不设置默认是一个回车符号。这个在EditView中再详细说明，此处无用。  

android:imeActionId 设置IME动作ID。在EditView再做说明，可以先看这篇帖子：这里。  

android:imeActionLabel  设置IME动作标签。在EditView再做说明。  

android:includeFontPadding  设置文本是否包含顶部和底部额外空白，默认为true。  

android:inputMethod 为文本指定输入法，需要完全限定名（完整的包名）。例如：com.google.android.inputmethod.pinyin，但是这里报错找不到。  

android:inputType   设置文本的类型，用于帮助输入法显示合适的键盘类型。在EditView中再详细说明，这里无效果。  

android:marqueeRepeatLimit  在ellipsize指定marquee的情况下，设置重复滚动的次数，当设置为marquee_forever时表示无限次。  

android:ems 设置TextView的宽度为N个字符的宽度。这里测试为一个汉字字符宽度，如图：   

android:maxEms  设置TextView的宽度为最长为N个字符的宽度。与ems同时使用时覆盖ems选项。  

android:minEms  设置TextView的宽度为最短为N个字符的宽度。与ems同时使用时覆盖ems选项。  

android:maxLength   限制显示的文本长度，超出部分不显示。  

android:lines   设置文本的行数，设置两行就显示两行，即使第二行没有数据。  
android:maxLines    设置文本的最大显示行数，与width或者layout_width结合使用，超出部分自动换行，超出行数将不显示。  

android:minLines    设置文本的最小行数，与lines类似。  

android:linksClickable  设置链接是否点击连接，即使设置了autoLink。  

android:lineSpacingExtra    设置行间距。  

android:lineSpacingMultiplier   设置行间距的倍数。如”1.2”  

android:numeric 如果被设置，该TextView有一个数字输入法。此处无用，设置后唯一效果是TextView有点击效果，此属性在EdtiView将详细说明。  

android:password    以小点”.”显示文本  

android:phoneNumber 设置为电话号码的输入方式。  

android:privateImeOptions  设置输入法选项，此处无用，在EditText将进一步讨论。  

android:scrollHorizontally  设置文本超出TextView的宽度的情况下，是否出现横拉条。 

android:selectAllOnFocus    如果文本是可选择的，让他获取焦点而不是将光标移动为文本的开始位置或者末尾位置。TextView中设置后无效果。  

android:shadowColor 指定文本阴影的颜色，需要与shadowRadius一起使用。效果：    

android:shadowDx    设置阴影横向坐标开始位置。  

android:shadowDy    设置阴影纵向坐标开始位置。  

android:shadowRadius    设置阴影的半径。设置为0.1就变成字体的颜色了，一般设置为3.0的效果比较好。  
android:singleLine  设置单行显示。如果和layout_width一起使用，当文本不能全部显示时，后面用“…”来表示。如android:text="test_ singleLine " android:singleLine="true" android:layout_width="20dp"将只显示“t…”。如果不设置singleLine或者设置为false，文本将自动换行  

android:text    设置显示文本.  

android:textAppearance  设置文字外观。如“?android:attr/textAppearanceLargeInverse  
”这里引用的是系统自带的一个外观，？表示系统是否有这种外观，否则使用默认的外观。可设置的值如下：textAppearanceButton/textAppearanceInverse/textAppearanceLarge/textAppearanceLargeInverse/textAppearanceMedium/textAppearanceMediumInverse/textAppearanceSmall/textAppearanceSmallInverse  

android:textColor   设置文本颜色  

android:textColorHighlight  被选中文字的底色，默认为蓝色  

android:textColorHint   设置提示信息文字的颜色，默认为灰色。与hint一起使用。  

android:textColorLink   文字链接的颜色.  

android:textScaleX  设置文字之间间隔，默认为1.0f。分别设置0.5f/1.0f/1.5f/2.0f效果如下：  

android:textSize    设置文字大小，推荐度量单位”sp”，如”15sp”  

android:textStyle   设置字形[bold(粗体) 0, italic(斜体) 1, bolditalic(又粗又斜) 2] 可以设置一个或多个，用“|”隔开  

android:typeface    设置文本字体，必须是以下常量值之一：normal 0, sans 1, serif 2, monospace(等宽字体) 3]   

android:height  设置文本区域的高度，支持度量单位：px(像素)/dp/sp/in/mm(毫米)  

android:maxHeight   设置文本区域的最大高度  

android:minHeight   设置文本区域的最小高度  

android:width   设置文本区域的宽度，支持度量单位：px(像素)/dp/sp/in/mm(毫米)，与layout_width的区别看这里。  

android:maxWidth    设置文本区域的最大宽度  

android:minWidth    设置文本区域的最小宽度  
```
### 3.TextView中设置多种字体大小
### 4.TextView中设置超链接
### 5.插入图片
```
xml:
android:drawableLeft
android:drawableTop
android:drawableRight
android:drawableBottom
android:drawablePadding 

java:
setCompoundDrawablesWithIntrinsicBounds(int left, int top, int right, int bottom) // left，top等需传入资源id，不需要的话传null或0。
setCompoundDrawablePadding(int pad)
```
### 6.阴影
```
android:shadowColor //指定文本阴影的颜色
android:shadowDx //设置阴影横向坐标开始位置
android:shadowDy //设置阴影纵向坐标开始位置
android:shadowRadius //设置阴影的半径。设置为0.1会变成字体的颜色

java:
public void setShadowLayer (float radius, float dx, float dy, int color)

```

### 7.字体加粗或者倾斜
```
xml:
android:textStyle=”bold”

java:
tv.getPaint().setFakeBoldText(true);

textstyle可设置的属性有：normal、bold、italic，talic为倾斜，多属性可用”|”分开。
```



### 8.文字过长显示省略号或者跑马灯效果

```
省略号：
android:maxEms="6" //限制显示的字符长度,一个汉字占1个字符,1个英文/符号占半个字符;从第maxEms+1位置开始用省略号代替
android:maxLines="1"// 单行显示
android:ellipsize="end"//在结尾用省略号

跑马灯：
android:marqueeRepeatLimit="marquee_forever"
android:ellipsize="marquee"
android:singleLine="true"
android:focusableInTouchMode="true"
android:focusable="true"

android:ellipsize设置当文字过长时,该控件该如何显示。有如下值设置：
”start”—–省略号显示在开头；
”end”——省略号显示在结尾；
”middle”—-省略号显示在中间；
”marquee” ——以跑马灯的方式显示(动画横向移动)
```

### 9.多文字展示中常见的【显示全部-收起】
```
// 根据数据内容来动态判断是该展示 [显示全部]按钮.此时需要动态获取textview加载了内容后占据了几行,比如我们需求规定超过3行,末尾就要省略+展示[显示更多] . 
// 不能直接在onCreate()中调用,因为此时TextView的内容可能还没有加载完毕导致获取到得行数为0.
textview.post(new Runnable() {
    @Override
    public void run() {
        if(textview.getLineCount() > 3){
            //textview内容大于行数限制,展示[显示全部]按钮"
        }else {
            //textview内容小于等于行数限制"
        }
    }
});
// 核心方法
showAll = !showAll;
if(showAll){
      tvShowAll.setMaxLines(2);
      btnShowAll.setText("查看全部");
  }else {
      tvShowAll.setMaxLines(20);
      btnShowAll.setText("收起");
  }
```

### 10.价格标签与下划线
```
//中间横线
textview.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG );
//下划线
textview.getPaint().setFlags(Paint.UNDERLINE_TEXT_FLAG);
```

### 11.设置字间距与行间距
```
设置字间距：
android:letterSpacing="0.5"(0.0~1.0之间的小数,以一个字母为空间标准)

设置行间距：
android:lineSpacingExtra="3dp"

设置行间距的倍数:
android:lineSpacingMultiplier="1.2"

设置字间距：
setLetterSpacing(float letterSpacing)

设置行间距和行间距的倍数:
setLineSpacing(float add, float mult)
```

### 12.关于字体
```
xml使用android默认字体:
android:typeface="normal|sans|serif|monospace"

java引入其他字体：
Typeface mTypeFace = Typeface.createFromAsset(getAssets(), "kaiti.ttf");
textview.setTypeface(mTypeFace);
```

### 13.TextView中的内容是否可被选中
```
xml:
android:textIsSelectable=""

java:
setTextIsSelectable(boolean selectable)
```

### 14.获取焦点后是否选中全部内容
```
xml:
android:selectAllOnFocus=""

java:
setSelectAllOnFocus(boolean selectAllOnFocus)
```

### 15.TextView中被选中内容的高亮背景色
```
xml:
android:android:textColorHighlight=""

java:
setHighlightColor(@ColorInt int color)
```

### 16.TextView实例中文字横向拉伸倍数
```
xml:
android:textScaleX=""

java:
setTextScaleX(float size)
```

### 17.设置TextView高度为行数相关
```
// 设置TextView最小高度为指定行高度
xml:
android:minLines

java:
setMinLines(int minlines)

// 设置TextView最大高度为指定行高度
xml:
android:maxLines

java:
setMaxLines(int maxlines)

// 设置TextView精确高度为指定行高度
xml:
android:lines

java:
setLines(int lines)
```

### 18.TextView中内容是否为明文
```
android:password="true"
```
### 19.TextView中的字符是否全部显示为大写形式
```
setAllCaps(boolean allCaps)
```

### 20.TextView实例的文字尺寸为指定单位unit的指定值size
```
setTextSize(int unit, float size)
```

### 21.Drawable居中的TextView（only "center"，exclude"center_horizontal"、"center_vertical"）
```
public class DrawableCenterTextView extends android.support.v7.widget.AppCompatTextView {

    public DrawableCenterTextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // 获取TextView的Drawable对象，返回的数组长度应该是4，对应左上右下
        Drawable[] drawables = getCompoundDrawables();
        if (drawables != null) {
            Drawable drawable = drawables[0];
            if (drawable != null) {
                // 当左边Drawable的不为空时，测量要绘制文本的宽度
                float textWidth = getPaint().measureText(getText().toString());
                int drawablePadding = getCompoundDrawablePadding();
                int drawableWidth = drawable.getIntrinsicWidth();
                // 计算总宽度（文本宽度 + drawablePadding + drawableWidth）
                float bodyWidth = textWidth + drawablePadding + drawableWidth;
                // 移动画布开始绘制的X轴
                canvas.translate((getWidth() - bodyWidth) / 2, 0);
            } else if ((drawable = drawables[1]) != null) {
                // 否则如果上边的Drawable不为空时，获取文本的高度
                Rect rect = new Rect();
                getPaint().getTextBounds(getText().toString(), 0, getText().toString().length(), rect);
                float textHeight = rect.height();
                int drawablePadding = getCompoundDrawablePadding();
                int drawableHeight = drawable.getIntrinsicHeight();
                // 计算总高度（文本高度 + drawablePadding + drawableHeight）
                float bodyHeight = textHeight + drawablePadding + drawableHeight;
                // 移动画布开始绘制的Y轴
                canvas.translate(0, (getHeight() - bodyHeight) / 2);
            } else
                if ((drawable = drawables[2]) != null) {
                // 当左边Drawable的不为空时，测量要绘制文本的宽度
                float textWidth = getPaint().measureText(getText().toString());
                int drawablePadding = getCompoundDrawablePadding();
                int drawableWidth = drawable.getIntrinsicWidth();
                // 计算总宽度（文本宽度 + drawablePadding + drawableWidth）
                float bodyWidth = textWidth + drawablePadding + drawableWidth;
                setPadding(0, 0, (int) (getWidth() - bodyWidth), 0);
                // 移动画布开始绘制的X轴
                canvas.translate((getWidth() - bodyWidth) / 2, 0);
            } else if ((drawable = drawables[3]) != null) {
                // 否则如果上边的Drawable不为空时，获取文本的高度
                Rect rect = new Rect();
                getPaint().getTextBounds(getText().toString(), 0, getText().toString().length(), rect);
                float textHeight = rect.height();
                int drawablePadding = getCompoundDrawablePadding();
                int drawableHeight = drawable.getIntrinsicHeight();
                // 计算总高度（文本高度 + drawablePadding + drawableHeight）
                float bodyHeight = textHeight + drawablePadding + drawableHeight;
                setPadding(0, 0, 0, (int) (getWidth() - bodyHeight));
                // 移动画布开始绘制的Y轴
                canvas.translate(0, (getHeight() - bodyHeight) / 2);
            }
        }
        super.onDraw(canvas);
    }
}
```

## 二、TextView之SpannableString使用
```
String text = "您已经连续走了5963步";
int start = text.indexOf('5');
int end = text.length();
Spannable textSpan = new SpannableStringBuilder(text);
textSpan.setSpan(new AbsoluteSizeSpan(16), 0, start, Spannable.SPAN_INCLUSIVE_INCLUSIVE);
textSpan.setSpan(new AbsoluteSizeSpan(26), start, end - 1, Spannable.SPAN_INCLUSIVE_INCLUSIVE);
textSpan.setSpan(new AbsoluteSizeSpan(16), end - 1, end, Spannable.SPAN_INCLUSIVE_INCLUSIVE);
TextView textView = (TextView) findViewById(R.id.text);
textView.setText(textSpan);
```

## 三、TextView之Html使用
```
<a href="...">  //定义链接内容
<b> //定义粗体文字   b 是blod的缩写
<big> //定义大字体的文字
<blockquote> //引用块标签 
<br> //定义换行
<cite> //表示引用的URI
<dfn> //定义标签  dfn 是defining instance的缩写
<div align="...">
<em> //强调标签  em 是emphasis的缩写
<font color="..." face="...">  //不支持size属性
<h1>
<h2>
<h3>
<h4>
<h5>
<h6>
<i> //定义斜体文字
<img src="...">
<p> // 段落标签,里面可以加入文字,列表,表格等
<small> //定义小字体的文字
<strike> // 定义删除线样式的文字   不符合标准网页设计的理念,不赞成使用.   strike是strikethrough的缩写
<strong> //重点强调标签
<sub> //下标标签   sub 是subscript的缩写
<sup> //上标标签   sup 是superscript的缩写
<tt> //定义monospaced字体的文字  不赞成使用.  此标签对中文没意义  tt是teletype or monospaced text style的意思
<u> //定义带有下划线的文字  u是underlined text style的意思
```
代码示例：显示多种颜色的字
```
TextView textth = (TextView) findViewById(R.id.textth);
String textStr1 = "<font color=\"#123569\">如果有一天，</font>";
String textStr2 = "<font color=\"#00ff00\">我悄然离去</font>";
textth.setText(Html.fromHtml(textStr1 + textStr2));
```
代码示例：字体加粗
```
String textStr1 = "<b>sdfa</b>";
textth.setText(Html.fromHtml(textStr1));
```
代码示例：插入图片
```
String imgStr = "<b>sdfa</b><br><img src=\"" + R.mipmap.ic_launcher + "\"/>";
Html.ImageGetter imageGetter = new Html.ImageGetter() {
        @Override
        public Drawable getDrawable(String source) {
            int id = Integer.parseInt(source);
            Drawable draw = getResources().getDrawable(id);
            draw.setBounds(0, 0, 300, 200);
            return draw;
        }
 };
 TextView textfi = (TextView) findViewById(R.id.textfiv);
 textfi.append(Html.fromHtml(imgStr, imageGetter, null));
 ```





