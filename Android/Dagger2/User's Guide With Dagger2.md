# User's Guide With Dagger2

The best classes in any application are the ones that do stuff: the BarcodeDecoder, the KoopaPhysicsEngine, and the AudioStreamer. These classes have dependencies; perhaps a BarcodeCameraFinder, DefaultPhysicsEngine, and an HttpStreamer.

在任何应用程序中最好的类是那些真正做事的：比如BarcodeDecoder，KoopaPhysicsEngine，AudioStreamer。这些类具有依赖性：也许是BarcodeCameraFinder，DefaultPhysicsEngine或是HttpStreamer。

To contrast, the worst classes in any application are the ones that take up space without doing much at all: the BarcodeDecoderFactory, the CameraServiceLoader, and the MutableContextWrapper. These classes are the clumsy duct tape that wires the interesting stuff together.

相比下，任何应用程序中最坏的类是那些占用空间却根本不做任何事情的：譬如BarcodeDecoderFactory，CameraServiceLoader，MutableContextWrapper。这些类是笨拙的管道胶带，把有趣的东西串在一起。

Dagger is a replacement for these FactoryFactory classes that implements the dependency injection design pattern without the burden of writing the boilerplate. It allows you to focus on the interesting classes. Declare dependencies, specify how to satisfy them, and ship your app.

Dagger是替代这些FactoryFactory实现依赖注入设计模式的类， 而不需要编写样板文件。它可以让你专注于有趣的类。声明依赖关系，指定如何满足它们，并发布您的应用程序。

By building on standard javax.inject annotations (JSR 330), each class is easy to test. You don’t need a bunch of boilerplate just to swap the RpcCreditCardService out for a FakeCreditCardService.

通过构建标准javax.inject注释（JSR 330），每个类都很容易测试。你不需要一堆样板就可以更换RpcCreditCardService为一个FakeCreditCardService。

Dependency injection isn’t just for testing. It also makes it easy to create reusable, interchangeable modules. You can share the same AuthenticationModule across all of your apps. And you can run DevLoggingModule during development and ProdLoggingModule in production to get the right behavior in each situation.

依赖注入不仅仅用于测试。它还可以轻松创建可重复使用的可互换模块。您可以在所有应用中共享相同的AuthenticationModule。并且您可以在开发过程中运行DevLoggingModule，在生产过程中运行ProdLoggingModule ，以在各自情况下获得正确的行为。

## Why Dagger 2 is Different

Dependency injection frameworks have existed for years with a whole variety of APIs for configuring and injecting. So, why reinvent the wheel? Dagger 2 is the first to implement the full stack with generated code. The guiding principle is to generate code that mimics the code that a user might have hand-written to ensure that dependency injection is as simple, traceable and performant as it can be. For more background on the design, watch this talk (slides) by +Gregory Kick.

拥有多种用于配置和注入API的依赖注入框架已经存在多年。那么，为什么要重新发明轮子？Dagger2是第一个使用生成的代码实现完整堆​​栈的产品。指导原则是生成模仿用户可能手写的代码，以确保依赖注入尽可能简单，可追踪和高性能。有关设计的更多背景，请观看 + Gregory Kick的演讲（幻灯片） 。

## Using Dagger

We’ll demonstrate dependency injection and Dagger by building a coffee maker. For complete sample code that you can compile and run, see Dagger’s coffee example.

我们将通过构建咖啡机来演示依赖注入和Dagger。有关可以编译和运行的完整示例代码，请参阅Dagger的咖啡示例。

### Declaring Dependencies

Dagger constructs instances of your application classes and satisfies their dependencies. It uses the javax.inject.Inject annotation to identify which constructors and fields it is interested in.

Dagger构造应用程序类的实例并满足它们的依赖关系。它使用javax.inject.Inject注释来标识它感兴趣的构造函数和字段。

Use @Inject to annotate the constructor that Dagger should use to create instances of a class. When a new instance is requested, Dagger will obtain the required parameters values and invoke this constructor.

使用@Inject注释那些Dagger用来创建一个类的实例的构造函数。当一个新的实例被请求时，Dagger将获得所需要的参数并调用这个构造函数。

```java
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) {
    this.heater = heater;
  }

  ...
}
```

Dagger can inject fields directly. In this example it obtains a Heater instance for the heater field and a Pump instance for the pump field.

Dagger可以直接注入字段。在这个例子中，它获得了heater字段的Heater实例和pump字段的Pump实例。

