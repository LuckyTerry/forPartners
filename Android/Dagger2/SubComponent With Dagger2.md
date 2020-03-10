# SubComponent With Dagger2

Subcomponents are components that inherit and extend the object graph of a parent component. You can use them to partition your application’s object graph into subgraphs either to encapsulate different parts of your application from each other or to use more than one scope within a component.

*子组件是继承和扩展父组件的对象图的组件。您可以使用它们将应用程序的对象图划分为子图，以将应用程序的不同部分彼此封装或在组件中使用多个范围。*

An object bound in a subcomponent can depend on any object that is bound in its parent component or any ancestor component, in addition to objects that are bound in its own modules. On the other hand, objects bound in parent components can’t depend on those bound in subcomponents; nor can objects bound in one subcomponent depend on objects bound in sibling subcomponents.

*绑定在子组件中的对象可以依赖于绑定在其父组件或任何祖先组件中的任何对象，以及绑定在其自己的模块中的对象。另一方面，在父组件中绑定的对象不能依赖于在子组件中绑定的对象; 在一个子组件中绑定的对象也不能依赖于在同级子组件中绑定的对象。*

In other words, the object graph of a subcomponent’s parent component is a subgraph of the object graph of the subcomponent itself.

*换句话说，子组件的父组件的对象图是子组件本身的对象图的子图。*

## Declaring a subcomponent（*声明一个子组件*）

Just like for top-level components, you create a subcomponent by writing an abstract class or interface that declares abstract methods that return the types your application cares about. Instead of annotating a subcomponent with @Component, you annotate it with @Subcomponent and install @Modules. Similar to component builders, a @Subcomponent.Builder specifies an interface to supply necessary modules to construct the subcomponent.

*就像顶级组件一样，通过编写一个抽象类或接口来创建子组件，该抽象类或接口声明抽象方法，以返回您的应用程序关心的类型。您可以使用@Subcomponent注解标记subcomponent，并安装@Modules，而不是使用@Component注解。类似于组件构建器，@Subcomponent.Builder指定一个接口来提供构建子组件的必要模块。*

```java
@Subcomponent(modules = RequestModule.class)
interface RequestComponent {
  RequestHandler requestHandler();

  @Subcomponent.Builder
  interface Builder {
    Builder requestModule(RequestModule module);
    RequestComponent build();
  }
}
```

## Adding a subcomponent to a parent component

To add a subcomponent to a parent component, add the subcomponent class to the subcomponents attribute of a @Module that the parent component installs. Then the subcomponent’s builder can be requested from within the parent.

```java
@Module(subcomponents = RequestComponent.class)
class ServerModule {}

@Singleton
@Component(modules = ServerModule.class)
interface ServerComponent {
  RequestRouter requestRouter();
}

@Singleton
class RequestRouter {
  @Inject RequestRouter(
      Provider<RequestComponent.Builder> requestComponentProvider) {}

  void dataReceived(Data data) {
    RequestComponent requestComponent =
        requestComponentProvider.get()
            .data(data)
            .build();
    requestComponent.requestHandler()
        .writeResponse(200, "hello, world");
  }
}
```

## Subcomponents and scope（*子组件和范围*）

One reason to break your application’s component up into subcomponents is to use scopes. With normal, unscoped bindings, each user of an injected type may get a new, separate instance. But if the binding is scoped, then all users of that binding within the scope’s lifetime get the same instance of the bound type.

*将应用程序组件分解为子组件的一个原因是使用 范围。使用普通的无限制绑定，注入类型的每个用户可能会获得一个新的独立实例。但是如果绑定是有作用域的，那么在该作用域生命周期内该绑定的所有用户将获得绑定类型的相同实例。*

The standard scope is @Singleton. Users of singleton-scoped bindings all get the same instance.

*标准范围是@Singleton。单例作用域绑定的用户都获得相同的实例。*

In Dagger, a component can be associated with a scope by annotating it with a @Scope annotation. In that case, the component implementation holds references to all scoped objects so they can be reused. Modules with @Provides methods annotated with a scope may only be installed into a component annotated with the same scope.

*在Dagger中，一个组件可以通过注释与@Scope注释相关联 。在这种情况下，组件实现保存对所有范围对象的引用，以便它们可以重用。带有@Provides用作用域注释的方法的模块 可能只能安装到使用相同作用域注释的组件中。*

(Types with @Inject constructors may also be annotated with scope annotations. These “implicit bindings” may be used by any component annotated with that scope or any of its descendant components. The scoped instance will be bound in the correct scope.)

*（带有@Inject构造函数的类型也可以使用作用域注释来注释，这些“隐式绑定”可以被任何使用该作用域或其任何后代组件注释的组件使用，范围实例将被绑定在正确的作用域中。*

No subcomponent may be associated with the same scope as any ancestor component, although two subcomponents that are not mutually reachable can be associated with the same scope because there is no ambiguity about where to store the scoped objects. (The two subcomponents effectively have different scope instances even if they use the same scope annotation.)

