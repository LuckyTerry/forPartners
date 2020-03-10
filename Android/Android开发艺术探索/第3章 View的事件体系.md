# 第3章 View的事件体系 

[TOC]


----------


## 1. View基础知识

### 1.1 什么是View
View是Android中所有控件的基类

### 1.2 View的位置参数

> * left，view左边界距离父View左边界的距离
> * right，view右边界距离父View左边界的距离
> * top，view上边界距离父View上边界的距离
> * bottom，view下边界距离父View上边界的距离
> * elevation，view于Z轴方向距离父View的距离，垂直屏幕向外为＋
> * width，view的宽度
> * height，view的高度
> * x，view左上角横坐标
> * y，view左上角纵坐标
> * z，view于Z轴方向的坐标
> * translationX，view左上角相对于父容器的横向偏移量，初始值为0
> * translationY，view左上角相对于父容器的纵向偏移量，初始值为0
> * translationZ，view左上角相对于父容器的Z轴偏移量，初始值为0

top、left、elevation表示的是原始信息，其值并不会改变，此时发生改变的是translationX、translationY、translationZ、x、y、z

```
//一些属性、方法
left = getLeft();
right = getRight();
top = getTop();
bottom = getBottom();
elevation = getElevation();
width = getWidth();
height = getHeight();
x = getX();
y = getY();
z = getZ();
translationX = getTranslationX();
translationY = getTranslationY();
translationZ = getTranslationZ();
//它们的关系
width == right - left;
height == bottom - top;
x = left + translationX;
y = top + translationY;
z = elevation + translationZ;
```

### 1.3 MotionEvent

> * ACTION_DOWN-手指刚接触屏
> * ACTION_MOVE-手指在屏幕上移动
> * ACTION_UP-手机从屏幕上松开的一瞬间

```
getX()、getY() //返回的是相对于当前View左上角的x和y坐标
getRawX()、getRawY() //返回的是相对于手机屏幕左上角的x和y坐标。
```

### 1.4 TouchSlop

系统所能识别出的可以被认为是滑动的最小距离。
```
ViewConfiguration.get(getContext()).getScaledTouchSlope();
```

### 1.5 VelocityTracker


```
//初始化
VelocityTracker mVelocityTracker = VelocityTracker.obtain();

//在onTouchEvent方法中追踪单击事件
mVelocityTracker.addMovement(event);

//获取当前速度，x像素/1000ms，则代表速度是x; y像素/100ms，则代表速度是y; 并不是以pix/s作为单位。
mVelocityTracker.computeCurrentVelocity(1000);
int xVelocity = (int) mVelocityTracker.getXVelocity();
int yVelocity = (int) mVelocityTracker.getYVelocity();

//重置和回收
mVelocityTracker.clear(); //一般在MotionEvent.ACTION_UP的时候调用
mVelocityTracker.recycle(); //一般在onDetachedFromWindow中调用
```

### 1.6 GestureDetector
* 日常开发中，比较常用的有:
onSingleTapUp(单击)、onFling(快速滑动)、onScroll（拖动）、onLongPress(长按)、onDoubleTap(双击)。
* 建议：如果只是监听滑动相关的事件在onTouchEvent中实现；如果要监听双击这种行为的话，那么就使用GestureDetector。

