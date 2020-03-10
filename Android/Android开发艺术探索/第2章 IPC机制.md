# 第2章 IPC机制

[TOC]

----------


## 1 Android IPC简介

Inter-Process Communication 进程间通信（跨进程通信）
----------


## 2 Android中的多进程模式

### 2.1 开启多进程模式 /

给四大组件在AndroidMenifest中指定android:process属性
```
android:process=":remote"//进程名为com.terry.ipctest:remote
android:process="com.terry.ipctest.remote"//进程名为com.terry.ipctest.remote
```

### 2.2 多进程模式的运行机制 /

静态成员和单例模式完全失效
线程同步机制完全失效
SharedPreference可靠性下降
Application会多次创建


----------


## 3 IPC基础概念介绍

### 3.1 Serializable接口

```
public class ... implements Serializable {
    private static final long serialVersionUID = 7536984512086478913L;
}

//序列化
User user = new User(0, "jake", true);
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("cache.txt"));
out.writeObject(user);
out.close;

//反序列化
ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.txt"));
User newUser = (User)in.readObject();
in.close;

```
* java提供的序列化方式
* 恢复后的newUser和user内容完全相容，但两者并不是同一个对象
* serialVersionUID是为了反序列化时最大限度地恢复数据（若反序列化都让系统重新计算id值，只要有不同就crash）

### 3.2 Parcelable接口
* Android提供的序列化方式

```
//其实 Android Studio 会自动生成，有木有很方便呀 ^_^
public class Book implements Parcelable{
    public int bookId;
    public String bookName;
    public User user;

    public Book(int bookId, String bookName, User user) {
        this.bookId = bookId;
        this.bookName = bookName;
        this.user = user;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(bookId);
        parcel.writeString(bookName);
        parcel.writeParcelable(User, 0);
    }

    protected Book(Parcel in) {
        bookId = in.readInt();
        bookName = in.readString();
        user = in.readParcelable(Thread.currentThread().getContextClassLoader());
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };
}
```

### 3.3 Binder

* 从IPC角度，是Android中的一种跨进程通讯方式
* 从Android Framework角度，是ServiceManager连接各种Manager和相应的ManagerService的桥梁
* 从Android Application角度，是客户端和服务端进行通讯的媒介

```
public interface IBookManager extends android.os.IInterface {

    public static abstract class Stub extends android.os.Binder implements com.terry.ipctest.IBookManager {
        
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }
        
        public static com.terry.ipctest.IBookManager asInterface(android.os.IBinder obj) {
            //略
        }
        
         @Override
        public android.os.IBinder asBinder() {
            return this;
        }
        
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            //略
        }
        private static class Proxy implements com.terry.ipctest.IBookManager {
            //略
        }
        
        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }
    
    public java.util.List<com.terry.ipctest.Book> getBookList() throws android.os.RemoteException;

    public void addBook(com.terry.drawablenotetest.Book book) throws android.os.RemoteException;
}
```

分析：
> 1. IBookManager，接口，继承自IInterface，故IBookManager现在含有包括anBinder()在内的三个抽象方法
> 2. Stub内部类，抽象类，继承自Binder，implements（未实现，还是抽象方法）IBookManager，可以认为它是一个含有IBookManager接口抽象方法的Binder
> 3. Proxy内部类(Stub内部的)，类，implements（已实现）IBookManager
> 4. asInterface将Stub对象作为IBookManager接口返回使用，如其名，内部有逻辑判断是否是进程内通信，true则强制类型转换使用，false则使用Proxy代理类包装使用
> 5. asBinder将Stub对象作为Binder返回使用，如其名

```
private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipiect(){
    @Overrider
    public void binderDied) {
        if (mBookManager == null){
            return;
        }
        mBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
        mBookManager = null;
        // TODO : 这里重新绑定
    }
}

mService = IMessageBoxManager.Stub.asInterface(binder);
binder.linkToDeath(mDeathRecipient, 0);
```
> 1. 连接成功后linkToDeath
> 2. 死亡时binderDied被回调，内部unlinkToDeath，再重新绑定

----------


## 4 Android中的IPC方式

### 4.1 使用Bundle

### 4.2 使用文件共享

注意：不能使用SharedPreference，系统对它的读/写有一定的缓存策略，在内存中会有一份SharedPreference文件的缓存，因此在多进程模式下，系统对它的读写就变得不可靠。

### 4.3 使用Messenger

轻量级的IPC方案，底层是AIDL。

