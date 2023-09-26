# IO，java8，规范，设计模式速记

1. BIO,NIO,AIO 总结
- BIO
- NIO
- AIO
2. 同步与异步，阻塞与非阻塞  如何区分 “同步/异步 ”和 “阻塞/非阻塞” 呢？
- 同步/异步是从行为角度描来述事物的，而阻塞和非阻塞是从状态角度来描述。
- 比如：当前任务对另一个任务的调用行为是同步或异步
- 当前任务调用另一个任务时，当前任务的状态是阻塞或非阻塞
- 说的更直观一点就是：结果通过return返回是同步，通过Callback返回是异步。调用时当前任务不能做其他事儿是阻塞，能够干其他事儿是非阻塞。
3. BIO (Blocking I/O) 通信模型
- 一请求一应答
4. 客户端并发访问量增加后这种模型会出现什么问题？

5. 什么是伪异步 BIO
- 通过一个线程池来处理多个客户端请求，客户端请求可以远远大于线程池数量
- 这样子，线程池可以灵活地调配线程资源，设置最大值，防止由于海量请求导致线程耗尽。
6. NIO (New I/O)
- 同步非阻塞，基于缓冲区的I/O模型
7. NIO的特性3
- Buffer 缓冲区
- Channel 通道
- Selector 选择器
- ServerChannel注册到Selector上，Selector阻塞轮循，发现Accept事件后将对应连接的Channel也注册到Selector。然后Selector发现Read事件后便通知Channel从缓冲区读取数据。
7. NIO与IO区别
- IO流是阻塞的，NIO流是不阻塞的
- IO 面向流，而 NIO 面向缓冲区
8. NIO 读数据和写数据方式
- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
- 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。
9. 用 JDK 原生 NIO 进行开发的问题
- JDK的 NIO 底层由 epoll 实现，空轮询 bug 会导致 cpu 飙升 100%
- 自行实现的 NIO 很容易出现 bug，维护成本较高
10. AIO (Asynchronous I/O)
- 异步非阻塞的IO模型，基于事件和回调机制实现的
11. Java8
- 接口的默认方法(Default Methods for Interfaces)
- Lambda表达式(Lambda expressions)
- 函数式接口(Functional Interfaces)
- 方法和构造函数引用(Method and Constructor References)
- Lamda 表达式作用域(Lambda Scopes)
- Parallel Streams
- Annotations
12. 设计模式
- 创建型模式5
    - SF_F_BP
    - 单例、工厂方法、抽象工厂、建造者、原型
    - Singleton/Factory/AbsFactory/Builder/Prototype
- 结构型模式7
    - ABCDFFP
    - Adapter/Bridge/Composite/Decorator/Facade/Flyweight/Proxy
    - 适配器/桥接/组合/装饰/外观/享元/代理
- 行为型模式11
    - IOC_RST
    - Iterator/Observer/Command/Responsibility/Strategy/Template
    - 迭代器模式、观察者模式、命令模式、职责链模式、策略模式、模板方法模式
    - MIS_MV
    - Memento/Interpreter/State/Mediator/Visitor
    - 备忘录模式、解释器模式、状态模式、中介者模式、访问者模式
- 1.什么是设计模式？
- 你是否在你的代码里面使用过任何设计模式？
    - 用过呀，创建型都用过，结构型也基本用过，行为型用过常用的6个
- 你可以说出几个在JDK库中使用的设计模式吗？
    - 创建型
        - Runtime#getRuntime() 单例
        - Proxy#newProxyInstance() 工厂方法
        - DriverManager#getConnection() 抽象工厂
        - StringBuilder() 建造者
        - Object#clone() 原型
    - 结构型
        - InputStreamReader 适配器模式
        - java.util.logging包中的Handler和Formatter 桥接模式
        - Collection#addAll 组合模式
        - BufferedReader 装饰者模式
        - java.lang.Class 外观模式
        - Integer#valueOf(int) 享元模式
        - Proxy#newProxyInstance() 代理模式
    - 行为型    
        - Iterator 迭代器模式
        - EventListener 观察者模式
        - Runnable#run() 命令模式
        - Filter#doFilter() 责任链模式
        - Comparator#compare() 策略模式
        - Filter#doFilter() 模板方法模式
