# Material Design 控件

[TOC]

---

## Immersive Mode

相关API
```
Window#setFlags
View#setSystemUiVisibility (Android 3.0开始提供)
```

相关Flag
```
WindowManager.LayoutParams.FLAG_FULLSCREEN
隐藏状态栏

View.SYSTEM_UI_FLAG_VISIBLE API 14
默认标记

View.SYSTEM_UI_FLAG_LOW_PROFILE API 14
低调模式, 会隐藏不重要的状态栏图标

View.SYSTEM_UI_FLAG_LAYOUT_STABLE API 16
保持整个View稳定, 常和控制System UI悬浮, 隐藏的Flags共用, 使View不会因为System UI的变化而重新layout

View.SYSTEM_UI_FLAG_FULLSCREEN API 16
状态栏隐藏，效果同设置WindowManager.LayoutParams.FLAG_FULLSCREEN

View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN API 16
视图延伸至状态栏区域，状态栏上浮于视图之上

View.SYSTEM_UI_FLAG_HIDE_NAVIGATION API 14
隐藏导航栏

View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION API 16
视图延伸至导航栏区域，导航栏上浮于视图之上

View.SYSTEM_UI_FLAG_IMMERSIVE API 19
沉浸模式, 隐藏状态栏和导航栏, 并且在第一次会弹泡提醒, 并且在状态栏区域滑动可以呼出状态栏（这样会系统会清楚之前设置的View.SYSTEM_UI_FLAG_FULLSCREEN或View.SYSTEM_UI_FLAG_HIDE_NAVIGATION标志）。使之生效，需要和View.SYSTEM_UI_FLAG_FULLSCREEN，View.SYSTEM_UI_FLAG_HIDE_NAVIGATION中的一个或两个同时设置。

View.SYSTEM_UI_FLAG_IMMERSIVE_STIKY API 19
与上面唯一的区别是, 呼出隐藏的状态栏后不会清除之前设置的View.SYSTEM_UI_FLAG_FULLSCREEN或View.SYSTEM_UI_FLAG_HIDE_NAVIGATION标志，在一段时间后将再次隐藏系统栏）
```

隐藏状态栏

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        View decorView = getWindow().getDecorView();
        int option = View.SYSTEM_UI_FLAG_FULLSCREEN;
        decorView.setSystemUiVisibility(option);
        ActionBar actionBar = getSupportActionBar();
        actionBar.hide();
    }
}
```

透明状态栏

```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

if (Build.VERSION.SDK_INT >= 21) {
    View decorView = getWindow().getDecorView();
    int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
    decorView.setSystemUiVisibility(option);
    getWindow().setStatusBarColor(Color.TRANSPARENT);//也可以设置成灰色透明的，比较符合MD风格
}
ActionBar actionBar = getSupportActionBar();
actionBar.hide();
```

隐藏导航栏

```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

View decorView = getWindow().getDecorView();
int option = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION;
decorView.setSystemUiVisibility(option);
ActionBar actionBar = getSupportActionBar();
actionBar.hide();
```

透明导航栏

```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

if (Build.VERSION.SDK_INT >= 21) {
    View decorView = getWindow().getDecorView();
    int option = View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
    decorView.setSystemUiVisibility(option);
    getWindow().setNavigationBarColor(Color.TRANSPARENT);
}
ActionBar actionBar = getSupportActionBar();
actionBar.hide();
```

隐藏状态栏和导航栏

```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

View decorView = getWindow().getDecorView();
int option = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
        | View.SYSTEM_UI_FLAG_FULLSCREEN;
decorView.setSystemUiVisibility(option);
ActionBar actionBar = getSupportActionBar();
actionBar.hide();
```

透明状态栏和导航栏

```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

if (Build.VERSION.SDK_INT >= 21) {
    View decorView = getWindow().getDecorView();
    int option = View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
    decorView.setSystemUiVisibility(option);
    getWindow().setNavigationBarColor(Color.TRANSPARENT);
    getWindow().setStatusBarColor(Color.TRANSPARENT);
}
ActionBar actionBar = getSupportActionBar();
actionBar.hide();
```
沉浸模式（默认全屏 + 点击内容区域控制System UI的显示和隐藏）
即 training 所谓的 Use Non-Sticky Immersion
API19及以上使用，因为View.SYSTEM_UI_FLAG_IMMERSIVE从API19才引进

```
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.ActionBar;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.GestureDetector;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;

public class MainActivity extends AppCompatActivity {

    private static final int INITIAL_DELAY = 1500;
    private View mDecorView;
    private View mContentView;
    private ActionBar mActionBar;
    private GestureDetector mGestureDetector;
    
    private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            hideSystemUI();
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mContentView = LayoutInflater.from(this).inflate(R.layout.activity_immersive, null);
        setContentView(contentView);
        mDecorView = getWindow().getDecorView();

