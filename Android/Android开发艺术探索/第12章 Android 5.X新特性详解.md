# 第12章 Android 5.X新特性详解

[TOC]

原文讲解较为基础，故整合了相关知识点

Android 5.X UI特色
==

> * 材料的形态模拟
> * 更加真实的动画
> * 大色块的使用

---

主题和布局
==

Material Design 主题
--

三种默认主题

    @android:style/Theme.Material
    @android:style/Theme.Material.Light
    @android:style/Theme.Material.Light.DarkActionBar

style配置：colorPrimary，ActionBar背景色；colorPrimaryDark，状态栏背景色；colorAccent，content背景色

    //style.xml中配置，推荐这个，可以进行其他属性配置
    <resources>
        <style name="AppTheme" parent="Theme.Material.Light.DarkActionBar">
            <item name="colorPrimary">@color/colorPrimary</item>
            <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
            <item name="colorAccent">@color/colorAccent</item>
        </style>
    </resources>
    
    //或者在AndroidManifest.xml中直接设置主题
    android:theme="@android:style/Theme.Material.Light"  
    
    

布局
--

CoordinatorLayout

    <android.support.design.widget.CoordinatorLayout                                            
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/line_coordinatorLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        android:background="#ffffff" >

        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:theme="@style/AppTheme.AppBarOverlay">
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary" />
        </android.support.design.widget.AppBarLayout>
        
        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/line_swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">
            <android.support.v7.widget.RecyclerView
                android:id="@+id/line_recycler"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />
        </android.support.v4.widget.SwipeRefreshLayout>

    </android.support.design.widget.CoordinatorLayout>

    coordinatorLayout=(CoordinatorLayout)findViewById(R.id.line_coordinatorLayout); 
    Snackbar.make(coordinatorLayout,"hello", Snackbar.LENGTH_SHORT).show();

Toolsbar

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary" />

    </android.support.design.widget.AppBarLayout>
    
    Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
    toolbar.setLogo(R.drawable.ic_launcher);
    toolbar.setTitle("");
    toolbar.setSubTitle("");
    setSupportActionBar(toolbar);//模拟出ActionBar效果

SwipeRefreshLayout

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/line_swipe_refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <android.support.v7.widget.RecyclerView
            android:id="@+id/line_recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
          />
    </android.support.v4.widget.SwipeRefreshLayout>
    
    swipeRefreshLayout=(SwipeRefreshLayout) findViewById(R.id.line_swipe_refresh) ;
    swipeRefreshLayout.setColorSchemeResources(R.color.colorPrimary,R.color.colorPrimaryDark,R.color.colorAccent);
    swipeRefreshLayout.setProgressViewOffset(false, 0,  (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 24, getResources().getDisplayMetrics()));
        
    swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
        @Override
        public void onRefresh() {

        }
    }); 
        
    swipeRefreshLayout.setRefreshing(true);
    swipeRefreshLayout.setRefreshing(false);
    
DrawerLayout  ---DrawerToggle状态变化的动态效果
    
    <include layout="@layout/toolbar"/>
    
    <android.support.v4.widget.DrawerLayout
	    android:id=""
	    android:layout_width=""
	    android:layout_height="">
	
	    <!--内容界面-->
	    <Linearlayout
		    android:layout_width=""
		    android:layout_height=""
		    android:orientation="" >
		    <Button
			    android:layout_width=""
			    android:layout_height=""
			    android:text=""/>
	    </Linearlayout>
	
	    <!--侧滑菜单界面-->
	    <Linearlayout
		    android:layout_width=""
		    android:layout_height=""
		    android:layout_gravity="start"
		    android:orientation="" >
		    <Button
			    android:layout_width=""
			    android:layout_height=""
			    android:text=""/>
	    </Linearlayout>
	
    </android.support.v4.widget.DrawerLayout>	
    
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);   
    mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer);.
    mDrawerToggle = new ActionBarDrawerToggle(
                this, mDrawerLayout, mToolbar,
                R.string.aabc_action_bar_home_description,
                R.string.aabc_action_bar_home_description_format);
    mDrawerToggle.syncState();
    mDrawerLayout.setDrawerListener(mDrawToggle);
	
