- 什么是进程和线程？他们有何区别？
- 程序计数器为什么是私有的?
- 并发与并行的区别是？
- 讲一讲多线程优缺点？
- 你了解线程的生命周期吗？
- 说说线程的各状态及对应的转移条件？
- 什么是上下文切换?
- 什么是线程死锁?
- 产生死锁的四个必要条件？
- 如何避免线程死锁?
- 说说 sleep() 方法和 wait() 方法区别和共同点?
- 为什么我们要调用 start() 方法，而不是直接调用 run() 方法？


synchronized
- synchronized最主要的三种使用方式？
- 手写DoubleCheck单例模式？为什么使用volatile修饰？
- synchronized同步语句块 底层原理？
- synchronized修饰方法 底层原理？
- JDK1.6 之后synchronized的底层优化？（偏向锁、轻量级锁、自旋锁和自适应自旋、锁消除、锁粗化）
- Synchronized 和 ReenTrantLock 的对比？

volatile
- 内存模型？
- 与synchronized的区别？


ThreadLocal
- ThreadLocal原理
- ThreadLocal 内存泄露问题
- ThreadLocal 扩展之 transmittable-thread-local
- ThreadLocal 扩展之 taskDecorator（executor, @Async, Hystrix）

线程池
- 

Atomic
- Atomic 原子类中原子的含义？
- JUC 包中的原子类是哪4类?
- AtomicInteger 的使用？
- 简单介绍一下 AtomicInteger 类的原理？

AQS
- 什么是AQS？
- 自定义同步器的步骤？
- 详细描述下accqure(1)的具体逻辑？

乐观锁和悲观锁
- 悲观锁
- 乐观锁
  - 版本号
  - CAS
    - 什么是CAS算法？
    - 什么是自旋锁？
    - Java如何实现自旋锁？
    - 如何在上述基础上实现可重入？
    - TicketLock、CLHLock、MCSLock 各自的大体实现？他们是可重入的吗？
    - 自旋锁与互斥锁有何区别？