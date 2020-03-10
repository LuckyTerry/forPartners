# 开源库及源码分析

## ButterKnife

核心：使用Java Annotation Processing，编译时扫描和解析Java注解，然后生成新的Java代码

流程：ButterKnifeProcessor类的process()方法

编译时：

1、ButterKnifeProcessor会帮你生成一个Java类，名字类似className$ViewBinder，这个新生成的类实现了ViewBinder<T>接口

2、这个ViewBinder类中包含了所有对应的代码，比如@Bind注解对应findViewById(), @OnClick对应了view.setOnClickListener()等等

启动时：

1、当Activity启动ButterKnife.bind(this)执行时，调用findViewBinderForClass(targetClass)加载相应的ViewBinder类

2、然后调用ViewBinder的bind方法，动态注入Activity类中所有的View属性和监听器，这里的注入是赋值操作，所以要求注解标注的属性或方法必须是public或protected

## OkHttp

## Retrofit

### Retrofit && Retrofit.Builder

Retrofit.Builder使用建造者模式接收一系列的配置，如Url、okhttp3.Call.Factory、CallAdapter.Factory、Converter.Factory等，然后应用进Retrofit中。可以认为Retrofit是一个配置的持有类。

### Call && OkHttpCall 

Call是一个网络调用的抽象，它与okhttp3.Call是一脉相承，加入了泛型支持和异常处理。它继承自cloneable接口，提供clone方法，是原型模式的一种运用。

OkHttpCall是一个Call的实现类，它持有ServiceMethod, Arguments等构成了网络请求的基本要素和一些状态控制的变量，通过对okhttp3.Call的装饰，能够同步、异步地执行、取消网络请求。这里体现装饰者模式和观察者模式的运用。

### CallAdapter && CallAdapter.Factory

CallAdapter是网络调用适配的抽象，如其名，是一种对象适配器模式的运用，作为一个适配器，在adapt方法中将适配者Call以入参形式传入，返回自定义的类型。

CallAdapter.Factory是CallAdapter的抽象工厂，负责生产CallAdapter这一个产品，是工厂方法模式的一种运用。常见的RxJava2CallAdapterFactory则是实现了生产方法，并返回RxJava2CallAdapter。如果客户端没有配置CallAdapter.Factory，Retrofit会采用默认的实现DefaultCallAdapterFactory直接返回Call对象。从这里可以采用多种CallAdapter来看，这是一种策略模式的隐式运用，策略强调的是这些不同CallAdapter对象的adapt方法的具体实现。

### Converter && Converter.Factory

Converter是一个数据装换的抽象，负责Http表现层和DTO的装换。

Converter.Factory是Converter的抽象工厂，能生产分别装换RequestBody、ResponseBody、String这三种数据类型的Converter，是抽象工厂模式的一种运用。常见的GsonConverterFactory则是覆写了其中两个生产方法，分别生产GsonRequestBodyConverter和GsonResponseBodyConverter。如果客户端没有配置Converter.Factory时会使用默认的BuiltInConverters，根据生产方法、类型和注解返回若干Converter的单例实现。

### Method && ServiceMethod

ServiceMethod的生成也是使用建造者模式，它持有Retrofit配置类和Method，能从中取出用户配置、方法注解等进行自己的构建。最后，它作为OkHttpCall的一个成员，根据自己构建时保存的用户配置等信息辅助前者生成真正的Request和Call。

Method接口数相对于请求次数是有数量级差距，因此使用ServiceMethodCache对ServiceMethod进行缓存，实现大量细粒度对象的复用，这是一种享元模式的体现。

### okhttp3.Call && okhttp3.Call.Factory 

### 相关设计模式

建造者模式(Builder)
简单工厂模式(Static Factory)
工厂方法模式(Factory Method)
抽象工厂模式(Abstract Factory)
外观模式模式(Facade)
代理模式(Proxy)
装饰者模式(Decorator)
适配器模式(Adapter)
策略模式(Strategy)
观察者模式(Observer)
单例模式(Singleton)
原型模式(Prototype)
享元模式(Flyweight)

