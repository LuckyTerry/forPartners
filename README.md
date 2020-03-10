# TODO

## 目录

- [背景](#背景)
- [感谢](#感谢)
- [网络](#网络)
- [操作系统](#操作系统)
    - [Linux](#linux)
- [数据结构与算法](#数据结构与算法)
    - [数据结构](#数据结构)
    - [算法](#算法)
- [设计原则与模式](#设计原则与模式)
- [Java](#java)
    - [基础](#基础)
    - [集合](#集合)
    - [并发](#并发)
    - [JVM](#jvm)
    - [泛型](#泛型)
    - [反射](#反射)
    - [Servlet容器](#Servlet容器)
    - [IO](#IO)
    - [Java 8](#Java 8)
    - [Java内存模型](#Java内存模型)
    - [Java线程模型](#Java线程模型)
    - [JVM线程模型](#JVM线程模型)
    - [其他](#其他)
- [Java学习常见问题汇总](#Java学习常见问题汇总)
- [Web方向](#Web方向)
- [Java方向](#Java方向)
- [Android方向](#Android方向)
- [工具](#工具)
    - [Git](#git)
    - [Docker](#docker)
    - [其他](#其他)
- [说明](#说明)
    - [JavaGuide介绍](#javaguide介绍)
    - [关于转载](#关于转载)
    - [Contributor](#Contributor)

## 背景

在公司教科书式的变相裁员下，Java团队奋力抗争，浴火重生！

## 感谢

**感谢** [Snailclimb/JavaGuide](https://github.com/Snailclimb/JavaGuide) 项目

## 网络

1. [计算机网络常见面试题](docs/network/计算机网络.md)
2. [计算机网络基础知识总结](docs/network/干货：计算机网络知识总结.md)
3. [HTTPS中的TLS](docs/network/HTTPS中的TLS.md)

## 操作系统

操作系统相关概念总结

### Linux

* [后端程序员必备的 Linux 基础知识](docs/operating-system/后端程序员必备的Linux基础知识.md)  
* [Shell 编程入门](docs/operating-system/Shell.md) 

## 数据结构与算法

### 数据结构

- [不了解布隆过滤器？一文给你整的明明白白！](docs/dataStructures-algorithms/data-structure/bloom-filter.md)
- [数据结构知识学习与面试](docs/dataStructures-algorithms/数据结构.md)

### 算法

- [算法学习资源推荐](docs/dataStructures-algorithms/算法学习资源推荐.md)  
- [几道常见的字符串算法题总结 ](docs/dataStructures-algorithms/几道常见的子符串算法题.md)
- [几道常见的链表算法题总结 ](docs/dataStructures-algorithms/几道常见的链表算法题.md)   
- [剑指offer部分编程题](docs/dataStructures-algorithms/剑指offer部分编程题.md)
- [公司真题](docs/dataStructures-algorithms/公司真题.md)
- [回溯算法经典案例之N皇后问题](docs/dataStructures-algorithms/Backtracking-NQueens.md)

## 设计原则与模式

TODO

## Java

### 基础

**基础知识系统总结：**

1. **[Java 基础知识](docs/java/Java基础知识.md)**
2. **[Java 基础知识疑难点/易错点](docs/java/Java疑难点.md)**
3. [【加餐】一些重要的Java程序设计题](docs/java/Java程序设计题.md)
4. [【选看】J2EE 基础知识](docs/java/J2EE基础知识.md)

**重要知识点详解：**

1. [枚举](docs/java/basic/用好Java中的枚举真的没有那么简单.md) （很重要的一个数据结构，用好枚举真的没有那么简单！）
2. [Java 常见关键字总结：final、static、this、super!](docs/java/basic/final、static、this、super.md)
3. [什么是反射机制?反射机制的应用场景有哪些?](docs/java/basic/reflection.md)

**其他：**

1. [JAD反编译](docs/java/JAD反编译tricks.md)

### 集合

1. **[Java容器常见面试题/知识点总结](docs/java/collection/Java集合框架常见面试题.md)**
2. [ArrayList 源码](docs/java/collection/ArrayList.md)  、[LinkedList 源码](docs/java/collection/LinkedList.md)   、[HashMap(JDK1.8)源码](docs/java/collection/HashMap.md)  

### 并发

**面试题总结：**

1. **[Java 并发基础常见面试题总结](docs/java/Multithread/JavaConcurrencyBasicsCommonInterviewQuestionsSummary.md)**
2. **[Java 并发进阶常见面试题总结](docs/java/Multithread/JavaConcurrencyAdvancedCommonInterviewQuestions.md)**

**必备知识点：**

1. [并发容器总结](docs/java/Multithread/并发容器总结.md)
2. **[Java线程池学习总结](./docs/java/Multithread/java线程池学习总结.md)**
3. [乐观锁与悲观锁](docs/essential-content-for-interview/面试必备之乐观锁与悲观锁.md)
4. [JUC 中的 Atomic 原子类总结](docs/java/Multithread/Atomic.md)
5. [AQS 原理以及 AQS 同步组件总结](docs/java/Multithread/AQS.md)

### JVM

1. **[Java内存区域](docs/java/jvm/Java内存区域.md)**
2. **[JVM垃圾回收](docs/java/jvm/JVM垃圾回收.md)**
3. [JDK 监控和故障处理工具](docs/java/jvm/JDK监控和故障处理工具总结.md)
4. [类文件结构](docs/java/jvm/类文件结构.md)
5. **[类加载过程](docs/java/jvm/类加载过程.md)**
6. [类加载器](docs/java/jvm/类加载器.md)
7. **[【待完成】最重要的 JVM 参数指南（翻译完善了一半）](docs/java/jvm/最重要的JVM参数指南.md)**
8. [JVM 配置常用参数和常用 GC 调优策略](docs/java/jvm/GC调优参数.md)
9. **[【加餐】大白话带你认识JVM](docs/java/jvm/[加餐]大白话带你认识JVM.md)**

### 泛型

TODO

### Servlet容器

TODO

### IO

TODO

### Java 8

TODO

### Java内存模型

TODO

### Java线程模型

TODO

### JVM线程模型

TODO

### 其他

1. **I/O** ：[BIO,NIO,AIO 总结 ](docs/java/BIO-NIO-AIO.md)
2. **Java 8**  ：[Java 8 新特性总结](docs/java/What's%20New%20in%20JDK8/Java8Tutorial.md)、[Java 8 学习资源推荐](docs/java/What's%20New%20in%20JDK8/Java8教程推荐.md)、[Java8 forEach 指南](docs/java/What's%20New%20in%20JDK8/Java8foreach指南.md)
3.  **[Java 编程规范以及优雅 Java 代码实践总结](docs/java/Java编程规范.md)**
4. 设计模式 :[设计模式系列文章](docs/system-design/设计模式.md)

## 面试指南

1. **[【备战面试1】程序员的简历就该这样写](docs/essential-content-for-interview/PreparingForInterview/程序员的简历之道.md)**
2. **[【备战面试2】初出茅庐的程序员该如何准备面试？](docs/essential-content-for-interview/PreparingForInterview/interviewPrepare.md)**
3. **[【备战面试3】7个大部分程序员在面试前很关心的问题](docs/essential-content-for-interview/PreparingForInterview/JavaProgrammerNeedKnow.md)**
4. **[【备战面试4】Github上开源的Java面试/学习相关的仓库推荐](docs/essential-content-for-interview/PreparingForInterview/JavaInterviewLibrary.md)**
5. **[【备战面试5】如果面试官问你“你有什么问题问我吗？”时，你该如何回答](docs/essential-content-for-interview/PreparingForInterview/面试官-你有什么问题要问我.md)**
6. [【备战面试6】应届生面试最爱问的几道 Java 基础问题](docs/essential-content-for-interview/PreparingForInterview/应届生面试最爱问的几道Java基础问题.md)
7. **[【备战面试6】美团面试常见问题总结(附详解答案)](docs/essential-content-for-interview/PreparingForInterview/美团面试常见问题总结.md)**
8. **[【备战面试7】一些刁难的面试问题总结](https://xiaozhuanlan.com/topic/9056431872)**

## Java学习常见问题汇总

1. [Java学习路线和方法推荐](docs/questions/java-learning-path-and-methods.md)
2. [Java培训四个月能学会吗？](docs/questions/java-training-4-month.md)
3. [新手学习Java，有哪些Java相关的博客，专栏，和技术学习网站推荐？](docs/questions/java-learning-website-blog.md)
4. [Java 还是大数据，你需要了解这些东西！](docs/questions/java-big-data)
5. [Java 后台开发/大数据？你需要了解这些东西！](https://articles.zsxq.com/id_wto1iwd5g72o.html)（知识星球）

## 工具

### Git

* [Git入门](docs/tools/Git.md)

### Docker

1. [Docker 基本概念解读](docs/tools/Docker.md)
2. [一文搞懂 Docker 镜像的常用操作！](docs/tools/Docker-Image.md )

### 其他

- [阿里云服务器使用经验](docs/tools/阿里云服务器使用经验.md)

## 说明

开源项目在于大家的参与，这才使得它的价值得到提升。感谢🙏有你！

### JavaGuide介绍

开源 JavaGuide 初始想法源于自己的个人那一段比较迷茫的学习经历。主要目的是为了通过这个开源平台来帮助一些在学习 Java 或者面试过程中遇到问题的小伙伴。

*  **对于 Java 初学者来说：** 本文档倾向于给你提供一个比较详细的学习路径，让你对于Java整体的知识体系有一个初步认识。另外，本文的一些文章
也是你学习和复习 Java 知识不错的实践；
*  **对于非 Java 初学者来说：** 本文档更适合回顾知识，准备面试，搞清面试应该把重心放在那些问题上。要搞清楚这个道理：提前知道那些面试常见，不是为了背下来应付面试，而是为了让你可以更有针对的学习重点。

Markdown 格式参考：[Github Markdown格式](https://guides.github.com/features/mastering-markdown/)，表情素材来自：[EMOJI CHEAT SHEET](https://www.webpagefx.com/tools/emoji-cheat-sheet/)。

利用 docsify 生成文档部署在 Github pages: [docsify 官网介绍](https://docsify.js.org/#/)

### 关于转载

如果你需要转载本仓库的一些文章到自己的博客的话，记得注明原文地址，并且注明借鉴项目Java-Guide的原文地址就可以了。

### Contributor

下面是本仓库的合作伙伴们，排名不分先后！

<a href="https://github.com/15198184721">
    <img src="https://avatars2.githubusercontent.com/u/16460404?s=400&v=4" width="45px">
</a>
<a href="https://github.com/caphehuan">
    <img src="https://avatars2.githubusercontent.com/u/20556760?s=400&v=4" width="45px">
</a>
<a href="https://github.com/chenchenaixuexi">
    <img src="https://avatars3.githubusercontent.com/u/23670998?s=400&v=4" width="45px">
</a>
<a href="https://github.com/crazyJjHe">
    <img src="https://avatars1.githubusercontent.com/u/34497590?s=400&v=4" width="45px">
</a>
<a href="https://github.com/forewei">
    <img src="https://avatars1.githubusercontent.com/u/40056555?s=460&v=4" width="45px">
</a>
<a href="https://github.com/guliangbiezou">
    <img src="https://avatars2.githubusercontent.com/u/15935710?s=400&v=4" width="45px">
</a>
<a href="https://github.com/Huangqr">
    <img src="https://avatars0.githubusercontent.com/u/49232400?s=400&v=4" width="45px">
</a>
<a href="https://github.com/JiangHao-1">
    <img src="https://avatars1.githubusercontent.com/u/53067327?s=400&v=4" width="45px">
</a>
<a href="https://github.com/mosvon">
    <img src="https://avatars2.githubusercontent.com/u/61421610?s=400&v=4" width="45px">
</a>
<a href="https://github.com/tocatch">
    <img src="https://avatars1.githubusercontent.com/u/24827471?s=400&v=4" width="45px">
</a>
<a href="https://github.com/wuhedong">
    <img src="https://avatars3.githubusercontent.com/u/25856750?s=400&v=4" width="45px">
</a>