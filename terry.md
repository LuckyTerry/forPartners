# Prepare

## 手画架构图

![手画架构图](https://terry-picture.oss-cn-qingdao.aliyuncs.com/public/2020-03-24/0d025f2d0e144e39bfa8d15dbcdde622-95010641-5D1F-4A1A-AE38-05FC04B8FECF.jpeg)

## 遇到的问题

1. ThreadLocal中的数据在切换线程的时候会丢失，需要手动取出来，放入到子线程中，导致代码里充斥着这样的业务逻辑无关的代码。，在装饰逻辑里完成取出并设置到子线程的动作。

- 对于hystrix的线程池模式，实现自定义HystrixConcurrencyStrategy，对Callbale进行包装
- 对于Async注解修饰的方法，直接设置TaskDecorator以包装Runnable
- 对于ThreadPoolTaskExecutor，直接设置TaskDecorator以包装Runnable
- 对于自定义的Executor，实现自定义ThreadFactoryBuilder,并注入TaskDecorator以包装Runnable。

如此依赖，彻底解决了上下文传递的问题。并且MDC中的信息也能正常传递，日志打印出来包含了完整的上下文数据，便于排错。

2. MybatisPlus的in方法和Mybatis的where标签导致全表查询

- MP的某一个版本及以前的in方法存在bug，当传入的Collection容量为0时，便不会动态拼接where语句，导致出现无where语句的查询sql，结果查了整张表的数据，导致OOM。最后，第一阶段是代码层次做参数验证，第二阶段是升级MP的版本(因为版本升级有API的破环，然后我们内部也涉及很多服务，在未经过充分测试的情况下，不能随意升级并发布，故分了两阶段来解决)
- 同样的问题出现在Mybatis的where标签上，where标签中的if条件全都没成立，导致没有where语句。然后再业务层加了参数校验以解决问题。

3. 有一个接口因为排序条件特别复杂，无法在数据库内完成排序，所以需要在业务层来处理，并且业务层存在着一条记录拆分为多条，多条记录合并为1条，的需求，频繁大量的深拷贝也会占用大量内存。一旦遇上有效数据超过10万左右时，便很容易出现OOM。

- 首先是生产环境只有很小概率会出现这么多有效数据
- 其次，经分析，查出所有数据不可避免，能做的就是减小在内存中的占用。于是，参照B+树的思想，分两阶段查询，第一阶段只查用于排序的若干字段，内存中完成排序；第二阶段就根据排序之后的有效数据再次查询数据库得到完整数据。

4. Rest远程调用的连接时间和超时时间

- Ribbon有连接时间和超时时间
- Feign有连接时间和超时时间
- Hystrix有连接时间和超时时间
- 那么到底谁配置的连接和超时时间是真正有效的呢？

查阅了很多文章，都讲解的不清晰甚至不正确。最后是在多篇文章 + 源码断点调试的配合下才确认了正确的配置。

5. k8s中部分服务pod的内存占用率高

- Pod设置内存最大为2G，JMV最大为512M，但是在运行一段时间后，部分服务对应的Pod的内存占用会达到1500+M，在buff/cache上能看到这一部分内存占用，而且即使没有什么请求了，内存仍然降不下来。后来有一次，因为某种原因需要升级某个服务的Spring版本，升级后突然发现该服务对应Pod的内存占用正常了

6. KDS响应时间过长

- 事故
    - 圣诞节前1周，有家大客户反馈，kds移动端应用在使用过程中相比较几天前能感受到明显的网络卡顿，经过日志分析，未发现任何异常。
    - 平安夜前一天晚上7点左右，该商户再次反馈，kds使用过程中网络卡顿更加频繁，几近无法使用，经日志分析，任未发现任何异常。调取链路分析，发现接口耗时最多的有四五十秒，远远大于正常的20ms。
    - 为了尽快恢复可用性，我们将服务伸缩到10个副本，然后准备分析dump文件，然而没撑到2分钟。再次扩大副本数，仍然没撑住。而且还引起了严重的服务雪崩的问题。因为业务服务的上层还有聚合服务层，聚合服务层目前采用的是信号量隔离，未做到服务或接口级别的线程池隔离，导致聚合层服务的线程被kds请求占满且阻塞，其他的业务请求无法处理。线上客户反馈激增，于是只好紧急扩大聚合层副本，并下线KDS服务。待问题解决后再次上线。
    - 回到另一边dump出来的堆栈信息，我们未发现死锁或gc等问题。再结合那么多副本都没撑住，那估计问题应该出现在数据库上面。
    - 于是查看MySQL的执行日志和正在执行的进程，发现存在大量SQL在等待获取表级锁。然后根据相关sql进行分析，发现update语句执行了锁表操作，而表内数据很多，一次update语句执行非常耗时，大量的用户请求阻塞在MySQL。那问题找到了，但是并没有马上修复后上线，而是花了一晚上的时间，对该服务的所有sql执行explain以查看执行过程，然后对几乎所有的sql进行了索引优化，并在预发布环境进行性能压测，最后在第二天大部分商户营业前上线了修复后的应用。
- 复盘：
    - 事故过程/原因/解决进行复盘。
    - 然后对 服务间调用的线程隔离、开发阶段的索引调优、测试阶段的性能压测、线上环境的日志/堆栈/慢SQL分析、服务雪崩后的紧急处理流程 等进行规范拟定

7. 过年期间 BootCDN 挂了2次

- 将静态文件部署在公司的OSS中，提高可用性

## 负责哪部分工作

todo

## 最自豪的项目？

todo

## 最难/最有技术含量的项目？

todo

## 最痛苦/最艰难的问题？

todo

## 技术故障/犯的错误？

todo

## Quick3Way

```
package com.alibaba.ttl;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Quick {

    public static void main(String[] args) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            data.add(i + 1);
        }
        Collections.shuffle(data);
        Comparable[] array = data.toArray(new Comparable[0]);
        sort(array, 0, data.size() - 1);
        for (Comparable comparable : array) {
            System.out.println(comparable);
        }
    }

    public static void sort(Comparable[] a, int lo, int hi) {
        if (lo >= hi) return;
        int lt = lo;
        int gt = hi;
        Comparable v = a[lo];
        int i = lo;
        while (i <= gt) {
            int cmp = a[i].compareTo(v);
            if (cmp < 0) {
                exchange(a, i++, lt++);
            } else if (cmp > 0) {
                exchange(a, i, gt--);
            } else {
                i++;
            }
        }
        sort(a, lo, lt -1);
        sort(a, gt + 1, hi);
    }

    private static boolean less(Comparable a, Comparable b) {
        return a.compareTo(b) < 0;
    }

    private static void exchange(Object[] a, int i, int j) {
        Object temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
}
```