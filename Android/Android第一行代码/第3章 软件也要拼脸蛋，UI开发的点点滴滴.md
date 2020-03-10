# 第3章 软件也要拼脸蛋，UI开发的点点滴滴


------

## 常见控件的使用方法

### 1. TextView

    <TextView
        android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textSize="24sp"
        android:textColor="#00ff00"
        android:text="This is TextView" />

### 2. Button

    <Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Button" />
        
    button.setOnClickListener(...);

### 3. EditText

    <EditText
        android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Type something here"
        android:inputType="textPassword"
        android:maxLines="2"/>
        
    String inputText = editText.getText().toString();

### 4. ImageView

    <ImageView
        android:id="@+id/image_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_launcher"/>
        
    imageView.setImageResource(R.drawable.jelly_bean);

### 5. ProgressBar

标准progressBar

    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    
横向显示进度的progressBar
    
    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="?android:attr/progressBarStyleHorizontal"
        android:max="100"/>

基础使用方法

    if (progressBar.getVisibility() == View.GONE) {
        progressBar.setVisibility(View.VISIBLE);
    } else {
        progressBar.setVisibility(View.GONE);
    }
    
    int progress = progressBar.getProgress();
    progressBar.setProgress(progress);

### 6. AlertDialog

    AlertDialog.Builder dialog = new AlertDialog.Builder(MainActivity.this);
    dialog.setTitle("This is Dialog");
    dialog.setMessage("Something important.");
    dialog.setCancelable(false);
    dialog.setPositiveButton("OK", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
        }
    });
    dialog.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
        }
    });
    dialog.show();


### 7. ProgressDialog

    ProgressDialog progressDialog = new ProgressDialog(MainActivity.this);
    progressDialog.setTitle("This is ProgressDialog");
    progressDialog.setMessage("Loading...");
    progressDialog.setCancelable(true);
    progressDialog.show();

------

## 四种基本布局

### 1. LinearLayout

LinearLayout 又称作线性布局，是一种非常常用的布局

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
    
    </LinearLayout>

### 2. RelativeLayout

RelativeLayout 又称作相对布局，也是一种非常常用的布局

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        
    </RelativeLayout>

相对于父布局进行定位

    android:layout_alignParentTop
    android:layout_alignParentBottom
    android:layout_alignParentLeft
    android:layout_alignParentRight
    android:layout_centerInParent

相对于子控件的位置

    android:layout_above
    android:layout_below
    android:layout_toLeftOf
    android:layout_toRightOf

相对于子控件对齐

    android:layout_alignTop 
    android:layout_alignBottom
    android:layout_alignLeft 
    android:layout_alignRight



### 3. FrameLayout

FrameLayout 这种布局没有任何的定位方式，所有的控件都会摆放在布局的左上角。

    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    
    </FrameLayout>

### 4. TableLayout

    <TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" 
        android:stretchColumns="1" >
        <TableRow>
            <TextView
                android:layout_height="wrap_content"
                android:text="Account:" />
            <EditText
                android:id="@+id/account"
                android:layout_height="wrap_content"
                android:hint="Input your account" />
        </TableRow>
        <TableRow>
            <Button
                android:id="@+id/login"
                android:layout_height="wrap_content"
                android:layout_span="2"
                android:text="Login" />
        </TableRow>
    </TableLayout>
    
android:stretchColumns="1"将第二列进行拉伸
android:layout_span="2"让按钮占据两列的空间

------

## include标签

新建一个布局 title.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/title_bg" >
        <Button
            android:id="@+id/title_back"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="5dip"
            android:background="@drawable/back_bg"
            android:text="Back"
            android:textColor="#fff" />
        <TextView
            android:id="@+id/title_text"
            android:layout_width="0dip"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_weight="1"
            android:gravity="center"
            android:text="Title Text"
            android:textColor="#fff"
            android:textSize="24sp" />
        <Button
            android:id="@+id/title_edit"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="5dip"
            android:background="@drawable/edit_bg"
            android:text="Edit"
            android:textColor="#fff" />
    </LinearLayout>

在另一个activity_main.xml中使用如下

    <include layout="@layout/title" />