```java
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  ...
}
```

If your class has @Inject-annotated fields but no @Inject-annotated constructor, Dagger will inject those fields if requested, but will not create new instances. Add a no-argument constructor with the @Inject annotation to indicate that Dagger may create instances as well.

如果您的类具有@Inject-nnotated字段但没有@Inject-annotated构造函数，则Dagger将根据请求注入这些字段，但不会创建新实例。添加一个不带参数并标记@Inject的构造函数以指示Dagger也可以创建实例。

Dagger also supports method injection, though constructor or field injection are typically preferred.

Dagger也支持方法注入，虽然通常建议使用构造函数或字段注入。

Classes that lack @Inject annotations cannot be constructed by Dagger.

缺乏@Inject注释的类不能由Dagger构建。

### Satisfying Dependencies

By default, Dagger satisfies each dependency by constructing an instance of the requested type as described above. When you request a CoffeeMaker, it’ll obtain one by calling new CoffeeMaker() and setting its injectable fields.

默认情况下，Dagger通过构建上述请求类型的实例来满足每个依赖。当你请求CoffeeMaker时，它会通过调用new CoffeeMaker()来获得一个实例并设置它的注入域。

But @Inject doesn’t work everywhere:

- Interfaces can’t be constructed.
- Third-party classes can’t be annotated.
- Configurable objects must be configured!

但@Inject无处不在：

- 接口不能被构造。
- 第三方类不能被注释。
- 可配置的对象必须配置！

For these cases where @Inject is insufficient or awkward, use an @Provides-annotated method to satisfy a dependency. The method’s return type defines which dependency it satisfies.

对于这些@Inject无效或尴尬的情况，请使用@Provides-annotated方法来满足依赖关系。该方法的返回类型定义了它满足哪个依赖关系。

For example, provideHeater() is invoked whenever a Heater is required:

例如，任何时刻需要Heater时provideHeater()就会被调用：

```java
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```

It’s possible for @Provides methods to have dependencies of their own. This one returns a Thermosiphon whenever a Pump is required:

@Provides方法可能具有自己的依赖关系。每当需要一个Pump时这个方法会返回一个Thermosiphon：
```java
@Provides static Pump providePump(Thermosiphon pump) {
  return pump;
}
```

All @Provides methods must belong to a module. These are just classes that have an @Module annotation.

所有@Provides方法都必须属于一个module。这些是具有@Module注释的类。

```java
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }

  @Provides static Pump providePump(Thermosiphon pump) {
    return pump;
  }
}
```

By convention, @Provides methods are named with a provide prefix and module classes are named with a Module suffix.

按照惯例，@Provides方法以provide前缀命名，module类以Module后缀命名。

### Building the Graph

The @Inject and @Provides-annotated classes form a graph of objects, linked by their dependencies. Calling code like an application’s main method or an Android Application accesses that graph via a well-defined set of roots. In Dagger 2, that set is defined by an interface with methods that have no arguments and return the desired type. By applying the @Component annotation to such an interface and passing the module types to the modules parameter, Dagger 2 then fully generates an implementation of that contract.

标注有@Inject和@Provides-annotated的类构成了一幅对象的图形，通过依赖关系链接在了一起。像应用程序的main方法或Android Application的代码通过一组定义好的根来访问该图形。在Dagger2中，该集合由一个接口定义，该接口的方法没有参数并返回所需的类型。通过将@Component注释应用于此类接口并将module类型传递给modules参数，Dagger2就会完全生成该契约的实现。

```java
@Component(modules = DripCoffeeModule.class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

The implementation has the same name as the interface prefixed with Dagger. Obtain an instance by invoking the builder() method on that implementation and use the returned builder to set dependencies and build() a new instance.

该实现与接口名字一致，并使用Dagger作为前缀。通过调用该实现的builder()方法来获取实例，并使用返回的builder来设置依赖关系并调用build()方法获取新实例。

```java
CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
    .dripCoffeeModule(new DripCoffeeModule())
    .build();