---


视图和阴影
==

视图高度
--

垂直屏幕向外为Z轴正方向
elevation 静态成员
translationZ 实现动画
Z = elevation + translationZ

    android:elevation="xxdp"
    View.setElevation(xx);
    View.setTranslationZ(xx);
    ViewPropertyAnimator.z = xx;
    ViewPropertyAnimator.translationZ = xx;

阴影和轮廓
--

在Android L中设置一个阴影很简单，只需要两点：
1.设置eleavation值
2.添加一个背景或者outline


  
Tinting（着色）
--

修改图像的Alpha遮罩来修改图像的颜色，有三种效果：

    android:tint="@android:color/holo_blue_bright"
    
    android:tint="@android:color/holo_blue_bright"
    android:tintMode="add"
    
    android:tint="@android:color/holo_blue_bright"
    android:tintMode="multiply"
    

Clipping（剪裁）
--

    ViewOutlineProvider viewOutlineProvider = new ViewOutlineProvider(){
        @Override
        public void getOutline(View view, Outline outline){
            outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), 30);
        }
    }
    mImageView.setOutlineProvider(viewOutlineProvider);
    
Palette
--

> * Vibrant(充满活力的)
> * Vibrant dark(充满活力的黑)
> * Vibrant light(充满活力的亮)
> * Muted(柔和的)
> * Muted dark(柔和的黑)
> * Muted light(柔和的亮)

    //引用
    com.android.support:palette-v7:21.0.2
    
    //使用
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.test);
    Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener(){
        @Override
        public void onGenerated(Palette palette){
            Palette.Swatch vibrant = palette.getDarkVibrantSwatch();
            getActionBar().setBackgroundDrawable(new ColorDrawable(vibrant.getRgb()));
            getWindow().setStatusBarColor(vibrant.getRgb());
        }
    })
    
    //六种方法提取不同颜色
    palette.getVibrantSwatch();
    palette.getDarkVibrantSwatch();
    palette.getLightVibrantSwatch();
    palette.getMutedSwatch();
    palette.getDarkMutedSwatch();
    palette.getLightMutedSwatch();

---

UI控件
==

    

1. RecyclerView
--

LayoutManager使用

    mLayoutManager=new LinearLayoutManager(this);
    mLayoutManager=new LinearLayoutManager(this, orientation);
    mLayoutManager=new GridLayoutManager(this, spanCount);
    mLayoutManager=new GridLayoutManager(this, spanCount, orientation);
    mLayoutManager=new StaggeredGridLayoutManager(spanCount, orientation);
    recyclerview.setLayoutManager(mLayoutManager);

Adapter使用

    class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> implements View.OnClickListener {
        @Override
        public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
                View view = LayoutInflater.from(LineActivity.this)
                                            .inflate(R.layout.line_meizi_item, parent, false);
                MyViewHolder holder = new MyViewHolder(view);
                view.setOnClickListener(this);
                return holder;
        }
        @Override
        public void onBindViewHolder(MyViewHolder holder, int position) {
            holder.tv.setText("图"+position);
            Picasso.with(LineActivity.this).load(meizis.get(position).getUrl()).into(holder.iv);
        }
        @Override
        public int getItemCount(){
            return meizis.size();
        }
        @Override
        public void onClick(View v) {
            int position=recyclerview.getChildAdapterPosition(v);
            Snackbar.make(coordinatorLayout,"点击第"+position+"个", nackbar.LENGTH_SHORT).show();
        }
        class MyViewHolder extends RecyclerView.ViewHolder{
            private ImageView iv;
            private TextView tv;
            public MyViewHolder(View view){
                super(view);
                iv = (ImageView) view.findViewById(R.id.line_item_iv);
                tv=(TextView) view.findViewById(R.id.line_item_tv);
            }
        }
        public void addItem(Meizi meizi, int position) {
            meizis.add(position, meizi);
            notifyItemInserted(position);
            recyclerview.scrollToPosition(position);
        }
        public void removeItem(final int position) {
            final Meizi removed=meizis.get(position);
            meizis.remove(position);
            notifyItemRemoved(position);
            SnackbarUtil.ShortSnackbar(coordinatorLayout,"你删除了第"+position+"个item",SnackbarUtil.Warning).setAction("撤销", new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    addItem(removed,position);
                    SnackbarUtil.ShortSnackbar(coordinatorLayout,"撤销了删除第"+position+"个item",SnackbarUtil.Confirm).show();
                }
            }).setActionTextColor(Color.WHITE).show();
        }
    }
    
