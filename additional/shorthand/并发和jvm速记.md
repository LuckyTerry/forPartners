# 并发和jvm速记

[答案](../shortanswer/并发和jvm速记.md)

1. 线程，进程，协程
2. 介绍下 Java 内存区域（运行时数据区）线程私有3为什么私有，线程共享3
- 程序计数器 作用
- 虚拟机栈
- 本地方法栈
- 栈帧中都拥有信息
-  Java 虚拟机栈和本地方法栈会出现哪两种异常
- 怎么模拟两种异常
- 栈调用深度和溢出内存大小固定的么
- 堆
- GC堆细分
- Java8内存模型—永久代(PermGen)和元空间(Metaspace)
- 运行时常量池属于哪个区域，存了什么东西
- 直接内存
3. Java 对象的创建过程（五步，建议能默写出来并且要知道每一步虚拟机做了什么）
- HotSpot 虚拟机在 Java 堆中对象分配、布局和访问的全过程
- 内存分配的两种方式 指针碰撞 空闲列表  什么情况用哪个
- 内存分配并发问题两种解决方式
- 对象在内存中的布局可以分为3块区域
- Hotspot虚拟机的对象头2部分
- 实例数据
- 对齐填充
4. 对象的访问定位的两种方式（句柄和直接指针两种方式）优缺点
5. String类和常量池
- String 对象的两种创建方式
- String s1 = new String("abc");这句话创建了几个对象？
- String 类型的常量池比较特殊。它的主要使用方法有哪两种？
- String 字符串拼接
6. 8种基本类型的包装类和常量池
- 哪些实现了常量池技术，哪些没有
7. 􏰍􏴿􏱆􏷗􏻥􏵠􏰃􏷋􏷈􏳺 多线程
- 为什么要使用多线程
- 线程的生命周期和状态6
- 􏴿􏱆􏱸􏲛􏱤􏷚􏿐􏳷什么是上下文切换
- 什么是线程死锁，如何避免
- sleep方法和wait方法区别
- 调用start会执行run 为什么不直接调用run
8. synchronized关键字
- 对synchronized的了解
- 3种使用方式
- 手写单例模式，双重检验锁方式实现单例的原理 为什么要使用volatile 
- 不使用volatile怎么导致对象逸出
- javap命令用处
- synchronized关键字的底层原理
- monitor对象存在哪里
- monitorenter 􏴛􏻶􏰫 monitorexit ACC_SYNCHRONIZED _
- 1.6之后对synchronized底层做了哪些优化
- 锁主要存在的4种状态
- 偏向锁
- 轻量级锁
- 自旋锁和自适应自旋
- 锁消除
- 锁粗化
- synchronized􏴎和ReentrantLock 􏰢􏷵􏶦 区别  ReentrantLock3点增强
- 公平锁和非公平锁
9. volatile 关键字
- 作用
- synchronized 和 volatile区别
10. ThreadLocal 
- 简介用途
- 内存泄漏问题
- (img)
11. 线程池
- 为什么要用线程池
- 实现Runnable􏷎􏸰􏴎和Callable接口区别􏷎􏸰 
- execute()􏳻􏶸􏴎submit()􏳻􏶸􏰢􏷵􏶦 区别
- Executors 创建线程池的缺点
- ThreadPoolExecutor 构造参数
- ThreadPoolExecutor饱和策略和执行原理
- 线程池大小确定

12. Atomic原子类
- 介绍
- juc包哪4类原子类
- AtomicInteger 原理
- longaddr
- 知道哪些并发容器
- CopyOnWriteArrayList
- CopyOnWriteArrayList怎么做到写读不互斥 读不加锁
- ConcurrentLinkedQueue
- 阻塞队列和非阻塞队列区别
- BlockingQueue
- ConcurrentSkipListMap
- 跳表概念
13. AQS 
- 原理 理解
- clh队列 FIFO cas
- AQS对资源的共享方式
- AQS使用了什么设计模式
- 自定义AQS同步器需要重写哪些方法
- ReentrantLock  Semaphore CountDownLatch  CyclicBarrier 底层实现
14. gc
- Minor Gc􏴎 和 Major GC Full GC 
- 如何判断对象是否死亡
- 强软弱虚引用
- 如何判断一个类是无用的类
- gc有哪些算法，各自特点
- 标记清除，复制，标记整理，分代收集
- 有哪些垃圾收集器
- Serial ParNew Parallel Scavenge Serial Old  Parallel Old 
- CMS运行4个步骤 G1􏸉􏹇􏼀 收集器
15. 类文件结构
- 文件结构和组件
- 类加载过程
- 加载阶段做了哪3件事
- 知道哪些类加载器
- 介绍一下双亲委派模型
- 双亲委派模型有什么好处
- 如何自定义类加载器
## 面经：
- java线程模型和jvm线程模型
- 说一下java类加载器的工作机制？类加载在那个区域进行的？
- 说说java泛型，为什么称java泛型为伪泛型？泛型的好处有哪些？int可以作为泛型类型吗？
- 线程run和start的区别？两次start同一个线程会怎么样？
- 对concureent包了解吗？什么是cas？cas怎么解决ABA问题？讲一下CountDownLatch和cyclicBarrier的区别？并发包了解吗？假如几个线程之间相互等待，可以用哪个并发类来实现，他的原理是什么？2
- 没有做过GC调优，讲一下这么做的？
- GC的过程
- 强制young gc会有什么问题？
- 知道G1么？
- 回收过程是怎么样的？
- 你提到的Remember Set底层是怎么实现的？
- CMS4个阶段 CMS GC有什么问题？
- 怎么避免产生浮动垃圾？
- Jvm了解吗？jvm中哪些可以作为垃圾回收的gcroot?为什么呢？ 分析哪些是gc root对象
- 什么时候需要自定义类加载器？
- 什么时候静态方法加锁  什么时候实例方法加锁
- jvm安全点
- lock.interrupt底层原理