```

Note: If your @Component is not a top-level type, the generated component’s name will include its enclosing types’ names, joined with an underscore. For example, this code:

注意：如果您@Component不是顶级类型，则生成的组件的名称将包含其封闭类型的名称，并用下划线连接。例如，这个代码：

```
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```

would generate a component named DaggerFoo_Bar_BazComponent.

会生成一个名为DaggerFoo_Bar_BazComponent的组件。

Any module with an accessible default constructor can be elided as the builder will construct an instance automatically if none is set. And for any module whose @Provides methods are all static, the implementation doesn’t need an instance at all. If all dependencies can be constructed without the user creating a dependency instance, then the generated implementation will also have a create() method that can be used to get a new instance without having to deal with the builder.

任何具有可访问的默认构造函数的module都可以省略，因为如果没有设置，builder将自动构造一个实例。对于任何其@Provides都是静态方法的module，实现类根本不需要实例。如果所有依赖关系都可以在用户不创建任何依赖项实例的情况下构建，那么生成的实现也会有一个create()方法，可以用来直接获取新实例，而不必处理builder。

```java
CoffeeShop coffeeShop = DaggerCoffeeShop.create();
```

Now, our CoffeeApp can simply use the Dagger-generated implementation of CoffeeShop to get a fully-injected CoffeeMaker.

现在，我们CoffeeApp可以简单地使用Dagger生成的实现类CoffeeShop来获得完全注入的CoffeeMaker。

```java
public class CoffeeApp {
  public static void main(String[] args) {
    CoffeeShop coffeeShop = DaggerCoffeeShop.create();
    coffeeShop.maker().brew();
  }
}
```

Now that the graph is constructed and the entry point is injected, we run our coffee maker app. Fun.

现在，图形被构建，并且入口点被注入，我们运行我们的咖啡机应用程序。真有趣。

```java
$ java -cp ... coffee.CoffeeApp
~ ~ ~ heating ~ ~ ~
=> => pumping => =>
 [_]P coffee! [_]P
```

#### Bindings in the graph

The example above shows how to construct a component with some of the more typical bindings, but there are a variety of mechanisms for contributing bindings to the graph. The following are available as dependencies and may be used to generate a well-formed component:

- Those declared by @Provides methods within a @Module referenced directly by @Component.modules or transitively via @Module.includes
- Any type with an @Inject constructor that is unscoped or has a @Scope annotation that matches one of the component’s scopes
- The component provision methods of the component dependencies
- The component itself
- Unqualified builders for any included subcomponent
- Provider or Lazy wrappers for any of the above bindings
- A Provider of a Lazy of any of the above bindings (e.g., Provider<Lazy<CoffeeMaker>>)
- A MembersInjector for any type

上面的例子展示了如何使用一些更典型的绑定来构建一个component，但是有多种机制可以为依赖图提供绑定。以下是可用的依赖关系，可用于生成格式良好的conponent：

- 那些被@Component.modules直接引用、被@Module.includes包含的@Module中的@Provides方法声明的依赖关系
- 任何未标注@Scope的@Inject构造函数或匹配component的任何一个scope的@Inject构造函数的类型
- component的依赖提供的provide方法
- component本身
- 任何被包含子组件的unqualified的builders
- Provider或上述任何绑定的Lazy包装
- Lazy或上述任何绑定的Provider（例如，Provider<Lazy<CoffeeMaker>>）
- 任何类型的MembersInjector

### Singletons and Scoped Bindings

Annotate an @Provides method or injectable class with @Singleton. The graph will use a single instance of the value for all of its clients.

标注有@Singleton注解的@Provides方法或可注入类。依赖图将为所有客户端使用该值的单例。

```java
@Provides @Singleton static Heater provideHeater() {
  return new ElectricHeater();
}
```

The @Singleton annotation on an injectable class also serves as documentation. It reminds potential maintainers that this class may be shared by multiple threads.

可注入类的@Singleton注解也可以作为文档。它提醒潜在的维护者，这个类可以被多个线程共享。

```java
@Singleton
class CoffeeMaker {
  ...
}
```

Since Dagger 2 associates scoped instances in the graph with instances of component implementations, the components themselves need to declare which scope they intend to represent. For example, it wouldn’t make any sense to have a @Singleton binding and a @RequestScoped binding in the same component because those scopes have different lifecycles and thus must live in components with different lifecycles. To declare that a component is associated with a given scope, simply apply the scope annotation to the component interface.

由于Dagger2将依赖图中的scoped实例与component实现的实例相关联，所以组件本身需要声明它们想要表示的scope。例如，在同一个component中同时使用@Singleton绑定和@RequestScoped绑定没有任何意义， 因为这些范围具有不同的生命周期，因此必须生存在具有不同生命周期的component中。要声明一个与给定scope关联的component，只需将scope注解应用到component接口即可。

```java
@Component(modules = DripCoffeeModule.class)
@Singleton
interface CoffeeShop {
  CoffeeMaker maker();
}
```

Components may have multiple scope annotations applied. This declares that they are all aliases to the same scope, and so that component may include scoped bindings with any of the scopes it declares.

Components可能会拥有多个scope注解。这表明它们都是相同scope的别名，所以组件可能包含它声明的任何scope相关的范围绑定。

### Reusable scope

Sometimes you want to limit the number of times an @Inject-constructed class is instantiated or a @Provides method is called, but you don’t need to guarantee that the exact same instance is used during the lifetime of any particular component or subcomponent. This can be useful in environments such as Android, where allocations can be expensive.

有时候你想限制一个@Inject-constructed类被实例化或一个@Provides方法被调用的次数，但是你不需要保证在任何特定组件或子组件的生命周期中使用完全相同的实例。这对于像Android这样的资源分配可能很昂贵的环境很有用。

For these bindings, you can apply @Reusable scope. @Reusable-scoped bindings, unlike other scopes, are not associated with any single component; instead, each component that actually uses the binding will cache the returned or instantiated object.

对于这些绑定，您可以使用@Reusable范围注解。被@Reusable范围注解修饰的绑定与其他范围绑定不同，它不与任何单个组件关联; 相反，实际使用绑定的每个组件都会缓存返回的或实例化的对象。

That means that if you install a module with a @Reusable binding in a component, but only a subcomponent actually uses the binding, then only that subcomponent will cache the binding’s object. If two subcomponents that do not share an ancestor each use the binding, each of them will cache its own object. If a component’s ancestor has already cached the object, the subcomponent will reuse it.

这意味着如果您在组件中安装了具有@Reusable绑定的module，但只有子组件实际使用该绑定，则只有该子组件才会缓存绑定的对象。如果两个不共享祖先的子组件都使用该绑定，则它们中的每一个都会缓存它自己的对象。如果组件的祖先已经缓存了该对象，则该子组件将复用它。

There is no guarantee that the component will call the binding only once, so applying @Reusable to bindings that return mutable objects, or objects where it’s important to refer to the same instance, is dangerous. It’s safe to use @Reusable for immutable objects that you would leave unscoped if you didn’t care how many times they were allocated.

不保证组件只会调用绑定一次，因此将@Reusable应用于返回可变对象的绑定或引用相同实例会被认为是很重要的对象的绑定是很危险的。相反，如果您不关心分配了多少次，那么将@Reusable应用于可能会离开范围的不可变对象是很安全的。

```java
@Reusable // It doesn't matter how many scoopers we use, but don't waste them.
class CoffeeScooper {
  @Inject CoffeeScooper() {}
}

