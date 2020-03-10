# 第3章 保证APP流畅的关键因素——多线程 （未完待续）

[TOC]

## 1 Android 中的消息机制

### 1 从main函数到Handler创建都发生了什么？
处理消息的手段——Handler、Looper与MessageQueue
ActivityThread
```
public static void main(String[] args) {
    //代码省略
    Looper.prepareMainLooper();//1.创建消息循环Looper
    //代码省略
    Looper.loop();//2.执行消息循环
}
```

Looper
```
public static void prepareMainLooper() {
    prepare();
    setMainLooper(myLooper());
    myLooper().mQueue.mQuitAllowed = false;
}

public static void prepare() {
    if (sThreadLocal.get() != null) {
        throw /*代码省略*/;
    }
    sThreadLocal.set(new Looper());
}

public static Looper myLooper() {
    return sThreadLocal.get();
}

private synchronized static void setMainLooper(Looper looper) {
    mMainLooper = looper;
}
```

Handler
```
public Handler() {
    //代码省略
    mLooper = Looper.myLooper();
    //代码省略
    mQueue = mLooper.mQueue;
    mCallback = null;
}
```

Activity
```
class MyHandler extends Handler {
    @Override
    public void handleMessage (Message msg) {
        //更新UI
    }
}
MyHandler mHandler = new MyHandler();
new Thread {
    public void run() {
        //耗时操作
        mHandler.sendEmptyMessage(123);
    }
}.start();
```

### 2 Looper是怎样取出消息并分发处理消息的？

Looper
```
public static void loop() {
    Looper me = myLooper();
    //代码省略
    MessageQueue queue = me.mQueue;//1.获取消息队列
    //代码省略
    while (true) {                  //2.死循环，即消息循环
        Message msg = queue.next();     //3.获取消息
        //代码省略
        msg.target.dispatchMessage(msg);    //4.处理消息
        //代码省略
        msg.recycle();                          //回收消息
    }
}
```

Message
```
public final class Message implements Parcelable {
    Handler target;         //target处理
    Runnable callback;      //Runnable类型的callback
    Message next;           //下一条消息，消息队列时链式存储的
    //代码省略
}
```

Handler
```
//分发消息
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handlerCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

private final void handlerCallback(Message message) {
    message.callback.run();
}

//消息处理函数，之类覆写
public void hanlderMessage(Message msg) {
}
```

### 3 Handler是怎样发送消息的？

Handler (以下是api23的源码，与原书略有差异)

归纳以下

* 最后都会执行到 sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage

```
sendMessage -> sendMessageDelayed -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

```
sendEmptyMessage -> sendEmptyMessageDelayed -> sendMessageDelayed -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

```
sendEmptyMessageAtTime -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

```
post -> sendMessageDelayed -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

```
postAtTime -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

```
postDelayed -> sendMessageDelayed -> sendMessageAtTime -> enqueueMessage -> queue.enqueueMessage
```

常用之一 sendMessage

```
public final boolean sendMessage(Message msg){
    return sendMessageDelayed(msg, 0);
}

public final sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

常用之二 post

```
public final boolean post(Runnable r) {
    return sendMessageDelayed(getPostMessage(r), 0);
}

private final Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

public final sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

最后调用

```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
        return enqueueMessage(queue, msg, uptimeMillis);
}
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

```
public final boolean sendEmptyMessage(int what){
    return sendEmptyMessageDelayed(what, 0);
}
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageAtTime(msg, uptimeMillis);
}

```

### 4 子线程创建Handler注意事项

错误示例
```
new Thread() {
    Handler handler = null;
    public void run() {
        handler = new Handler();
    }
}.start();

```

原因分析：如下，子线程无Looper时创建Handler，Looper.myLooper()返回空，抛出异常
```
public Handler() {
    //代码省略
    mLooper = Looper.myLooper();
    
    if(mLooper == null) {
        throw new RuntimeException("略")；//抛出异常
    }
    
    mQueue = mLooper.mQueue;
    mCallback = null;
}
```

正确示例
```
new Thread() {
    Handler handler = null;
    public void run() {
        Looper.prepare();//1.为当前线程创建Looper，并且会绑定到ThreadLocal中
        handler = new Handler();
        Looper.loop();//2.启动消息循环
    }
}.start();

```

## 2 Android中的多线程

### 1 多线程的实现—— Thread 和 Runnable
```
new Thread() {
    @Overrride
    public void run() {
        //耗时操作
    }
}.start();
```

```
new Thread(new Runnable() {
        @Overrride
        public void run() {
            //耗时操作
        }
    }
).start();
```
区别：当启动一个线程时，如果Tread的target不为空，则会在子线程中执行这个target的run函数（第一段代码），否则虚拟机就会执行该线程自身的run函数（第二段代码）。

