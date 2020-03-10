# RxJava2 操作符解析

[TOC]

---

## 创建型
## 过滤型
## 组合型
## 转换型
## 创建型
## 创建型

## concat() 简单拼接
```
Observable.concat(Observable.just(1, 2, 3), Observable.just(5, 6, 7))
.subscribe(...);

输出：

0 onSubscribe
0 onNext:1
0 onNext:2
0 onNext:3
0 onNext:5
0 onNext:6
0 onNext:7
0 onComplete

分析：

```

## distinct() 简单去重
```
Observable.just(0, 1, 1, 2, 3)
.distinct()
.subscribe(getObserver(0));

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onNext:2
0 onNext:3
0 onComplete

分析：

```
## filter() 筛选
```
Observable.just(0, 1, 3, 4)
.filter(i -> i > 2)
.subscribe(getObserver(0));

输出：

0 onSubscribe
0 onNext:3
0 onNext:4
0 onComplete

分析：

```

## buffer() 
```
Observable.just(1, 2 ,3, 4, 5)
.buffer(3, 2)
.subscribe(getListObserver(0));

输出：

0list onSubscribe
0list onNext:[1, 2, 3]
0list onNext:[3, 4, 5]
0list onNext:[5]
0list onComplete

分析：
buffer(count, skip) 从定义就差不多能看出作用了，将 observable 中的数据按 skip（步长）分成最长不超过 count 的 buffer，然后生成一个 observable
```
## debounce()
```
Observable.create((ObservableOnSubscribe) e -> {
    e.onNext(1);
    Thread.sleep(1000);
    e.onNext(2);
    Thread.sleep(400);
    e.onNext(3);
    Thread.sleep(1000);
    e.onNext(4);
    Thread.sleep(400);
    e.onNext(5);
    Thread.sleep(400);
    e.onNext(6);
    Thread.sleep(1000);
    e.onComplete();
}).subscribeOn(Schedulers.newThread())
.observeOn(Schedulers.newThread())
.debounce(500, TimeUnit.MILLISECONDS)
.subscribe(getObserver(0));
Thread.sleep(10000);

输出：

0 onSubscribe
0 onNext:1
0 onNext:3
0 onNext:6
0 onComplete
```
## defer()
```
Observable observable = Observable.defer(new Callable>() {
    @Override
    public ObservableSource call() throws Exception {
        return Observable.just(0, 1, 2, 3);
    }
});

observable.subscribe(...);
observable.subscribe(...);

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onNext:2
0 onNext:3
0 onComplete
1 onSubscribe
1 onNext:0
1 onNext:1
1 onNext:2
1 onNext:3
1 onComplete

分析：
就是在每次订阅的时候就会创建一个新的 Observable
```
##  interval()
```
Observable.interval(3, 2, TimeUnit.SECONDS)
.subscribe(new Consumer() {
    @Override
    public void accept(Long aLong) throws Exception {
        p("accept " + aLong + " at " + System.currentTimeMillis());
    }
});

输出：

accept 0 at 1487498704028
accept 1 at 1487498706028
accept 2 at 1487498708028
accept 3 at 1487498710028
accept 4 at 1487498712028
...

分析：
interval(long initialDelay, long period, TimeUnit unit)
类似于一个定时任务
```
## last()
```
Observable.just(0, 1, 2)
.last(3)
.subscribe(...);

输出：

0 onSubscribe
0 onSuccess :2

分析：
就是取出最后一个值，参数是没有值的时候的默认值，比如这样：
Observable.create((ObservableOnSubscribe) Emitter::onComplete).last(3)
.subscribe(getSingleObserver(0));

输出就是 3 了
另见 lastOrError() 方法，区别就是 lastOrError() 没有默认值，没值就触发错误
```
## map()
```
Observable.just(0, 1, 2, 3)
.map(i -> "string" + i)
.subscribe(...)

输出：

0 onSubscribe
0 onNext:string0
0 onNext:string1
0 onNext:string2
0 onNext:string3
0 onComplete

分析：
把持有泛型A的Observable 转成 持有泛型B的Observable
```
## merge()
```
Observable.merge(Observable.just(0, 1), Observable.just(3, 4))
.subscribe(...);

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onNext:3
0 onNext:4
0 onComplete

分析：
将多个 Observable 合起来，例子中只是两个
参数也支持使用迭代器将更多的组合起来
```
## reduce()
```
Observable.just(1, 2, 3)
.reduce((i1, i2) -> i1 + i2)
.subscribe(...);

输出：

0 onSubscribe
0 onSuccess 6

分析：
就是依次用一个方法处理每个值，可以有一个 seed 作为初始值
不指定 seed 则第一次传入的就是第一第二个
```
## replay()
```

PublishSubject publishSubject = PublishSubject.create();
Observable observable = publishSubject.replay().autoConnect();

observable.subscribe(getObserver(0));
publishSubject.onNext(0);
observable.subscribe(getObserver(1));
publishSubject.onNext(1);
publishSubject.onComplete();

输出：

0 onSubscribe
0 onNext:0
1 onSubscribe
1 onNext:0
0 onNext:1
1 onNext:1
0 onComplete
1 onComplete

分析：
从结果可以很明显的看出来，使用了 replay() 后，subscribe(getObserver(1)) 之前的数据也被传入了
相对于 cache() 来说，replay() 提供了更多可控制的选项，在实际使用中可以通过指定 bufferSize 来防止内存占用过大等

还有一个 ReplaySubject 功能和这个应该是差不多的，下面就不单独说了，代码在 GitHub 上都有。
```
## scan()
```
Observable.just(0, 1, 2, 3)
.scan((i1, i2) -> i1 + i2)
.subscribe(getObserver(0));

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onNext:3
0 onNext:6
0 onComplete

分析：
scan() 和上面提到的 reduce() 差不多
区别在于 reduce() 只输出最终结果，而 scan() 会将过程中的每一个结果输出
```