        mGestureDetector = new GestureDetector(ImmersiveActivity.this, new GestureDetector.SimpleOnGestureListener(){
            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                if ((mDecorView.getSystemUiVisibility() & View.SYSTEM_UI_FLAG_HIDE_NAVIGATION) == 0){
                    hideSystemUI();
                }else {
                    showSystemUI();
                }
                return true;
            }
        });

        mContentView.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                mGestureDetector.onTouchEvent(event);
                return true;
            }
        });

        mDecorView.setOnSystemUiVisibilityChangeListener(new View.OnSystemUiVisibilityChangeListener() {
            @Override
            public void onSystemUiVisibilityChange(int visibility) {
                if ((visibility & View.SYSTEM_UI_FLAG_HIDE_NAVIGATION) == 0){
                    //systemUI is visible
                    mActionBar.show();
                }else {
                    //systemUI is invisible
                    mActionBar.hide();
                }
            }
        });

        mActionBar = getSupportActionBar();
        mActionBar.setShowHideAnimationEnabled(true);
    
        showSystemUI();
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus){
            delayedHide(INITIAL_DELAY);
        }else {
            handler.removeMessages(0);
            showSystemUI();
        }
    }

    private void delayedHide(int delay){
        handler.removeMessages(0);
        handler.sendEmptyMessageDelayed(0, delay);
    }

    private void hideSystemUI(){
        mDecorView.setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                |View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                |View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                |View.SYSTEM_UI_FLAG_FULLSCREEN
                |View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                |View.SYSTEM_UI_FLAG_IMMERSIVE);
    }

    private void showSystemUI(){
        mDecorView.setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                |View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                |View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
    }
}
```

沉浸式模式（默认全屏 + 触摸滑动状态栏区域出现透明状态栏 + 触摸滑动导航栏区域出现导航栏）
即 training 所谓的 Use Sticky Immersion（API19及以上使用）
API19及以上使用，因为View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY从API19才引进

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus && Build.VERSION.SDK_INT >= 19) {
            View decorView = getWindow().getDecorView();
            decorView.setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);
        }
    }

}

//如果需要横屏，添加如下代码
<activity android:name=".MainActivity" 
          android:screenOrientation="landscape">
    ...
</activity>
```


----------


## DrawerLayout && NavigationView

抽屉式的导航控件
Drawerlayout继承自ViewGroup，NavigationView继承自FrameLayout
NavigationView一定需要放在最后面，因为NavigationView是在最上层

DrawerLayout的xml属性解析：
```
android:fitsSystemWindows="true" --> 设置状态栏透明化与否
tools:openDrawer="start" --> 不清楚作用（但是tools属性只由IDE使用，不会在代码中起作用）
```
NavigationView的xml属性解析：
```
android:layout_gravity="start" --> 设置抽屉（即NavigationView）从左边或是右边打开。
android:fitsSystemWindows="true" --> 
app:headerLayout="@layout/nav_header_main" --> 设置其头部的布局
app:menu="@menu/activity_main_drawer" --> 设置其菜单项
```

菜单项的xml属性解析：
```
<menu>
--> 声明其容纳的是菜单项

<group>
android:id --> 设置组id，只有给group设置了id，才会出现分割线。
android:checkableBehavior --> 设置选中策略（none|single）

<item>
android:id -->  设置菜单Item的id
android:icon --> 设置菜单Item的icon
android:title --> 设置菜单Item的名称
android:checked --> 设置菜单Item默认选中
```      

示例xml代码：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:openDrawer="start">

    <!--内容区-->
    <include
        layout="@layout/app_bar_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <!--左侧导航菜单-->
    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main"
        app:menu="@menu/activity_main_drawer"/>

</android.support.v4.widget.DrawerLayout>

```

layout/nav_header_main

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="@dimen/nav_header_height"
    android:background="@drawable/side_nav_bar"
    android:gravity="bottom"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:theme="@style/ThemeOverlay.AppCompat.Dark">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        app:srcCompat="@android:drawable/sym_def_app_icon"/>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        android:text="Android Studio"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1"/>

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="android.studio@android.com"/>

</LinearLayout>
```

menu/activity_main_drawer

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_camera"
            android:icon="@drawable/ic_menu_camera"
            android:title="Import"/>
        <item
            android:id="@+id/nav_gallery"
            android:icon="@drawable/ic_menu_gallery"
            android:title="Gallery"/>
        <item
            android:id="@+id/nav_slideshow"
            android:icon="@drawable/ic_menu_slideshow"
            android:title="Slideshow"/>
        <item
            android:id="@+id/nav_manage"
            android:icon="@drawable/ic_menu_manage"
            android:title="Tools"/>
    </group>

    <item android:title="Communicate">
        <menu>
            <item
                android:id="@+id/nav_share"
                android:icon="@drawable/ic_menu_share"
                android:title="Share"/>
            <item
                android:id="@+id/nav_send"
                android:icon="@drawable/ic_menu_send"
                android:title="Send"/>
        </menu>
    </item>

</menu>
```

示例java代码：
```
// 将DrawerLayout和Toolbar结合使用 之 方法一
ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(
                                this, mDrawerLayout, mToolbar, "open", "close");
mDrawerLayout.setDrawerListener(toggle);
toggle.syncState();
     
// 将DrawerLayout和Toolbar结合使用 之 方法二
mToolbar.setNavigationIcon(R.mipmap.ic_launcher);
mToolbar.setNavigationOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        mDrawerLayout.openDrawer(GravityCompat.START);
    }
});

// 添加DrawerLayout监听
mDrawerLayout.addDrawerListener(new DrawerLayout.DrawerListener() {
    @Override
    public void onDrawerSlide(View drawerView, float slideOffset) {
                
    }

    @Override
    public void onDrawerOpened(View drawerView) {

    }

    @Override
    public void onDrawerClosed(View drawerView) {

    }

    @Override
    public void onDrawerStateChanged(int newState) {

    }
});
        
...

mNavigationView.setNavigationItemSelectedListener(this);

