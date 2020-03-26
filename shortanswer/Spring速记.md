# 框架速记
5. SpringAOP实现原理
- 实现接口，用JDK动态代理
- 未实现接口，用CGLIB动态代理
6. 静态代理、JDK动态代理、CGLIB动态代理
- 手动创建代理类，注入被代理对象，调用被代理对象的方法，并在调用前后加入增强逻辑
- 使用反射生成代理类，Proxy.newProxyInstance
- 使用字节码技术生成代理类
7. Spring AOP 基于AspectJ注解如何实现AOP
- 使用了aspectj5的注解，而未使用其编译器，底层是依靠动态代理实现的
8. Aop通知5种类型
- before, around, after, after return, after throwing
```
[Aspect1] around advise 1
[Aspect1] before advise
test OK
[Aspect1] around advise2
[Aspect1] after advise
[Aspect1] afterReturning advise
```
11. 怎么在程序运行的时候动态往 Spring IOC 容器注册新的 bean
12. @RestController vs @Controller
- RestController = ResponseBody + Controller
13. Spring 中的 bean 的作用域有哪些?5
- singleton、prototype、request、session、global session
14. Spring 中的单例 bean 的线程安全问题了解吗？
- 不使用共享变量，使用Threadloca，使用并发工具
15. @Component 和 @Bean 的区别是什么？
- Component 修饰类，对于系统/框架类就无能为力了
- Bean 修饰方法，比Component灵活
16. 将一个类声明为Spring的 bean 的注解有哪些?
- Controller、Service、Repository、Component
17.  Spring 中的 bean 生命周期?
- 反射创建bean
- setter方法设置值
- setXxxAware等方法的调用
- 前置处理器、PostConstruct、afterPropertySet、init-method、
- 后置处理器、PreDestroy、destroy、destroy-method
18. singleton 作用域下 bean 的生命周期和prototype作用域的bean的生命周期
- singleton下的bean的销毁有框架负责，prototype则不是，需要使用者自己去释放资源
18. 什么是 MVC 模式
- Model、View、Controller，用于数据层和视图层解耦
19. SpringMVC 工作原理  流程8
- 略
20. SpringMVC 重要组件6
- DispatcherServlet
- HandlerMapping
- HandlerAdapter
- HandlerChain
- ViewResolver
- View
21. Spring 框架中用到了哪些设计模式？
22. Spring 管理事务的方式有几种？
- 编程式
- 声明式
23.  Spring 事务中的隔离级别有哪几种?
- ru
- rc
- rr
- serial
24. Spring 事务中哪几种事务传播行为?
- required
- supports
- mandatory
- require_new
- not_supported
- never
- nested
25. @Transactional(rollbackFor = Exception.class)注解了解吗？
- 事务注解，不叫rollbackFor则默认只回滚RuntimeException
26. SpringBootApplication注解
- SpringBootConfiguration
- EnableAuthConfiguration
- ComponentScan
27. @PostConstruct和@PreDestroy
- java的注解，在前置处理器后和后置处理器前紧接着执行，方法无参无返回值
28. Servlet 生命周期
- init 只执行一次
- service 执行多次，内部调用doGet、doPost等
- destroy 只执行一次
29. 什么是 Spring Boot Starters?
- 一系列依赖关系的集合，并且一般都是实现自动配置的功能
30. Spring Boot 的自动配置是如何实现的?
- @EnableAutoConfiguration用于启用 SpringBoot 的自动配置机制
- 会扫描包路径下的spring.factories文件中的配置项，这些配置信息会被Spring容器当作bean来管理。
31. Spring Boot支持哪些嵌入式web容器？
- tomcat, jetty, undertow, netty
32. 什么是Spring Security ?
- 功能强大的框架，用于认证和授权。相比shiro要重一些，所以我们使用的shiro。
33. \#\{\}和|$\{\}的区别是什么？
- # 占位符
- $ 拼接符
34. Xml 映射文件中，除了常见的 select|insert|updae|delete 标签之外，还有哪些标签？
- resultMap, paramMap, assosiation, collection, sql, include, case when
35. 最佳实践中，通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？
- 不能，因为使用接口全限定名+方法名来确定一个dao接口的
36. Mybatis 是如何进行分页的？分页插件的原理是什么？
- RowBounds的内存分页，不用
- Interceptor实现的物理分页（对StatementHandler拦截）
37. 简述 Mybatis 的插件运行原理，以及如何编写一个插件。
- 原理就是动态代理，InvocationHandler的invoke方法
- 编写的话：实现interceptor接口，在注解里面指定要拦截哪一个接口的哪些方法
39. Mybatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？
- resultMap
- 列别名
40. Mybatis 能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。
- 
41. Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？
- association和collection支持
- 使用了cglib生成代理对象，调用get方法时如果不为null直接返回，如果为null发送事先保存好的关联查询sql把B查出来并赋值。
42. Mybatis 中如何执行批处理？
- 使用BatchExecutor
43. Mybatis 都有哪些 Executor 执行器？它们之间的区别是什么？
- SimpleExecutor，每执行一次update或select，就开一个statement
- ReuseExecutor，使用hashmap，以sql作为key，复用statement
- BatchExecutor，没用过。。。
44. 为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？
- 关联查询需要手动编写sql实现
45. 21.说一下springboot的启动过程？平时开发中都用哪些注解？
spring容器的启动过程？2
- todo
12、spring中Bean的作用域，springMVC的controller是线程安全的吗？怎么去保证线程安全呢？
- 略
Spring的生命周期吧
- 同 Bean 的生命周期。Bean、Spring、BeanFactory、FactoryBean的生命周期都这么回答
Spring的单例是怎么实现的？
- 单例注册表，本质上是个HashMap，通过通过在适当位置对hashMap进行加锁以实现并发安全
SpringMVC不同用户登录的信息怎么保证线程安全的？
- ThreadLocal来实现即可


















21.说一下springboot的启动过程？平时开发中都用哪些注解？
spring容器的启动过程？2
 
12、spring中Bean的作用域，springMVC的controller是线程安全的吗？怎么去保证线程安全呢？
 
Spring的生命周期吧
 
Spring的单例是怎么实现的？
 
SpringMVC不同用户登录的信息怎么保证线程安全的？