@Module
class CashRegisterModule {
  @Provides
  @Reusable // DON'T DO THIS! You do care which register you put your cash in.
            // Use a specific scope instead.
  static CashRegister badIdeaCashRegister() {
    return new CashRegister();
  }
}

@Reusable // DON'T DO THIS! You really do want a new filter each time, so this
          // should be unscoped.
class CoffeeFilter {
  @Inject CoffeeFilter() {}
}

```

### Releasable references

DEPRECATED: This feature is deprecated and scheduled for removal in July 2018.

DEPRECATED：此功能已弃用，并计划于2018年7月删除。

When a binding uses a scope annotation, that means that the component object holds a reference to the bound object until the component object itself is garbage-collected. In memory-sensitive environments such as Android, you may want to let scoped objects that are not currently being used be deleted during garbage collection when the application is under memory pressure.

当绑定使用范围注解时，这意味着组件对象持有对绑定对象的引用，直到组件对象本身被垃圾回收为止。在Android等内存敏感的环境中，当应用程序处于内存压力下时，您可能希望在垃圾回收期间删除当前未使用的范围对象。

In that case, you can define a scope and annotate it with @CanReleaseReferences.

在这种情况下，您可以定义一个范围并使用它进行注释@CanReleaseReferences。

```java
@Documented
@Retention(RUNTIME)
@CanReleaseReferences
@Scope
public @interface MyScope {}
```

When you determine that you want to allow objects held in that scope to be deleted during garbage collection if they’re not currently being used by some other object, you can inject a ReleasableReferenceManager object for your scope and call releaseStrongReferences() on it, which will make the component hold a WeakReference to the object instead of a strong reference:

如果您确定要允许在垃圾回收期间删除在该范围内的对象，如果它们当前未被某个其他对象使用，则可ReleasableReferenceManager以为您的范围注入一个对象并调用releaseStrongReferences()它，这将使组件坚持一个WeakReference对象而不是一个强有力的参考：

```java
@Inject @ForReleasableReferences(MyScope.class)
ReleasableReferenceManager myScopeReferenceManager;

