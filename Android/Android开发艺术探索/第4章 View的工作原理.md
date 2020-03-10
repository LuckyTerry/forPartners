# 第4章 View的工作原理

[TOC]

----------


## 1 初识ViewRoot和DecorView
### 1.1 两者的关系
当Activity被创建完毕后，PhoneWindow中会创建ViewRootImpl对象，，并将ViewRootImpl对象和DecorView建立关联，源码如下：
```
root = new ViewRootImpl(view.getContext(), display);
root.setView(view, wparams, panelParentView);
```
### 1.2 整体流程

```
//入口，依次调用performMeasure、performLayout、performDraw
ViewRootImpl的performTraversals

//测量
ViewRootImpl的performMeasure ->
ViewGroup的measure ->
ViewGroup的onMeasure ->
View的measure ->
View的onMeasure -> end

//布局
ViewRootImpl的performLayout ->
ViewGroup的layout ->
ViewGroup的onLayout ->
View的layout ->
View的onLayout -> end

//绘制
ViewRootImpl的performDraw ->
ViewGroup的draw ->
ViewGroup的onDraw ->
View的draw ->
View的onDraw -> end
```

## 2 理解MeasureSpec

### 2.1 MeasureSpec介绍

MeasureSpec，32位int值，高两位mode，低30位size

```
//通过size和mode整合MeasureSpec
public static int makeMeasureSpec(int size, int mode)

//通过MeasureSpec拆分出mode
public static int getMode(int measureSpec)

//通过MeasureSpec拆分出size
public static int getSize(int measureSpec)
```

mode有三种模式

* UNSPECIFIED 要多大有多大
* EXACTLY 精确大小，对应match_parent 和 dp/px
* AT_MOST 不大于 parentSize，对应 wrap_content

### 2.2 MeasureSpec是如何确定的

MeasureSpec 由父容器 MeasureSpec 及自身 LayoutParam 共同确定，特别的，DecorView 由屏幕大小和自身 LayoutParam 共同确定

看下面ViewRootImpl和ViewGrooup中的两段代码

```
/**
* ViewRootImpl中产生DecorView的MeasureSpec的过程（DecorView是一个FrameLayout，相当于ViewGroup）
* @desiredWindowWidth 窗口宽
* @desiredWindowHeight 窗口高
* @childWidthMeasureSpec 目标MeasureSpec
* @childHeightMeasureSpec 目标MeasureSpec
*/
private boolean measureHierarchy(final View host, final WindowManager.LayoutParams lp,
            final Resources res, final int desiredWindowWidth, final int desiredWindowHeight) {
    ...
    childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
    childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    ...
}
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {
        case ViewGroup.LayoutParams.MATCH_PARENT:
        // Window can't resize. Force root view to be windowSize.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        // Window can resize. Set max size for root view.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        // Window wants to be an exact size. Force root view to be that size.
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
```

```
/**
* ViewGroup中产生ViewGroup或View的MeasureSpec的过程
* @parentWidthMeasureSpec 父MeasureSpec
* @parentHeightMeasureSpec 父MeasureSpec
* @childWidthMeasureSpec 目标MeasureSpec
* @childHeightMeasureSpec 目标MeasureSpec
*/
protected void measureChild(View child, int parentWidthMeasureSpec,
    int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
        mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
        mPaddingTop + mPaddingBottom, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);

    int size = Math.max(0, specSize - padding);

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

## 3 View的工作流程

先看三组API

* class View （关注下面6个函数）
```
public final void measure(int widthMeasureSpec, int heightMeasureSpec){
    ...
    onMeasure(...);
    ...
}
public void layout(int l, int t, int r, int b){
    ...
    onLayout(...);
    ...
}
public void draw(Canvas canvas){
    ...
    drawBackground(canvas);
    ...
    onDraw(canvas);
    ...
    dispatchDraw(canvas);
    ...
    onDrawForeground(canvas);
    ...
}
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
protected void	onLayout(boolean changed, int left, int top, int right, int bottom){
}，这是一个空实现
protected void onDraw(Canvas canvas) {
}，这是一个空实现
```

* abstract class ViewGroup extends View （除了onLayout，其他5各都是继承自View，添加了measureChildren等方法）
```
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
public abstract void onLayout(boolean changed, int l, int t, int r, int b);
```

* LinearLayout extends ViewGroup
```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
    ...
}
protected void onLayout(boolean changed, int l, int t, int r, int b){
    ...
}
protected void onDraw(Canvas canvas){
    ...
}
```

### 3.1 measure 过程 （3.1.3 -> 3.1.4 -> 3.1.1 -> 3.1.2）

#### 3.1.1 View 的 measure
参考上面第一组API，内部调用onMeasure。

#### 3.1.2 View 的 onMeasure
参考上面第一组API，内部只是setMeasuredDimension，设置进去的值就是测量值。

#### 3.1.3 ViewGroup 的 measure
继承自View的measure，没有重写。

#### 3.1.4 ViewGroup 的 onMeasure
继承自View的onMeasure，没有重写。因为不同容器的特性不同，所以这个需要子类来实现。
但是提供了measureChildren方法（内部遍历调用child的measure方法），供开发者调用以测量child。一般来说该方法使用在重写的onMeasure()或onLayout()中，先测量child，再根据child的测量高宽等参数布局该容器。图方便可以使用在重写的onLayout()里。

#### 3.1.5 Activity 启动的时候正确获取测量宽高
由于measure与Activity生命周期不同步，无法保证onCreate、onStart、onResume时某个View已经测量完毕，此时想要获取测量宽高，有如下四种方法：

```
@Override
public void onWindowsFocusChanged(boolean hasFocus) {
    super.onWindowsFocusChanged(hasFocus);
    if(hasFocus) {
        int width = view.getMeasureWidth();
        int height = view.getMeasureHeight();
    }
}
```
```
protect void onStart() {
    super.onStart();
    view.post(new Runnable() {
        @Override
        public void run () {
            int width = view.getMeasureWidth();
            int height = view.getMeasureHeight();
        }
    });
}
```
```
protect void onStart() {
    super.onStart();
    ViewTreeObserver observer = view.getViewTreeObserver();
    observer.addOnGloabalLayoutListener(new onGlobalLayoutListener() {
        @SuppressWarnings("deprecation")
        @Override
        public void onGlobalLayout() {
            view.getViewTreeObserver().removeGloabalOnLayoutListener(this);
            int width = view.getMeasureWidth();
            int height = view.getMeasureHeight();
        }
    });
}
```
```
//手动测量，根据View的LayoutParams分三种情况