### 2 线程的 wait 、 sleep 、 join 和 yield
* wait() 线程休眠，释放对象机锁，可调用notify、notifyAll或者指定睡眠时间来唤醒
* sleep 线程休眠，仍未释放对象的机锁
```
private static Object sLockObject = new Object();

static void waitAndNotifyAll() {
    System.out.println("主线程运行");
    //创建并启动子线程
    Thread thread = new WaitThread();
    thread.start();
    long startTime = System.currentTimeMillis();
    try{
        synchronized (sLockObject){
            System.out.println("主线程等待");
            sLockObject.wait();
        }
    } catch (Exception e) {
    }
    long timesMs = System.currentTimeMillis() - startTime;
    System.out.println("主线程继续——>等待耗时： " = timesMs + " ms");
}

static class WaitThread extends Thread {
    @Override
    public void run() {
        try{
            Thread.sleep(3000);
            sLockObject.notifyAll();
        } catch (Exception e) {
        }
    }
}

执行结果：
主线程运行
主线程等待
主线程继续——>等待耗时： 3000 ms
```

* join 等待目标线程执行完成之后再继续执行
```
static void joinDemo() {
    Worker worker1 = new Worker("worker-1");
    Worker worker2 = new Worker("worker-2");
    worker1.start();
    System.out.println("启动线程1");
    try {
        //调用线程1的join函数，主线程会阻塞直到worker1执行完毕
        worker.join();
        System.out.println("启动线程2");
        //再启动线程2，并且调用线程2的join函数，主线程会阻塞直到worker2执行完毕
        worker2.start();
        worker2.join();
    } catch (InterruptedException e) {
            e.printStackTrace();
    }
    System.out.println("主线程继续执行");
}

static class Worker extends Thread {
    public Worker(String name) {
        super(name);
    }
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("work in " + getName());
    }
}

执行结果：
启动线程1
work in worker-1
启动线程2
work in worker-2
主线程继续执行

总结：阻塞当前调用join函数时所在的线程，直到接收线程执行完毕之后再继续
```
* yield 线程礼让，让出执行权限，但其他线程能否优先执行是未知的;
```
static void yieldDemo() {
    YieldThread t1 = new YieldThread("thread-1");
    YieldThread t2 = new YieldThread("thread-2");
    t1.start();
    t2.start();
}

static class YieldThread extends Thread {
    public Yield(String name) {
        super(name);
    }
    
    public synchronized void run() {
        for (int i = 0; i < MAX; i++) {
            System.out.printf("%s ，优先级为： %d ----> %d");
            if (i == 2) {
                Thread.yield();
            }
        }
    }
}

执行结果：
thread-1 ,优先级为： 5 ----> 0
thread-1 ,优先级为： 5 ----> 1
thread-1 ,优先级为： 5 ----> 2
thread-2 ,优先级为： 5 ----> 0
thread-2 ,优先级为： 5 ----> 1
thread-2 ,优先级为： 5 ----> 2
thread-1 ,优先级为： 5 ----> 3
thread-1 ,优先级为： 5 ----> 4
thread-2 ,优先级为： 5 ----> 3
thread-2 ,优先级为： 5 ----> 4

总结：使调用该函数的线程让出执行时间给其他已就绪的线程。
```

### 3 与多线程相关的方法—— Callable、 Future 和 FutureTask
Runnnable 既能运用在 Thread 中，也能运用在线程池中
Callable、 Future 和 FutureTask 则只能运用在线程池中

* Runnable 接口
```
public interface Runnable {
    public void run();
}
```

* Callable 接口

```
public interface Callable<T> {
    V call() throws Exception;
}
```

* Future 接口
```
public interface Future {
    boolean cancel(booelan mayInterruptIfRunning);
    
    //该任务是否已取消
    boolean isCancelled();
    
    //判断是否完成
    boolean isDone();
    
    //获取结果，如果任务未完成，则等待，直到返回结果，因此该函数会阻塞
    V get() throws InterruptedException, ExecutionExcepption;
    
    //获取结果，如果任务未完成，则等待，直到timeout或者返回结果，该函数会阻塞
    V get(long timeout, TimeUnit unit) 
        throws InterruptedException, ExecutionExcepption TimeoutException;
}
```

* RunnableFuture 接口 --- 实现了 Runable 和 Future<V> 这两个接口
```
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

* FutureTask类 --- 实现了RunnableFuture 接口
```
public class FutureTask<V> implements RunnableFuture<V> {
    //代码省略
}
```

> * 一句话总结：FutureTask既是Future、又是Runnable，而且还包装了Callable；

### 4 线程池
* 线程池都实现了 ExecutorService 接口，该接口定义了线程池需要实现的接口，如 submit、execute、shutdown等

#### 1 实现类之一 ThreadPoolExecutor

> * 运行状态 创建后便进入运行状态
> * 关闭状态 调用了shutdown()方法时便进入了关闭状态，此时ExecutorService不再接收新的任务，还在执行已经提交的任务
> * 停止状态 但所有已经提交了的任务执行完成后，就变成了停止状态

#### 2 实现类之二 ScheduledThreadPoolExecutor

### 5 同步集合
### 6 同步锁
### 7 AsyncTask 的原理