RecyclerView的滚动监听
    
    recyclerview.addOnScrollListener(new RecyclerView.OnScrollListener() {
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
            if (newState == RecyclerView.SCROLL_STATE_IDLE
                        && mLastVisibleItem + 2 >= mLayoutManager.getItemCount()) {
                
            }
        }
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            mLastVisibleItem = mLayoutManager.findLastVisibleItemPosition();
        }
    });
    
RecyclerView的点击事件监听

    public OnItemClickListener itemClickListener;
    public void setOnItemClickListener(OnItemClickListener itemClickListener){
        this.itemClickListener = itemClickListener;
    }
    public interface OnItemClickListener{
        void onItemClick(View view, int position);
    }
    //在OnClick()中使用如下
    if(itemClickListener != null){
        itemClickListener.onItemClick(v, getPosition());
    }
    
ItemTouchHelper使用

    itemTouchHelper=new ItemTouchHelper(new ItemTouchHelper.Callback() {
        @Override
        public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
            int dragFlags=0, swipeFlags=0;
            if(recyclerView.getLayoutManager() instanceof StaggeredGridLayoutManager){
                dragFlags=ItemTouchHelper.UP|ItemTouchHelper.DOWN|ItemTouchHelper.LEFT|ItemTouchHelper.RIGHT;
            }else if(recyclerView.getLayoutManager() instanceof LinearLayoutManager){
                dragFlags=ItemTouchHelper.UP|ItemTouchHelper.DOWN;
                //设置侧滑方向为从左到右和从右到左都可以
                swipeFlags = ItemTouchHelper.START|ItemTouchHelper.END;
            }
            return makeMovementFlags(dragFlags,swipeFlags);
        }
        @Override
        public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
                int from=viewHolder.getAdapterPosition();
                int to=target.getAdapterPosition();
                Collections.swap(meizis,from,to);
                mAdapter.notifyItemMoved(from,to);
                return true;
        }
        @Override
        public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
                mAdapter.removeItem(viewHolder.getAdapterPosition());
        }
        @Override
        public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
            super.onSelectedChanged(viewHolder, actionState);
            if(actionState==ItemTouchHelper.ACTION_STATE_DRAG){
                viewHolder.itemView.setBackgroundColor(Color.LTGRAY);
            }
        }
        @Override
        public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
            super.clearView(recyclerView, viewHolder);
            viewHolder.itemView.setBackgroundColor(Color.WHITE);
        }
        @Override
        public void onChildDraw(Canvas c, RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive) {
            viewHolder.itemView.setAlpha(1- Math.abs(dX)/screenwidth);
            super.onChildDraw(c, recyclerView, viewHolder, dX, dY, actionState, isCurrentlyActive);
        }
    });

2. CardView
--

    <android.support.v7.widget.CardView 
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id=""
        android:layout_width=""
        android:layout_height=""
        android:elevation=""
        android:layout_marginTop=""
        android:layout_marginLeft=""
        android:layout_marginRight=""
        card_view：cardBackgroundColor="@color/card_view_backgroud"
        card_view：cardCornerRadius="8dp" >
        <TextView 
            android:layout_width=""
            android:layout_height=""
            android:text="" />
    </android.support.v7.widget.CardView> 

