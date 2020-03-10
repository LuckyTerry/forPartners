# Android Studio 精品插件使用指南（A-Z）


[TOC]

---

## [1. ADB WIFI][1]
1. 功能：
无线调试应用

2. 使用：
将安卓手机与电脑连在同一无线网络下；
将手机用usb数据线连接至电脑，确保手机打开USB调试；
点击工具栏 -> Tools -> Android -> Android USB to WIFI；
拔掉USB线即可，此时已可以无线调试。

3. 补充说明：
重启 Android Studio 调试失效，需要再次设置。

## [2. Android ButterKnife Zelezny][2]
 1. 功能：
自动生成相关注解。

 2. 使用：
在Activity，Fragment，Adapter中选中布局xml的资源id；
右键，选择 Generate（或 alt+insert）；
在出现的对话框中选择 Generate Butterknife Injections 自动生成butterknife注解。

 3. 补充说明：
由于版本升级，相关函数变动由 Bind 到 Inject 到 BindView，最新版代码样式如下
```
@BindView(R.id.textView)
TextView mTextView;

ButterKnife.bind(this, view);
```

## [3. Android Code Generator][3]
 1. 功能：
根据布局文件快速生成对应的Activity，Fragment，Adapter，Menu。

 2. 使用：
布局文件中右键，选择 Generate Android Code ，根据需要选择Activity/Fragment/Menu等

 3. 补充说明：

## [4. Android DPI Calculator][4]
 1. 功能：
DPI计算插件

 2. 使用：
shift shift ，输入 DPI Calculator，回车即可。

 3. 补充说明：

## [5. Android Drawable Importer][5]
 1. 功能：
导入Android图标与Material图标的Drawable ，支持批量导入 ，多源导入（即导入某张图片各种dpi对应的图片）

 2. 使用：
File -> 
New -> 
Icon Pack Drawable Importer 
Vector Drawable Importer 导入矢量图
Batch Drawable Import 导入Batch
Multisource-Drawable 导入多源图

 3. 补充说明：

## [6. Android File Grouping （原 folding-plugin ）][6]
 1. 功能：
布局文件智能分组。

 2. 使用：
布局文件名第一个单词相同的归为一组，且位于以第一个单词命名的文件夹下面。
右键 layout，选择 Group 即可。
 
 3. 补充说明：


## [7. Android Material Design Icon Generator][7]
 1. 功能：
将 Material Design 的图标导入到程序中。

 2. 使用：
File -> New -> Materail design icon（或者 Ctrl+Alt+M，亲测已有该快捷键，故不生效）

 3. 补充说明：


## [8. Android Methods Count][8]
 1. 功能：
显示依赖库中的方法数。

 2. 使用：
不需要设置。
 
 3. 补充说明：


## [9. Android Parcelable code generator][9]
 1. 功能：
JavaBean 序列化，快速实现 Parcelable 接口。

 2. 使用：
Bean 类中 alt + insert 选择 Parcelable。

 3. 补充说明：
内部使用到的非基本类型对象对应的类需要预先实现序列化。

## [10. Android Postfix Completion][10]
1. 功能：
根据后缀快速完成代码，系统已经有这些功能，如sout、notnull等，这个插件在原有的基础上增添了一些新的功能。

2. 使用：
expr.toast -> Toast.makeText(context, expr, Toast.LENGTH_SHORT).show();
expr.log -> Log.d("log", expr);
expr.logd -> Log.d("log", expr);
expr.find -> (ViewType) findViewById(expr);
expr.isemp -> TextUtils.isEmpty(expr);
expr.vg -> (expr) ? View.VISIBLE : View.GONE;
等后缀操作

3. 补充说明：

## [11. Android Styler][11]
 1. 功能：
根据xml自动生成style代码。

 2. 使用：
在layout文件中复制选中的style代码；
在styles.xml文件中使用Ctrl+Shift+D粘贴；
在弹出的对话框中输入style的名字即可生成；

 3. 补充说明：


## [12. CheckStyle-IDEA][12]
 1. 功能：
检查代码风格的插件，比如像命名约定，Javadoc，类设计等方面进行代码规范和风格的检查，你们可以遵从像Google Oracle 的Java 代码指南 ，当然也可以按照自己的规则来设置配置文件，从而有效约束你自己更好地遵循代码编写规范。

 2. 使用：

 3. 补充说明：


## [13. CodeGlance][13]
 1. 功能：
在右边可以预览代码，实现快速定位。

 2. 使用：
不需要设置。

 3. 补充说明：


## [14. Codota][14]
 1. 功能：
搜索超过七百万精品代码实例；

 2. 使用：
Ctrl + K 打开 Codota 视图，输入关键词搜索。

 3. 补充说明：
第一次使用会请求一个token，会在浏览器中打开，登陆后复制 Token 到 Android Studio 弹出的对话框中并回车就可以使用了。

## [15. GenerateSerialVersionUID][15]
 1. 功能：
为Serializable序列化的Bean自动生成的serialVersionUID。

 2. 使用：
Bean类实现Serializable接口；
alt + insert 选择 SerialVersionUID 。
 
 3. 补充说明：


## [16. Gradle Dependencies Helper][16]
1. 功能：
maven gradle 依赖支持自动补全。

