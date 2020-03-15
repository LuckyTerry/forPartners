<!-- TOC -->

- [atomic](#atomic)
- [locks](#locks)
- [工具类](#工具类)
    - [TimeUnit](#TimeUnit)
    - [ThreadLocalRandom](#ThreadLocalRandom)
    - [CountDownLatch](#CountDownLatch)
    - [CyclicBarrier](#CyclicBarrier)
    - [Semaphore](#Semaphore)
- [执行类](#执行类)
    - [Executors](#Executors)
    - [ExecutorCompletionService](#ExecutorCompletionService)
- [CompletableFuture](#CompletableFuture)
    - [Exchanger](#Exchanger)
    - [Phaser](#Phaser)
    - [FutureTask](#FutureTask)
    - [ForkJoin](#ForkJoin)
    - [CountedCompleter](#CountedCompleter)
- [集合类](#集合类)
    - [队列双端队列](#队列双端队列)
    - [列表集合](#列表集合)
    - [字典](#字典)
- [Stream类中的仿JUC实现](#Stream类中的仿JUC实现)

<!-- /TOC -->

# JUC包实战指南

## atomic

- AtomicBoolean
- AtomicInteger
- AtomicIntegerArray
- AtomicIntegerFieldUpdater
- AtomicLong
- AtomicLongArray
- AtomicLongFieldUpdater
- AtomicMarkableReference
- AtomicReference
- AtomicReferenceArray
- AtomicReferenceFieldUpdater
- AtomicStampedReference
- DoubleAccumulator
- DoubleAdder
- LongAccumulator
- LongAdder
- Striped64

## locks

- StampedLock

核心思想在于，在读的时候如果发生了写，应该通过重试的方式来获取新的值，而不应该阻塞写操作。

StampedLock在读线程非常多而写线程非常少的场景下非常适用，同时还避免了写饥饿情况的发生。

[读写锁中的性能之王：StampedLock](https://juejin.im/post/5bacf523f265da0a951ee418)

```java
public class Point {

	private double x, y;
	
	private final StampedLock stampedLock = new StampedLock();
	
	//写锁的使用
	void move(double deltaX, double deltaY){
		
		long stamp = stampedLock.writeLock(); //获取写锁
		try {
			x += deltaX;
			y += deltaY;
		} finally {
			stampedLock.unlockWrite(stamp); //释放写锁
		}
	}
	
	//乐观读锁的使用
	double distanceFromOrigin() {
		
		long stamp = stampedLock.tryOptimisticRead(); //获得一个乐观读锁
		double currentX = x;
		double currentY = y;
		if (!stampedLock.validate(stamp)) { //检查乐观读锁后是否有其他写锁发生，有则返回false
			
			stamp = stampedLock.readLock(); //获取一个悲观读锁
			
			try {
				currentX = x;
			} finally {
				stampedLock.unlockRead(stamp); //释放悲观读锁
			}
		} 
		return Math.sqrt(currentX*currentX + currentY*currentY);
	}
	
	//悲观读锁以及读锁升级写锁的使用
	void moveIfAtOrigin(double newX,double newY) {
		
		long stamp = stampedLock.readLock(); //悲观读锁
		try {
			while (x == 0.0 && y == 0.0) {
				long ws = stampedLock.tryConvertToWriteLock(stamp); //读锁转换为写锁
				if (ws != 0L) { //转换成功
					
					stamp = ws; //票据更新
					x = newX;
					y = newY;
					break;
				} else {
					stampedLock.unlockRead(stamp); //转换失败释放读锁
					stamp = stampedLock.writeLock(); //强制获取写锁
					x = newX;
					y = newY;
				}
			}
		} finally {
			stampedLock.unlock(stamp); //释放所有锁
		}
	}
}
```

- ReentrantLock

[ReentrantLock与AQS](https://juejin.im/post/5b7235e951882560ed075893)

```java
Lock nonFairLock = new ReentrantLock();
Lock fairLock = new ReentrantLock(true);

fairLock.lock();
fairLock.unlock();
```

- ReentrantReadWriteLock

[你真的了解 ReentrantReadWriteLock 吗](https://juejin.im/post/5b9df6015188255c8f06923a)

```java
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

public static class ReadLock implements Lock, java.io.Serializable {
    public void lock() {
        sync.acquireShared(1); //共享
    }

    public void unlock() {
        sync.releaseShared(1); //共享
    }
}

public static class WriteLock implements Lock, java.io.Serializable {
    public void lock() {
        sync.acquire(1); //独占
    }

    public void unlock() {
        sync.release(1); //独占
    }
}
```

## 工具类

### TimeUnit

- TimeUnit.NANOSECONDS  纳秒
- TimeUnit.MICROSECONDS 微秒
- TimeUnit.MILLISECONDS 毫秒
- TimeUnit.SECONDS 秒
- TimeUnit.MINUTES 分
- TimeUnit.HOURS 时
- TimeUnit.DAYS  天

> TimeUnit.SECONDS.timedWait(1) 

同 Object.wait

调用某个对象的wait()方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor（即锁）

调用某个对象的notify()方法能够唤醒一个正在等待这个对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程；

调用notifyAll()方法能够唤醒所有正在等待这个对象的monitor的线程；


> TimeUnit.SECONDS.timedJoin(1)

 同 Thread.join

thread.join的含义是当前线程需要等待thread线程终止之后才从thread.join返回

```java
public void joinDemo(){
   Thread t = new Thread(payService);
   t.start();
   // 其他业务逻辑处理,不需要确定t线程是否执行完
   insertData();
   // 后续的处理，需要依赖t线程的执行结果，可以在这里调用join方法等待t线程执行结束
   t.join();
}
```

> TimeUnit.SECONDS.sleep(1) 

同 Thread.sleep

> TimeUnit.SECONDS.toMillis(1)

时间转换

### ThreadLocalRandom

线程安全的随机数生成器

```java
// 错误示例：seed唯一，产生了相同的随机数
public class ThreadLocalRandomDemo {

    private static final ThreadLocalRandom RANDOM = ThreadLocalRandom.current();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable {
                System.out.println(getName() + ": " + RANDOM.nextInt(100));
            }).start();
        }
    }
}

// 正确示例：给每个线程初始化一个 seed
public class ThreadLocalRandomDemo {

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable {
                System.out.println(getName() + ": " + ThreadLocalRandom.current().nextInt(100));
            }).start();
        }
    }
}
```

### CountDownLatch

> 等待直到计数减为0，可用于当前线程等待异步线程执行完毕

```java
public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(10);
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (int i=0; i<10; i++){
            executor.submit(new Runnable {
                try {
                    Thread.sleep(new Random().nextInt(10) * 1000);
                } finally {
                    // 计数减一
                    latch.countDown();
                }
            });
        }
        // 等待检查
        latch.await();
        // 关闭线程池
        executor.shutdown();
}
```

### CyclicBarrier

> 等待直到到达栅栏，可用于任务的绝对并发执行

```java
public static void main(String[] args) throws Exception {
    CyclicBarrier barrier = new CyclicBarrier(5);
    ExecutorService executor = Executors.newCachedThreadPool();

    for (int i = 0; i < 10; i++) {
        final int threadNum = i;
        Thread.sleep(1000);
        executor.execute(() -> {
            try {
                    Thread.sleep(1000);
                log.info("{} is ready", threadNum);
                try {
                    barrier.await(2000, TimeUnit.MILLISECONDS);
                } catch (Exception e) {
                    log.warn("BarrierException", e);
                }
                log.info("{} continue", threadNum);
            } catch (Exception e) {
                log.error("exception", e);
            }
        });
    }
    executor.shutdown();
}
```

### Semaphore

用途：用来控制同时执行某个指定操作的数量。

原理：Semaphore管理着一组许可（permit）,许可的初始数量可以通过构造函数设定，操作时首先要获取到许可，才能进行操作，操作完成后需要释放许可。如果没有获取许可，则阻塞到有许可被释放。如果初始化了一个许可为1的Semaphore，那么就相当于一个不可重入的互斥锁（Mutex）。

```java
public static void main(String[] args) {
    Semaphore s = new Semaphore(10);
    ExecutorService threadPool = Executors.newFixedThreadPool(30);
    for (int i = 0; i < 30; i++) {
        threadPool.execute(new Runnable {
             try {
                semaphore.acquire();
                System.out.println(i + "is using the toilet");
                TimeUnit.MILLISECONDS.sleep(rand.nextInt(2000));
                semaphore.release();
                System.out.println(i + "is leaving");
            } catch (InterruptedException e) {

            }
        });
    }

    threadPool.shutdown();
}
```

## 执行类

### Executors

Executors、ThreadPoolExecutor 和 ScheduledThreadPoolExecutor

1. 类继承关系

![hierarchy.jpg](https://note.youdao.com/yws/res/2572/WEBRESOURCE09e10e9a7138ed2496bfa7ce43921f74)

2. 任务类型

   - Runnable接口
   - Callable接口

    两者的区别：

    > Runnable接口不会返回结果但是Callable接口可以返回结果

3. 任务执行

    ```
    ExecutorService.execute(Runnable task)
    ExecutorService.submit(Runnable task)
    ExecutorService.submit(Callable task)
    ```

    > execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；

    > submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

4. 任务调度

   - FixedThreadPool： 适用于为了满足资源管理需求，而需要限制当前线程数量的应用场景。它适用于负载比较重的服务器；
   - SingleThreadExecutor： 适用于需要保证顺序地执行各个任务并且在任意时间点，不会有多个线程是活动的应用场景。
   - CachedThreadPool： 适用于执行很多的短期异步任务的小程序，或者是负载较轻的服务器；
   - ScheduledThreadPoolExecutor： 适用于需要多个后台执行周期任务，同时为了满足资源管理需求而需要限制后台线程的数量的应用场景，
   - SingleThreadScheduledExecutor： 适用于需要单个后台线程执行周期任务，同时保证顺序地执行各个任务的应用场景。

   四个参数
   - corePoolSize 核心线程池大小
   - maximumPoolSize 最大线程池大小
   - queue 暂时保存任务的工作队列
   - rejectedExecutionHandler 拒绝策略

   如何创建

   - 静态工厂方法创建

   ```java
   Executors.newFixedThreadPool();
   Executors.newCachedThreadPool();
   Executors.newScheduledThreadPool();
   Executors.newWorkStealingPool();
   Executors.newSingleThreadExecutor();
   Executors.newSingleThreadScheduledExecutor();
   ```

   - 构造函数创建

   ```java
   new ThreadPoolExecutor();
   new ScheduledThreadPoolExecutor();
   ```

### ExecutorCompletionService

> 使用场景：主线程提交多个异步任务，然后希望有任务完成就处理结果，并且按任务完成顺序逐个处理

```java
public void testJuc() {
    ExecutorService executor = Executors.newFixedThreadPool(10);
    CompletionService<Integer> completionService = new ExecutorCompletionService<>(executor);
    try {
        for (int i = 0; i < 10; i++) {
            completionService.submit(new Callable<Integer>() {
                @Override
                public Integer call() throws Exception {
                    int timeout = new Random().nextInt(1000);
                    TimeUnit.MILLISECONDS.sleep(timeout);
                    return timeout;
                }
            });
        }
        for (int i = 0; i < 10; i++) {
            try {
                Future<Integer> result = completionService.take();
                System.out.println(result.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
    } finally {
        executor.shutdown();
    }
}
```

## CompletableFuture

1. 创建 CompletableFuture

使用 complete() 手动完成任务

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();
new Thread(() -> {
    try {
        TimeUnit.MILLISECONDS.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    completableFuture.complete("Future's Result");
}).start();
try {
    System.out.println(completableFuture.get());
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

使用 runAsync() 运行异步计算

```java
// Run a task specified by a Runnable Object asynchronously.
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // Simulate a long-running Job   
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    System.out.println("I'll run in a separate thread than the main thread.");
});
// Block and wait for the future to complete
System.out.println(future.get());
```

使用 supplyAsync() 运行一个异步任务并且返回结果

```java
// Run a task specified by a Supplier object asynchronously
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Result of the asynchronous computation";
});
// Block and get the result of the Future
System.out.println(future.get());
```

2. 转换和运行

thenApply()

```java
CompletableFuture<String> welcomeText = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return "Terry";
}).thenApply(name -> {
    return "Hello " + name;
}).thenApply(greeting -> {
    return greeting + ", Welcome to the CalliCoder Blog";
});

// Prints - Hello Rajeev, Welcome to the CalliCoder Blog
System.out.println(welcomeText.get());
```

thenAccept() 和 thenRun()

```java
CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Terry";
}).thenAccept(name -> System.out.println("Hello " + name));

CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Terry";
}).thenRun(() -> System.out.println("Finished"));
```

thenApplyAsync() 、 thenAcceptAsync() 和 thenRunAsync()

```java
CountDownLatch latch = new CountDownLatch(3);

System.out.println("Main Thread: " + Thread.currentThread().getName());

// task1
CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("task 1 step 1 sleeps at " + Thread.currentThread().getName());
        // print task 1 step 1 sleeps at ForkJoinPool.commonPool-worker-2
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Terry";
}).thenApply(name -> {
    // Executed in the same thread where the supplyAsync() task is executed
    // or in the main thread If the supplyAsync() task completes immediately (Remove sleep() call to verify)
    try {
        System.out.println("task 1 step 2 sleeps at " + Thread.currentThread().getName());
        // print task 1 step 2 sleeps at ForkJoinPool.commonPool-worker-2
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Hello " + name;
}).thenAccept(name -> {
    System.out.println("task 1 step 3 happens at " + Thread.currentThread().getName());
    // print task 1 step 3 happens at ForkJoinPool.commonPool-worker-2
    latch.countDown();
});

// task2
CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("task 2 step 1 sleeps at " + Thread.currentThread().getName());
        // print task 2 step 1 sleeps at ForkJoinPool.commonPool-worker-2
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Terry";
}).thenApplyAsync(name -> {
    // Executed in a different thread from ForkJoinPool.commonPool()
    try {
        System.out.println("task 2 step 2 sleeps at " + Thread.currentThread().getName());
        // print task 2 step 2 sleeps at ForkJoinPool.commonPool-worker-2
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Hello " + name;
}).thenAccept(name -> {
    System.out.println("task 2 step 3 happens at " + Thread.currentThread().getName());
    // print task 2 step 3 happens at ForkJoinPool.commonPool-worker-2
    latch.countDown();
});

// task3
CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("task 3 step 1 sleeps at " + Thread.currentThread().getName());
        // print task 3 step 1 sleeps at ForkJoinPool.commonPool-worker-6
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Terry";
}).thenApplyAsync(name -> {
    // Executed in a thread obtained from the executor
    try {
        System.out.println("task 2 step 3 sleeps at " + Thread.currentThread().getName());
        // print task 3 step 2 sleeps at pool-10-thread-1
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Hello " + name;
}, Executors.newFixedThreadPool(2)).thenAccept(name -> {
    System.out.println("task 3 step 3 happens at " + Thread.currentThread().getName());
    // print task 3 step 3 happens at pool-10-thread-1
    latch.countDown();
});

try {
    latch.await(10, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

1. 组合

使用 thenCompose()组合两个独立的future

> 用于请求的依赖调用，类似于 RxJava.flatmap

```java
CompletableFuture<User> getUsersDetail(String userId) {
	return CompletableFuture.supplyAsync(() -> {
		UserService.getUserDetails(userId);
	});	
}

CompletableFuture<Double> getCreditRating(User user) {
	return CompletableFuture.supplyAsync(() -> {
		CreditRatingService.getCreditRating(user);
	});
}

// 以下并不是想要的结果，future嵌套了
CompletableFuture<CompletableFuture<Double>> result = getUserDetail(userId).thenApply(user -> getCreditRating(user));

// 这才是想要的结果，结果compose了
CompletableFuture<Double> result = getUserDetail(userId).thenCompose(user -> getCreditRating(user));
```

使用thenCombine()组合两个独立的 future

> 用于请求的合并调用，类似于 Rxjava.combine 

```java
System.out.println("Retrieving weight.");
CompletableFuture<Double> weightInKgFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return 65.0;
});

System.out.println("Retrieving height.");
CompletableFuture<Double> heightInCmFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return 177.8;
});

System.out.println("Calculating BMI.");
CompletableFuture<Double> combinedFuture = weightInKgFuture
        .thenCombine(heightInCmFuture, (weightInKg, heightInCm) -> {
    Double heightInMeter = heightInCm/100;
    return weightInKg/(heightInMeter*heightInMeter);
});

System.out.println("Your BMI is - " + combinedFuture.get());
```

4. 组合多个CompletableFuture

CompletableFuture.allOf() 


```java
CompletableFuture<String> downloadWebPage(String pageLink) {
	return CompletableFuture.supplyAsync(() -> {
		// Code to download and return the web page's content
	});
} 

public void test() {
    List<String> webPageLinks = Arrays.asList(...)	// A list of 100 web page links

    // Download contents of all the web pages asynchronously
    List<CompletableFuture<String>> pageContentFutures = webPageLinks.stream()
            .map(webPageLink -> downloadWebPage(webPageLink))
            .collect(Collectors.toList());

    // Create a combined Future using allOf()
    CompletableFuture<Void> allFutures = CompletableFuture.allOf(
            pageContentFutures.toArray(new CompletableFuture[pageContentFutures.size()])
    );

    // When all the Futures are completed, call `future.join()` to get their results and collect the results in a list -
    // 花一些时间理解下以上代码片段。当所有future完成的时候，我们调用了future.join()，因此我们不会在任何地方阻塞。
    // join()方法和get()方法非常类似，这唯一不同的地方是如果最顶层的CompletableFuture完成的时候发生了异常，它会抛出一个未经检查的异常。
    CompletableFuture<List<String>> allPageContentsFuture = allFutures.thenApply(v -> {
    return pageContentFutures.stream()
            .map(pageContentFuture -> pageContentFuture.join())
            .collect(Collectors.toList());
    });

    // Count the number of web pages having the "CompletableFuture" keyword.
    CompletableFuture<Long> countFuture = allPageContentsFuture.thenApply(pageContents -> {
        return pageContents.stream()
                .filter(pageContent -> pageContent.contains("CompletableFuture"))
                .count();
    });

    System.out.println("Number of Web Pages having CompletableFuture keyword - " + countFuture.get());
}
```

CompletableFuture.anyOf()

当任何一个CompletableFuture完成的时候【相同的结果类型】，返回一个新的CompletableFuture

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return "Result of Future 1";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return "Result of Future 2";
});

CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return "Result of Future 3";
});

CompletableFuture<Object> anyOfFuture = CompletableFuture.anyOf(future1, future2, future3);

System.out.println(anyOfFuture.get()); // Result of Future 2
```

5. CompletableFuture 异常处理

- 使用 exceptionally() 回调处理异常

> 从异常恢复，仅当发生异常时才会调用

```java
Integer age = -1;

CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
    if(age < 0) {
        throw new IllegalArgumentException("Age can not be negative");
    }
    if(age > 18) {
        return "Adult";
    } else {
        return "Child";
    }
}).exceptionally(ex -> {
    System.out.println("Oops! We have an exception - " + ex.getMessage());
    return "Unknown!";
});

System.out.println("Maturity : " + maturityFuture.get()); 
```

- 使用 handle() 方法处理异常

> 从异常恢复，无论一个异常是否发生它都会被调用，结果取决于用户对入参result和exception的逻辑处理。

```java
Integer age = -1;

CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
    if(age < 0) {
        throw new IllegalArgumentException("Age can not be negative");
    }
    if(age > 18) {
        return "Adult";
    } else {
        return "Child";
    }
}).handle((res, ex) -> {
    if(ex != null) {
        System.out.println("Oops! We have an exception - " + ex.getMessage());
        return "Unknown!";
    }
    return res;
});

System.out.println("Maturity : " + maturityFuture.get());
```

### Exchanger

> 如果一个线程先执行exchange方法，那么它会同步等待另一个线程也执行exchange方法，这个时候两个线程就都达到了同步点，两个线程就可以交换数据

```java
Exchanger<String> exchanger = new Exchanger<>();

//代表男生和女生
ExecutorService service = Executors.newFixedThreadPool(2);

CountDownLatch latch = new CountDownLatch(2);

service.execute(() -> {
    try {
        //男生对女生说的话
        String girl = exchanger.exchange("我其实暗恋你很久了......");
        System.out.println("女孩儿说：" + girl);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        latch.countDown();
    }
});
service.execute(() -> {
    try {
        System.out.println("女生慢慢的从教室你走出来......");
        TimeUnit.SECONDS.sleep(3);
        //男生对女生说的话
        String boy = exchanger.exchange("我也很喜欢你......");
        System.out.println("男孩儿说：" + boy);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        latch.countDown();
    }
});
try {
    latch.await();
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    service.shutdown();
}
```

### Phaser

#### 个人最简单理解

CyclicBarrier的多阶段实现，每个阶段需要参与者到达屏障。（实际并不是，细节差别挺大，建议看完以下详情）

#### 简介

Phaser是java7中引入的,属于线程同步辅助工具类.有一类场景,例如比赛,一个比赛分为3个阶段(phase): 初赛、复赛和决赛,现在规定只要所有运动员完成上一个阶段的比赛就可以进行下一阶段的比赛,并且比赛的过程中允许退赛(deregister),这个场景就很适合Phaser.

#### phaser和phase

正如其名,Phaser可以把一组线程的执行分为多个阶段(phase),并在每个阶段实现线程同步,而且每个阶段可以减少或者增加线程.概言之,一个phaser可以包含一个或者多个phase.

#### party

通过phaser同步的线程被称为party.这个party并没有特别的含义,只是Oracle官方的命名.所有需要同步的party必须持有同一个phaser对象.party需要向phaser注册,执行phaser.register()方法注册,该方法仅仅是增加phaser中的线程计数.也可以通过构造器注册,比如new Phaser(5)就会在创建phaser对象时注册5个party,随后这5个party只要持有该phaser对象并调用该对象的api就能实现同步.

#### arrived和unarrived

party到达一个phaser某个阶段之前处于unarrived状态,到达时处于arrived状态.一个arrived的party也被称为arrival.

#### deregister

一个线程可以在到达(arrive)某个阶段(phase)后退出(deregister),此时可以使用arriveAndDeregister()方法.

#### phase计数

Phaser类有一个phase计数,初始阶段为0.当一个阶段的所有线程到达(arrive)时,会将phase计数加1,这个动作被称为advance.当这个计数达到Integer.MAX_VALUE时,会被重置为0,开始下一轮循环,不过相信很多程序不可能达到这个值.advace这个词出现在Phaser类的很多api里,比如arriveAndAwaitAdvance()、awaitAdvance(int phase)等.在advance过程中,会触发onAdvance(int phase, int registeredParties)方法的执行.

#### onAdvance(int phase, int registeredParties)

可以在这个方法中定义advance过程中需要执行何种操作,如果进入下一阶段(phase)执行,返回false.如果返回true,会导致phaser结束,因此该方法也是终止phaser的关键所在.

#### Tiering

Tiering即分层的意思.Phaser支持分层结构，即通过构造函数Phaser(Phaser parent)和Phaser(Phaser parent, int parties)构造一个树形结构。这有助于减轻因在单个的Phaser上注册过多的任务而导致的竞争，从而提升吞吐量，代价是增加单个操作的开销。

#### api

主要有以下api:

- arrive(): 通知phaser该线程已到达,并且不需等待其它线程,直接进入下一个执行阶段(phase)
- arriveAndAwaitAdvance(): 通知phaser该线程已到达,并且等待其它线程.
- arriveAndDeregister()
- awaitAdvance(int phase): 阻塞线程,直到phaser的phase计数从参数中的phase变化成为另一个值.比如awaitAdvance(2),会导致线程阻塞,直到phaser的phase计数变为3以后才会继续执行.
- awaitAdvanceInterruptibly(int phase)
- onAdvance(int phase, int registeredParties)
- register(): phaser的线程计数加1.

#### 使用案例

```java
// 6个游泳选手完成一场比赛需要分为3个阶段: 到达赛场、准备、完成比赛

// 自定义Phase类
public class MyPhaser extends Phaser{

    // 定义结束阶段.这里是完成3个阶段以后结束
    private static final int phaseToTerminate = 2;

    @Override
    protected boolean onAdvance(int phase, int registeredParties) {
        System.out.println("*第"+phase+"阶段完成*");
        // 到达结束阶段,或者还没到结束阶段但是party为0,都返回true,结束phaser
        return phase==phaseToTerminate || registeredParties==0;
    }
}

public class Swimmer implements Runnable{

    private Phaser phaser;

    public Swimmer(Phaser phaser) {
        this.phaser = phaser;
    }

    @Override
    public void run() {
        // 从这里到第一个phaser.arriveAndAwaitAdvance()是第一阶段做的事
        System.out.println("游泳选手-"+Thread.currentThread().getName()+":已到达赛场");
        phaser.arriveAndAwaitAdvance();

        // 从这里到第二个phaser.arriveAndAwaitAdvance()是第二阶段做的事
        System.out.println("游泳选手-"+Thread.currentThread().getName()+":已准备好");
        phaser.arriveAndAwaitAdvance();

        // 从这里到第三个phaser.arriveAndAwaitAdvance()是第三阶段做的事
        System.out.println("游泳选手-"+Thread.currentThread().getName()+":完成比赛");
        phaser.arriveAndAwaitAdvance();
    }
}

@Test
public void testPhaser() {
    MyPhaser phaser = new MyPhaser();

    // 注册主线程,用于控制phaser何时开始第二阶段
    phaser.register();

    for(int i = 0; i < 6; i++) {
        phaser.register();
        new Thread(new Swimmer(phaser),"swimmer" + i).start();
    }

    // 此时某个游泳者所在的线程都已经完成了第一阶段,但是没法进入第二阶段,因为主线程还没到达第一阶段
    // 主线程到达第一阶段并且不参与后续阶段.其它线程从此时可以进入后面的阶段.
    phaser.arriveAndDeregister();

    // 加while是为了防止其它线程没结束就打印了"比赛结束”
    while (!phaser.isTerminated()) {

    }

    System.out.println("===== 比赛结束 =====");
}
```

结果如下

```java
游泳选手-swimmer0:已到达赛场
游泳选手-swimmer2:已到达赛场
游泳选手-swimmer1:已到达赛场
游泳选手-swimmer3:已到达赛场
游泳选手-swimmer5:已到达赛场
游泳选手-swimmer4:已到达赛场
*第0阶段完成*

游泳选手-swimmer4:已准备好
游泳选手-swimmer3:已准备好
游泳选手-swimmer2:已准备好
游泳选手-swimmer5:已准备好
游泳选手-swimmer1:已准备好
游泳选手-swimmer0:已准备好
*第1阶段完成*

游泳选手-swimmer0:完成比赛
游泳选手-swimmer2:完成比赛
游泳选手-swimmer3:完成比赛
游泳选手-swimmer1:完成比赛
游泳选手-swimmer4:完成比赛
游泳选手-swimmer5:完成比赛
*第2阶段完成*

===== 比赛结束 =====
```

### FutureTask

> Future

Future接口代表异步计算的结果，通过Future接口提供的方法可以查看异步计算是否执行完成，或者等待执行结果并获取执行结果，同时还可以取消执行。Future接口的定义如下:

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

- cancel():cancel()方法用来取消异步任务的执行。如果异步任务已经完成或者已经被取消，或者由于某些原因不能取消，则会返回false。如果任务还没有被执行，则会返回true并且异步任务不会被执行。如果任务已经开始执行了但是还没有执行完成，若mayInterruptIfRunning为true，则会立即中断执行任务的线程并返回true，若mayInterruptIfRunning为false，则会返回true且不会中断任务执行线程。
- isCanceled():判断任务是否被取消，如果任务在结束(正常执行结束或者执行异常结束)前被取消则返回true，否则返回false。
- isDone():判断任务是否已经完成，如果完成则返回true，否则返回false。需要注意的是：任务执行过程中发生异常、任务被取消也属于任务已完成，也会返回true。
- get():获取任务执行结果，如果任务还没完成则会阻塞等待直到任务执行完成。如果任务被取消则会抛出CancellationException异常，如果任务执行过程发生异常则会抛出ExecutionException异常，如果阻塞等待过程中被中断则会抛出InterruptedException异常。
- get(long timeout,Timeunit unit):带超时时间的get()版本，如果阻塞等待过程中超时则会抛出TimeoutException异常。

> FutureTask

FutureTask是Future的实现类，FutureTask实现了RunnableFuture接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。接口的定义如下：

```java
public class FutureTask<V> implements RunnableFuture<V> {

}

public interface RunnableFuture<V> extends Runnable, Future<V> {

}
```

Callable+Future获取多线程的执行结果

```java
@Test
public void testFuture() {
    ExecutorService executor = Executors.newCachedThreadPool();
    Future<Integer> result = executor.submit(new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            System.out.println("子线程在进行计算");
            Thread.sleep(3000);
            int sum = 0;
            for(int i = 0; i < 100; i++){
                sum += i;
            }
            return sum;
        }
    });
    executor.shutdown();

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e1) {
        e1.printStackTrace();
    }

    System.out.println("主线程在执行任务");

    try {
        System.out.println("task运行结果" + result.get());
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }

    System.out.println("所有任务执行完毕");
}
```

使用Callable+FutureTask获取多线程的执行结果

```java
@Test
public void testJuc() {
    //第一种方式
    ExecutorService executor = Executors.newCachedThreadPool();
    FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {

        @Override
        public Integer call() throws Exception {
            System.out.println("子线程在进行计算");
            Thread.sleep(3000);
            int sum = 0;
            for(int i = 0; i < 100; i++){
                sum += i;
            }
            return sum;
        }
    });
    executor.submit(futureTask);
    executor.shutdown();

    //第二种方式，注意这种方式和第一种方式效果是类似的，只不过一个使用的是ExecutorService，一个使用的是Thread
    /*
    Task task = new Task();
    FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
    Thread thread = new Thread(futureTask);
    thread.start();
    */

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e1) {
        e1.printStackTrace();
    }

    System.out.println("主线程在执行任务");

    try {
        System.out.println("task运行结果" + futureTask.get());
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }

    System.out.println("所有任务执行完毕");
}
```

> PS：

有个值得关注的问题就是当任务还在执行的时候用户调用cancel(true)方法能否真正让任务停止执行呢？
当调用cancel(true)方法的时候，实际执行还是Thread.interrupt()方法，而interrupt()方法只是设置中断标志位，如果被中断的线程处于sleep()、wait()或者join()逻辑中则会抛出InterruptedException异常。
因此结论是:cancel(true)**并不一定能够停止正在执行的异步任务**。

### ForkJoin

#### ForkJoin框架

Fork/Join框架是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。它的主要思想是：分而治之。

#### 工作窃取算法

> 工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。

大任务分割为若干互不依赖的子任务。
减少线程间的竞争，将这些子任务分别放到不同的队列里。
为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

优点：充分利用线程进行并行计算，并减少了线程间的竞争；

缺点：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列


#### ForkJoin介绍

Fork/Join框架的设计分为两步：

第一步分割任务。首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join使用两个类来完成以上两件事情：

- ForkJoinTask：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
RecursiveAction：用于没有返回结果的任务。
RecursiveTask ：用于有返回结果的任务。

- ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。


#### ForkJoin使用方式

```java
public class CountTask extends RecursiveTask<Long> {
    // 阀值
    private static final long THRESHOLD = 10000;
    // 开始数
    private long start;
    // 结束数
    private long end;

    public CountTask(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long sum = 0;
        // 如果足够小就计算
        if ((end - start) <= THRESHOLD) {
            for (long i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 否则，对大任务进行拆分
            // 对半分
            long middle = (start + end) / 2;
            // 进行递归
            CountTask left = new CountTask(start, middle);
            CountTask right = new CountTask(middle + 1, end);
            // 执行子任务
            ForkJoinTask.invokeAll(left, right);
            // 获取结果
            long leftResult = left.join();
            long rightResult = right.join();
            sum = leftResult + rightResult;
        }
        return sum;
    }
}

@Test
public void testForkJoin() {
    long s = System.currentTimeMillis();
    ForkJoinPool pool = ForkJoinPool.commonPool();
    // 参数为起始值与结束值
    CountTask countTask = new CountTask(1, 100000000);
    ForkJoinTask<Long> result = pool.submit(countTask);
    // 如果任务完成
    if (!result.isCompletedAbnormally()) {
        try {
            // 获取任务结果
            System.out.println("fork/join计算为：" + result.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    } else {
        countTask.getException().printStackTrace();
    }
    System.out.println("fork/join计算花费时间：" + (System.currentTimeMillis() - s) + "ms");

    s = System.currentTimeMillis();
    long sum = 0;
    for (int i = 1; i <= 100000000; i++) {
        sum += i;
    }
    System.out.println("计算结果：" + sum);
    System.out.println("普通计算花费时间：" + (System.currentTimeMillis() - s) + "ms");
}
```

#### 任务运行状态

- 无论以什么方式结束任务，isDone() 方法返回true；
- 如果完成任务过程中没有被取消或者发生异常，isCompletedNormally() 方法返回true；
- 如果任务被取消， isCancelled() 方法返回true；
- 如果任务被取消或者遇到异常，isCompletedAbnormally() 方法返回true

#### 异常处理

ForkJoinTask在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，所以ForkJoinTask提供了isCompletedAbnormally()方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过ForkJoinTask的getException方法获取异常。使用如下代码：

```java
if (!result.isCompletedAbnormally()) {
    try {
        System.out.println("fork/join计算为：" + result.get());
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
} else {
    countTask.getException().printStackTrace();
}
```

isCompletedAbnormally不阻塞当前线程。getException方法返回Throwable对象，如果任务被取消了则返回CancellationException。如果任务没有完成或者没有抛出异常则返回null。

### CountedCompleter

## 集合类

### 队列双端队列

- ArrayBlockingQueue
- ConcurrentLinkedQueue
- ConcurrentLinkedDeque
- DelayQueue
- LinkedBlockingDeque
- LinkedBlockingQueue
- LinkedTransferQueue
- PriorityBlockingQueue
- SynchronousQueue


### 列表集合

- CopyOnWriteArrayList
- CopyOnWriteArraySet

### 字典

- ConcurrentHashMap
- ConcurrentSkipListMap

## Stream类中的仿JUC实现

大数据量任务分片处理，结果无序

```java
List<Integer> list = new ArrayList<>();
for (int i = 0; i < 100000; i++) {
    list.add(i);
}

int step = 1000;
List<Integer> result = new ArrayList<>();
IntStream.range(0, (list.size() + step - 1) / step)
        .parallel()
        .mapToObj(i -> list.stream()
                .skip(i * step).limit(step)
                .collect(Collectors.toList()))
        .peek(data -> {
            try {
                // simulated time-consuming tasks
                TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        })
        .collect(Collectors.toList());
System.out.println(result);
```

大数据量任务分片处理，结果有序

```java
List<Integer> list = new ArrayList<>();
for (int i = 0; i < 100000; i++) {
    list.add(i);
}

int step = 1000;
List<Integer> result = new ArrayList<>();
IntStream.range(0, (list.size() + step - 1) / step)
        .parallel()
        .mapToObj(i -> list.stream()
                .skip(i * step).limit(step)
                .collect(Collectors.toList()))
        .peek(data -> {
            try {
                // simulated time-consuming tasks
                TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        })
        .forEachOrdered(result::addAll);
System.out.println(result);
```