3. Notification
--

    //基本的Notification
    Notification.Builder builder = new Notification.Builder(this);
    Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
    PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
    builder.setSmallIcon(R.drawable.ic_launcher);
    builder.setContentIntent(pi);
    builder.setAutoCancel(true);
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher));
    builder.setContentTitle("Basic Notifications");
    builder.setContentText("I am a basic notificaiton");
    builder.setSubText("it is really basic");
    NotificationManager notificaitonManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    notificaitonManager.notify(NOTIFICATION_ID_BASIC, builder.build());

    //折叠式Notification
    
    //拥有两个视图状态 一个是普通状态下的视图状态，一个是展开状态下的视图状态
    //notification.xml
    <Linearlayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center_horizontal"
        android:orientation="vertical" >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
		<ImageView
		    android:src="@drawable/ic_launcher"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </Linearlayout>
    
    //notification_expand.xml
    <Linearlayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center_horizontal"
        android:orientation="vertical" >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
		<ImageView
		    android:src="@drawable/ic_launcher"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
		<ImageView
		    android:src="@drawable/ic_launcher"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </Linearlayout>
    
    Notification.Builder builder = new Notification.Builder(this);
    Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.sina.com"));
    PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
    builder.setSmallIcon(R.drawable.ic_launcher);
    builder.setContentIntent(pi);
    builder.setAutoCancel(true);
    builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher));
    Notification notification = builder.build();
    
    RemoteViews contentView = new RemoteViews(getPackageName(),R.layout.notification);
    contentView.setTextViewText(R.id.textView, "show me when collapsed");
    notification.contentView = contentView;
    
    RemoteViews expandedView = new RemoteViews(getPackageName(),R.layout.notification_expand);
    expandedView.setTextViewText(R.id.textView, "i am now expanded");
    notification.bigContentView = expandedView;
    
    NotificationManager notificaitonManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    notificaitonManager.notify(NOTIFICATION_ID_COLLAPSE, notification);
    
    //悬挂式Notification
    Notification.Builder builder = new Notification.Builder(this);
    builder.setSmallIcon(R.drawable.ic_launcher);
    builder.setPriority(Notification.PRIORITY_DEFAULT);
    builder.setCategory(Notification.CATEGORY_MESSAGE);
    builder.setContentTitle("Headsup Notification");
    builder.setContentText("I am a Headsup notification.");

    Intent intent = new Intent();
    intent.setClass(this,MainActivity.class);
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    PendingIntent pi = PendingIntent.getActivity(this, 0, pi, PendingIntent.FLAG_CANCEL_CURRENT);
    builder.setFullScreenIntent(pi, true);//貌似是这一句起决定性作用？

    NotificationManager notificaitonManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    notificaitonManager.notify(NOTIFICATION_ID_HEADSUP, builder.build());
    
    //显示等级的Notification
    builder.setVisibility(otification.VISIBILITY_PUBLIC);//任何情况都显示
    builder.setVisibility(otification.VISIBILITY_PRIVATE);//没有锁屏时显示
    builder.setVisibility(otification.VISIBILITY_SECRECT);//安全锁和屏幕都没有锁定时显示

---

动画
==

1. Activity transitions（Activity转换效果）
--

1.1 进入和退出效果
--

explode（分解）------从屏幕中间进或出
slide（滑动）------从屏幕边缘进或出
fade（淡出）------改变屏幕上视图的不透明度


代码中使用

    //Activity A 启动
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this).toBundle());

    //Activity B 设置允许使用Transition
    getWindow().requeastFeature(WINDOW.FEATURE_CONTENT_TARNSITIONS);
    
    //Activity A 设置离开动画
    getWindow().setExitTransition(new Explode());
    getWindow().setExitTransition(new Slide);
    getWindow().setExitTransition(new Fade());

    //Activity B 设置进入动画
    getWindow().setEnterTransition(new Explode());
    getWindow().setEnterTransition(new Slide());
    getWindow().setEnterTransition(new Fade());
    