```
//6个函数都得Override
GestureDetector mGestureDetector = new GestureDetector(this, new GestureDetector.OnGestureListener() {
    @Override
    public boolean onDown(MotionEvent event) {
        return false;
    }

    @Override
    public void onShowPress(MotionEvent event) {

    }

    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        return false;
    }

    @Override
    public boolean onScroll(MotionEvent event, MotionEvent event1, float v, float v1) {
        return false;
    }

    @Override
    public void onLongPress(MotionEvent event) {

    }

    @Override
    public boolean onFling(MotionEvent event, MotionEvent event1, float v, float v1) {
        return false;
    }
});

//6个函数可自由选择Override哪些
GestureDetector mGestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent event) {
        return false;
    }

    @Override
    public void onShowPress(MotionEvent event) {

    }

    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        return false;
    }

    @Override
    public boolean onScroll(MotionEvent event, MotionEvent event1, float v, float v1) {
        return false;
    }

    @Override
    public void onLongPress(MotionEvent event) {

    }

    @Override
    public boolean onFling(MotionEvent event, MotionEvent event1, float v, float v1) {
        return false;
    }
});

//解决长按屏幕无法拖动的现象
mGestureDetector.setIsLongPressEnabled(false);

//接管目标view的onTouchEvent方法
boolean consume = mGestureDetector.onTouchEvent(event);
return consume;

//双击事件监听
mGestureDetector.setOnDoubleTapListener(new GestureDetector.OnDoubleTapListener() {
    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        return false;
    }

    @Override
    public boolean onDoubleTap(MotionEvent event) {
        return false;
    }

    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        return false;
    }
});
```

### 1.7 Scroller

弹性滑动对象，用于实现View的弹性滑动。
Scroller本身无法让View弹性滑动，它需要和View的computeScroll方法配合使用才能共同完成这个功能。
请把Scroller当成一个计算器，传进去起点、偏移量、时间，然后Scroller会根据逝去的时间算出当前的XY。

```
Scroller mScroller = new Scroller(mContext);

private void smoothScrollTo(int destX, int destY) {
    int scrollX = getScrollX();
    int scrollY = getScrollY();
    int deltaX = destX - scrollX;
    int deltaY = destY - scrollY;
    mScroller.startScroll(scrollX, scrollY, deltaX, deltaY, 1000);
    invalidate();
}

@Override
public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
        postInvalidate();
    }
}
```
----------


## 2. View的滑动

* 通过View本身提供的scrollTo/scrollBy方法来实现滑动
* 通过动画给View施加平移效果来实现滑动
* 通过改变View的LayoutParams使得View重新布局从而实现滑动

### 2.1 scrollTo和scrollBy

scrollTo和scrollBy方法只能改变view内容的位置而不能改变View在布局中的位置。
mScrollX，View左边缘和view内容左边缘在水平方向的距离，View左边缘在View内容左边缘右边时为+ 。
mScrollY，View上边缘和view内容上边缘在竖直方向的距离，View上边缘在View内容上边缘下边时为+ 。

一准则：
> * View在滑动，View的内容始终不滑动。而对人眼来说惯性坐标系是View，因此看起来就是View的内容在滑动（并且实质上应该反向的）

四个原则：
> * 手指右滑，View应该是在左滑动。并且mScrollX是在减小
> * 手指下滑，View应该是在上滑动。并且mScrollY是在减小
> * View左边缘向左滑动到View内容左边缘左边时，mScrollX开始为-
> * View上边缘向上滑动到View内容上边缘上边时，mScrollY开始为-


### 2.2 使用动画

这个实现的方式有很多：
View动画 TranslateAnimation
属性动画 ObjectAnimator，ValueAnimator，ViewPropertyAnimator等等，不赘述了。


### 2.3 改变布局参数

结合Android群英传，总结如下

```
//方式一，推荐这个，推荐理由我也不知道-.-
ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
layoutParams.leftMargin = getLeft() + offsetX;
layoutParams.topMargin = getTop() + offsetY;
setLayoutParams(layoutParams);
//方式二
父布局.LayoutParams layoutParams = (父布局.LayoutParams) getLayoutParams();
layoutParams.leftMargin = getLeft() + offsetX;
layoutParams.topMargin = getTop() + offsetY;
setLayoutParams(layoutParams);
//方式三
layout(getLeft() + offsetX, getTop() + offsetY, getRight() + offsetX, getBottom() + offsetY);
//方式四
offsetLeftAndRight(offsetX);
offsetTopAndBottom(offsetY);
```
----------


## 3. View的弹性滑动

### 3.1 Scroller
贴一记源码，啥都明白了，如下

