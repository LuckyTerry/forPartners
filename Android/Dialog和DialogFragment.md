# Dialog和DialogFragment

---

## 一、背景

Android 官方推荐使用 DialogFragment 来代替 Dialog ，可以让它具有更高的可复用性（降低耦合）和更好的便利性（很好的处理屏幕翻转的情况）。
## 二、Dialog
```
public class FullScreenDialog extends Dialog {
    public FullScreenDialog(Context context) {
        super(context);
      }
}
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        mWindow = getWindow();
        
        //无标题
        requestWindowFeature(Window.FEATURE_NO_TITLE);// 内部即调用以下代码
        mWindow.requestFeature(Window.FEATURE_NO_TITLE);

        //透明状态栏
        mWindow.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);

        //退出,进入动画
        mWindow.setWindowAnimations(getAnimStyles());

        //清理背景变暗 
        mWindow.clearFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND);

        //点击window外的区域 是否消失
        setCanceledOnTouchOutside(canCanceledOnOutside());

        //是否可以取消,会影响上面那条属性
        setCancelable(canCancelable());

        //window外可以点击,不拦截窗口外的事件
        mWindow.addFlags(WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL);

        //设置背景颜色,只有设置了这个属性,宽度才能全屏MATCH_PARENT
        mWindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        WindowManager.LayoutParams mWindowAttributes = mWindow.getAttributes();
        mWindowAttributes.width = getWindowWidth();//这个属性需要配合透明背景颜色,才会真正的 MATCH_PARENT
        mWindowAttributes.height = WindowManager.LayoutParams.WRAP_CONTENT;

        //gravity
        mWindowAttributes.gravity = getGravity();
        mWindow.setAttributes(mWindowAttributes);
        
        // 隐藏标题栏
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        或 mWindow.requestFeature(Window.FEATURE_NO_TITLE);
        
        // 设置ContentView
        setContentView(R.layout.dialog_dish_note);

        // 按空白处不能取消对话框
        setCanceledOnTouchOutside(false);
        
        // 不能取消对话框
        setCancelable(false);

    }
```
## 三、DialogFragment基本用法
创建 DialogFragment 有两种方式：覆写其 onCreateDialog或覆写其 onCreateView

### 1. 覆写其 onCreateDialog

#### 1.1 创建一个 Dialog 并返回它即可：
```
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
  	//为了样式统一和兼容性，可以使用 V7 包下的 AlertDialog.Builder
    AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
  	// 设置主题的构造方法
	// AlertDialog.Builder builder = new AlertDialog.Builder(getActivity(), R.style.CustomDialog);
    builder.setTitle("注意：")
           .setMessage("是否退出应用？")
           .setNegativeButton("取消", null)
           .setPositiveButton("确定", null)
           .setCancelable(false);
    //builder.show(); // 不能在这里使用 show() 方法，应该外部调用
    return builder.create();
}
```
#### 1.2 使用自定义 View 来创建：
```
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
	LayoutInflater inflater = getActivity().getLayoutInflater();  
    View view = inflater.inflate(R.layout.fragment_dialog, null);  
	AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
  	// 设置主题的构造方法
	// AlertDialog.Builder builder = new AlertDialog.Builder(getActivity(), R.style.CustomDialog);
    builder.setView(view) 
 	// do something
    return builder.create();
}
```
```
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    // inflate布局
	LayoutInflater inflater = getActivity().getLayoutInflater();
    View view = inflater.inflate(R.layout.fragment_dialog, null);
    
    // 创建dialog
	Dialog dialog = new Dialog(getActivity());
  	// Dialog dialog = new Dialog(getActivity(), R.style.CustomDialog);
  	
  	// 隐藏标题栏，setContentView() 之前调用
	dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
	
	// 设置ContentView
    dialog.setContentView(view);
    
    // 设置点击外部区域不可关闭
    dialog.setCanceledOnTouchOutside(true);
    
     // 设置宽度为屏宽、位置靠近屏幕底部
	Window window = dialog.getWindow();
	window.setBackgroundDrawableResource(R.color.transparent);
	WindowManager.LayoutParams wlp = window.getAttributes();
	wlp.gravity = Gravity.BOTTOM;
	wlp.width = WindowManager.LayoutParams.MATCH_PARENT;
  	wlp.height = WindowManager.LayoutParams.WRAP_CONTENT;
	window.setAttributes(wlp);
    
  	// do something
  	
	return dialog;
}
```

### 2. 覆写其 onCreateView

