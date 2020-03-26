# Spring Cloud速记
1. 什么是Spring cloud
- 微服务架构的一站式解决方案
- 包括了：服务注册发现、负载均衡、断路器、链路追踪、配置中心、消息总线 等
2. Eureka 的一些基础概念和功能
- 服务注册
- 服务发现
- 服务提供者
- 服务消费者
- 服务续约
- 获取注册表
- 服务下线
- 服务剔除
3. 为什么需要 Ribbon？Ribbon 的几种负载均衡算法
- 一般为了高可用都会部署服务集群，为了将请求负载均衡到不同的节点，就需要一套完整的负载均衡框架
- 随机 random
- 轮询 round robin
- 重试 retry
- 区域 zone
- 加权 weighted
- 哈希 hash
- 一致性哈希 consistent hash
4. 什么是 Open Feign
- 集成 eureka, restTemplate, ribbon ，使用者只需定义接口及url等，feign便会生成代理类，完成远程调用逻辑增强
5. 什么是 Hystrix之熔断和降级
- 指定时间窗内的请求失败率达到阈值，便通过断路器将请求链路断开
- 远程调用异常后，应该执行降级逻辑来给用户更好的体验
6. Hystrix资源隔离方式https://blog.csdn.net/javaer\_lee/article/details/87942816
- 信号量隔离
- 线程池隔离
7. 网关
8. 配置中心
- 既能对配置文件统一管理，也能在运行时动态修改配置文件，如 apollo
9. 消息总线
- 可以配合 spring cloud config 完成配置刷新，因为项目中使用的时apollo，所以不涉及到bus
10. redis补充 HyperLoglog GeoHash查找附近的人
- 基于经纬度的地理位置算法工具，能计算出两个点的距离，一个点的附近 等等

踩过哪些坑，怎么解决的？
- 略 
链路追踪的信息是怎么传递的？
- 保存在线程的ThreaLocalMap中
SpanId怎么保证唯一性？
- 在网关进行生成，可以用分布式唯一ID算法
RpcContext是在什么维度传递的？
- 线程维度
Dubbo的远程调用怎么实现的？
- client clientStub server serverStub
分布式追踪的上下文是怎么存储和传递的？
- 通过threadlocal和header