```
public void startScroll(int startX, int startY, int dx, int dy, int duration) {
    mMode = SCROLL_MODE;
    mFinished = false;
    mDuration = duration;
    mStartTime = AnimationUtils.currentAnimationTimeMillis();
    mStartX = startX;
    mStartY = startY;
    mFinalX = startX + dx;
    mFinalY = startY + dy;
    mDeltaX = dx;
    mDeltaY = dy;
    mDurationReciprocal = 1.0f / (float) mDuration;
}

public boolean computeScrollOffset() {
    ...
    int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
   
    if (timePassed < mDuration) {
        switch (mMode) {
            case SCROLL_MODE:
                final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.round(x * mDeltaY);
                break;
            ...
       }
    }
    return true;
}

public final int getCurrX() {
    return mCurrX;
}

public final int getCurrY() {
    return mCurrY;
}

//设计思想很赞有木有，整个过程中对View没有丝毫引用，甚至在它的内部连计时器都没有
```

### 3.2 通过动画

同2.2 不叨逼了

### 3.3 使用延时策略

```
private static final int MESSAGE_SCROLL_TO = 1;
private static final int FRAME_COUNT = 30;
private static final int DELAYED_TIME = 33;
private int mCount = 0;

private Handler mHandler = new Handlder(){
    public void handleMessage(Message msg){
        mCount++;
        if(mCount <= FRAME_COUNT){
            float fraction = mCount / (float)FRAME_COUNT;
            int scrollX = (int) (fraction * 100);
            mButton.scrollTo(scrollX, 0);
            mHandler.sendEmptyMessageDelayed(MESSAGE_SCROLL_TO, DELAYED_TIME);
        }
    }
}
```

----------

## 4. View的事件分发机制

```
public boolean dispatchTouchEvent(MotionEvent event)

public boolean onInterceptTouchEvent(MotionEvent event)

public boolean onTouchEvent(MotionEvent event)

public boolean dispatchTouchEvent(MotionEvent event){
    boolean consume = false;
    if (onInterceptTouchEvent(ev)) {
        consume = onTouchEvent(ev);
    } else {
        consume = child.dispatchTouchEvent(ev);
    }
}
```

关于事件传递的机制，给出一些结论：

* (1) 同一个事件序列是以down事件开始，中间含有数量不定的move事件，最终以up事件结束

* (2) 正常情况下，一个事件序列只能被一个View拦截且消耗。一旦一个元素拦截了某次事件，那么同一个事件序列内的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由两个View同时处理，但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View处理

* (3) 某个View一旦决定拦截，那么这一个事件序列都只能由它处理，并且它的onInterceptTouchEvent不会再调用
* (5) 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续事件，最终这些消失的点击事件会传递给Activity处理。

* (4) 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件，那么同一事件序列的其他事情都不会再交给它来处理，并且事件将重新交给它的父容器去处理（调用父容器的onTouchEvent方法）；如果它消耗ACTION_DOWN事件，但是不消耗其他类型事件，那么这个点击事件会消失，父容器的onTouchEvent方法不会被调用，当前view依然可以收到后续的事件，但是这些事件最后都会传递给Activity处理。

* (6) ViewGroup默认不拦截任何事件。Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false，View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会调用。

* (7) View没有onInterceptTouchEvent方法，一旦有事件传递给它，那么它的onTouchEvent方法就会被调用。

* (8) View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认都为false，clickable要分情况，比如Button的clickable属性默认为true，而TextView的clickable属性默认为false。

* (9) View的enable属性不影响onTouchEvent的默认返回值，哪怕一个View是disable状态的，只要它的clickable或者longClickable有一个为true，那么它的onTouchEvent就返回true

* (10) onClick会发生的前提是当前View是可点击的，并且它收到了down和up事件。

* (11) 事件传递过程总是先传递给父元素，然后再由父元素分发给子view，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外，即当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件。


### 4.1 Activity对点击事件的处理过程