## RxJava

### 核心：

1、基于观察者模式，实现响应式编程

2、简单、安全的线程切换

3、链式操作

4、丰富的操作符

### 类型：

Observable：能单独发射一个onSubscribe、若干onNext、一个onError、一个onComplete事件

Flowable：能单独发射一个onSubscribe、若干onNext、一个onError、一个onComplete事件，事件可以使用背压策略进行管理

Single：能单独发射一个onSuccess或者onError事件

Completable：能单独发射一个onComplete或者onError事件

Maybe：能单独发射一个onSuccess事件(对应1个item)、onComplete(对应0个item)事件或者onError事件

MaybeObserver：onSubscribe()、onSuccess()、onError()、onComplete()

### 工厂创建方法：

create、defer、just、zip、merge、concat、timer、interval、fromIterable、combineLatest、fromArray、range、amb、empty、error、never、fromFuture、fromCallable等等

### 操作符拓展：

1、过滤

常用的：filter、first、timeout、throttleFirst

其他：take、takeLast、debounce等

2、转换

常用：map、flatMap、concatMap、flatMapIterator、switchMap、scan

其他：groupby、buffer、window、cast

3、组合

常用：zipWith、startWith

其他：withLatestFrom

4、Schedulers

subscribeOn、observeOn

### 源码分析

1、Observable如何将数据发送出去的，Observer如何接收到数据的，何时将源头和终点关联起来的

以Observable.create()为例：主要关注subscribe()方法，该方法内部实际调用subscribeActual()，该方法内部主要有三个步骤：一是创建内部持有真正Observer引用，并且实现了ObservableEmitter和Disposable接口的CreateEmitter，该类作为中间层，负责转发Observable发送的事件到真正的Observer。二是调用真正Observer的onSubscribe方法并传入CreateEmitter作为参数，至此CreateEmmiter就可以响应Observer的dispose动作，并判断接下来的事件是否可以发送。三是调用Observable的subscribe方法并传入CreateEmitter作为参数，至此CreateEmmiter就可以响应Observable的各种事件。通过第一、三步，以CreateEmitter作为中间人，就建立起上下游的关联。

2、线程调度的实现

subscribeOn：该操作符核心与上述的Observable.create()基本类似，都是创建了一个中间者，但是这个中间者第三步却是在Scheduler的Runnable.run()方法中进行调用。也就是事件的发送都是在Scheduler指定的线程中执行。值得注意的是该方法如果切换线程多次，那么总是以第一次为准，因为订阅subscribe是从下往上传递的，那么第一个线程切换是执行在最内部的，所以以它为准。

observeOn：该操作符核心与上述的subscribeOn不太一样，它先创建一个Worker，然后创建一个中间者，并传入Worder作为参数，在中间者的onSubscribe方法回调下游的onSubscribe方法，并且在每一个onNext、onError、onComplete内部切换线程然后回调下游的对应方法，因此observeOn方法决定事件的接收线程。值得注意的是该方法如果多次切换线程，那么总是以最新的为准，因为事件的发送是从上往下，每一次调用总会影响下游。

3、操作符的实现

4、并行转串行的算法

### 与REST的无缝结合Retrofit


## RxLifecycle

核心：RxLifecycle的原理是不是基于自动调用unsubscribe的，而是基于takeUtil操作符，该操作符接受一个Activity或Fragment所持有的Observable作为参数，一旦该Observable发送事件，takeUtil所在事件流就会终止，从而达到根据Activity或Fragment生命周期自动解除订阅的目的；

bindUtilEvent：绑定到Activity或Fragment的指定生命周期对应的事件

bindLifecyle：内部由一个Mapper进行事件的对称映射，比如：OnCreate事件映射为OnDestroy事件，从而自动在合适的生命周期解除订阅。

## Lifecycle Component

## Glide

## GreenDao

## Room

## Permission