void lowMemory() {
  myScopeReferenceManager.releaseStrongReferences();
}
```

If you determine that the memory pressure has receded, then you can restore the strong references for any cached objects that have not yet been deleted during garbage collection by calling restoreStrongReferences():

如果确定内存压力已降低，则可以通过调用restoreStrongReferences()以下命令来恢复垃圾回收期间尚未删除的任何缓存对象的强引用：

```java
void highMemory() {
  myScopeReferenceManager.restoreStrongReferences();
}
```

### Lazy injections

Sometimes you need an object to be instantiated lazily. For any binding T, you can create a Lazy<T> which defers instantiation until the first call to Lazy<T>’s get() method. If T is a singleton, then Lazy<T> will be the same instance for all injections within the ObjectGraph. Otherwise, each injection site will get its own Lazy<T> instance. Regardless, subsequent calls to any given instance of Lazy<T> will return the same underlying instance of T.

有时你需要一个对象懒惰地实例化。对于任何的T，你可以创建一个延迟实例化过程直到第一次调用其get()方法的Lazy<T>，。如果T是单例，那么Lazy<T>将是ObjectGraph中所有注入的同一个实例。否则，每个注入站点将获得它自己的Lazy<T>实例。无论如何，随后对任何给定Lazy<T>实例的调用都将返回相同的底层实例T。

```java
class GrindingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to .get() and cached.
      lazyGrinder.get().grind();
    }
  }
}
```

### Provider injections

Sometimes you need multiple instances to be returned instead of just injecting a single value. While you have several options (Factories, Builders, etc.), one option is to inject a Provider<T> instead of just T. A Provider<T> invokes the binding logic for T each time .get() is called. If that binding logic is an @Inject constructor, a new instance will be created, but a @Provides method has no such guarantee.

有时你需要返回多个实例，而不是只注入一个值。虽然你有几个选择（工厂，建设者等），但一种选择是注入Provider<T>而不仅仅是T。每当Provider的.get()方法被调用时Provider<T>就会调用其绑定逻辑以生成T。如果该绑定逻辑是一个@Inject构造函数，则会创建一个新实例，但@Provides方法没有这种保证。

```java
class BigCoffeeMaker {
  @Inject Provider<Filter> filterProvider;

  public void brew(int numberOfPots) {
  ...
    for (int p = 0; p < numberOfPots; p++) {
      maker.addFilter(filterProvider.get()); //new filter every time.
      maker.addCoffee(...);
      maker.percolate();
      ...
    }
  }
}
```

Note: Injecting Provider<T> has the possibility of creating confusing code, and may be a design smell of mis-scoped or mis-structured objects in your graph. Often you will want to use a factory or a Lazy<T> or re-organize the lifetimes and structure of your code to be able to just inject a T. Injecting Provider<T> can, however, be a life saver in some cases. A common use is when you must use a legacy architecture that doesn’t line up with your object’s natural lifetimes (e.g. servlets are singletons by design, but only are valid in the context of request-specfic data).

注意：注入Provider<T>可能会产生混淆的代码，并且可能是依赖图中的错误范围或错误结构对象的设计意味。通常你会想要使用工厂或者 Lazy<T>重新组织代码的生命周期和结构，以便能够注入一个T。然而，Provider<T>在某些情况下，注射可以成为救生员。一个常见的用法是，您必须使用与您的对象的自然生命周期不一致的遗留体系结构（例如，servlet是设计成单例的，但仅在请求特定数据的情况下有效）。

### Qualifiers

Sometimes the type alone is insufficient to identify a dependency. For example, a sophisticated coffee maker app may want separate heaters for the water and the hot plate.

有时，仅仅只有类型不足以识别依赖性。例如，一个复杂的咖啡机应用程序可能需要为水和热板使用不同的加热器。

In this case, we add a qualifier annotation. This is any annotation that itself has a @Qualifier annotation. Here’s the declaration of @Named, a qualifier annotation included in javax.inject:

在这种情况下，我们添加一个限定符注解。这是是任何注解，只要它本身被@Qualifier注解修饰即可。以下是一个@Named注解的声明，一个包含在javax.inject包下的限定符注解：

```java
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {
  String value() default "";
}
```

You can create your own qualifier annotations, or just use @Named. Apply qualifiers by annotating the field or parameter of interest. The type and qualifier annotation will both be used to identify the dependency.

您可以创建自己的限定符注解，或者只是使用@Named。通过注解感兴趣的字段或参数来应用限定符。类型和限定符注解都将用于标识依赖关系。

```java
class ExpensiveCoffeeMaker {
  @Inject @Named("water") Heater waterHeater;
  @Inject @Named("hot plate") Heater hotPlateHeater;
  ...
}
```

Supply qualified values by annotating the corresponding @Provides method.

通过注解相应的@Provides方法来提供相应的值。

```java
@Provides @Named("hot plate") static Heater provideHotPlateHeater() {
  return new ElectricHeater(70);
}