*没有子组件可以与任何祖先组件相同的范围相关联，尽管不可相互可达的两个子组件可以与相同范围相关联，因为关于在哪里存储范围对象没有歧义。（即使使用相同的作用域注释，这两个子组件实际上也有不同的作用域 实例。）*

For example, in the component tree below, BadChildComponent has the same @RootScope annotation as its parent, RootComponent, and that is an error. But SiblingComponentOne and SiblingComponentTwo can both use @ChildScope because there is no way to confuse a binding in one with a binding of the same type in another.

*例如，在下面的组件树中，BadChildComponent与@RootScope其父代具有相同的 注释RootComponent，并且这是一个错误。但SiblingComponentOne和SiblingComponentTwo可以都使用@ChildScope ，因为没有办法在一个有约束力的另一种相同类型的混淆绑定。*

```java
@RootScope @Component
interface RootComponent {
  BadChildComponent.Builder badChildComponent(); // ERROR!
  SiblingComponentOne.Builder siblingComponentOne();
  SiblingComponentTwo.Builder siblingComponentTwo();
}

@RootScope @Subcomponent
interface BadChildComponent {...}

@ChildScope @Subcomponent
interface SiblingComponentOne {...}

@ChildScope @Subcomponent
interface SiblingComponentTwo {...}
```

Because a subcomponent is created from within its parent, its lifetime is strictly smaller than its parent’s. That means that it makes sense to consider subcomponents’ scopes as “smaller” and parent components’ scopes as “larger”. In fact, you almost always want the root component to use the @Singleton scope.

*由于子组件是从其父项创建的，因此其生命周期严格小于其父项。这意味着将子组件的范围视为“更小”和父组件的范围“更大”是有意义的。实际上，你几乎总是希望根组件使用@Singleton范围。*

In the example below, RootComponent is in @Singleton scope. @SessionScope is nested within @Singleton scope, and @RequestScope is nested within @SessionScope. Note that FooRequestComponent and BarRequestComponent are both associated with @RequestScope, which works because they are siblings; neither is an ancestor of the other.

*在下面的例子中，RootComponent在@Singleton范围内。 @SessionScope嵌套在@Singleton作用域内，并@RequestScope嵌套在作用域内@SessionScope。请注意，FooRequestComponent并 BarRequestComponent都与相关的@RequestScope，这工作，因为他们是兄弟姐妹; 既不是另一个的祖先。*

```java
@Singleton @Component
interface RootComponent {
  SessionComponent.Builder sessionComponent();
}

@SessionScope @Subcomponent
interface SessionComponent {
  FooRequestComponent.Builder fooRequestComponent();
  BarRequestComponent.Builder barRequestComponent();
}

@RequestScope @Subcomponent
interface FooRequestComponent {...}

@RequestScope @Subcomponent
interface BarRequestComponent {...}
```

## Subcomponents for encapsulation（*用于封装的子组件*）

Another reason to use subcomponents is to encapsulate different parts of your application from each other. For example, if two services in your server (or two screens in your application) share some bindings, say those used for authentication and authorization, but each have other bindings that really have nothing to do with each other, it might make sense to create separate subcomponents for each service or screen, and to put the shared bindings into the parent component.

*使用子组件的另一个原因是将应用程序的不同部分彼此封装在一起。例如，如果服务器中的两个服务（或应用程序中的两个屏幕）共享一些绑定，例如用于身份验证和授权的绑定，但每个服务都有其他绑定，而这些绑定实际上没有任何关系，那么创建为每个服务或屏幕分开子组件，并将共享绑定放入父组件。*

In the following example, the Database is provided within the @Singleton component, but all of its implementation details are encapsulated within the DatabaseComponent. Rest assured that no UI will have access to the DatabaseConnectionPool to schedule their own queries without going through the Database since that binding only exists in the subcomponent.

*在下面的例子中， 组件中Database提供@Singleton了它，但是它的所有实现细节都封装在组件中 DatabaseComponent。请放心，没有用户界面可以访问 DatabaseConnectionPool它们来安排他们自己的查询，而不必经过Database这个绑定， 因为绑定只存在于子组件中。*

```java
@Singleton
@Component(modules = DatabaseModule.class)
interface ApplicationComponent {
  Database database();
}

@Module(subcomponents = DatabaseComponent.class)
class DatabaseModule {
  @Provides
  @Singleton 
  Database provideDatabase(
      @NumberOfCores int numberOfCores,
      DatabaseComponent.Builder databaseComponentBuilder) {
    return databaseComponentBuilder
        .databaseImplModule(new DatabaseImplModule(numberOfCores / 2))
        .build()
        .database();
  }
}

@Module
class DatabaseImplModule {
  DatabaseImplModule(int concurrencyLevel) {}
  @Provides DatabaseConnectionPool provideDatabaseConnectionPool() {}
  @Provides DatabaseSchema provideDatabaseSchema() {}
}

@Subcomponent(modules = DatabaseImplModule.class)
interface DatabaseComponent {
  @PrivateToDatabase Database database();
}
```