Activity
```
  public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
    //所有View的onTouchEvent都返回了false，那么Activity的onTouchEvent被调用
    //onTouchEvent返回true表示整个事件循环结束，返回false表示事件没人处理
}  
```

PhoneWindow
```
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
    //mDecor是getWindow().getDecorView()返回的View。
}
```

### 4.2 顶级View对点击事件的处理过程

顶级View，setContentView所设置的View
```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    ...
    // Check for interception.
    final boolean intercepted;
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
        //ACTION_DOWN事件，进入if
        //其他ACTION事件，由于未拦截ACTION_DOWN，mFirstTouchTarget不为空，进入if
        
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
        if (!disallowIntercept) {
            //FLAG_DISALLOW_INTERCEPT标志可由子类控制，默认为false，ACTION_DOWN事件会重置该标志位
            //那么disallowIntercept为false，进入if，后续拦截与否看onInterceptTouchEvent
            intercepted = onInterceptTouchEvent(ev);
            ev.setAction(action); // restore action in case it was changed
        } else {
            //子类调用函数修改为true后，disallowIntercept为true，进入else，后续事件始终不拦截
            intercepted = false;
        }
    } else {
        //当拦截ACTION_DOWN并自己处理，则mFirstTouchTarget为空，
        //后续事件始终进入else，该View始终处理后续事件
        intercepted = true;
    }
    
    ...
    
    //当intercepted为false（即不拦截）时，继续如下
    final View[] children = mChildren;
    for (int i = childrenCount - 1; i >= 0; i--) {
        final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
        final View child = getAndVerifyPreorderedView(preorderedList, children, childIndex);
    
        if (childWithAccessibilityFocus != null) {
            if (childWithAccessibilityFocus != child) {
                continue;
            }
            childWithAccessibilityFocus = null;
            i = childrenCount - 1;
        }
    
        if (!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)){
            //子元素不能播动画 或者 点击事件坐标没有落在子元素的区域内，此时continue，下一个循环
            ev.setTargetAccessibilityFocus(false);
            continue;
        }
        
        //子元素能播动画 且 点击事件坐标落在子元素的区域内 继续执行以下代码
        newTouchTarget = getTouchTarget(child);
        if (newTouchTarget != null) {
            newTouchTarget.pointerIdBits |= idBitsToAssign;
            break;
        }   

        resetCancelNextUpFlag(child);
        //第三个参数child，不为null，执行child的dispatchTouchEvent，见代码块儿A
        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
            mLastTouchDownTime = ev.getDownTime();
            if (preorderedList != null) {
                for (int j = 0; j < childrenCount; j++) {
                    if (children[childIndex] == mChildren[j]) {
                        mLastTouchDownIndex = j;
                            break;
                    }
                }
            } else {
                mLastTouchDownIndex = childIndex;
            }
            mLastTouchDownX = ev.getX();
            mLastTouchDownY = ev.getY();
            //mFirstTouchTarget在addTouchTarget里被赋值，见代码块儿B
            newTouchTarget = addTouchTarget(child, idBitsToAssign);
            alreadyDispatchedToNewTouchTarget = true;
            break;
        }
    }
    ...
    
    
    if (mFirstTouchTarget == null) {
        //第三个参数是null，执行View的dispatchTouchEvent方法，见代码块儿A
        handled = dispatchTransformedTouchEvent(ev, canceled, null, TouchTarget.ALL_POINTER_IDS);
    }
    ...
```

代码块儿A dispatchTransformedTouchEvent
```
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
    ...
    if (child == null) {
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        ...
        handled = child.dispatchTouchEvent(transformedEvent);
    }
    ...
}
```

代码块儿B addTouchTarget
```
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;
    return target;
}
```
    
### 4.3 View对点击事件的处理过程

dispatchTouchEvent
    
```
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean result = false;
    ...
    if (onFilterTouchEventForSecurity(event)) {
        ...
        
        ListenerInfo li = mListenerInfo;
        //如果有OnTouchListener，则调用OnTouchListener的onTouch方法
        //并且当onTouch方法返回true，result为true，下一个语句中onTouchEvent就不会被调用
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }

        //如果上面的if条件为真，则result为true，则下面这个if条件为假，onTouchEvent也就不会执行
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }
    ...
    return result;
}
```