@Provides @Named("water") static Heater provideWaterHeater() {
  return new ElectricHeater(93);
}
```

Dependencies may not have multiple qualifier annotations.

依赖项可能没有多个限定符注释。

### Optional bindings

If you want a binding to work even if some dependency is not bound in the component, you can add a @BindsOptionalOf method to a module:

如果组件中没有绑定某个依赖关系，但你仍然想让绑定生效，那么，你可以将一个@BindsOptionalOf方法添加到module中：

```java
@BindsOptionalOf abstract CoffeeCozy optionalCozy();
```

That means that @Inject constructors and members and @Provides methods can depend on an Optional<CoffeeCozy> object. If there is a binding for CoffeeCozy in the component, the Optional will be present; if there is no binding for CoffeeCozy, the Optional will be absent.

这意味着@Inject构造函数、@Inject成员和@Provides方法可以依赖于一个Optional<CoffeeCozy>对象。如果组件中存在一个CoffeeCozy的绑定，那么Optional则是有值的; 如果不存在CoffeeCozy的绑定，这个Optional将是缺省的。

Specifically, you can inject any of the following:

- Optional<CoffeeCozy> (unless there is a @Nullable binding for CoffeeCozy; see below)
- Optional<Provider<CoffeeCozy>>
- Optional<Lazy<CoffeeCozy>>
- Optional<Provider<Lazy<CoffeeCozy>>>

具体而言，您可以注入以下任何一项：
- Optional<CoffeeCozy> (除非有@Nullable约束力 CoffeeCozy；见下文)
- Optional<Provider<CoffeeCozy>>
- Optional<Lazy<CoffeeCozy>>
- Optional<Provider<Lazy<CoffeeCozy>>>

(You could also inject a Provider or Lazy or Provider of Lazy of any of those, but that isn’t very useful.)

（你也可以注入的这些类型的Provider、Lazy或Provider<Lazy>，但不是非常有用的。）

If there is a binding for CoffeeCozy, and that binding is @Nullable, then it is a compile-time error to inject Optional<CoffeeCozy>, because Optional cannot contain null. You can always inject the other forms, because Provider and Lazy can always return null from their get() methods.

如果存在一个CoffeeCozy的绑定，并且该绑定是@Nullable，那么注入Optional<CoffeeCozy>将导致一个编译时错误，因为Optional不能包含null。您可以随时注入其他形式，因为Provider和Lazy的get()方法可以随时返回null。

An optional binding that is absent in one component can be present in a subcomponent if the subcomponent includes a binding for the underlying type.

如果子组件包含对基础类型的绑定，则可以在子组件中存在一个父组件中不存在的可选绑定。

You can use either Guava’s Optional or Java 8’s Optional.

您可以使用Guava Optional或Java 8 Optional。

### Binding Instances

Often you have data available at the time you’re building the component. For example, suppose you have an application that uses command-line args; you might want to bind those args in your component.

通常，在构建组件时，您可以获得可用的数据。例如，假设您有一个使用命令行参数的应用程序; 你可能想要在你的组件中绑定这些参数。

Perhaps your app takes a single argument representing the user’s name that you’d like to inject as @UserName String. You can add a method annotated @BindsInstance to the component builder to allow that instance to be injected in the component.

也许你的应用程序只需要一个参数来表示你想注入的用户名@UserName字符串，可以为组件生成器添加一个修饰有@BindsInstance注解的方法，以允许将该实例注入组件。

```java
@Component(modules = AppModule.class)
interface AppComponent {
  App app();

