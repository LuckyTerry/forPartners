# 第1章 Activity的生命周期和启动模式

[TOC]

## 1 Activity的生命周期全面分析

### 1.1 典型情况下的生命周期分析
* onCreate 正在创建
* onRestart 正在重启
* onStart 正在启动
* onResume 出现在前台并开始活动
* onPause 正在停止，存储数据、停止动画等工作，不能太耗时
* onStop 即将停止，稍微重量级的回收工作，不能太耗时
* onDestroy 即将被销毁，回收工作和最终的资源释放

问题1：onStart和onResume、onPause和onStop有什么实质上的不同？
答案：从实际使用过程中，的确差不多，甚至可以只保留一对；两个配对的回调表示不同的意义，onStart和onStop是从是否可见这个角度来回调的，onResume和onPause是从是否位于前台这个角度来回调的。

问题2：ActivityA进入ActivityB，B的onResum和A的onPause哪个先执行？ 
答案：A的onPause -> B的onCreate -> B的onStart -> B的onResume -> A的onStop。

### 1.2 异常情况下的生命周期分析

#### 1.2.1 资源相关的系统配置发生改变导致Activity被杀死并重新创建

异常发生：

销毁：onSaveInstanceState在onStop之前执行，与onPause并没有先后顺序之分
```
onPause onSaveInstanceState
onStop
onDestroy
```

重新创建：onRestoreInstanceState在onStart之后执行，与onResume并没有先后之分
```
onCreate
onStart
onResume onRestoreInstanceState
```

当Activity异常情况下需要重新创建时，系统默认在onSaveInstanceState中为我们保存当前Activity中**定义了id属性**的视图，并且在Activity重启后在onRestoreInstanceState中为我们恢复这些数据。

#### 1.2.2 资源内存不足导致低优先级的Activity被杀死

> * 前台Activity，优先级最高
> * 可见非前台Activity，优先级次之
> * 后台Activity，优先级最低
> * 当系统内存不足时，会按照以上优先级去杀死进程，并后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据
> * 一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程很容易被杀死，比较好的方法是将后台工作放入Service中从而保证进程有一定的优先级，这样就不会轻易地被系统杀死。

为Activity指定如下属性
```
android:configChanges="orientation|screenSize"
```
这样的话，Activity并不会重建，取而代之会回调以下函数，可以做一些自己的特殊处理
```
onConfigurationChanged()
```

## 2 Activity的启动模式

### 2.1 LaunchMode

> * standard 标准模式，也是系统的默认模式

Activity A启动了Activity B(B是标准模式)，那么B就会进入到A所在的栈中；

一种特殊情况：

非Activity类型的Context启动一个Activity
---过程--->会报错，因为没有默认的任务栈
---解决--->为待启动的Activity添加FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就会为它创建一个新的任务栈，这个时候待启动的Activity实际上是以singleTask模式启动的。

> * singleTop 栈顶复用模式

两种情况：

任务栈S1的情况为ABCD，Activity A以singleTask模式请求启动
---过程---> 直接创建A的实例并入栈S1
---结果---> 任务栈S1的情况为ABCDA
任务栈S1的情况为ABCD，，Activity D以singleTask模式请求启动
---过程---> D不会重建，同时它的onNewIntent方法会被回调
---结果---> 任务栈S1的情况为ABCD

> * singleTask 栈内复用模式

三种情况：

任务栈S1的情况为ABC，Activity D 以singleTask模式请求启动，所需要任务栈为S1
---过程---> 直接创建D的实例并入栈到S1 
---结果---> 任务栈S1的情况为ABCD
任务栈S1的情况为ADBC，Activity D 以singleTask模式请求启动，所需要任务栈为S1
---过程---> D上面的Activity全部出栈，D切换到栈顶并调用onNewIntent方法
---结果---> 任务栈S1的情况为AD
任务栈S1的情况为ADBC，Activity D 以singleTask模式请求启动，所需要任务栈为S2
---过程---> 先创建任务栈S2，再创建D的实例并入栈到S2
---结果---> 任务栈S2的情况为D


> * singleInstance 单实例模式

只能单独地位于一个任务栈，后续地请求均不会创建新的Activity，除非这个独特地任务栈被系统销毁了。

> * 两种启动方式
通过AndroidMenifest为Activity指定启动模式，优先级低，无法设定FLAG_ACTIVITY_CLEAR_TOP标识
在Intent中设置标志位来为Activity指定启动模式，优先级高，无法指定singleInstance模式

### 2.2 任务栈详解

任务栈可分为前台任务栈和后台任务栈，后台任务栈的Activity处于暂停状态。

一个重要属性：TaskAffinity  译 任务相关性
默认情况下，所有Activity所需的任务栈的名字为应用的包名，我们可以为每个Activity都单独指定新的TaskAffinity属性，这个属性值必须不能和包名相同。

* TaskAffinity和singleTask启动模式配对使用：
待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。
* TaskAffinity和allowTaskReparenting配对使用：
---操作1--->应用A启动应用B的Activity（该Activity的allowTaskReparenting属性为true）
---操作2--->按Home键回到桌面
---操作3--->当应用B被启动后，此Activity会直接从应用A的任务栈转移到应用B的任务栈。

### 2.3 Flags

> * FLAG_ACTIVITY_NEW_TASK -> 同singleTask
> * FLAG_ACTIVITY_SINGLE_TOP -> 同singleTop
> * FLAG_ACTIVITY_CLEAR_TOP ->  与FLAG_ACTIVITY_NEW_TASK配合使用，它连同它上面的Activity都要出栈，系统会创建新的Activity实例并放入栈顶。相比较只使用FLAG_ACTIVITY_NEW_TASK（或singleTask），它也会出栈并重建，而不是回调onNewIntent（这里有点懵逼！？）
> * FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS -> 同android:excuteFromRecents="true" 具有这个标记的Activity不会出现在历史Activity列表中

## 3 IntentFilter的匹配规则

> * 一个Activity可以有多个intent-filter（过滤规则），一个Intent只要能匹配任意一组intent-filter即可成功启动
> * 一个Intent同时匹配action类别、category类别、data类别才算完全匹配该组intent-filter

### 3.1 action的匹配规则

* Intent只能含有一个action，而一组intent-filter中可以有多个action，只要Intent中的action能够和intent-filter中的任何一个action相同即可成功

### 3.2 category的匹配规则

* Intent可以没有category，系统会添加默认category，需要intent-filter含有android.intent.category.DEFAULT才能匹配
* Intent含有一个或多个category，要求每个category在该组intent-filter都存在

### 3.3 data的匹配规则

* Intent不含data规则，intent-filter也不含data规则，该项匹配
* Intent含有一组data规则，而一组intent-filter可以含有多组data规则，只要Intent中的data能够和intent-filter中的任何一个data相同即可匹配该项

intent.setDataAndType(?,?);

#### 3.3.1 mimeType匹配规则

* Intent中的mimeType必须等同于intent-filter的mimeType或者是它的通配符实现

```
mimeType数据结构：
image/*、image/jpeg、audio/*、audio/mpeg4-generic、video/*等，表示图片、文本、视频等不同的媒体格式
```

#### 3.3.2 URI匹配规则

* Intent中的URI指定为content或file，由于intent-filter中含有默认的URI（content和file），该项匹配
* 如果intent-filter显式指定含有?（不再默认包括content和file了），那么Intent只能指定URI为?才能匹配

```
URI数据结构：
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
示例如下
content://com.example.project:200/folder/subfolder/etc
http://www.baidu.com:80/search/info

Scheme:URI的模式
Host:URI的主机名
Port:URI中的端口号
Path:完整路径
pathPatter:完整路径，可以包含通配符
pathPrefix:路径的前缀信息
```


### 3.4 可以通过如下的三个方法判断是否有Activity能够匹配我们的隐式Intent
```
Intent的resolveActivity方法
PackageManager的resolveActivity方法
PackageManager的queryIntentActivities方法
```