## Defining subcomponents with abstract factory methods（*用抽象工厂方法定义子组件*）

In addition to @Module.subcomponents, a subcomponent can be installed in a parent by declaring an abstract factory method in the parent component that returns the subcomponent. If the subcomponent requires a module that does not have a no-arg public constructor, and that module is not installed into the parent component, then the factory method must have a parameter of that module’s type. The factory method may have other parameters for any other modules that are installed on the subcomponent but not on the parent component. (The subcomponent will automatically share the instance of any module shared between it and its parent.) Alternatively, the abstract method on the parent component may return the @Subcomponent.Builder and no modules need to be listed as parameters.

*除此之外@Module.subcomponents，通过abstract在返回子组件的父组件中声明工厂方法，可以将子组件安装在父组件中。如果子组件需要一个没有无参数公共构造函数的模块，并且该模块未安装到父组件中，那么工厂方法必须具有该模块类型的参数。工厂方法可以为安装在子组件上但不在父组件上的任何其他模块提供其他参数。（子组件将自动共享它与其父代之间共享的任何模块的实例。）或者，abstract父组件上的方法可能返回@Subcomponent.Builder并且不需要将模块列为参数。*

Using @Module.subcomponents is better since it allows Dagger to detect if the subcomponent is ever requested. Installing a subcomponent via a method on the parent component is an explicit request for that component, even if that method is never called. Dagger can’t detect that, and thus must generate the subcomponent even if you never use it.

*使用@Module.subcomponents更好，因为它允许Dagger检测子组件是否曾被请求过。通过父组件上的方法安装子组件是对该组件的明确请求，即使该方法从未被调用。匕首无法检测到，因此即使您从不使用它也必须生成子组件。*

## Details（*细节*）

### Extending multibindings（*扩展多重绑定*）

Like other bindings, multibindings in a parent component are visible to bindings in subcomponents. But subcomponents can also add multibindings to maps and sets bound in their parent. Any such additional contributions are visible only to bindings within the subcomponent or its subcomponents, and are not visible within the parent.

*像其他绑定一样，父组件中的多重绑定对子组件中的绑定可见。但子组件也可以将多重绑定添加到其父级中的映射和集合。任何此类附加贡献仅对子组件或其子组件内的绑定可见，并且在父代中不可见。*

```java
@Component(modules = ParentModule.class)
interface Parent {
  Map<String, Integer> map();
  Set<String> set();

  Child child();
}

@Module
class ParentModule {
  @Provides @IntoMap
  @StringKey("one") static int one() {
    return 1;
  }

  @Provides @IntoMap
  @StringKey("two") static int two() {
    return 2;
  }

  @Provides @IntoSet
  static String a() {
    return "a"
  }

  @Provides @IntoSet
  static String b() {
    return "b"
  }
}

@Subcomponent(modules = ChildModule.class)
interface Child {
  Map<String, Integer> map();
  Set<String> set();
}

@Module
class ChildModule {
  @Provides @IntoMap
  @StringKey("three") static int three() {
    return 3;
  }

  @Provides @IntoMap
  @StringKey("four") static int four() {
    return 4;
  }

  @Provides @IntoSet
  static String c() {
    return "c"
  }

  @Provides @IntoSet
  static String d() {
    return "d"
  }
}

Parent parent = DaggerParent.create();
Child child = parent.child();
assertThat(parent.map().keySet()).containsExactly("one", "two");
assertThat(child.map().keySet()).containsExactly("one", "two", "three", "four");
assertThat(parent.set()).containsExactly("a", "b");
assertThat(child.set()).containsExactly("a", "b", "c", "d");
```

### Repeated modules（*重复的模块*）

When the same module type is installed in a component and any of its subcomponents, then each of those components will automatically use the same instance of the module. This means that it is an error if you call a subcomponent builder method for a repeated module or if a subcomponent factory method defines a repeated module as a parameter. (The former cannot be checked at compile time, and is thus a runtime error.)

*当相同的模块类型安装在组件及其任何子组件中时，那么每个组件都将自动使用该模块的相同实例。这意味着如果您针对重复模块调用子组件构建器方法，或者子组件工厂方法将重复模块定义为参数，则这是错误的。（前者不能在编译时检查，因此是运行时错误。）*

```java
@Component(modules = {RepeatedModule.class, ...})
interface ComponentOne {
  ComponentTwo componentTwo(RepeatedModule repeatedModule); // COMPILE ERROR!
  ComponentThree.Builder componentThreeBuilder();
}

@Subcomponent(modules = {RepeatedModule.class, ...})
interface ComponentTwo { ... }

@Subcomponent(modules = {RepeatedModule.class, ...})
interface ComponentThree {
  @Subcomponent.Builder
  interface Builder {
    Builder repeatedModule(RepeatedModule repeatedModule);
    ComponentThree build();
  }
}

DaggerComponentOne.create().componentThreeBuilder()
    .repeatedModule(new RepeatedModule()) // UnsupportedOperationException!
    .build();
```