xml中使用

    //Activity A 启动
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this).toBundle());

    <style name="myTheme" parent="android:Theme.Material">  
        <!-- 允许使用transitions -->  
        <item name="android:windowContentTransitions">true</item>  
      
        <!-- 指定进入和退出transitions动画 -->  
        <item name="android:windowEnterTransition">@transition/explode</item>  
        <item name="android:windowExitTransition">@transition/explode</item>  
    </style> 

1.2 共享元素
--

changeBounds------改变目标视图的布局边界
changeClipBounds------裁剪目标视图边界
changeTransform------改变目标视图的缩放比例和旋转角度
changeImageTransform------改变目标视图的大小和缩放比例



代码中使用

    //transitionName命名必须相同，系统才能找到共享元素
    android:transitionName="XXX"
    android:transitionName="XXX"
    android:transitionName="YYY"
    android:transitionName="YYY"

    //Activity A 启动
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this,view,"XXX").toBundle());
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this,Pair.create(view,"XXX"),Pair.create(view,"YYY")).toBundle());

    //Activity B 设置允许使用Transition
    getWindow().requeastFeature(WINDOW.FEATURE_CONTENT_TARNSITIONS);

    //共享元素transition的进入、退出效果
    getWindow().setSharedElementEnterTransition();
    getWindow().setSharedElementExitTransition();

xml中使用
    
    //transitionName命名必须相同，系统才能找到共享元素
    android:transitionName="XXX"
    android:transitionName="XXX"
    android:transitionName="YYY"
    android:transitionName="YYY"
    
    //Activity A 启动
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this,view,"XXX").toBundle());
    startActivity(intent,ActivityOptions.makeSceneTranslationAnimation(this,Pair.create(view,"XXX"),Pair.create(view,"YYY")).toBundle());
    
    <style name="myTheme" parent="android:Theme.Material">  
        <!-- 允许使用transitions -->  
        <item name="android:windowContentTransitions">true</item>  
      
        <!-- 指定共享元素transitions -->  
        <item name="android:windowSharedElementEnterTransition">  
            @transition/change_image_transform</item>  
        <item name="android:windowSharedElementExitTransition">  
            @transition/change_image_transform</item>  
    </style> 

    //定义要使用的动画change_image_transform.xml
    <transitionSet xmlns:android="http://schemas.android.com/apk/res/android">  
        <explode/>  
        <changeBounds/>  
        <changeTransform/>  
        <changeClipBounds/>  s
        <changeImageTransform/>  
    </transitionSet>  
    
当需要结束当前Activity并回退这个动画时调用Activity.finishAfterTransition()方法

2. Touch feedback（触摸反馈）
--
## 2.1 Ripple效果

    //波纹有边界
    android:background="?android:attr/selectableItemBackground"
    //波纹超出边界
    android:background="?android:attr/selectableItemBackgroundBorderless"
    
    android:colorControlHighlight="" 设置波纹颜色
    android:colorAccent="" 设置checkbox等控件的选中颜色
    
    //自定义ripple.xml
    <ripple xmlns:android="http://schemas.android.com/apk/res/android"
    		android:color="@android:color/holo_blue_bright">
    		<item>
    			<shape android:shape="oval">
    				<solid android:color="?android:colorAccent">
    			</shape>
    		</item>
    </ripple>
    android:background="@drawable/ripple"

