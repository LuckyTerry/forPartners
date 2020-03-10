# Producers With Dagger2（*生产者*）

Dagger Producers is an extension to Dagger that implements asynchronous dependency injection in Java.

*Dagger Producers是Dagger的扩展，它实现了Java中的异步依赖注入。*

## Overview（*概观*）

This document assumes familiarity with the Dagger 2 API and with Guava’s ListenableFuture.

*本文档假设您熟悉Dagger 2 API和Guava的ListenableFuture。*

Dagger Producers introduces new annotations @ProducerModule, @Produces, and @ProductionComponent as analogues of @Module, @Provides, and @Component. We refer to classes annotated @ProducerModule as producer modules, methods annotated @Produces as producer methods, and interfaces annotated @ProductionComponent as producer graphs (analogous to modules, provider methods, and object graphs).

*匕首生产者引入了新的注释@ProducerModule， @Produces，和@ProductionComponent作为类似物@Module，@Provides，和 @Component。我们引用注释@ProducerModule为 生产者模块的类，注释@Produces为生产者方法的方法，以及注释@ProductionComponent为生产者图的接口（类似于 模块，提供者方法和对象图）。*

The key difference from ordinary Dagger is that producer methods may return ListenableFuture<T> and subsequent producer methods may depend on the unwrapped T; the framework handles scheduling the subsequent methods when the future is available.

*与普通Dagger的主要区别在于生产者方法可能会返回， ListenableFuture<T>并且随后的生产者方法可能取决于解包T; 当未来可用时，框架处理后续方法的调度。*

Here is a simple example that mimics a server’s request-handling flow:

*下面是一个模仿服务器请求处理流程的简单例子：*

```java
@ProducerModule(includes = UserModule.class)
final class UserResponseModule {
  @Produces
  static ListenableFuture<UserData> lookUpUserData(
      User user, UserDataStub stub) {
    return stub.lookUpData(user);
  }

  @Produces
  static Html renderHtml(UserData data, UserHtmlTemplate template) {
    return template.render(data);
  }
}

@Module
final class ExecutorModule {
  @Provides
  @Production
  static Executor executor() {
    return Executors.newCachedThreadPool();
  }
}
```

In this example, the computation we’re describing here says:

*在这个例子中，我们在这里描述的计算说：*

- call the lookUpUserData method（*调用该lookUpUserData方法*）
- when the future is available, call the renderHtml method with the result（*当未来可用时，调用renderHtml结果的方法*）

(Note that we haven’t explicitly indicated where User, UserDataStub, or UserHtmlTemplate come from; we’re assuming that the Dagger module UserModule provides those types.)

*（请注意，我们没有明确指出位置User，UserDataStub或 UserHtmlTemplate来自;我们假设Dagger模块UserModule 提供了这些类型。）*

Each of these producer methods will be scheduled on the thread pool that we specify via the binding to @Production Executor. Note that even though the provides method that specifies the executor is unscoped, the production component will only ever get one instance from the provider; so in this case, only one thread pool will be created.

*这些生产者方法中的每一个都将安排在我们通过绑定指定的线程池中@Production Executor。请注意，即使指定执行程序的提供方法未被缩小，生产组件也只会从提供程序获取一个实例; 所以在这种情况下，只会创建一个线程池。*

To build this graph, we use a ProductionComponent:

*为了建立这个图，我们使用ProductionComponent：*

```java
@ProductionComponent(modules = {UserResponseModule.class, ExecutorModule.class})
interface UserResponseComponent {
  ListenableFuture<Html> html();
}

// ...

UserResponseComponent component = DaggerUserResponseComponent.create();

ListenableFuture<Html> htmlFuture = component.html();
```

Dagger generates an implementation of the interface UserResponseComponent, whose method html() does exactly what we described above: first it calls lookUpUserData, and when that future is available, it calls renderHtml.