2. 使用：
在 build.gradle 文件中编辑依赖时会自动补全。

3. 补充说明：

## [17. GsonFormat][17]
 1. 功能：
 快速将Json字符串转换成一个Java Bean。

 2. 使用：
新建一个Bean类；
Alt+Insert，在出现的对话框选择GsonFormate（或者 Alt+S一步到位）;
粘贴Json内容，回车即可。

 3. 补充说明：
自动生成变量切不可手动改变，如将id改为mId，会导致Gson库无法正确解析。

## [18. JSONOnlineViewer][18]
 1. 功能：
在Android Studio中请求、调试接口。

 2. 使用：
工具栏 -> 
View ->
JSONViewer
 
 3. 补充说明：


## [19. Lifecycle Sorter][19]
 1. 功能：
根据 Activity 或者 Fragment 的生命周期对其生命周期方法位置进行先后排序。

 2. 使用：
工具栏 ->
Code -> 
Sort Lifecycle Methods -> 
Place at Start of Class
Place at End of Class
 
 3. 补充说明：
排序的是当前正在编辑、显示的这个类

## [20. PermissionsDispatcher plugin][20]
 1. 功能：
自动生成6.0权限的代码。

 2. 使用：
-> build.gradle中添加如下
compile 'com.github.hotchemi:permissionsdispatcher:2.2.0'
annotationProcessor 'com.github.hotchemi:permissionsdispatcher-processor:2.2.0'
->  Alt+Insert，选择Generate Runtime Permissions。
（或者Ctrl+Shift+A 或者Shift Shift，输入 Generate Runtime Permissions，回车即可打开插件）
-> 选择需要的权限及响应函数即可。
 
 3. 补充说明：
该插件使用第三方库 [PermissionsDispatcher][21] 来进行权限管理，app/build 目录下会自动生成相应的 XxxPermissionsDispatcher.java 文件。

## [21. SelectorChapek for Android][22]
 1. 功能：
通过资源文件命名自动生成Selector Drawable。

 2. 使用：
资源文件需要遵循命名规则，右键drawable/drawable-hdpi/drawable-xhdpi等，选择 Generate Android Selectors 。
```
// 资源文件后缀规则如下
_normal	(默认状态)
_pressed	state_pressed
_focused	state_focused
_disabled	state_enabled(假)
_checked	state_checked
_selected	state_selected
_hovered	state_hovered
_checkable	state_checkable
_activated	state_activated
_windowfocused	state_window_focused

// 示列如下
actionbar_btn_normal.png
actionbar_btn_pressed.png
actionbar_btn_focused.png
actionbar_btn_disabled.png
actionbar_btn_disabled_focused.png
```

 3. 补充说明：

## [22. Statistic][23]
1. 功能：
统计行数的插件。

2. 使用：
工具栏 ->
View -> 
Tool Windows ->
Statistic -> 
下方 SlideBar 会出现 Statistic 工具 tab ->
Refresh 统计所有.java文件的行数
Refresh on selection 统计当前编辑的.java文件的行数

3. 补充说明：
右键 SlideBar 上的 tab 可以 Remove from SlideBar。

## 23. [Alibaba Java Coding Guidelines][24]
1. 功能
使用规约来检查代码，并提供修改方案

2. 使用
代码中变色提示

3. 补充说明


  [1]: https://plugins.jetbrains.com/plugin/7856?pr=androidstudio
  [2]: http://plugins.jetbrains.com/plugin/7369?pr=androidstudio
  [3]: http://plugins.jetbrains.com/plugin/7595?pr=androidstudio
  [4]: https://plugins.jetbrains.com/plugin/7832?pr=androidstudio
  [5]: https://plugins.jetbrains.com/plugin/7658?pr=androidstudio
  [6]: https://github.com/dmytrodanylyk/folding-plugin/releases
  [7]: https://plugins.jetbrains.com/plugin/7647?pr=androidstudio
  [8]: https://plugins.jetbrains.com/plugin/8076?pr=androidstudio
  [9]: https://plugins.jetbrains.com/plugin/7332?pr=androidstudio
  [10]: https://plugins.jetbrains.com/plugin/7775?pr=androidstudio
  [11]: https://plugins.jetbrains.com/plugin/7972?pr=androidstudio
  [12]: https://plugins.jetbrains.com/plugin/1065?pr=androidstudio
  [13]: https://plugins.jetbrains.com/plugin/7275?pr=androidstudio
  [14]: https://plugins.jetbrains.com/plugin/7638?pr=androidstudio
  [15]: https://plugins.jetbrains.com/plugin/185?pr=androidstudio
  [16]: https://plugins.jetbrains.com/plugin/7299?pr=androidstudio
  [17]: http://plugins.jetbrains.com/plugin/7654?pr=androidstudio
  [18]: https://plugins.jetbrains.com/plugin/7838?pr=androidstudio
  [19]: http://plugins.jetbrains.com/plugin/7742?pr=androidstudio
  [20]: https://plugins.jetbrains.com/plugin/8349?pr=androidstudio
  [21]: https://github.com/hotchemi/PermissionsDispatcher
  [22]: http://plugins.jetbrains.com/plugin/7298?pr=androidstudio
  [23]: https://plugins.jetbrains.com/plugin/4509?pr=androidstudio
  [24]: https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines