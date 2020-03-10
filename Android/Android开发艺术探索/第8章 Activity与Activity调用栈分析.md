# 第8章 Activity与Activity调用栈分析

[TOC]

----------


## 1 Activity

### 1.1 起源

略

### 1.2 Activity形态

* Active/Running
Activity处于Activity栈的最顶层，可见，并与用户进行交互
* Paused
当Activity失去焦点，被一个非全屏的Activity或者一个透明的Activity放置在栈顶时，Activity就进入Paused状态
* Stopped
当一个Activity被另一个Activity完全覆盖，Activity就进入Stopped状态
* Killed
Activity被系统回收掉或者从未创建过，Activity就处于Killed状态

### 1.3 生命周期

7个生命周期函数

* onCreate() onStart() onResume() onPause() onStop() onDestroy onRestart()

6个生命周期状态

* Created Started Resumed Paused Stopped

其中只有如下3个状态是稳定的，其他状态都是过渡状态，很快就会结束

* Resumed Paused Stopped

#### 1.3.1 启动与销毁过程

在系统调用onCreate()之后，就会马上调用onStart()，然后继续调用onResume()以进入Resumed状态，最后就会停在状态
onCreate()中：创建基本的UI元素
onDestroy()中：清除开启的线程等

#### 1.3.2 暂停与恢复过程

onPause()中：释放系统资源，如Camera、sensor、receiver
onResume()中：需要重新初始化在onPause()中释放的资源

#### 1.3.3 停止过程

onStop()中：清除Activity的资源，避免浪费

#### 1.3.4 重新启动过程

onRestart()中：一般不做处理

#### 1.3.5 重新创建过程

Activity因为某种原因被系统Kill掉了：系统会将Activity状态通过onSaveInstanceState()方法保存到Bundle对象中
重新创建时：通过onRestoreInstanceState()方法和onCreate()方法取出保存的Bundle对象

----------


## 2 Android任务栈简介

* 栈结构：后进先出(Last In First Out)的线性表。
* 当一个App启动时，如果当前环境不存在该App的任务栈，那么系统就会创建一个任务栈，此后，这个App所启动的Activity都将在这个任务栈中被管理，这个栈也被称为一个task，即表示若干Activity的集合，他们组合在一起形成一个Task。
* 一个Task中的Activity可以来自不同的App，同一个App的Activity也可能不在一个Task中。
* 可以给Activity设置”特权“(android:launchMode和Intent的flas)，来改变Activity的行为模式

----------


## 3 AndroidMainifest启动模式

此处不够详细，参考Android开发艺术探索 [第1章 Activity的生命周期和启动模式][1]

### 3.1 standard  

默认启动模式。

### 3.2 singleTop

栈顶复用模式。若复用，则在Activity启动时会调用onNewIntent()方法；否则，标准启动。
通过startActivityForResult()方法来启动另一个，系统会直接返回Activity.RESULT_CANCELED

### 3.3 singleTask

栈内复用模式，若复用，则在Activity启动时会调用onNewIntent()方法；否则，标准启动
不是在新的应用栈中被打开(使用TaskAffinity)，就是将已打开的Activity切换到前台

### 3.4 singlelnstance

会出现在一个新的任务栈中，而且该任务栈中只存在这一个Activity
通过startActivityForResult()方法来启动另一个，系统会直接返回Activity.RESULT_CANCELED

----------


## 4 Intent Flag启动模式

常用的Flag

* Intent.FLAG_ACTIVITY_NEW_TASK　通常使用在server中启动Activity
* Intent.FLAG_ACTIVITY_SINGLE_TOP 同"singleTop"
* Intent.FLAG_ACTIVITY_CLEAR_TOP 同"singleTask"
* Intent.FLAG_ACTIVITY_NO_HISTORY AB以这种模式启动C，再启动D，则当前Activity栈为ABD


----------


## 5 清空任务栈

* clearTaskOnLaunch
每次返回该Activity时，都将该Activity之上的所有Activity清除。

* finishOnTaskLaunch
当离开这个Activity所处的Task，那么用户再返回该Task时，该就会被finish掉。
    
* alwaysRetainTaskState
若设置为true，那么该Activity所处的Task将不接受任何清理命令，一直保持当前Task状态。

----------


## 6 Activity任务栈使用

一定要谨慎使用，任务栈导致的Bug很难调试的。


  [1]: https://github.com/LuckyTerry/ReadingNotes/blob/master/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2/%E7%AC%AC1%E7%AB%A0%20Activity%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F.md