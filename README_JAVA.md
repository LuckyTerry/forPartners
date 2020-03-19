# TODO

## 目录

- [数据库](#数据库)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MongoDB](#mongodb)
        - [范式/反范式](#范式/反范式)
        - [分布式主键](#objectId作为唯一键)
    - [Elasticsearch](#elasticsearch)
        - [倒排索引](#倒排索引)
- [系统设计](#系统设计)
    - [MyBatis](#MyBatis)
    - [Servlet容器](#Servlet容器)
    - [认证授权(JWT、SSO)](#认证授权)
        - [JWT](#JWT)
        - [SSO](#SSO)
        - [SpringSecurity](#SpringSecurity)
    - [分布式](#分布式)
        - [一致性协议](#一致性协议)
        - [RPC](#rpc)
        - [消息队列](#消息队列)
        - [API 网关](#api-网关)
        - [唯一id生成](#唯一-id-生成)
        - [分布式锁](#分布式锁)
        - [分布式事务](#分布式事务)
        - [分布式任务调度](#分布式任务调度)
        - [ZooKeeper](#zookeeper)
        - [数据库扩展](#数据库扩展)
    - [大型网站架构](#大型网站架构)
        - [性能测试](#性能测试)
        - [高并发](#高并发)
        - [高可用](#高可用)
    - [微服务](#微服务)
        - [Spring MVC](#spring-mvc)
        - [Spring Webflux](#spring-webflux)
        - [Spring Boot](#spring-boot)
        - [Spring Cloud](#spring-cloud)
        - [配置中心](#配置中心)
        - [注册中心](#注册中心)
    - [服务治理](#服务治理)
        - [灰度发布](#灰度发布)
        - [链路追踪](#链路追踪)
        - [日志采集](#日志采集)
        - [APM工具](#APM工具)
    - [团队管理](#团队管理)
        - [scrum](#scrum)
        - [白板站会](#白板站会)
        - [代码审查](#代码审查)
        - [分支规范](#分支规范)
        - [项目管理](#项目管理)
        - [协同工作](#协同工作)
    - [架构图](#架构图)
        - [业务架构图](#业务架构图)
        - [技术架构图](#技术架构图)
    - [线上部署相关](#线上部署相关)
        - [应用上云](#应用上云)
        - [混合云方案](#混合云方案)
        - [目前的线上部署](#目前的线上部署)
    - [持续开发集成](#持续开发集成)
        - [Jenkins](#Jenkins)
        - [Gitlab](#Gitlab)
    - [个人真实案例](#个人真实案例)
        - 待大家补充
- [面试指南](#面试指南)
    - [为什么离职](#为什么离职)
    - [自我介绍](#自我介绍)
    - [引导面试官](#引导面试官)
    - [未来的规划](#未来的规划)
    - [自己的提问](#自己的提问)
- [Java学习常见问题汇总](#Java学习常见问题汇总)
- [工具](#工具)
    - [Docker](#docker)
    - [其他](#其他)

## 数据库

### MySQL

- **ACID**
- **索引**
- **锁**
- 分库分表
- layer分层

1. **[【推荐】MySQL/数据库 知识点总结](docs/database/MySQL.md)**
2. **[阿里巴巴开发手册数据库部分的一些最佳实践](docs/database/阿里巴巴开发手册数据库部分的一些最佳实践.md)**
3. **[一千行MySQL学习笔记](docs/database/一千行MySQL命令.md)**
4. [MySQL高性能优化规范建议](docs/database/MySQL高性能优化规范建议.md)
5. [数据库索引总结](docs/database/MySQL%20Index.md)
6. [事务隔离级别(图文详解)](docs/database/事务隔离级别(图文详解).md)
7. [一条SQL语句在MySQL中如何执行的](docs/database/一条sql语句在mysql中如何执行的.md)
8. [MVCC多版本并发控制](https://github.com/LuckyTerry/forPartners/blob/master/docs/database/MySQL.md)

### Redis

- **五种数据类型**
- 构成五种数据结构的底层数据结构
- 单线程、守护线程、NIO
- redis单例、集群
- lua脚本
- **缓存雪崩，缓存穿透，缓存击穿**
- 主从机制
- 跳表

* [Redis 总结](docs/database/Redis/Redis.md)
* [Redlock分布式锁](docs/database/Redis/Redlock分布式锁.md)
* [如何做可靠的分布式锁，Redlock真的可行么](docs/database/Redis/如何做可靠的分布式锁，Redlock真的可行么.md)
* [几种常见的 Redis 集群以及使用场景](docs/database/Redis/redis集群以及应用场景.md) 

### MongoDB

- 范式/反范式
- 分布式唯一主键

todo

### Elasticsearch

- 倒排索引

TODO

## 系统设计

### MyBatis

- [MyBatis常见面试题总结](docs/system-design/framework/mybatis/mybatis-interview.md)

### Servlet容器

- tomcat
- jetty
- undertow
- netty

todo

### 认证授权

**[认证授权基础:搞清Authentication,Authorization以及Cookie、Session、Token、OAuth 2、SSO](docs/system-design/authority-certification/basis-of-authority-certification.md)**

#### JWT

- **[JWT 优缺点分析以及常见问题解决方案](docs/system-design/authority-certification/JWT-advantages-and-disadvantages.md)**

#### SSO(单点登录)

SSO(Single Sign On)即单点登录说的是用户登陆多个子系统的其中一个就有权访问与其相关的其他系统。举个例子我们在登陆了京东金融之后，我们同时也成功登陆京东的京东超市、京东家电等子系统。相关阅读：**[SSO 单点登录看这篇就够了！](docs/system-design/authority-certification/sso.md)**

#### SpringSecurity

- **[适合初学者入门 Spring Security With JWT 的 Demo](https://github.com/Snailclimb/spring-security-jwt-guide)**

### 分布式

[分布式相关概念入门](docs/system-design/website-architecture/分布式.md)

#### 一致性协议

- Raft
- Paxos
- Zab协议(zookeeper自研)

代办......

#### RPC

让调用远程服务调用像调用本地方法那样简单。

- [Dubbo 总结：关于 Dubbo 的重要知识点](docs/system-design/data-communication/dubbo.md)
- [服务之间的调用为啥不直接用 HTTP 而用 RPC？](docs/system-design/data-communication/why-use-rpc.md)

#### 消息队列

消息队列在分布式系统中主要是为了接耦和削峰。相关阅读： **[消息队列总结](docs/system-design/data-communication/message-queue.md)** 。

**RabbitMQ:**

1. [RabbitMQ 入门](docs/system-design/data-communication/rabbitmq.md)

**RocketMQ:**

- 单例、集群
- 死信队列的应用
- 延迟队列的应用
- **高可用部署**

1. [RocketMQ 入门](docs/system-design/data-communication/RocketMQ.md)
2. [RocketMQ的几个简单问题与答案](docs/system-design/data-communication/RocketMQ-Questions.md)

**Kafka:**

1. **[Kafka 入门+SpringBoot整合Kafka系列](https://github.com/Snailclimb/springboot-kafka)**
2. [Kafka系统设计开篇-面试看这篇就够了](docs/system-design/data-communication/Kafka系统设计开篇-面试看这篇就够了.md)
3. [【加餐】Kafka入门看这一篇就够了](docs/system-design/data-communication/Kafka入门看这一篇就够了.md)

**EMQ**

轻量级消息队列：EMQ(基于mqtt轻量级协议)

- topic规则

todo

#### API 网关

网关主要用于请求转发、安全认证、协议转换、容灾。

[浅析如何设计一个亿级网关(API Gateway)](docs/system-design/micro-service/API网关.md)

#### 唯一id生成

- UUID/GUID
- 数据库自增ID
- Twitter-Snowflake
- 基于redis的分布式ID生成器
- MongoDB的ObjectId
- Zookeeper的znode
- 百度的UidGenerator
- 美团的Leaf
- ecp-uid

[分布式id生成方案总结](docs/system-design/micro-service/分布式id生成方案总结.md)

#### 分布式锁

- redis
- redisson
- zookeeper

todo

#### 分布式事务

- 2PC
- TCC
- 阿里Seata

todo

#### 分布式任务调度

todo

#### ZooKeeper

> 前两篇文章可能有内容重合部分，推荐都看一遍。

1. [【入门】ZooKeeper 相关概念总结](docs/system-design/framework/ZooKeeper.md)
2. [【进阶】Zookeeper 原理简单入门！](docs/system-design/framework/ZooKeeper-plus.md)
3. [【拓展】ZooKeeper 数据模型和常见命令](docs/system-design/framework/ZooKeeper数据模型和常见命令.md)

#### 其他

- 接口幂等性（代办）：分布式系统必须要考虑接口的幂等性。

#### 数据库扩展

读写分离、分库分表。

代办.....

### 大型网站架构

- [8 张图读懂大型网站技术架构](docs/system-design/website-architecture/8%20张图读懂大型网站技术架构.md)
- [关于大型网站系统架构你不得不懂的10个问题](docs/system-design/website-architecture/关于大型网站系统架构你不得不懂的10个问题.md)

#### 性能测试

- jps、jstat、jmap、jstack、jvisualVM、阿里Arthas

- [后端程序员也要懂的性能测试知识](https://articles.zsxq.com/id_lwl39teglv3d.html) （知识星球）

#### 高并发

待办......

#### 高可用

高可用描述的是一个系统在大部分时间都是可用的，可以为我们提供服务的。高可用代表系统即使在发生硬件故障或者系统升级的时候，服务仍然是可用的 。相关阅读： **《[如何设计一个高可用系统？要考虑哪些地方？](docs/system-design/website-architecture/如何设计一个高可用系统？要考虑哪些地方？.md)》** 。

### 微服务=

#### Spring

1. [Spring 学习与面试](docs/system-design/framework/spring/Spring.md)
2. **[Spring 常见问题总结](docs/system-design/framework/spring/SpringInterviewQuestions.md)**
3. [Spring中 Bean 的作用域与生命周期](docs/system-design/framework/spring/SpringBean.md)
4. [SpringMVC 工作原理详解](docs/system-design/framework/spring/SpringMVC-Principle.md)
5. [Spring中都用到了那些设计模式?](docs/system-design/framework/spring/Spring-Design-Patterns.md)

#### SpringBoot

- beanFactory
- bean单例问题
- springboot的启动过程
- spring容器的启动过程
- to be added

- **[SpringBoot 指南/常见面试题总结](https://github.com/Snailclimb/springboot-guide)**

#### Spring Cloud

- [ 大白话入门 Spring Cloud](docs/system-design/micro-service/spring-cloud.md)

#### SpringWebflux

TODO

#### 配置中心

待办......

#### 注册中心

待办......

### 服务治理

TODO

#### 灰度发布

todo

#### 链路追踪

todo

#### 日志采集

todo

#### APM工具

todo

### 团队管理

TODO

#### scrum

todo

#### 白板站会

todo

#### 代码审查

todo

#### 分支规范

todo

#### 项目管理

禅道

#### 协同工作

钉钉、石墨、蓝湖

### 架构图

![image](https://github.com/LuckyTerry/forPartners/blob/master/images/1.png)

#### 业务架构图

todo

#### 技术架构图

todo

### 线上部署相关

TODO

#### 应用上云

todo

#### 混合云方案

todo

#### 目前的线上部署

request -> dns -> loadBalancer -> nginx -> gateway -> aggregation -> service -> repository 

### 持续开发集成

- [持续集成之理论篇](https://zhuanlan.zhihu.com/p/57021068)

Start -> 获取JenkinsCICD文件 -> 拉取代码 -> mvn打包 -> 获取Dockerfile和yaml文件 -> 制作并上传Docker镜像 -> 部署服务 -> 清空工作区 -> End

### 个人真实案例

亲自经历的优化点、问题解决案例

TODO

## 面试指南

1. **[【备战面试1】程序员的简历就该这样写](docs/essential-content-for-interview/PreparingForInterview/程序员的简历之道.md)**
2. **[【备战面试2】初出茅庐的程序员该如何准备面试？](docs/essential-content-for-interview/PreparingForInterview/interviewPrepare.md)**
3. **[【备战面试3】7个大部分程序员在面试前很关心的问题](docs/essential-content-for-interview/PreparingForInterview/JavaProgrammerNeedKnow.md)**
4. **[【备战面试4】Github上开源的Java面试/学习相关的仓库推荐](docs/essential-content-for-interview/PreparingForInterview/JavaInterviewLibrary.md)**
5. **[【备战面试5】如果面试官问你“你有什么问题问我吗？”时，你该如何回答](docs/essential-content-for-interview/PreparingForInterview/面试官-你有什么问题要问我.md)**
6. [【备战面试6】应届生面试最爱问的几道 Java 基础问题](docs/essential-content-for-interview/PreparingForInterview/应届生面试最爱问的几道Java基础问题.md)
7. **[【备战面试6】美团面试常见问题总结(附详解答案)](docs/essential-content-for-interview/PreparingForInterview/美团面试常见问题总结.md)**
8. **[【备战面试7】一些刁难的面试问题总结](https://xiaozhuanlan.com/topic/9056431872)**

### 真实面试题

Java编程思想5大原则？

Spring的三种注入方式？

Redis持久化机制？

## Java学习常见问题汇总

1. [Java学习路线和方法推荐](docs/questions/java-learning-path-and-methods.md)
2. [Java培训四个月能学会吗？](docs/questions/java-training-4-month.md)
3. [新手学习Java，有哪些Java相关的博客，专栏，和技术学习网站推荐？](docs/questions/java-learning-website-blog.md)
4. [Java 还是大数据，你需要了解这些东西！](docs/questions/java-big-data)
5. [Java 后台开发/大数据？你需要了解这些东西！](https://articles.zsxq.com/id_wto1iwd5g72o.html)（知识星球）

## 工具

### Docker

1. [Docker 基本概念解读](docs/tools/Docker.md)
2. [一文搞懂 Docker 镜像的常用操作！](docs/tools/Docker-Image.md )

### 其他

- [阿里云服务器使用经验](docs/tools/阿里云服务器使用经验.md)