onTouchEvent

```
public boolean onTouchEvent(MotionEvent event) {
    ...
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        // A disabled view that is clickable still consumes the touch
        // events, it just doesn't respond to them.
        return (((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
    }
    //如果View设置有代理，还会执行TouchDelegate的onTouchEvent方法
    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
    
    ...
    
    //对点击事件的处理，只要CLICKABLE和LONG_CLICKABLE有一个为true，就会消耗这个事件
    //不管它是不是DISABLE状态
    if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
        switch (action) {
            //当ACTION_UP事件发生后，触发performClick方法
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                    ...
                    if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                        removeLongPressCallback();

                        // Only perform take click actions if we were in the pressed state
                        if (!focusTaken) {
                            // Use a Runnable and post this rather than calling
                            // performClick directly. This lets other visual state
                            // of the view update before click actions start.
                            if (mPerformClick == null) {
                                mPerformClick = new PerformClick();
                            }
                            if (!post(mPerformClick)) {
                                performClick();
                            }
                        }
                    }
                    ...
                }
                break;
            ...
        }
    }
    ...
    return true;
}
```

代码块儿 A

```
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    //如果View设置了OnClickListener，则调用它的onClick方法
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}
```

代码块儿 B

```
setClickable 可以改变View的CLICKABLE属性
setLongClickable 可以改变LONG_CLICKABLE属性
setOnClickListener 会将View的CLICKABLE属性设为true
setOnLongClickListener 会将View的LONG_CLICKABLE属性设为true
```

----------


## 5. View的滑动冲突

### 5.1 常见的滑动冲突场景

* 场景1 外部滑动与内部滑动方向不一致 
* 场景2 外部滑动与内部滑动方向一致
* 场景3 上面两种情况嵌套

### 5.2 滑动冲突的处理规则

场景1

* 水平方向和竖直方向的距离差
* 滑动路径和水平方向的夹角
* 水平和竖直方向的速度差

场景2

* 业务逻辑需求

### 5.3 滑动冲突的解决方式


#### 5.3.1 外部拦截法

父View->HorizontalScrollViewEx

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    boolean intercepted = false;
    int x = (int) ev.getX();
    int y = (int) ev.getY();
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            intercept = false;
            if (!mScroller.isFinished()) {
                mScroller.abortAnimation();
                intercept = true;
            }
            break;
        case MotionEvent.ACTION_MOVE:
            int deltaX = x - mLastInterceptX;
            int deltaY = y - mLastInterceptY;
            if (父容器需要当前点击事件) {
                intercepted = true;
            } else {
                intercepted = false;
            }
            break;
        case MotionEvent.ACTION_UP:
            intercept = false;
            break;
        default:
            break;
    }
    mLastInterceptX = x;
    mLastInterceptY = y;
    return intercepted;
}

```

#### 5.3.2 内部拦截法

父View->HorizontalScrollViewEx2

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        mLastX = x;
        mLastY = y;
        if (!mScroller.isFinished()) {
            mScroller.abortAnimation();
            return true;
        }
        return false;
    } else {
        return true;
    }
}
```

子View->ListViewEx

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    int x = (int) ev.getX();
    int y = (int) ev.getY();
    
    switch (ev.getAction()){
        case MotionEvent.ACTION_DOWN:
            parent.requestDisallowInterceptTouchEvent(true);
            break;
        case MotionEvent.ACTION_MOVE:
            int deltaX = x - mLastX;
            int deltaY = y - mLastY;
            if(父类需要此点击事件){
                parent.requestDisallowInterceptTouchEvent(false);
            }
            break;
        case MotionEvent.ACTION_UP:
            break;
        }
    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(ev);
}
```

具体Demo示例见/AndroidDevArt/SlidingConflictDemo

----------