  @Component.Builder
  interface Builder {
    @BindsInstance Builder userName(@UserName String userName);
    AppComponent build();
  }
}
```

Your app then might look like

你的应用程序可能看起来像

```java
public static void main(String[] args) {
  if (args.length > 1) { exit(1); }
  App app = DaggerAppComponent
      .builder()
      .userName(args[0])
      .build()
      .app();
  app.run();
}
```

In the above example, injecting @UserName String in the component will use the instance provided to the Builder when calling this method. Before building the component, all @BindsInstance methods must be called, passing a non-null value (with the exception of @Nullable bindings below).

在上面的示例中，当main方法调用时,组件中注入@UserName字符串时将使用提供给Builder的实例。在构建组件之前，所有标注有@BindsInstancem注解的方法必须调用，并传递一个非空值（@Nullable下面的绑定除外）。

If the parameter to a @BindsInstance method is marked @Nullable, then the binding will be considered “nullable” in the same way as a @Provides method is nullable: injection sites must also mark it @Nullable, and null is an acceptable value for the binding. Moreover, users of the Builder may omit calling the method, and the component will treat the instance as null.

对于标记有@BindsInstance注解的方法，如果其参数被标记为@Nullable，那么绑定将被认为是“可空的”，@Provides方法与空方法相同：注入站点也必须标记为@Nullable，并且null是该绑定的可接受值。而且，用户Builder可能会省略调用该方法，并且该组件会将该实例视为null。

@BindsInstance methods should be preferred to writing a @Module with constructor arguments and immediately providing those values.

在设计一个使用构造函数参数方式来实例化的Module，并要求立即返回这些参数时，应该优先使用标记有@BindsInstance注解的方法

### Compile-time Validation

The Dagger annotation processor is strict and will cause a compiler error if any bindings are invalid or incomplete. For example, this module is installed in a component, which is missing a binding for Executor:

Dagger注解处理器是严格的，如果任何绑定无效或不完整，将导致编译器错误。例如，该模块安装在一个组件中，该组件缺少到Executor的绑定关系：

```java
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater(Executor executor) {
    return new CpuHeater(executor);
  }
}
```

When compiling it, javac rejects the missing binding:

编译时，javac拒绝缺失的绑定：

```java
[ERROR] COMPILATION ERROR :
[ERROR] error: java.util.concurrent.Executor cannot be provided without an @Provides-annotated method.
```

Fix the problem by adding an @Provides-annotated method for Executor to any of the modules in the component. While @Inject, @Module and @Provides annotations are validated individually, all validation of the relationship between bindings happens at the @Component level. Dagger 1 relied strictly on @Module-level validation (which may or may not have reflected runtime behavior), but Dagger 2 elides such validation (and the accompanying configuration parameters on @Module) in favor of full graph validation.

通过在component中任何一个modul里添加标记有@Provides注解、返回类型为Executor的方法来修复该问题。虽然@Inject，@Module和 @Provides注解分别进行验证，但是所有绑定关系的验证发生在@Component级别。Dagger1严格依赖@Module级别验证（可能会或可能不会反映运行时行为），但Dagger2将这种验证（以及随附的配置参数@Module）取消，以支持完整的依赖图验证。

#### Compile-time Code Generation

Dagger’s annotation processor may also generate source files with names like CoffeeMaker_Factory.java or CoffeeMaker_MembersInjector.java. These files are Dagger implementation details. You shouldn’t need to use them directly, though they can be handy when step-debugging through an injection. The only generated types you should refer to in your code are the ones Prefixed with Dagger for your component.

Dagger的注解处理器也可以生成类似于CoffeeMaker_Factory.java或CoffeeMaker_MembersInjector.java等名称的源文件。这些文件是Dagger实现的细节。您不需要直接使用它们，但当进行分步调试时，注入它们可能非常有用。在您的代码中，您唯一应该引用的生成类型是添加了Dagger前缀的组件。

## Using Dagger In Your Build

You will need to include the dagger-2.X.jar in your application’s runtime. In order to activate code generation you will need to include dagger-compiler-2.X.jar in your build at compile time. See the README for more information.

您将需要在应用程序的运行时包含dagger-2.X.jar。为了自动生成代码，您需要在编译时包含dagger-compiler-2.X.jar。请参阅自述文件以获取更多信息。