//match_parent 
无法知道parentSize，测量不出，放弃

//具体数值
int widthMeasureSpec = MeasureSpec.makeMeasureSpec(100, MeasureSpec.EXACTLY);
int heightMeasureSpec = MeasureSpec.makeMeasureSpec(100, MeasureSpec.EXACTLY);
view.measure(widthMeasureSpec, heightMeasureSpec);

//wrap_content
int widthMeasureSpec = MeasureSpec.makeMeasureSpec( (1 << 30) - 1, MeasureSpec.AT_MOST);
int heightMeasureSpec = MeasureSpec.makeMeasureSpec( (1 << 30) - 1, MeasureSpec.AT_MOST);
view.measure(widthMeasureSpec, heightMeasureSpec);
```

#### 3.1.6 示例：LineaLayout 的 onMeasure 过程
有时间把原文的LineaLayout的onMeasure过程多看几遍

### 3.2 layout 过程 （3.2.3 -> 3.2.4 -> 3.2.1 -> 3.2.2）

#### 3.2.1 View 的 layout

* setFrame(l, t, r, b)设置4个顶点的位置，View的顶点一旦确定，那么View的位置也就确定了。（竟然是layout确定的View的位置，我还一直以为是onLayout确定的呢。。。）
* onLayout方法，用于父容器确定子元素的位置。（骚年，记清楚，layout确定View位置，onLayout确定子元素位置）

```
@SuppressWarnings({"unchecked"})
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```

#### 3.2.2 View 的 onLayout
View没有子元素，所以当然是空实现咯。当然自定义View也没必要且不应该重写该方法，毕竟View位置都是在layout中确定的。

#### 3.2.3 ViewGroup 的 layout
继承自View的layout，没有重写。确定ViewGroup的位置

#### 3.2.4 ViewGroup 的 onLayout
重写View的layout，并且改变了限定符，现在是个abstract class了。继承自ViewGroup的子类必须重写该方法，以确定子元素的位置。

#### 3.2.5 示例：LineaLayout的onLayout过程
有时间把原文的LineaLayout的onLayout过程多看几遍。

### 3.3 draw 过程 （3.3.3 -> 3.3.4 -> 3.3.1 -> 3.3.2）

#### 3.3.1 View 的 draw
上面第一组API看看就明白了。画背景、画自己、画children、画装饰。

#### 3.3.2 View 的 onDraw
都不知道是什么View，当然是空实现。自定义View的画就重写该方法自己画去。

#### 3.3.3 ViewGroup 的 draw
继承自View，没有重写。

#### 3.3.4 ViewGroup 的 onDraw
继承自View，没有重写。一般来说ViewGroup也不需要绘制自己，因为只需要“容纳”子元素就功成身退了。

## 4 自定义View

### 4.1 自定义View的分类

 1. 继承View重写onDraw方法
 2. 继承ViewGroup派生特殊的Layout
 3. 继承特定的View（比如TextView）
 4. 继承特定的ViewGroup（比如LinearLayout）

找到一种代价最小、最高效的方法去实现。
 
### 4.2 自定义View须知

 1. 让View支持wrap_content
 2. 支持padding
 3. 不要使用Handler ？
 4. View中如果有线程或者动画，需要及时停止
 5. 滑动嵌套，需要处理好滑动冲突

### 4.3 自定义View示例

 1. 继承View重写onDraw方法 -> 略，这个还是看书吧
 2. 继承ViewGroup派生特殊的Layout -> 略，这个还是看书吧

### 4.4 自定义View的思想

我觉得自定义View还是等我真正上手项目后再认真研究，看过谁说的一句话，千万不要刻意去自定义View。。。弥足深陷。。。