@SuppressWarnings("StatementWithEmptyBody")
@Override
public boolean onNavigationItemSelected(MenuItem item) {
    // Handle navigation view item clicks here.
    int id = item.getItemId();

    if (id == R.id.nav_camera) {
        // Handle the camera action
    } else if (id == R.id.nav_gallery) {

    }

    DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
    drawer.closeDrawer(GravityCompat.START);
    return true;
}
```
----------


## CoordinatorLayout

官方介绍：
```
CoordinatorLayout is a super-poweredFrameLayout.
CoordinatorLayout is intended for two primary use cases:
1.As a top-level application decor or chrome layout
2.As a container for a specific interaction with one or more child views
```

它是一个增强版的FrameLayout，他可以协调其子View的交互，控制手势机滚动技巧。通常把CoordinatorLayout作为顶层布局来协调其子布局之间的动画效果。

CoordinatorLayout的使用核心是Behavior，Behavior就是执行你定制的动作。在讲Behavior之前必须先理解两个概念：Child和Dependency，什么意思呢？Child当然是子View的意思了，是谁的子View呢，当然是CoordinatorLayout的子View；其实Child是指要执行动作的CoordinatorLayout的子View。而Dependency是指Child依赖的View。比如上面的gif图中，蓝色的View就是Dependency，黄色的View就是Child，因为黄色的View的动作是依赖于蓝色的View。简而言之，就是如过Dependency这个View发生了变化，那么Child这个View就要相应发生变化。发生变化是具体发生什么变化呢？这里就要引入Behavior，Child发生变化的具体执行的代码都是放在Behavior这个类里面。

定义一个类MyBehavior，继承CoordinatorLayout.Behavior<T>，其中，泛型参数T是我们要执行动作的View类，也就是Child。然后就是去实现Behavior的两个方法：
```
// 此处Button是child，DependencyView是dependency
public class MyBehavior extends CoordinatorLayout.Behavior<Button> {
    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, Button child, View dependency) {
        return dependency instanceof DependencyView;
    }

    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, Button child, View dependency) {
        // 设置具体依赖行为
        
        return true;
    }
}

app:layout_behavior="com.terry.gank.MyBehavior"
```

常用xml属性解析：
```
android:fitsSystemWindows="true" --> 
```

常用情景1：结合AppBarLayout、ToolBar
```
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/main_content"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>

    </android.support.design.widget.AppBarLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end|bottom"
        android:layout_margin="15dp"
        android:src="@drawable/add_2"/>

</android.support.design.widget.CoordinatorLayout>
```

常用情景2：结合AppBarLayout、ToolBar(或ImageView)、TabLayout、ViewPager
```
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/main_content"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="250dp">

        <ImageView 
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="?attr/colorPrimary"
            android:scaleType="fitXY"
            android:src="@drawable/tangyan"
            app:layout_scrollFlags="scroll|enterAlways"/>

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentBottom="true"
            android:background="?attr/colorPrimary"
            app:tabIndicatorColor="@color/colorAccent"
            app:tabIndicatorHeight="4dp"
            app:tabSelectedTextColor="#000"
            app:tabTextColor="#fff"/>

    </android.support.design.widget.AppBarLayout>


    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end|bottom"
        android:layout_margin="15dp"
        android:src="@drawable/add_2"/>

</android.support.design.widget.CoordinatorLayout>
```

常用情景3：结合AppBarLayout、CollapsingToolbarLayout、ToolBar
```
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.terry.gank5.ScrollingActivity3">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                android:scaleType="centerCrop"                
                android:src="@mipmap/ic_launcher"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7" />
                
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay"/>

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" >

        <android.support.v7.widget.RecyclerView
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </android.support.v4.widget.NestedScrollView>


    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"
        app:srcCompat="@android:drawable/ic_dialog_email"/>

</android.support.design.widget.CoordinatorLayout>
```

----------


## AppBarLayout

AppBarLayout继承自LinearLayout，布局方向为垂直方向，它可以让包含在其中的子控件能响应被标记了ScrollingViewBehavior的滚动事件（要与CoordinatorLayout一起使用），利用它我们可以很容易的去实现视差滚动效果。简单的来说，AppBarLayout可以协调其子view随着同为CoordinatorLayout子view的兄弟view发生滚动手势的时候发生相应滚动。
　　
AppBarLayout常用xml属性解析：
```
android:layout_height="@dimen/app_bar_height" --> 设置AppBarLayout高度（即下拉到底的高度）
// 一般使用者wrap_content(前提是某个子view设置了具体数值)或者具体数值
        
android:theme="@style/AppTheme.AppBarOverlay" --> 设置AppBarLayout主题
// <style name="AppTheme.AppBarOverlay" parent="ThemeOverlay.AppCompat.Dark.ActionBar"/>

