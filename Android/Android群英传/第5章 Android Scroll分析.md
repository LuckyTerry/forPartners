# 第5章 Android Scroll分析

滑动效果是如何产生的
==

## Android坐标系

> * 屏幕左上角的顶点作为Android坐标系的原点
> * 向右X轴正方向
> * 向下Y轴正方向

    //
    getLocationOnScreen(int location[])
    //触控事件在Android坐标系中的横坐标
    getRawX()
    //触控事件在Android坐标系中的纵坐标
    getRawY() 

## 视图坐标系

> * 父视图左上角的顶点作为视图坐标系的原点
> * 向右X轴正方向
> * 向下Y轴正方向

    //触控事件在视图坐标系中的横坐标
    getX() 
    //触控事件在视图坐标系中的纵坐标
    getY() 

## MotionEvent

触控事件的类型

    //单指触摸按下
    public static final int ACTION_DOWN = 0;
    //单指触摸离开
    public static final int ACTION_UP = 1;
    //单指触摸移动
    public static final int ACTION_MOVE = 2;
    //触摸动作取消
    public static final int ACTION_CANCEL = 3;
    //触摸动作超出边界
    public static final int ACTION_OUTSIDE = 4;
    //多指触摸按下
    public static final int ACTION_POINTER_DOWN = 5;
    //多指触摸离开
    public static final int ACTION_POINTER_UP = 6;
    
触控事件的方法

    //相对于自身View
    getX()
    getY()
    //相对于屏幕
    getRawX()
    getRawY()
    //相对于父容器ViewGroup
    getLeft()
    getRight()
    getTop()
    getBottom()

---

实现滑动的三种(七种)方法
==

## layout()方法
    
    layout(getLeft() + offsetX,
             getTop() + offsetY,
               geteRight() + offsetX,
                 getBottom() + offsetY);

## offsetLeftAndRight() offsetTopAndBottom()

    offsetLeftAndRight(offsetX);
    offsetTopAndBottom(offsetY);

## LayoutParams

    LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
    layoutParams.leftMargin = getLeft() + offsetX();
    layoutParams.topMargin = getTop() + offsetY();
    setLayoutParams(layoutParams);
    //requestLayout();//也可使用这一句，其实setLayoutParams内部调用了requestLayout
    
    ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
    layoutParams.leftMargin = getLeft() + offsetX();
    layoutParams.topMargin = getTop() + offsetY();
    setLayoutParams(layoutParams);
    //requestLayout();

## scrollTo() scrollBy()

    //scrollTo() scrollBy()方法改变的是内容的位置
    ((View)getParent()).scrollBy(-offsetX, -offsetY);

## Scroller

可以将Scroller当成一个计算器，startScroll()传入开始坐标、偏移量、时间。Scroller根据每次调用computeScrollOffset()时所逝去的时间计算mCurrentX()、mCurrentY()，再交由View的Parent使用scrollTo()进行平移。
详细原理请参考Android开发艺术探讨3.3.1节

    //初始化
    mScroller = new Scroller(mContext);
    
    View viewGroup = ((View) getParent());
    mScroller.startScroll(viewGroup.getScrollX(), viewGroup.getScrollY(),
                            -viewGroup.getScrollX(), -viewGroup.getScrollY())
    invalidate();
    
    @Override
    public void computeScroll(){
        super.computeScroll();
        if(mScroller.computeScrollOffset()){
            ((View)getParent()).scrollTo(mScroller.getCurrentX(), mScroller.getCurrentY());
            invalidate();
        }
    }
    
## 属性动画

这个so easy了

    ObjectAnimator.ofFloat(mButton,"translationX",0,300).setDuration(3000).start();

## 究极方法之ViewDragHelper

    //初始化，ViewDragHelper通常定义在一个ViewGroup的内部，并通过静态工厂方法进行初始化
    mViewDragHelper = ViewDragHelper.create(this,callback);
	
	//拦截事件
	@Override
	public boolean onInterceptTouchEvent(MotionEvent event){
		return mViewDragHelper.shouldInterceptTouchEvent(event);
	}
	@Override
	public boolean onTouchEvent(MotionEvent event){
		mViewDragHelper.processTouchEvent(event);
		return true;
	}
	
	//处理computeScroll()
	@Override
	public void computeScroll(){
		if(mViewDragHelper.continueSettling(true)){
			ViewCompat.postInvalidateOnAnimation(this);
		}
	}
	
	//处理回调Callback
	private ViewDragHelper.Callback callback = new ViewDragHelper.Callback(){
		@Override
		public boolean tryCaptureView(View child, int pointerId){
			//如果当前触摸的child是mMainView时才开始检测
			return mMainView == child;
		}
		@Override
		public int clampViewPositonVertical(View child, int top, int dy){
			return top;//return 0;则表示不发生滑动。如果需要精确计算padding等属性时，就要对top进行一些处理
		}
		@Override
		public int clampViewPositonHorizontal(View child, int left, int dx){
			return left;
		}
		@Override
		public void onViewReleased(View releasedChild, float xvel, float yvel){
			super.onViewReleased(releasedChild, xvel, yvel);
			if(?){
				mViewDragHelper.smoothSlideViewTo(mMainView, 0, 0);
				ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);//与startScroll方法非常像
			}else{
				mViewDragHelper.smoothSlideViewTo(mMainView, 300, 0);
				ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
			}
		}
	}