------

## merge标签

作用：减少布局层级结构

a.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
        
        <Button
            android:id="@+id/login"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    
    </LinearLayout>
    
b.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
        
        <include layout="@layout/a" />
    
    </LinearLayout>

很明显，b的LinearLayout和a的LinearLayout嵌套增加了复杂度，此时可以将a的LinearLayout换成merge，被include进b后相当于直接使用login按钮

------

## StubView标签

略

------

## 自定义控件

熟练使用接口实现回调函数

    public class TitleLayout extends LinearLayout {
        private OnButtonClickListener mOnButtonClickListener;
        public interface OnButtonClickListener{
            void onLeftButtonClick();
            void onRightButtonClick();
        }
        public void setOnButtonClickListener(OnButtonClickListener listener){
            mOnButtonClickListener = listener;
        }
        public TitleLayout(Context context, AttributeSet attrs) {
            super(context, attrs);
            LayoutInflater.from(context).inflate(R.layout.title, this);
            Button titleBack = (Button) findViewById(R.id.title_back);
            Button titleEdit = (Button) findViewById(R.id.title_edit);
            titleBack.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    if(mOnButtonClickListener != null){
                        mOnButtonClickListener.onLeftButtonClick();
                    }
                }
            });
            titleEdit.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    if(mOnButtonClickListener != null){
                        mOnButtonClickListener.onRightButtonClick();
                    }
                }
            });
        }
    }

------

## ListView

基本使用

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
        <ListView
            android:id="@+id/list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent" >
        </ListView>
    </LinearLayout>

    private String[] data = { "Apple", "Banana", "Orange", "Watermelon",
                                    "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango" };
    ArrayAdapter<String> adapter = new ArrayAdapter<String>(
                                        MainActivity.this,android.R.layout.simple_list_item_1, data);
    ListView listView = (ListView) findViewById(R.id.list_view);
    listView.setAdapter(adapter);
    listView.setOnItemClickListener(new OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            String fruit = data[position];
            Toast.makeText(MainActivity.this, fruit, Toast.LENGTH_SHORT).show();
        }
    });

    
复用converView,使用ViewHolder。
    
    ViewHolder viewHolder = null;
    if (convertView == null) {
        convertView = LayoutInflater.from(getContext()).inflate(resourceId, null);
        viewHolder = new ViewHolder();
        viewHolder.fruitImage = (ImageView) convertView.findViewById(R.id.fruit_image);
        viewHolder.fruitName = (TextView) convertView.findViewById(R.id.fruit_name);
        view.setTag(viewHolder); // 将ViewHolder存储在View中
    } else {
        viewHolder = (ViewHolder) convertView.getTag(); // 重新获取ViewHolder
    }
    viewHolder.fruitImage.setImageResource(fruit.getImageId());
    viewHolder.fruitName.setText(fruit.getName());
    return convertView;

------

## 单位和尺寸

### px

像素,即屏幕中可以显示的最小元素单元

### pt 

磅数,1 磅等于 1/72 英寸，一般 pt 都会作为字体的单位来使用。

### dp

设备独立像素,也被称作 dip（device independent pix）

### sp

可伸缩像素,一般作为字体的单位来使用

### 密度

屏幕每英寸所包含的像素数，通常以 dpi 为单位

    int width = getResources().getDisplayMetrics().widthPixels;
    int height = getResources().getDisplayMetrics().heightPixels;
    
    DisplayMetrics metrics = new DisplayMetrics();
    getWindowManager().getDefaultDisplay().getMetrics(metrics);
    int width = metrics.widthPixels;
    int height = metrics.heightPixels;

### Android 的规定

在 160dpi 的屏幕上，1dp 等于 1px

------

## Nine-Patch 图片

在 Android sdk 目录下有一个tools文件夹，使用draw9patch.bat文来制作 Nine-Patch 图片的。
上边框和左边框黑点标记的区域就表示拉伸区域，下边框和右边框绘制的部分则表示内容会被放置的区域。
文件名为 message_left.9.png 的图片按以下方式使用，非android:background="@drawable/message_left.9"

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/message_left" >
        
    </LinearLayout>