android:background --> 设置AppBarLayout背景图片或颜色
// 这里不建议设置该属性，因为会将设置的主题样式给改变了，若包含有CollapsingToolbarLayout，留给CollapsingToolbarLayout去做（可以设置背景颜色，但是不建议CollapsingToolbarLayout去设置背景图片，使用单独一个ImageView去加载这个图片比较好）
```

AppBarLayout使用方法如下：    
　　
第一步，给依赖View设置 app:layout_behavior 属性：
```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```

第二步，给AppBarLayout子View设置 app:layout_scrollFlags 属性，该属性有四个枚举值：
```
scroll --> 所有想滚动出屏幕的view都需要设置这个flag， 没有设置这个flag的view将被固定在屏幕顶部
enterAlways --> 设置这个flag时，向下的滚动都会导致该view变为可见，启用快速“返回模式”
enterAlwaysCollapsed --> 当你的视图已经设置minHeight属性又使用此标志时，你的视图只能已最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度
exitUntilCollapsed --> 滚动退出屏幕，最后折叠在顶端
snap --> 在Scroll滑动事件结束以前 ，如果这个View部分可见，那么这个View会停在最接近当前View的位置
```

----------


## CollapsingToolbarLayout

来源：由于Toolbar 只能固定到屏幕顶端并且不能做出好的动画效果，所以才有了这个Layout的出现。
功能：让Toolbar可伸缩，在伸缩的时候决定ToolBar是移除屏幕和固定在最上面。
简介：CollapsingToolbarLayout继承至FrameLayout，主要是用于实现折叠效果。它需要放在AppBarLayout布局里面，并且作为AppBarLayout的直接子View。

CollapsingToolbarLayout的常用xml属性解析：
```
android:fitsSystemWindows="true" --> 
android:background --> 背景图片或颜色（建议只用于设置背景颜色，想要展示图片单独用一个ImageView去加载比较好）
app:collapsedTitleTextAppearance --> 在收缩时Title文字外形设置 
app:expandedTitleTextAppearance --> 展开时Title文字外形的设置 
app:contentScrim --> 标题文字停留在顶部时候背景的设置 
app:collapsedTitleMarginStart --> 收缩时title向左填充的距离
app:expandedTitleMarginStart --> 展开时title向左填充的距离 
```

CollapsingToolbarLayout 通过设置layout_scrollFlags属性来控制其子View在响应layout_behavior事件时作出相应的scrollFlags滚动事件：
```
app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed|exitUntilCollapsed" 
```

CollapsingToolbarLayout 的子View中通过设置layout_collapseMode属性来响应折叠模式，该属性有两个枚举值：
```
app:layout_collapseMode="pin"：固定模式，在折叠的时候最后固定在顶端 
app:layout_collapseMode="parallax"：视差模式，在折叠的时候会有个视差折叠的效果
```

FloatingActionButton中通过设置以下两个属性，可以使FloatingActionButton跟随CollapsingToolbarLayout的收缩而显示隐藏（所依赖的AppBarLayout的id和相对AppBarLayout的摆放位置）。
```
app:layout_anchor="@id/appBarLayout"
app:layout_anchorGravity="bottom|right|end" 
```
----------


## Toolbar

Toolbar可以看做是android 5.0以后用来替代ActionBar的控件。

ToolBar使用前提：给ToolBar所在Activity使用如下不带ActionBar的主题样式
```
<style name="AppTheme.NoActionBar">
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
</style>

或者

<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>
```

ToolBar常用xml属性解析：
```
android:layout_height="?attr/actionBarSize" --> 设置ToolBar高度为ActionBar的高度

android:background="?attr/colorPrimary" --> 设置ToolBar背景颜色

app:navigationIcon="@drawable/menu_icon" --> 设置导航按钮
app:logo="@drawable/logo" --> 设置应用程序的标志logo

app:title="标题" --> 设置标题
app:titleTextColor="@color/white" --> 设置标题文字颜色
app:titleTextAppearance="@style/ToolbarTextAppearanceTitle" --> 设置标题文字样式
//<style name="ToolbarTextAppearanceTitle">
//     <item name="android:textSize">16sp</item>
//</style>

app:subtitle="子标题" --> 设置子标题
app:subtitleTextColor="@color/white" --> 设置子标题文字颜色
app:subtitleTextAppearance="@style/ToolbarTextAppearanceSubTitle" --> 设置子标题文字样式
//<style name="ToolbarTextAppearanceSubTitle">
//     <item name="android:textSize">14sp</item>
//</style>

app:popupTheme="@style/AppTheme.PopupOverlay" --> 设置ToolBar弹出菜单的主题样式
//<style name="AppTheme.PopupOverlay" parent="ThemeOverlay.AppCompat.Light"/>
```

ToolBar常用方法属性解析：
```
mAppCompatActivity.setSupportActionBar(toolbar); --> 设置支持Toolbar

mToolbar.inflateMenu(R.menu.toolbar_menu); --> 将menu加入到toolbar中，等效于如下Override
@Override
public boolean onCreateOptionsMenu(Menu menu) {
     getMenuInflater().inflate(R.menu.toolbar_menu, menu);
     return true;
}

mToolbar.setOnMenuItemClickListener() --> 设置菜单项的点击事件

mToolbar.setNavigationOnClickListener() --> 设置导航按钮的点击事件

mToolbar.setNavigationIcon() --> 设置导航按钮
mToolbar.setLogo() --> 设置应用程序的标志logo

mToolbar.setTitle() --> 设置标题
mToolbar.setTitleTextColor() --> 设置标题文字颜色
mToolbar.setTitleTextAppearance() --> 设置标题文字样式

mToolbar.setSubtitle() --> 设置子标题
mToolbar.setSubtitleTextColor() --> 设置子标题文字颜色
mToolbar.setSubtitleTextAppearance() --> 设置子标题文字样式
```

当使用在CollapsingToolbarLayout内部时（且CollapsingToolbarLayout在AppBarLayout内部时）设置：
```
app:layout_scrollFlags 属性以控制TooBar的滑动效果
app:layout_collapseMode 属性以控制TooBar的折叠效果。
```

----------


## SearchView

在menu/xxxx.xml的菜单布局文件将SearchView以菜单条目的方式加入到ToolBar中
```
<item
    android:id="@+id/action_search"
    android:title="@String/action_search"
    android:icon="@drawable/ic_action_search"
    app:showAsAction="ifRoom|collapseActionView"
    app:actionViewClass="android.support.v7.widget.SearchView" />