记住这两个构造函数：
```
private final IMessenger mTarget;

public Messenger(Handler target){
    mTarget = target.getIMessenger;
}

public Messenger (IBinder target) {
    mTarget = IMessenger.Stub.asInterface(target);
}
```

如何使用呢？
服务端
```
<service 
    android:name="com.terry.ipctest.messenger.MessengerService"
    android:process=":remote">

public class MessengerService extends Service {

    private final Messenger mMessenger = new Messenger(new MessengerHandler());
    
    private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage (Message msg) {
            //处理消息
            
            //回复消息
            Messenger clientMessenger = msg.replyTo;
            Message replyMsg = Message.obtain(null, 0);
            Bundle data = new Bundle ();
            data.putString("reply", "你发的消息我收到了");
            msg.setData(data);
            try {
                clientMessenger.send(replyMsg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    }
    
    @Override
    public IBinder onBind (Intent intent) {
        return mMessenger.getBinder();
    }
}
```

客户端
```
private Messenger mServiceMessenger;

private final Messenger mGetReplyMessenger = new Messenger(new MessengerHandler());
    
private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage (Message msg) {
            //处理服务端的消息
        }
    }

private ServiceConnection mConnection = new ServiceConnection () {
    public void onServiceConnected(ComponentName className, IBinder service) {
        mServiceMessenger = new Messenger(service);
        Message msg = Message.obtain(null, 0);
        Bundle data = new Bundle ();
        data.putString("msg", "hello");
        msg.setData(data);
        msg.replyTo = mGetReplyMessenger;//把Messenger传给了服务端，可以理解为我把电话号码给了服务端
        //服务端可以通过该电话号码（Messenger）给我发消息了
        try {
            mServiceMessenger.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
     public void onServiceDisconnected(ComponentName className) {
     }
}

... onCreate ...{
    ...
    Intent intent = new Intent(this, MessengerService.class);
    bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    ...
}

... onDestroy ... {
    ...
    unbindService(mConnection);
}

```

分析：
> onServiceConnected，服务端将联系方式（服务端Handler构造的Messenger的IBinder成员变量）给了客户端，客户端利用IBinder生成对应的Messenger，并给服务端发送消息，消息中可以利用replyTo参数传递联系方式（客户端Handler构造的Messenger）
> 服务端回调hanleMessage处理消息或通过replyTo参数（如果有的话）回复消息
> 以上的分析都是基于两个构造函数，IBinder和Handler都能构造出Messenger，核心所在

### 4.4 使用AIDL

基本使用
```
说明
    AndroidStudio的aidl文件默认放在src/main/aidl目录下，aidl目录和java目录同级别。
右键 main -> New -> AIDL -> AIDL File -> 输入接口名，系统会自动创建aidl文件夹并生成该接口aidl。
and then do what you need to do .
最后make project即可，在 build -> source -> aidl -> debug -> 包名 -> 下能看到生成的Binder类

Code
1. 服务端新建Stub实例，并在onBind中返回该实例
2. 客户端在onServiceConnected中执行asInterface作为接口使用（实际上我们想要使用的也是接口中的函数，Binder的存在是为了数据传输）
```

注册观察者

修改原接口，添加注册、解注册抽象函数

```
//服务端中

private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new RemoteCallbackList<IOnNewBookArrivedListener>();

//注册观察者
@Override
public void registerListener(IOnNewBookArrivedListener listener)
        throws RemoteException {
    mListenerList.register(listener);
}

//解注册观察者
@Override
public void unregisterListener(IOnNewBookArrivedListener listener)
        throws RemoteException {
    mListenerList.unregister(listener);
};

//通知已注册用户
final int N = mListenerList.beginBroadcast();
for (int i = 0; i < N; i++) {
    IOnNewBookArrivedListener l = mListenerList.getBroadcastItem(i);
    if (l != null) {
        try {
            l.onNewBookArrived(book);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
mListenerList.finishBroadcast();
```

```
//客户端中

onServiceConnected中注册
mRemoteBookManager.registerListener(mOnNewBookArrivedListener);

onDestroy中解注册
mRemoteBookManager.unregisterListener(mOnNewBookArrivedListener);
```

更详细的见原书和源码

### 4.5 使用ContentProvider

略，见原书和源码

### 4.6 使用Socket

略，见原书和源码

----------


## 5 Binder连接池

略，见原书和源码

----------


## 6 选用合适的IPC方式

略，见原书和源码

----------