*Dagger生成接口的实现UserResponseComponent，其方法html()完全如我们上面所描述的那样：首先调用 lookUpUserData，并且当未来可用时，调用它renderHtml。*

Both producer methods are scheduled on the provided executor, so the execution model is entirely user-specified.

*两个生产者方法都在提供的执行器上进行调度，因此执行模型完全由用户指定。*

Note that, as in the above example, producer modules can be used seamlessly with ordinary modules, subject to the restriction that provided types cannot depend on produced types.

*请注意，如上例所示，生产者模块可以与普通模块无缝地使用，受到提供类型不能依赖于生成类型的限制。*

## Exception handling（*异常处理*）

By default, if a producer method throws an exception, or the future that it returns failed, then any dependent producer methods will be skipped - this models “propagating” an exception up a call stack.

*默认情况下，如果一个生产者方法抛出一个异常，或者它返回的将来失败，那么任何依赖的生产者方法都会被跳过 - 这个模型会在调用堆栈中“传播”一个异常。*

If a producer method would like to “catch” that exception, the method can request a Produced<T> instead of a T. For example:

*如果生产者方法想要“捕捉”该异常，则该方法可以请求生产者<T>而不是T.例如：*

```java
@Produces
static Html renderHtml(
    Produced<UserData> data,
    UserHtmlTemplate template,
    ErrorHtmlTemplate errorTemplate) {
  try {
    return template.render(data.get());
  } catch (ExecutionException e) {
    return errorTemplate.render("user data failed", e.getCause());
  }
}
```

In this example, if the production of UserData threw an exception (either by a producer method throwing an exception, or by a future failing), then the renderHtml method catches it and returns an error template.

*在这个例子中，如果产生UserData抛出异常（通过生产者方法抛出异常或未来失败），那么该 renderHtml方法捕获它并返回一个错误模板。*

If an exception propagates all the way up to the component’s entry point without any producer method catching it, then the future returned from the component will fail with an exception.

*如果一个异常一直传播到组件的入口点，而没有任何生产者方法捕获它，那么从组件返回的未来将失败并出现异常。*

## Lazy execution（*懒惰的执行*）

Producer methods can request a Producer<T>, which is analogous to a Provider<T>: it delays the computation of the associated binding until a get() method is called. Producer<T> is non-blocking; its get() method returns a ListenableFuture, which can then be fed to the framework. For example:

*生产者方法可以请求一个Producer<T>类似于a的方法Provider<T>：它会延迟关联绑定的计算，直到get()调用一个方法。Producer<T>是无阻塞的; 其get() 方法返回一个ListenableFuture，然后可以将其馈送到框架。例如：*

```java
@Produces
static ListenableFuture<UserData> lookUpUserData(
    Flags flags,
    @Standard Producer<UserData> standardUserData,
    @Experimental Producer<UserData> experimentalUserData) {
  return flags.useExperimentalUserData()
      ? experimentalUserData.get()
      : standardUserData.get();
}
```

In this example, if the experimental user data is requested, then the standard user data is never computed. Note that the Flags may be a request-time flag, or even the result of an RPC, which lets users build very flexible conditional graphs.

*在这个例子中，如果请求了实验用户数据，那么永远不会计算标准用户数据。请注意，这Flags可能是请求时间标志，甚至是RPC的结果，它可以让用户构建非常灵活的条件图。*

## Multibindings（**）

Several bindings of the same type can be collected into a set or map, just like in ordinary Dagger. For example:

*就像在普通的Dagger中一样，几个相同类型的绑定可以被收集到一个集合或一个地图中。例如：*

```java
@ProducerModule
final class UserDataModule {
  @Produces @IntoSet static ListenableFuture<Data> standardData(...) { ... }
  @Produces @IntoSet static ListenableFuture<Data> extraData(...) { ... }
  @Produces @IntoSet static Data synchronousData(...) { ... }
  @Produces @ElementsIntoSet static Set<ListenableFuture<Data>> rest(...) { ... }

  @Produces static ... collect(Set<Data> data) { ... }
}
```