```

```
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_test, menu);
    MenuItem menuItem = menu.findItem(R.id.action_search);
        
    // 设置打开关闭动作监听，一般不用设置
    MenuItemCompat.setOnActionExpandListener(menuItem, new MenuItemCompat.OnActionExpandListener() {
        @Override
        public boolean onMenuItemActionExpand(MenuItem item) {
            Toast.makeText(MainActivity.this, "Expand", Toast.LENGTH_LONG).show();
            return true;
        }
        @Override
        public boolean onMenuItemActionCollapse(MenuItem item) {
            Toast.makeText(MainActivity.this, "Collapse", Toast.LENGTH_LONG).show();
            return true;
        }
    });
        
    // 设置menuItem点击动作监听，一般不用设置
    menuItem.setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
        @Override
        public boolean onMenuItemClick(MenuItem item) {
            Toast.makeText(MainActivity.this, "Click", Toast.LENGTH_LONG).show();
            return true;
        }
    });

    // SearchView相关操作，必要
    SearchView searchView = (SearchView) MenuItemCompat.getActionView(menuItem);
    searchView.setQueryHint("搜索...");
    searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
        @Override
        public boolean onQueryTextSubmit(String query) {
            Toast.makeText(MainActivity.this, "query: " + query, Toast.LENGTH_LONG).show();
            return false;
        }

        @Override
        public boolean onQueryTextChange(String newText) {
            Toast.makeText(MainActivity.this, "newText: " + newText, Toast.LENGTH_LONG).show();
            return false;
        }
    });
        
    return super.onCreateOptionsMenu(menu);
}
```
至此，其实就已经实现了一个基础的搜索功能。但是，如果为了能够让自己的应用的某些功能被Android系统的Search功能检索到，我们就需要做更进一步的操作，例如定义Searchable，实现一个SearchableActivity，响应系统的Search行为等等。国内的应用很少会去关注这个功能，这里就不展开了，感兴趣点击下面的链接进一步学习：https://developer.android.com/guide/topics/search/index.html

----------


## TabLayout

官方版ViewPagerIndicator，由大神JackWharton主导开发，一般都和ViewPager一起使用。

TabLayout常用xml属性解析：
```
android:background --> 
app:tabBackground  --> 
app:tabIndicatorColor --> 设置指示器颜色 
app:tabTextColor --> 设置标签文字颜色
app:tabSelectedTextColor --> 设置当前选中的标签文字颜色 

```

TabLayout的使用方法：

第零步：xml中引用，设置app:layout_scrollFlags属性可以让
```
<android.support.design.widget.TabLayout
        android:id="@+id/tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabBackground="?attr/colorPrimary"
        app:tabTextColor="?attr/cursorTextColor"
        app:tabSelectedTextColor="@color/white" />
   
<android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="?attr/backgroundColor"/>
```


第一步：配置ViewPager
```
mViewPager.setAdapter(mPagerAdapter);
```

第二步：配置TabLayout
```
mTabLayout.setTabMode(TabLayout.MODE_SCROLLABLE);//适合很多tab
//mTabLayout.setTabMode(TabLayout.MODE_FIXED);//tab均分,适合少的tab

mTabLayout.setTabGravity(TabLayout.GRAVITY_FILL);//tab均分,适合少的tab
//mTabLayout.setTabGravity(TabLayout.GRAVITY_CENTER);

mTabLayout.setOnTabSelectedListener (); //添加监听 

...等等...
```

第三步：循环添加Tab标题
```
mTabLayout.addTab(mTabLayout.newTab().setText("title1"));
mTabLayout.addTab(mTabLayout.newTab().setText("title2"));
mTabLayout.addTab(mTabLayout.newTab().setText("title3));
```

或者 重写ViewPager适配器的getPageTitle()方法以添加Tab标题
```
@Override
public CharSequence getPageTitle(int position) {
    return mTitleList.get(position);
}
```

或者 xml中直接定义（前提是每个tabItem都预先知道）
```
<android.support.design.widget.TabLayout
        android:id="@+id/tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabBackground="?attr/colorPrimary"
        app:tabTextColor="?attr/cursorTextColor"
        app:tabSelectedTextColor="@color/white" >
        
     <android.support.design.widget.TabItem
             android:text="title1"/>

     <android.support.design.widget.TabItem
             android:text="title2"/>

 </android.support.design.widget.TabLayout>
```

第四步，建立TabLayout和ViewPager的联系

```
mTabLayout.setupWithViewPager(mViewPager);
// 跟踪其源码发现内部调用如下两句核心代码，故如果使用setupWithViewPager出现问题可以使用如下代码
// mViewPager.addOnPageChangeListener(new TabLayout.TabLayoutOnPageChangeListener(mTabLayout));
// mTabLayout.addOnTabSelectedListener(new TabLayout.ViewPagerOnTabSelectedListener(mViewPager));
```
----------


## SwipeRefreshLayout

SwipeRefreshLayout常用方法解析：
```
setEnable() --> 设置刷新是否可用，true可用，false不可用
setRefreshing(boolean) --> 设置刷新状态，true表示正在刷新，false表示取消刷新
isRefreshing() --> 判断刷新状态，true表示正在刷新，false表示没有刷新
setOnRefreshListener() --> 设置监听
setProgressBackgroundColorSchemeResource() --> 设置下拉进度条的背景颜色，默认白色
setColorSchemeResources() --> 设置下拉进度条的颜色主题，参数为可变参数，并且是资源id，可以设置多种不同的颜色，每转一圈就显示一种颜色
```

SwipeRefreshLayout使用方法：
```
// 配置SwipeRefreshLayout
mSwipeRefreshView.setProgressBackgroundColorSchemeResource(android.R.color.white);
mSwipeRefreshView.setColorSchemeResources(R.color.colorAccent, R.color.colorPrimary, R.color.colorPrimaryDark);

// 下拉时触发SwipeRefreshLayout的下拉动画，动画完毕之后就会回调这个方法
mSwipeRefreshView.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
    @Override
    public void onRefresh() {
        1. mSwipeRefreshView.setRefreshing(true);
        2. 异步线程获取数据
        3. 切换到UI线程
        4. 显示数据
        5. mSwipeRefreshView.setRefreshing(false);
      }
  });