```
/**
 * 设置主题需要在 onCreate() 方法中调用 setStyle() 方法
 */
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	 // 隐藏标题栏（）
	 // 还有STYLE_NORMAL|STYLE_NO_FRAME|STYLE_NO_INPUT可选，具体看源码
	setStyle(DialogFragment.STYLE_NO_TITLE, R.style.CustomDialog);
}
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	
	// 设置点击外部区域不可取消
	getDialog().setCanceledOnTouchOutside(true);
	
	// 设置不可取消
	getDialog().setCancelable(true);
	
    // inflate布局
	View rootView = inflater.inflate(R.layout.fragment_dialog, container, false);
	
        
	// do something
	
	// return即设置ContentView
	return rootView;
}

@Override
public void onActivityCreated(Bundle savedInstanceState) {

    super.onActivityCreated(savedInstanceState);

	// 设置宽度为屏宽、靠近屏幕底部。
    Window window = getDialog().getWindow();
    window.setBackgroundDrawableResource(android.R.color.transparent);
    window.getDecorView().setPadding(0, 0, 0, 0);
    WindowManager.LayoutParams wlp = window.getAttributes();
    wlp.gravity = Gravity.BOTTOM;
    wlp.width = WindowManager.LayoutParams.MATCH_PARENT;
    wlp.height = WindowManager.LayoutParams.WRAP_CONTENT;
    window.setAttributes(wlp);
    // 设置宽度为自定义
    Window window = getDialog().getWindow();
    window.setBackgroundDrawableResource(R.color.self);
    window.getDecorView().setPadding(0, 0, 0, 0);
    WindowManager.LayoutParams wlp = window.getAttributes();
    wlp.gravity = Gravity.CENTER;
    wlp.width = 400;
    wlp.height = 300;
    window.setAttributes(wlp);

}
```


## 四、 全屏Dialog实现原理

```
<style name="Dialog.FullScreen" parent="Theme.AppCompat.Dialog">
    <item name="android:windowIsFloating">false</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowNoTitle">true</item>
</style>
```
第一个属性：windowIsFloating
Dialog主题与Activity主题，两者都是继承Theme
```
 <style name="Theme">
    ...
    <item name="windowIsFloating">false</item>
</style>
```
Activity采用了默认的false，而Dialog的一般是进行了覆写，为True，如下：
```
<style name="Base.V7.Theme.AppCompat.Dialog" parent="Base.Theme.AppCompat">
    ...
    <item name="android:windowIsFloating">true</item>
</style>
```
Window被新建的时候，WindowManager.LayoutParams默认采用的是MATCH_PARENT，但是如果windowIsFloating 被设置为True，WindowManager.LayoutParams参数中的尺寸就会被设置成WRAP_CONTENT

第二个属性: android:windowBackground
这个属性如果采用默认值，设置会有黑色边框，其实这里主要是默认背景的问题，默认采用了有padding的InsetDrawable,设置了一些边距，导致上面的状态栏，底部的导航栏，左右都有一定的边距

第三个属性：android:windowNoTitle
为什么需要在setContentView之前设置该属性？
答：setContentView会进一步调用generateLayout创建根布局，Android系统默认实现了多种样式的根布局应，以应对不同的场景，选择的规则就是用户设置的主题样式（Window属性），比如需不需要Title，而布局样式在选定后就不能再改变了（大小可以），有些属性是选择布局文件的参考，如果是在setContentView之后再设定，就是失去了意义，另外Android也不允许在选定布局后，设置一些影响布局选择的属性，会抛出异常。
## 五、 全屏dialog实现具体方法
### 1. Dialog

利用Theme主题来实现全屏对话框
```
    public FullScreenDialog(@NonNull Context context) {
        super(context, R.style.Dialog_FullScreen);
    }
```

利用java代码属性来实现全屏对话框
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.dialog_test);
    // 设置全屏、需在setContentView之后设置
    WindowManager.LayoutParams attributes = getWindow().getAttributes();
    getWindow().setBackgroundDrawableResource(android.R.color.transparent);
    attributes.gravity = Gravity.BOTTOM;
    attributes.width = WindowManager.LayoutParams.MATCH_PARENT;
    attributes.height = WindowManager.LayoutParams.MATCH_PARENT;
    getWindow().setAttributes(attributes);
    }
```

### 2. DialogFragment 之 onCreateDialog

利用Theme主题来实现全屏对话框
```
<style name="Dialog.FullScreen" parent="Theme.AppCompat.Dialog">
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowIsFloating">false</item>
</style>

@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setStyle(STYLE_NORMAL, R.style.Dialog_FullScreen);
}
```

利用java代码属性来实现全屏对话框
```

```

### 3. DialogFragment 之 onCreateView

利用Theme主题来实现全屏对话框
```
<style name="Dialog.FullScreen" parent="Theme.AppCompat.Dialog">
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowIsFloating">false</item>
</style>
    
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setStyle(STYLE_NORMAL, R.style.Dialog_FullScreen);
}
```

利用java代码属性来实现全屏对话框
```

```

### 4.因屏幕旋转导致重启时DialogFragment内回调的保存处理
```
        Type type = getClass().getGenericSuperclass();
        if (type instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) type;
            Class<T> listenerType = (Class<T>) parameterizedType.getActualTypeArguments()[0];
            if (listenerType.isInstance(getTargetFragment())) {
                mDialogListener = listenerType.cast(getTargetFragment());
            } else if (listenerType.isInstance(getParentFragment())) {
                mDialogListener = listenerType.cast(getParentFragment());
            } else if (listenerType.isInstance(getActivity())) {
                mDialogListener = listenerType.cast(getActivity());
            }
        }
```