## 2.2 Circular Reveal

    public static Animator createCircularReveal(
    	    View view,
    	    int centerX/*动画开始的中心点X*/,
    	    int centerY/*动画开始的中心点Y*/,
    	    float startRadius/*动画开始半径*/,
    	    float endRadius/*动画结束半径*/){
        return new RevealAnimator(view, centerX, centerY, startRadius, endRadius);	
    }
    //eg1
    Animator animator = ViewAnimationUtils.createCircularReveal(
    	oval,
    	oval.getWidth() / 2,
    	oval.getHeight() / 2,
    	oval.getWidth(),
    	0);
    animator.setInterpolator(new AccelerateDecelerateInterpolator());
    animator.setDuration(2000);
    animator.start();
    //eg2
    Animator animator = ViewAnimationUtils.createCircularReveal(
    	rect,
    	0,
    	0,
    	0,
    	(float) Math.hypot((rect.getWidth(), rect.getHeight())));
    animator.setInterpolator(new AccelerateInterpolator());
    animator.setDuration(2000);
    animator.start();

3. View state changes （视图状态改变）
--
## 3.1 View state changes Animation

StateListAnimator

    //定义
    <selector xmlns:android="http://schemas.android.com/apk/res/android">
         <item android:state_pressed="true">
		    <set>
		        <objectAnimator 
		            android:propertyName="rotationX"
		            android:duration="@android:integer/config_shortAnimTime"
		            android:valueTo="360"
		            android:valueType="floatType"/>
		    <set/>
	    </item>
	    
	     <item android:state_pressed="false">
		    <set>
		        <objectAnimator 
		            android:propertyName="rotationX"
		            android:duration="@android:integer/config_shortAnimTime"
		            android:valueTo="0"
		            android:valueType="floatType"/>
		    <set/>
	    </item>
    <selector/>
    
    //xml中使用
    <Button
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:stateListAnimator="@drawable/anim_change"/>
        
    //代码中使用 原书AnimationInflater错误，应该是AnimatorInflater
    StateListAnimator animator = AnimatorInflater.loadStateListAnimator(this, R.drawable.anim_change);
    view.setStateListAnimator(animator);

animated-selector

    //定义
    <animated-selector xmlns:android="http://schemas.android.com/apk/res/android">

        <item android:id="@+id/state_on" android:state_checked="true">
		    <bitmap android:src="@drawable/ic_done_anim_30"/>
	    </item>
	
	    <item android:id="@+id/state_off">
		    <bitmap android:src="@drawable/ic_plus_anim_30"/>
	    </item>

        <transition android:fromId="@+id/state_on" android:toId="@+id/state_off">
            <animation-list>
                <item android:duration="16">
				    <bitmap android:src="@drawable/ic_plus_anim_00"/>
			    </item>
                <item android:duration="16">
			    	<bitmap android:src="@drawable/ic_plus_anim_01"/>
		    	</item>
			    ...
			    ...
                <item android:duration="16">
			    	<bitmap android:src="@drawable/ic_plus_anim_29"/>
			    </item>
                <item android:duration="16">
			    	<bitmap android:src="@drawable/ic_plus_anim_30"/>
			    </item>
            </animation-list>
        </transition>
	
	    <transition android:fromId="@+id/state_off" android:toId="@+id/state_on">
            <animation-list>
                <item android:duration="16">
		    		<bitmap android:src="@drawable/ic_done_anim_00"/>
		    	</item>
                <item android:duration="16">
		    		<bitmap android:src="@drawable/ic_done_anim_01"/>
		    	</item>
		    	...
		    	...
                <item android:duration="16">
		    		<bitmap android:src="@drawable/ic_done_anim_29"/>
		    	</item>
                <item android:duration="16">
		    		<bitmap android:src="@drawable/ic_done_anim_30"/>
		    	</item>
            </animation-list>
        </transition>

    </animated-selector>
    
    //代码中使用
    private static final int[] STATE_CHECKED = new int[]{android.R.attr.state_checked};
    private static final int[] STATE_UNCHECKED = new int[]{};

    mDrawable = getResources().getDrawable(R.drawable.fab_anim);
    mImageView.setImageDrawable(mDrawable);

    mImageView.setImageState(STATE_CHECKED,true);
    mImageView.setImageState(STATE_UNCHECKED,true);

4. Reveal effect（揭露效果）
--

略

5. Curved motion（曲线运动）
--

略

6. Animate Vector Drawables（可绘矢量动画）
--

略