```

拓展延伸 --> [自定义View继承SwipeRefreshLayout，添加上拉加载更多功能][1]

----------


## NestedScrollView


----------


## CardView


----------


## RecyclerView


----------


## TextInputLayout

TextInputLayout继承自LinearLayout。EditText有一个叫hint的属性，它可以提示用户此处应该输入什么内容，然而当用户输入真实内容之后，hint的提示内容就消失了，用户的体验效果是十分不好的。使用TextInputLayout包裹EditText，这样当用户在输入的时候hint的内容就会跑到输入内容的上边去，其中TextInputLayout中字体的颜色是style文件中的colorAccent，还可以提示输入error信息。

TextInputLayout常用方法解析：
```
setHint() --> 设置提示语
setErrorEnabled() --> 设置错误信息是否显示。true显示，false不显示
setError() --> 设置错误显示信息，需要先设置setErrorEnabled(true)
getEditText() --> 得到EditText的控件实例
```

TextInputLayout使用方法示例：
xml
```
<!--TextInputLayout的颜色来自style中的colorAccent的颜色-->
<android.support.design.widget.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <EditText
        android:id="@+id/edit_username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="用户名/手机号"
        android:inputType="textEmailAddress" />
</android.support.design.widget.TextInputLayout>
```

java
```
//mTextInputLayout.setHint("请输入用户名");  
// 获取TextInputLayout下的输入框  
mEditText = mTextInputLayout.getEditText();  
// 设置对EditText输入的监听事件  
mEditText.addTextChangedListener(new TextWatcher() {  
    @Override  
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {  
    }
  
    @Override  
    public void onTextChanged(CharSequence s, int start, int before, int count) {  
        if(s.length() < 5){  
            mTextInputLayout.setErrorEnabled(true);  
            mTextInputLayout.setError("用户名不能小于6位");  
        }else{  
            mTextInputLayout.setErrorEnabled(false);  
        }  
    }  
  
    @Override  
    public void afterTextChanged(Editable s) {  
    }  
});  
```
----------

## TextInputEditText

TextInputEditText继承自AppCompatEditText（后者继承自EditText）。

TextInputEditText常用方法解析：
```
setError(“密码不能为空”) --> 设置错误提醒的文字（与TextInputLayout.setError()显示效果不一样）
```

TextInputEditText使用方法示例：

xml
```
<TextInputEditText
    android:id="@+id/edit_username"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="用户名/手机号"
    android:inputType="textEmailAddress" />
```

java
```
mTextInputEditText.addTextChangedListener(new TextWatcher() {  
    @Override  
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {  
    }
  
    @Override  
    public void onTextChanged(CharSequence s, int start, int before, int count) {  
        if(s.length() < 5){  
            mTextInputEditText.setError("用户名不能小于6位");  
        }
    }  
  
    @Override  
    public void afterTextChanged(Editable s) {  
    }  
});  
```

TextInputEditText拓展之 与TextInputLayout搭配使用

xml
```
<android.support.design.widget.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <TextInputEditText
        android:id="@+id/edit_username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="用户名/手机号"
        android:inputType="textEmailAddress" />
</android.support.design.widget.TextInputLayout>
```

java
```
mTextInputEditText.setError(“密码不能为空”)；//错误提醒的文字

或者使用： 
mTextInputLayout.setErrorEnabled(true); //开启错误提醒 
mTextInputLayout.setError(“密码不能为空”); //错误提醒的文字 
mTextInputLayout.setErrorEnabled(false); //关闭错误提醒
```


----------


## FloatingActionButton

FloatingActionButton继承自ImageView，从名字可以看出它是一个浮动的按钮，它是一个带有阴影的圆形按钮，可以通过fabSize来改变其大小，主要负责界面的基本操作。另外FloatingActionButton默认使用FloatingActionButton.Behavior。

常用属性
```
android:layout_gravity --> 设置 button 的放置位置（一般使用 "bottom|end"）
android:layout_margin --> 设置 button 的margin（一般使用 "@dimen/fab_margin"，即16dp）
android:src --> 设置 button 的drawable
app:srcCompat --> 设置 button 的drawable（与android:src的区别是......）
app:backgroundTint --> 设置 button 的背景颜色
app:fabSize --> 设置 button 的大小尺寸（"normal|mini"）
app:rippleColor --> 设置点击 button 时候的颜色（水波纹效果）
app:borderWidth --> 设置 button 的边框宽度
app:elevation --> 设置普通状态阴影的深度（默认是 6dp）
app:pressedTranslationZ --> 设置点击状态的阴影深度（默认是 12dp）
app:layout_anchor --> 设置链接于某个View（一般使用 "@id/appBarLayout"）
app:layout_anchorGravity --> 设置链接于某个View哪个位置（"top|bottom|start|end"）
```

常用方法
```
setBackgroundTintList(ColorStateList tint) 设置 button 背景颜色。
```

示例xml代码1
```
<android.support.design.widget.FloatingActionButton
        android:id="@+id/fab_search"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email"
        app:borderWidth="2dp"
        app:fabSize="normal"
        app:rippleColor="#ff0000" />