In this example, when all the producer methods that contribute to this set have completed futures, the Set<Data> is constructed and the collect() method is called.

*在这个例子中，当所有有助于这个集合的生产者方法都完成了期货时，Set<Data>就构造了这个方法，并调用了collect（）方法。*

### Map multibindings（*映射多重绑定*）

Map multibindings are similar to set multibindings:

*映射多重绑定与设置多重绑定类似：*

```java
@MapKey @interface DispatchPath {
  String value();
}

@ProducerModule
final class DispatchModule {
  @Produces @IntoMap @DispatchPath("/user")
  static ListenableFuture<Html> dispatchUser(...) { ... }

  @Produces @IntoMap @DispatchPath("/settings")
  static ListenableFuture<Html> dispatchSettings(...) { ... }

  @Produces
  static ListenableFuture<Html> dispatch(
      Map<String, Producer<Html>> dispatchers, Url url) {
    return dispatchers.get(url.path()).get();
  }
}
```

Note that here, dispatch() is requesting map values of the type Producer<Html>; this ensures that only the dispatch handler that was requested will be executed.

*请注意，这里dispatch()是请求类型的映射值 Producer<Html>; 这确保只有被请求的调度处理程序将被执行。*

Also note that DispatchPath is a [simple map key] (multibindings.md#simple-map-keys), but that [complex map keys] (multibindings.md#complex-map-keys) are supported as well.

*还要注意，这DispatchPath是一个[简单地图键]（multibindings.md＃simple-map-keys），但也支持[复杂地图键]（multibindings.md＃complex-map-keys）。*

## Scoping（*作用域*）

Producer methods and production components are implicitly scoped @ProductionScope. Like ordinary scoped bindings, each method will only be executed once within the context of a given component, and its result will be cached. This gives complete control over the lifetime of each binding — it is the same as the lifetime of its enclosing component instance.

*生产者方法和生产组件隐含作用域 @ProductionScope。像普通的作用域绑定一样，每个方法只会在给定组件的上下文中执行一次，并且其结果将被缓存。这可以完全控制每个绑定的生命周期 - 它与其封闭组件实例的生命周期相同。*

@ProductionScope may also be applied to ordinary provisions; they will then be scoped to the production component that they’re bound in. Production components may also additionally have other scopes, like ordinary components can.

*@ProductionScope也可以适用于普通条款; 它们将被限定到它们绑定的生产组件。生产组件还可以另外具有其他范围，如普通组件所能。*

To supply the executor, bind @Production Executor in a ProductionComponent or ProductionSubcomponent. This binding will be implicitly scoped @ProductionScope. For subcomponents, the executor may be bound in any parent component, and its binding will be inherited in the subcomponent (like all bindings are).

*提供执行者，绑定@Production Executor在一个 ProductionComponentor中ProductionSubcomponent。该绑定将隐含作用域@ProductionScope。对于子组件，执行程序可以绑定在任何父组件中，并且其绑定将在子组件中继承（与所有绑定一样）。*

The executor binding will only be executed once per component instance, even if it is not scoped to the component; that is, it will be implicitly scoped @ProductionScope.

*每个组件实例只能执行一次执行器绑定，即使它不在组件范围内; 也就是说，它将被隐式限制 @ProductionScope。*

## Component dependencies（*组件依赖性*）

Like ordinary Components, ProductionComponents may depend on other interfaces:

*像普通的Components 一样，ProductionComponents可能依赖于其他接口：*

```java
interface RequestComponent {
  ListenableFuture<Request> request();
}

@ProducerModule
final class UserDataModule {
  @Produces static ListenableFuture<UserData> userData(
      Request request, ...) { ... }
}

@ProductionComponent(
    modules = UserDataModule.class,
    dependencies = RequestComponent.class)
interface UserDataComponent {
  ListenableFuture<UserData> userData();
}
```

Since the UserDataComponent depends on the RequestComponent, Dagger will require that an instance of the RequestComponent be provided when building the UserDataComponent; and then that instance will be used to satisfy the bindings of the getter methods that it offers:

*由于UserDataComponent依赖于RequestComponent，Dagger将要求RequestComponent在构建时提供 一个实例UserDataComponent。然后该实例将用于满足它提供的getter方法的绑定：*

```java
ListenableFuture<UserData> userData = DaggerUserDataComponent.builder()
    .requestComponent(/* a particular RequestComponent */)
    .build()
    .userData();
```

## Subcomponents（*子组件*）

Dagger Producers introduces a new annotation @ProductionSubcomponent as an analogue to @Subcomponent. Production subcomponents may be subcomponents of either components or production components.

*Dagger Producers引入了一个新的注解 @ProductionSubcomponent作为@Subcomponent的模拟 。生产子组件可以是组件或生产组件的子组件。*

A subcomponent inherits all bindings from its parent component, and so it is often a simpler way of building nested scopes; see subcomponents for more details.

*子组件继承了其父组件的所有绑定，因此它通常是构建嵌套作用域的一种更简单的方法; 有关更多详细信息，请参阅 子组件。*

## Monitoring（*监控*）

ProducerMonitor can be used to monitor the execution of producer methods; its methods correspond to various places in a producer’s lifecycle.

*ProducerMonitor可用于监视生产者方法的执行情况; 其方法对应于生产者生命周期中的各个地方。*

To install a ProducerMonitor, contribute into a set binding of ProductionComponentMonitor.Factory. For example:

*要安装一个ProducerMonitor，贡献到一个集合的绑定中 ProductionComponentMonitor.Factory。例如：*

```java
@Module
final class MyMonitorModule {
  @Provides @IntoSet
  static ProductionComponentMonitor.Factory provideMonitorFactory(
      MyProductionComponentMonitor.Factory monitorFactory) {
    return monitorFactory;
  }
}

@ProductionComponent(modules = {MyMonitorModule.class, MyProducerModule.class})
interface MyComponent {
  ListenableFuture<SomeType> someType();
}
```

When the component is created, each monitor factory contributed to the set will be asked to create a monitor for the component. The resulting (single) instance will be held for the lifetime of the component, and will be used to create individual monitors for each producer method.

*当创建组件时，每个贡献该组的监视器工厂都将被要求为该组件创建一个监视器。生成的（单个）实例将在组件的生命周期中保存，并将用于为每个生产者方法创建单独的监视器。*

## Timing, Logging and Debugging（*定时，记录和调试*）

### As of March 2016, not implemented yet（*截至2016年3月，尚未实施

*）

Since graphs are constructed at compile-time, the graph will be able to be viewed immediately after compiling, likely via writing its structure to json or a proto.

*由于图形是在编译时构建的，因此可以在编译后立即查看该图，可能通过将其结构写入json或proto来进行查看。*

Statistics from producer execution (CPU usage of producer method invocations and latency of futures) will be available to export to many endpoints.

*生产者执行的统计数据（生产者方法调用的CPU使用率和期货延迟）将可用于输出到许多端点。*

While a graph is running, its current state will be able to be dumped.

*当一个图表正在运行时，其当前状态将能够被倾倒。*

## Optional bindings（*可选绑定*）

@BindsOptionalOf works for producers in much the same way as [for providers] (users-guide#optional-bindings). If Foo is produced, or if there is no binding for Foo, then a @Produces method can depend on any of:

*@BindsOptionalOf对于生产者来说就像[提供者]（用户指南＃可选的绑定）一样。如果Foo产生，或者如果没有约束力Foo，那么@Produces方法可以取决于以下任何一种：*

- Optional<Foo>
- Optional<Producer<Foo>>
- Optional<Produced<Foo>>