## skip()
```
Observable.just(0, 1, 2, 3)
.skip(2)
.subscribe(getObserver(0));

输出：

0 onSubscribe
0 onNext:2
0 onNext:3
0 onComplete

分析：
看名字也知道是干嘛的了，跳过一些数据，例子中跳过的是数据量，也可以跳过时间 skip(long time, TimeUnit unit)
```
##  take()
```
Observable.just(0, 1, 2, 3)
.take(2)
.subscribe(getObserver(0));

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onComplete

分析：
从数据中取前几个出来
和 skip() 类似，参数可以指定为时间，那就是取出这段时间里的数据了
```
## sample() throttleFirst() throttleLast()
```
Observable.create((ObservableOnSubscribe) e -> {
    e.onNext(0);
    Thread.sleep(200);
    e.onNext(1);
    Thread.sleep(600);
    e.onNext(2);
    Thread.sleep(300);
    e.onNext(3);
    Thread.sleep(1100);
    e.onNext(4);
    Thread.sleep(3000);
    e.onComplete();
})
.sample(1, TimeUnit.SECONDS) // throttleFirst 和 throttleLast 就将这里的 sample 改掉就行
.subscribe(getObserver(0));

输出：

sample:
0 onSubscribe
0 onNext:2
0 onNext:3
0 onNext:4
0 onComplete

throttleFirst:
0 onSubscribe
0 onNext:0
0 onNext:3
0 onNext:4
0 onComplete

throttleLast:
0 onSubscribe
0 onNext:2
0 onNext:3
0 onNext:4
0 onComplete

分析：
先从 sample() 开始吧，发出最接近周期点的事件;
对于 throttleFirst()，它和 sample() 是相反的;
而 throttleLast() 就是 sample() 换了个名字而已.
```
## timer()
```
Observable.timer(1, TimeUnit.SECONDS)
.subscribe(getLongObserver(0));

输出：

0 onSubscribe
0 onNext:0
0 onComplete

分析：
定时任务了
```
## window()
```
Observable.interval(1, TimeUnit.SECONDS)
.take(9)
.window(3, TimeUnit.SECONDS)
.subscribe(new Consumer<Observable>() {
    int n = 0 ;
    @Override
    public void accept(Observable longObservable) throws Exception {
        longObservable.subscribe(getLongObserver(n++));
    }
});

输出：

0 onSubscribe
0 onNext:0
0 onNext:1
0 onComplete
1 onSubscribe
1 onNext:2
1 onNext:3
1 onNext:4
1 onComplete
2 onSubscribe
2 onNext:5
2 onComplete

分析：
按照时间划分窗口，将数据发送给不同的 observable。
```
## zip()
```
Observable.zip(
    (ObservableSource) observer -> {
        observer.onNext(1);
        observer.onNext(2);
        observer.onNext(3);},
    (ObservableSource) observer -> {
        observer.onNext("str");
        observer.onNext("text");
        observer.onComplete();},
    (integer, s) -> integer + s)
.subscribe(getStringObserver(0));

输出：

0 onSubscribe
0 onNext:1str
0 onNext:2text
0 onComplete

分析：
功能是可以将不同的观察者进行组合，并且 onNext() 是一对一对的，例子里的 3 就没有对象了（3 真是惨 =.=）
然后只要有一个执行了 onComplete 就会结束掉
```
## PublishSubject
```
publishSubject.subscribe(getObserver(0));
publishSubject.onNext(0);
publishSubject.subscribe(getObserver(1));
publishSubject.onNext(1);
publishSubject.onComplete();

输出：

0 onSubscribe
0 onNext:0
1 onSubscribe
0 onNext:1
1 onNext:1
0 onComplete
1 onComplete

分析：
onNext() 会通知每个观察者，仅此而已
```
## AsyncSubject
```
AsyncSubject subscriber = AsyncSubject.create();
subscriber.subscribe(getObserver(1));
subscriber.onNext(1);

subscriber.subscribe(getObserver(2));
subscriber.onNext(2);
subscriber.onComplete();


输出：

Observer1 onSubscribe
Observer2 onSubscribe
Observer1 onNext: 2
Observer1 onComplete
Observer2 onNext: 2
Observer2 onCompelte

分析：
查看文档，关于 AsyncObject 的介绍，在 调用 onComplete() 之前，除了 subscribe() 其它的操作都会被缓存，在调用 onComplete() 之后只有最后一个 onNext() 会生效
```
## BehaviorObject
```
BehaviorSubject subscriber = BehaviorSubject.create();

subscriber.subscribe(getObserver(1));

subscriber.onNext(0);
subscriber.onNext(1);

subscriber.subscribe(getObserver(2));
subscriber.onNext(4);

subscriber.onComplete();

输出：

1 onSubscribe
1 onNext:0
1 onNext:1
2 onSubscribe
2 onNext:1
1 onNext:4
2 onNext:4
1 onComplete
2 onComplete

分析：
BehaviorSubject 的最后一次 onNext() 操作会被缓存，然后在subscribe() 后立刻推给新注册的 Observer
对于上面的例子 onNext(1) 这个操作会被缓存，在 subscribe(observer2) 之后会立刻传入 onNext(1) 从而执行
```
## Completable
```
Completable completable = Completable.timer(1000, TimeUnit.MILLISECONDS);
completable.subscribe(new CompletableObserver() {
@Override
public void onSubscribe(Disposable d) {
p("onSubscribe " + System.currentTimeMillis());
}

@Override
public void onComplete() {
p("onComplete " + System.currentTimeMillis());
}

@Override
public void onError(Throwable e) {
p("onError" + e);
}
});

输出：

onSubscribe 1487237357951
onComplete 1487237358952
```
## Flowable
```
Flowable.just(0, 1, 2, 3)
.reduce(50, (a, b) -> a + b)
.subscribe(getSingleObserver(0));

输出：

0 onSubscribe
0 onSuccess :56

分析：
Flowable 在 RxJava2 中是用来解决背压问题的，至于具体的可以看前面推荐的文章，这里就不展开了。
```
## CompositeDisposable
```
new CompositeDisposable().addAll(
    Observable.just(0, 1, 2, 3)
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.computation())
    .subscribeWith(getDisposableObserver(0)),
    Observable.just(6, 7, 8, 9)
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.computation())
    .subscribeWith(getDisposableObserver(1))
);

输出：

0
6
1
7
java.lang.InterruptedException: sleep interrupted
at java....
java.lang.InterruptedException: sleep interrupted
at java...

分析：

就是一个 Disposable 的集合
```
## 
```

```