```

示例xml代码2

```
//在FloatingActionButton中通过设置以下两个属性，可以使FloatingActionButton跟随CollapsingToolbarLayout的收缩而显示隐藏（所依赖的AppBarLayout的id和相对AppBarLayout的摆放位置）。
一些属性：
<android.support.design.widget.FloatingActionButton
        android:id="@+id/fab_search"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_anchor="@id/appBarLayout"
        app:layout_anchorGravity="bottom|right|end" />
```

拓展：实现滑动列表时，下滑显示和上滑隐藏的效果（继承FloatingActionButton.Behavior进行重写）
```
// 自定义Behavior
public class FloatingActionButtonScrollBehavior extends FloatingActionButton.Behavior {
    public FloatingActionButtonScrollBehavior(Context context, AttributeSet attrs) {
        super();
    }

    @Override
    public boolean onStartNestedScroll(final CoordinatorLayout coordinatorLayout, final
    FloatingActionButton child, final View directTargetChild, final View target, final int
            nestedScrollAxes) {
        // 确保是竖直判断的滚动
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL || super.onStartNestedScroll
                (coordinatorLayout, child, directTargetChild, target, nestedScrollAxes);
    }

    @Override
    public void onNestedScroll(final CoordinatorLayout coordinatorLayout, final
    FloatingActionButton child, final View target, final int dxConsumed, final int dyConsumed,
                               final int dxUnconsumed, final int dyUnconsumed) {
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed,
                dxUnconsumed, dyUnconsumed);
        if (dyConsumed > 0 && child.getVisibility() == View.VISIBLE) {
            child.hide();
        } else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
            child.show();
        }
    }
}

// values/styles文件中定义
<string name="fab_behavior" translatable="false">com.terry.gank5.FloatingActionButtonScrollBehavior</string>

// 使用
<android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        app:layout_behavior="@string/fab_behavior"
        app:srcCompat="@android:drawable/ic_dialog_email"/>
```
----------

## SnackBar

SnackBar通过在屏幕底部展示简洁的信息，为一个操作提供了一个轻量级的反馈，并且在Snackbar中还可以包含一个操作，在同一时间内，仅且只能显示一个Snackbar，它的显示依赖于UI，不像Toast那样可以脱离应用显示。它的用法和Toast很相似，唯一不同的就是它的第一个参数不是传入Context而是传入它所依附的父视图（建议使用CoordinatorLayout或其子View作为父视图传入，但实测传入子View之AppbarLayout会报错，其它好像都行），但是他比Toast更强大。

Snackbar常用方法解析：
```
setActionTextColor() --> 设置动作按钮颜色
setAction() --> 设置动作按钮监听
show() --> 显示Snackbar
getView() --> 获取 snackbar 视图
```

Snackbar使用方法：
```
Snackbar.make(mCoordinatorLayout, "SnackbarClicked", Snackbar.LENGTH_SHORT)
    .setAction("Action", new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(MainActivity.this, "I'm a Toast", Toast.LENGTH_SHORT).show();
        }
    })
    .setActionTextColor(Color.RED)
    .show();
```

Snackbar拓展之 修改Snackbar样式
```
// 设置动作按钮颜色
snackbar.setActionTextColor(getResources().getColor(android.R.color.holo_green_light));

// 获取snackbar视图
View snackbarView = snackbar.getView();

// 设置snackbar背景色
snackbarView.setBackgroundColor(Color.GRAY);

// 设置snackbar文本颜色
TextView tv = (TextView) snackbarView.findViewById(android.support.design.R.id.snackbar_text);
tv.setTextColor(getResources().getColor(android.R.color.holo_green_light));
```

Snackbar拓展之 添加icon
```
// 获取snackbar视图
View snackbarView = snackbar.getView();

// 定义icon
ImageView iconImage = new ImageView(MainActivity.this);
iconImage.setImageResource(R.mipmap.ic_launcher);

// 定义icon的布局参数
ViewGroup.LayoutParams vl = snackbarView.getLayoutParams();
Snackbar.SnackbarLayout.LayoutParams sl = new Snackbar.SnackbarLayout.LayoutParams(vl.WRAP_CONTENT, vl.WRAP_CONTENT);
sl.gravity = Gravity.CENTER_VERTICAL;

// icon应用该布局参数
iconImage.setLayoutParams(sl);

// snackbar视图插入icon
Snackbar.SnackbarLayout snackbarLayout = (Snackbar.SnackbarLayout) snackbarView;
snackbarLayout.addView(iconImage, 0);
```

Snackbar拓展之 改变Snackbar的位置
```
// 获取snackbar视图
View snackbarView = snackbar.getView();

// 定义snackbar视图的布局参数
ViewGroup.LayoutParams vl = snackbarView.getLayoutParams();
CoordinatorLayout.LayoutParams cl = new CoordinatorLayout.LayoutParams(vl.width, vl.height);
cl.gravity = Gravity.CENTER_VERTICAL;

// snackbar视图应用该布局参数
snackbarView.setLayoutParams(cl);
```

----------


## BottomNavigationView

BottomNavigationView 底部菜单只能是3-5个，如果个数少于3个或者多于5个，都会报错。

xml属性解析

```
android:layout_height --> 设置底部导航栏高度（默认"wrap_content"，即56dp）
app:itemIconTint --> 设置菜单图标颜色（默认@color/colorPrimary）
app:itemTextColor --> 设置菜单文本颜色（默认@color/colorPrimary）
app:itemBackground --> 设置菜单背景颜色（默认当前样式的背景色（白色/黑色））
app:menu --> 设置菜单布局
```

xml示例代码

```
<android.support.design.widget.BottomNavigationView
    android:id="@+id/bottomNavigationView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:itemIconTint="@color/bottom_item_text_color"
    app:itemTextColor="@color/bottom_item_text_color"
    app:menu="@menu/menu_bottom_navigation_items"/>
```

color/bottom_item_text_color.xml
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="?android:attr/textColorPrimary" android:state_checked="true"/>
    <item android:color="?android:attr/textColorPrimary" android:state_pressed="true"/>
    <item android:color="?android:attr/textColorSecondary"/>
</selector>
```

menu/menu_bottom_navigation_items.xml
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/item1"
        android:checked="true"
        android:icon="@mipmap/ic_launcher"
        android:title="Text1"/>
    <item
        android:id="@+id/item2"
        android:icon="@mipmap/ic_launcher"
        android:title="Text2"/>
    <item
        android:id="@+id/item3"
        android:icon="@mipmap/ic_launcher"
        android:title="Text3"/>
</menu>
```

java
```
BottomNavigationView bottomNavigationView = (BottomNavigationView) findViewById(R.id.bottomNavigationView);
bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()) {
            case R.id.item1:
                break;
            case R.id.item2:
                break;
            case R.id.item3:
                break;
        }
        return false;
    }
});
```


----------


## DialogFragment


----------


## BottomSheetDialog

BottomSheetDialog指从底部弹出的对话框。

与PopupWindow比较：PopupWindow要实现背景透明效果，必须使用代码进行设置，但是BottomSheetDialog不用，它的默认效果就是这样。

与DialogFragment比较：使用方便，效果不错，低版本上兼容性DialogFragment好点

使用 -->

BottomSheetDialog布局layout/layout_bottom

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="test" />
        
</LinearLayout>
```

BottomSheetActivity

```
BottomSheetDialog dialog = new BottomSheetDialog(BottomSheetActivity.this);
View dialogView = LayoutInflater.from(BottomSheetActivity.this)
                                .inflate(R.layout.layout_bottom, null);
dialog.setContentView(dialogView);
dialog.show();
```

当弹出的layout是一个ListView/RecyclerView的时候，若item比较多，弹出对话框时只会显示几个item，向上拖动时，才会显示全部item

----------


## BottomSheetBehavior

BottomSheetBehavior的常用方法解析

```
静态方法BottomSheetBehavior.from(View) --> 获取BottomSheetBehavior实例
实例方法getState() --> 获取BottomSheetBehavior实例的state
实例方法setState() --> 设置BottomSheetBehavior实例的state

有如下5种state
/**
 * The bottom sheet is dragging.
 */
public static final int STATE_DRAGGING = 1;

/**
 * The bottom sheet is settling.
 */
public static final int STATE_SETTLING = 2;

/**
 * The bottom sheet is expanded.
 */
public static final int STATE_EXPANDED = 3;

/**
 * The bottom sheet is collapsed.
 */
public static final int STATE_COLLAPSED = 4;

/**
 * The bottom sheet is hidden.
 */
public static final int STATE_HIDDEN = 5;
```

目标布局view应该设置如下属性，BottomSheetBehavior.from(view)才能获取到正确的实例对象

```
app:layout_behavior="@string/bottom_sheet_behavior"
```

xml

```
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:popupTheme="@style/AppTheme.PopupOverlay" />
    </android.support.design.widget.AppBarLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@+id/tab_layout"
        android:gravity="center"
        android:orientation="vertical"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <Button
            android:id="@+id/btn_bottom_sheet_control"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="sheet 显示/隐藏" />

    </LinearLayout>

    <LinearLayout
        android:id="@+id/bottom_sheet_layout"
        android:layout_width="match_parent"
        android:layout_height="?actionBarSize"
        android:layout_alignParentBottom="true"
        android:background="@android:color/holo_purple"
        app:elevation="4dp"
        app:behavior_hideable="true"
        app:behavior_peekHeight="300dp"
        app:layout_behavior="@string/bottom_sheet_behavior">

        <Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:text="第一" />

        <Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:text="第二" />

        <Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:text="第三" />

        <Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:text="第四" />
    </LinearLayout>

</android.support.design.widget.CoordinatorLayout>
```

java

```
mBottomSheetBehavior = BottomSheetBehavior.from(findViewById(R.id.bottom_sheet_layout));
mBottomSheetBehavior.setHideable(true);
mBottomSheetBehavior.setState(BottomSheetBehavior.STATE_HIDDEN);
Button button = (Button) findViewById(R.id.btn_bottom_sheet_control);
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        if (mBottomSheetBehavior.getState() == BottomSheetBehavior.STATE_HIDDEN) {
            mBottomSheetBehavior.setState(BottomSheetBehavior.STATE_COLLAPSED);
        } else {
            mBottomSheetBehavior.setState(BottomSheetBehavior.STATE_HIDDEN);
        }
    }
});

// app:behavior_hideable="true"
// mBottomSheetBehavior.setHideable(true);
// 以上两句至少设置一句，如果没有设置能够隐藏，setState隐藏会报错
```




----------


## 尺寸规范


  [1]: http://www.jianshu.com/p/d23b42b6360b